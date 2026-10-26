---
name: query
description: Answer threat-intelligence questions in natural language from the Sekoia MCP server, in conversation rather than as a report. Use when asked "is this IP / domain / hash known to Sekoia", "what does Sekoia know about actor X", "which actors target sector Y", "which actors carry out espionage / sabotage / lucrative campaigns against Y", "enrich this indicator", "are we exposed to Z", "profile this malware", "what is CVE-... linked to", "which reports cover this", "vet these domains", "has Sekoia CTI on X fired in our tenant", "did we see this indicator or actor in our environment", or any ad hoc CTI lookup. Picks the tool sequence, reads the STIX fields correctly, answers with IDs, dates, confidence and platform links, and says what was not found and why. Hands off to the exposure-assessment, threat-horizon-assessment or intelligence-delta skills when the user wants a document. Requires the Sekoia MCP server connected with View intelligence; the tenant check also needs the query builder permissions.
---

# Sekoia CTI query

Turn a natural-language question into the right Sekoia MCP tool sequence, read the results as an analyst would, and answer in a few sentences with evidence. This is the conversational counterpart of the report skills: same evidence discipline, no document.

## Before answering

1. **Answer from the MCP results only.** No web fetches and no model knowledge about actors, campaigns or indicators; `external_references` URLs on objects are provenance to cite, never to open. Unknown means "not in Sekoia CTI", bounded to the exact query run.
2. **Check the tools.** The server lists only the tools the API key allows. The five threat intelligence tools (`search_cti`, `get_cti_object_by_id`, `get_cti_object_relationships_by_id`, `get_cti_object_reports_by_id`, `search_observables`) need `View intelligence`. `run_sol_query` needs `View query builder data sources` and `Execute query`; `get_case` needs `View cases`. If a tool the recipe needs is absent, say which permission is missing and answer with the rest.
3. **Classify the question** into one recipe from `references/recipes.md`: actor profile, indicator enrichment, tenant check, sector or country brief, malware profile, vulnerability lookup, campaign lookup, report search, third-party vetting, what changed since a date, actors or campaigns by goal. A question that spans two recipes runs them in sequence.
4. **Decide the depth.** A yes-or-no lookup stops at the first tool that answers it. A profile pulls the object, its graph slices and its reports. Say which depth you took.

## Tool rules

- **Search is discovery.** `search_cti` returns about twenty objects of mixed types (identity, intrusion set, campaign, malware, report) ranked by relevance: the exact name or alias match first, then objects whose name contains the term, with weak token matches at the tail. The rank is not a date order and not an evidence weight, and the cut at about twenty drops lower-ranked objects on well-covered terms. Take the top hit as the first candidate for a name lookup, check its type, aliases and `revoked`, then retrieve it with `get_cti_object_by_id` before quoting a field. A sector or country search returns the `identity` object with `x_ic_is_sector` or a location marker plus the campaigns whose names mention the term; the identity ID is the pivot for "who targets this sector".
- **One `target_types` value per relationship call** on intrusion sets, malware families and sector identities. The documentation shows three types in one call; on a well-known actor that exceeds the response limit and the client persists the payload to a file instead of returning it. Call `["malware"]`, then `["tool"]`, then `["attack-pattern"]`, then `["identity"]`, then `["campaign"]`, then `["vulnerability"]`, as the question needs. Read confidence, `created`, `modified`, `source_ref` and `target_ref` from the nested `relationship`, not from the related object.
- **Overflow recovery.** When a result is reported as persisted to a file, read it with `jq` from the path given rather than dropping the slice or narrowing the question. Typical: `jq '.result[] | {name: .related_object.name, id: .related_object.id, type: .related_object.type, conf: .relationship.confidence, rel: .relationship.relationship_type}' <file>`.
- **Reports.** `get_cti_object_reports_by_id` returns the report objects with their `description` and `object_refs`, and overflows on well-covered objects; for names and dates alone, the report objects already returned by `search_cti` are enough. On FLINT reports the `description` is the introduction or key takeaways only ("To read more, please see the PDF view"). The MCP does not return the PDF body; the Sekoia API serves it from `x_inthreat_uploaded_files` (see `references/fields.md`). This skill does not download it; when the user needs the report's procedures, offer the `report-hunt` skill, which does.
- **Observables.** `search_observables` takes several values in one call. `found: false` means unknown to the database. `found: true` with an empty `x_ic_indicated_threats` means the value exists in the database (referenced by a report or by an indicator that is no longer valid) with no threat currently linked: report it as "known, no current indicated threat", never as malicious. A populated `x_ic_indicated_threats` names the threats; quote each with its ID and, when present, the indicator validity period. Name the source from `x_inthreat_sources_refs` on every found value; most indicators come from Sekoia C2 Tracker.
- **Own telemetry.** Only with `run_sol_query`, only in the tenant check (recipe 3) or when the user accepts it as a follow-up to an indicator, actor or vetting answer; never run it unasked. Follow `references/tenant_check.md`: queries on `events` bounded with `timestamp between (…)` (the server rejects a lower bound alone), aggregated, with a `limit`; results labelled customer-side evidence about the API key's community: they show whether the threat touched this environment, not what the threat is. For behaviours described in a report, hand off to the `report-hunt` skill.

