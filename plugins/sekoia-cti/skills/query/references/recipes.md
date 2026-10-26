# Recipes

One recipe per question type. Each gives the question shapes, the tool sequence with parameters, the answer shape and the traps observed on the live server. Prompts in the documentation how-to guides map onto these.

## 1. Actor profile

Question shapes: "what does Sekoia know about TeamTNT", "profile Salt Typhoon", "who is RedMike".

1. `search_cti` with the name. Pick the `intrusion-set` (or `threat-actor`) whose `name` or `aliases` match; ignore campaigns and reports at this step.
2. `get_cti_object_by_id` on it: description, aliases, `first_seen`, goals, resource level, markings.
3. Graph slices as the question needs, one `target_types` value per call: `["malware"]`, `["tool"]`, `["attack-pattern"]`, `["identity"]` (sectors and countries), `["campaign"]`, `["vulnerability"]`.
4. `get_cti_object_reports_by_id` for sourcing, or the report objects from step 1 when the payload overflows.

Answer: identity line (name, aliases as recorded, confidence, first seen, latest relationship date across the slices fetched), the most recent campaign by `first_seen` when the campaign slice was fetched, three or four capability lines with counts per slice, the sectors and countries as relevance ("Sekoia relates the actor to the Telecommunications sector at 100; that is not a victim list"), the two or three most recent reports with dates, then the gaps.

Traps: a revoked duplicate object may surface first; check `revoked`. Do not quote `last_seen` on the intrusion set: it is filled on a minority of actors and some values are period-end placeholders (2026-12-31). For recent activity, name the most recent campaign by `first_seen`, and do not let the user read it as activity on their surface. Counts are of the returned slice; say "in the returned slice" when the call overflowed and was recovered.

Follow-up to offer: has Sekoia CTI on this actor fired in the tenant (recipe 3, pattern 1 on the actor and its infrastructure and malware IDs).

## 2. Indicator enrichment

Question shapes: "is 203.0.113.42 known", "enrich this hash", "which threat is this domain linked to".

1. `search_observables` with all values in `searches`, `with_indicated_threats` default.
2. For each populated `x_ic_indicated_threats`, `get_cti_object_by_id` on the threat for its profile.

Answer, per value: not found; found with no current indicated threat (say what that means: referenced in the database, no valid indicator today); or indicator of threat X with validity period, `last_seen` when the indicator carries one, confidence and the threat's profile line. For every found value, name the source from `observable.x_inthreat_sources_refs`, resolving unknown identity IDs with `get_cti_object_by_id`: "reported by Sekoia C2 Tracker" (Sekoia first-party) reads differently from a value reported by a vendor or a government CERT (ingested).

Traps: `found: true` with empty threats is common, including for infrastructure named in old reports. Never upgrade it to malicious. Several values in one call come back in order; report each separately.

Follow-up to offer: did these values appear in the tenant's events (recipe 3, pattern 2).

## 3. Tenant check

Question shapes: "are we exposed to TeamTNT", "did any of this actor's infrastructure hit us", "has Sekoia CTI on APT28 fired in our tenant", "did we see this domain".

Patterns, constraints and field map: `references/tenant_check.md`.

1. Recipe 1 steps 1 to 3 for an actor (or recipe 5 for a malware family), keeping the IDs of the actor and of its related `["infrastructure"]`, `["malware"]`, `["tool"]` and `["campaign"]` objects.
2. Pattern 1: alerts already raised by the Sekoia CTI feed on those IDs (`alerts | where threats.id in [...]`), by threat and status, then the latest ones.
3. Indicators: `search_observables` on the values named in the reports, the campaign descriptions or the user's message (Sekoia objects rarely list indicators inline).
4. Pattern 2: sweep the events for those indicators on `related.ip`, `related.hosts`, `related.hash`, aggregated by host and intake; pattern 3 for first and last hit when rows come back.

