# Claim types, retrieval states and field semantics

Contents: claim types · retrieval states · date semantics · confidence semantics

## Claim types

Assign exactly one type to each material statement before drafting.

### FACT

Directly supported by validated evidence (a retrieved object or relationship, inspected).

> Sekoia associates infrastructure X with campaign Y.

### CALCULATED RESULT

Derived mechanically from validated evidence using a reproducible, documented methodology.

> Indicator overlap percentage · number of active infrastructures · time difference between two observations · distribution of objects by geography · "22 indicators: 15 URLs, 7 file hashes".

State the methodology (which objects, which fields, which operation).

### ASSESSMENT

Analytical interpretation based on available evidence. Marked as such in the text.

> This mechanism may be relevant to organisations using the affected identity stack.

### EXPOSURE MODEL

A conditional pathway describing how verified threat behaviour could intersect a customer environment.

> Identity compromise → cloud account access → business communications → financial-workflow exposure.

It never establishes that the recipient experienced the incident, and it implies no independent activity date.

### NOT ASSERTED

A material conclusion deliberately excluded because the evidence does not support it. Written out explicitly so the reader sees the line was considered and not crossed.

> Customer-specific targeting · confirmed customer compromise · deployment of a technology by the recipient · common attribution between technically similar clusters.

## Retrieval states

| State | Definition | May support |
|---|---|---|
| VALUE | Usable data successfully returned | FACT, CALCULATED RESULT |
| EMPTY | Query completed successfully, no match within the exact scope queried | An absence statement only where the query is known exhaustive for that claim (say the scope: "no relationship of type X with target_types Y was returned") |
| NOT_RETURNED | Failed, timed out, truncated or inconclusive | Nothing. Not evidence of absence. Retry bounded or change strategy; else the claim stays unresolved and the gap is stated |
| NOT_CHECKED | Never queried | Nothing. Listed as a limitation when material |

Practical mappings on the Sekoia MCP tools:
- `search_observables` returns `found: false` with a placeholder object for unknown indicators. That is EMPTY for "known to Sekoia CTI" within the searched values, nothing more.
- A truncated or context-overflowing `get_cti_object_relationships_by_id` is NOT_RETURNED; re-query with one `target_types` value per call. A bounded retry that succeeds upgrades that slice to VALUE while the full graph stays NOT_RETURNED unless every slice was retrieved.
- A truncated `get_cti_object_reports_by_id` is NOT_RETURNED for report completeness; report names and publication dates from `search_cti` results remain VALUE for what they show.
- A tool absent from the key's permissions makes its checks NOT_CHECKED.

## Date semantics

| Field | Means | Never means |
|---|---|---|
| `created` | When the object entered the intelligence base | When the activity happened |
| `modified` | Last edit of the object | Recent threat activity |
| `first_seen` | Start of observed activity of the object | The start date of every related object |
| `last_seen` | End of observed activity, read on indicators only | Activity end on any other object (rarely filled, sometimes a placeholder) |
| Latest relationship `created` | Last time Sekoia linked something to the object | An activity date |
| `valid_from` / `valid_until` | Indicator validity period | Current maliciousness after expiry |
| Report `published` | Publication date | Activity date |
| Relationship with no date | No independent activity date; say so | The date of either endpoint |

## Confidence semantics

- Confidence is read per object and per relationship and quoted exactly (a number, a range or absent).
- 100 means the source asserts it at maximum confidence, not that it is certain or current.
- An object's confidence does not transfer to its relationships, and a relationship's confidence does not transfer to conclusions built across several relationships.
- Third-party reporting reaches the analysis only as objects ingested in Sekoia Intelligence. Keep each ingested source's evidentiary role separate and never average confidences.