## Reading the fields

See `references/fields.md` for the field table. The rules that change answers:

- `first_seen` dates the start of activity on any object; `last_seen` is read on indicators only (rarely filled elsewhere, sometimes a placeholder). For how recent an actor, malware or campaign is, give the most recent campaign's `first_seen` (activity) and the latest relationship `created` (Sekoia's knowledge, see `fields.md`). `created` and `modified` date Sekoia's record. A recent `modified` is not recent activity. A relationship with no date of its own has "no independent activity date".
- Confidence is the STIX 2.1 field, 0 to 100, read where it sits: 100 confirmed by other sources, 70 probably true, 50 possibly true, 30 doubtful, absent when empty. An object's confidence says nothing about its relationships.
- `revoked: true` with `x_ic_revocation_reason` marks a duplicate or withdrawn object; use the surviving one. `x_ic_deprecated` marks retired content. `x_ic_is_in_flint: true` means the object is covered by a Sekoia FLINT report.
- Markings: `marking-definition--613f2e26…` is TLP:WHITE, `…34098fce…` TLP:GREEN, `…f88d31f6…` TLP:AMBER. Retrieve any other marking ID before quoting the object outside the session, and state the strictest marking used when the user intends to share the answer.
- `goals` on actors and `objective` on campaigns use one controlled vocabulary (`espionage`, `lucrative`, `sabotage`, `disruption`, `influence`, `reconnaissance`, `pre-positioning`); match it by stem, case-insensitively, and expect free-text synonyms on older records ("Financial gain", "Destructive"), reported as recorded. An actor's goal is not a campaign's objective.
- Sources: `x_inthreat_sources_refs` holds identity IDs. Sekoia has about a dozen source identities, all named `Sekoia …` (Sekoia, Sekoia C2 Tracker, Sekoia Honeypot, Sekoia Malware Watcher, Sekoia Dark Web Monitoring…); every one of them is Sekoia first-party evidence. Resolve an unknown ID with `get_cti_object_by_id` and name the specific source.
- Aliases: Sekoia merges vendor names onto one object (`aliases`); vendors describe overlaps, not identity. Say "Sekoia records X as an alias of Y", not "X is Y".

## Answer shape

Lead with the answer in one sentence: yes, no, or not established, and on what. Then the evidence, one line per object or relationship: name, type, confidence, the date that applies and its field name, and the link `https://app.sekoia.io/objects/<id>`. Keep facts and interpretation apart: "Sekoia records" for retrieved facts, "assessment" for anything inferred. Close with what was not found, bounded to the exact scope searched ("no identity relationship on the campaign object; the intrusion set carries one at 100"), and with the next pivot the user can ask for.

Never combine two true facts into a third (an actor targeting a sector and a country does not target that sector in that country). Never inherit a capability across clusters because the tradecraft matches. Never borrow a date from a linked object. When a question has no evidence answer, say so; do not fill the gap from general knowledge without labelling it as outside Sekoia CTI.

Length: a lookup answers in three to six sentences; a profile in one short section per pivot (profile, graph, reports, gaps). Tables only when the user asks for several objects side by side.

## When to hand off

The user wants a document, a customer-facing brief, a sector RFI, a forecast or a "prove the delta" brief: stop after the first evidence pass, say what you found, and offer the matching skill (`exposure-assessment`, `threat-horizon-assessment`, `intelligence-delta`). Those skills apply the full `evidence-protocol`, whose claim types and retrieval states this skill uses in light form.

The user wants to hunt for what a report or campaign describes ("how do we hunt for this FLINT", "can we see these techniques in our logs"): offer the `report-hunt` skill, which turns the report's behaviours into SOL queries on the tenant.

## Degradation

- No `View intelligence`: no CTI answer; say which permission is missing.
- Relationship or report payload persisted to a file: recover it with `jq`; if the file is unreadable, mark the slice NOT_RETURNED and say the answer is incomplete on that slice. Never present a truncated slice as complete or as absence.
- Rate limit or timeout: retry once, then report the slice as NOT_RETURNED.
- Ambiguous name (several objects match): list the candidates with type and ID and ask which one, unless one is clearly the actor asked about.