Answer in three labelled parts: what Sekoia CTI records about the threat (CTI evidence), which IDs and indicators were checked and where they came from, what the tenant returned over which window (customer-side evidence, the API key's community). "No CTI alert and no event on twelve indicators over 30 days" is the answer, not "not exposed".

Traps: needs `View query builder data sources` and `Execute query`; without them, deliver the first two parts and say the third was not run, giving the queries for the analyst. A query on `events` with only `timestamp > ago(30d)` is rejected: use `between`. Alerts point at infrastructure or malware more often than at the intrusion set, so include the related IDs. IPs of shared hosting match unrelated traffic; confirm with the domain or URL. Techniques on the actor were not necessarily used against every victim; for behaviours, offer the `report-hunt` skill instead of turning the technique list into a checklist.

## 4. Sector or country brief

Question shapes: "which actors target healthcare", "threats to French telecoms in 2026".

1. `search_cti` with the sector or country word. Take the `identity` object with `x_ic_is_sector: true` (or the location identity); the campaigns returned alongside are candidates whose names mention the term.
2. `get_cti_object_relationships_by_id` on the identity with `target_types = ["intrusion-set"]`, then `["campaign"]`. Read `relationship.confidence` and dates; a `targets` relationship with no date is relevance, not a dated attack.
3. For the actors the user cares about, recipe 1 steps 3 and 4 with `["malware"]` and the reports.

Answer: the actors as Sekoia relates them to the sector, each with confidence and one cited report with its date; the campaigns with `first_seen` in the period asked; then the boundary: sector relevance is not targeting of the user's organisation, and a period filter applies to `first_seen`, not to `modified`.

Traps: sector identities are the most connected objects in the database; the `["intrusion-set"]` slice on Healthcare alone is 56 entries and 443 KB, which overflows even with one target type. Expect the persisted file and rank it with `jq` by `relationship.created` or `relationship.confidence` before answering:

```
jq -r '.result[] | [.relationship.relationship_type, (.relationship.confidence|tostring), (.relationship.created[:10]), .related_object.name, .related_object.id] | @tsv' <file> | sort -t$'\t' -k3 -r | head -20
```

The sector search also returns campaigns from many years; filter on `first_seen` for the period asked and say so. Campaign names carry the vendor's framing ("Lazarus targeting healthcare"); the attribution lives in the `attributed-to` relationship and its confidence, quote that.

## 5. Malware profile

Question shapes: "what is Sliver used by", "profile the malware in case CA…".

1. If a case is named and `get_case` is available, take the malware name from its alerts; otherwise use the name given.
2. `search_cti` with the name; pick the `malware` (or `tool`) object.
3. `get_cti_object_by_id`: description, malware types, kill chain phases, aliases.
4. Graph slices: `["intrusion-set"]` (who uses it), `["infrastructure"]`, `["tool"]`, `["attack-pattern"]`.
5. `get_cti_object_reports_by_id`, most recent first.

Answer: what it is, who Sekoia records as using it with confidence per relationship, infrastructure if any, the most recent reports. Commodity tools (Sliver, Cobalt Strike, Mimikatz) are used by many actors; say that a `uses` relationship does not attribute an incident.

## 6. Vulnerability lookup

Question shapes: "which actors exploit CVE-2023-20198", "is this CVE linked to a campaign".

1. `search_cti` with the CVE identifier; pick the `vulnerability` object.
2. `get_cti_object_by_id` for the description (Sekoia's vulnerability descriptions carry the exploitation mechanism, useful for the answer).
3. Graph slices: `["intrusion-set"]`, `["campaign"]`, `["malware"]`.

Answer: the actors and campaigns with `targets` or `exploits` relationships, each with confidence and the relationship's `created` date when no activity date exists. A campaign's `first_seen` dates the campaign, not the exploitation of this CVE.

## 7. Campaign lookup

Question shapes: "tell me about the Gatling Gun campaign", "what campaigns did X run in 2024".

1. `search_cti` with the campaign name, or recipe 1 with `target_types = ["campaign"]` on the actor.
2. `get_cti_object_by_id` on the campaign: description, `first_seen`, objective, external references (the source URL).
3. Graph slices as needed: `["attack-pattern"]`, `["identity"]`, `["vulnerability"]`, `["malware"]`.

Answer: the campaign in three lines (`first_seen` and the latest relationship date from the slices fetched, what it did, source), attribution as recorded with its confidence, techniques if asked. Campaign objects built from a single vendor post carry that vendor's framing; the `external_references` URL is the primary source.

Traps: many older campaigns are one-line objects with no technique graph; say "description only" rather than inferring techniques.

## 8. Report search

Question shapes: "which Sekoia reports cover Salt Typhoon", "latest FLINT on this actor".

1. `search_cti` with the name; keep the `report` objects (name, `published`, external references) and sort them by `published`: search order is relevance, not date.
2. Or `get_cti_object_reports_by_id` on the object for the full list; expect overflow on well-covered actors and fall back to step 1.

Answer: reports by date, most recent first, with the FLINT reference when the name carries one, and whether each is Sekoia-authored or ingested from a vendor (the `x_inthreat_sources_refs` identity and the external reference say which; any `Sekoia …` source identity counts as Sekoia-authored, see `fields.md`).

## 9. Third-party vetting

Question shapes: "check these domains and IPs before we onboard the supplier".

1. `search_observables` with every value in one call.
2. `get_cti_object_by_id` on any indicated threat.

Answer: a line per value, three outcomes as in recipe 2, with the source and the validity period on every match; expired validity is historical context, current validity is a finding. State the search date. Offer recipe 3, pattern 2, to see whether the values already appear in the tenant's events.

## 10. What changed since a date

Question shapes: "anything new on this actor since June", "what did Sekoia add this month on healthcare".

1. Recipe 1 or 4 to reach the object.
2. Quick check: the latest relationship date on each slice (`fields.md`, Relationship entries). If it is older than the date given, nothing was linked since on that slice.
3. Compare `created` on the related objects and relationships with the date given; `modified` alone does not qualify.
4. For new campaigns, read `first_seen`.

Answer: objects and relationships created since the date, each with its `created` and, when present, `first_seen`; then the boundary: "created since" measures Sekoia's ingestion, and only `first_seen` measures activity. If nothing was created since the date, say so and name the most recent item.

## 11. Actors or campaigns by goal

Question shapes: "which actors carry out sabotage against energy", "who runs lucrative campaigns against French healthcare", "espionage actors targeting telecoms since 2025", "which of APT28's campaigns were sabotage".

Ready-made questions on this recipe: "Which actors engaged in espionage target <sector>?", "Which lucrative campaigns hit <country> since <date>?", "Which actors carrying out sabotage use <malware>?", "Which of <actor>'s campaigns were <goal>?". Goal values are listed in `fields.md` under Goal vocabulary.

No tool lists the database by goal, so the question needs a pivot: a sector, a country, a malware family, a CVE or an actor. Without one, say that a goal-wide list is not available through the MCP tools and ask for the scope. A `search_cti` on the goal word returns objects whose name contains it; present those as name matches, never as the list of actors with that goal.

1. Reach the pivot: recipe 4 step 1 for a sector or country, recipe 5 or 6 for a malware family or CVE, recipe 1 for an actor.
2. Actors: `get_cti_object_relationships_by_id` with `["intrusion-set"]`. Each `related_object` carries its `goals`. Filter on the stem of the value (`fields.md`, Goal vocabulary), which catches casing and typos, on the inline result or on the persisted file:

```
jq -r --arg g sabotag '.result[] | select(.related_object.revoked != true) | select(any(.related_object.goals[]?; test($g; "i"))) | [.related_object.name, .related_object.id, (.related_object.goals|join(",")), .relationship.relationship_type, (.relationship.confidence|tostring)] | @tsv' <file>
```

3. Campaigns: the same call with `["campaign"]`, filtered on the objective, then on `first_seen` for any period asked. The stem also catches each value of a multi-value string ("Sabotage / Espionage"):

```
jq -r --arg g sabotag '.result[] | select(.related_object.revoked != true) | select((.related_object.objective // "") | test($g; "i")) | [.related_object.name, .related_object.id, .related_object.objective, (.related_object.first_seen // "-")[:10], (.relationship.confidence|tostring)] | @tsv' <file>
```

4. List what the stem did not catch, so synonyms are reported rather than dropped, and count the objects with no value:

```
jq -r '.result[].related_object.goals[]?' <file> | sort | uniq -c | sort -rn
jq '[.result[] | select((.related_object.goals // []) | length == 0)] | length' <file>
```

(use `.related_object.objective` and `== null` for campaigns).

Answer: the actors whose recorded goals match, each with relationship type and confidence; the campaigns whose objective matches, each with `first_seen`; the synonyms found and on which objects ("two actors record 'Financial gain'"), kept apart from the matches; the count with no goal recorded; then the boundary: a goal is an actor-level label, and only a campaign's `objective` shows that a given activity had that goal. "Sandworm records sabotage among its goals and targets Energy at 70" does not say Sandworm sabotaged Energy; a campaign against Energy with objective `sabotage` does.

Traps: a sector `["intrusion-set"]` slice can overflow; read it from the persisted file. Some objects carry no goal. Free-text values ("Financial gain", "Ransomware", "Destructive", "Hacktivism") escape the stem filter; folding them into a vocabulary value is an assessment, labelled as such. Revoked duplicates appear in campaign slices; the filters drop them.
