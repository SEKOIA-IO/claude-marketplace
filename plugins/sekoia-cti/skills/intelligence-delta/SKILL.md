---
name: intelligence-delta
description: Produce a Sekoia Intelligence Delta technical brief, the "not another feed" answer for an MSSP or mature CTI team that already aggregates multiple sources. One verified first-party Sekoia finding traced end to end, plus a falsifiable test protocol the prospect runs against their existing stack, where "no material delta" is an allowed outcome. Use when asked to "prove the delta", "answer the not-another-feed objection", "show Sekoia adds value over their current sources", "technical brief for MSSP X". Outputs a markdown brief, an evidence manifest and a self-contained HTML rendering. Requires the Sekoia MCP server connected with View intelligence.
---

# CTI intelligence delta brief

Produce the Datacom-style brief: one first-party Sekoia finding, its full evidence chain and a test the prospect can fail us on. Credibility comes from falsifiability: the brief states plainly that if there is no measurable delta, the prospect should not add Sekoia.

Apply the `evidence-protocol` skill throughout. If unavailable: exact objects only, relationship confidences quoted per relationship, counts as CALCULATED RESULTs with methodology, dates keep their semantics, no claim beyond the retrieved evidence.

## Inputs to settle first

- **Prospect**: name, type (MSSP, managed services, integrator), what they already have (their stated stack and objection).
- **The one finding**: prefer a case sourced from **Sekoia-operated collection** (honeypots, TDR-run infrastructure) so the first-party claim is verifiable on the object's provenance. Verify provenance on the object itself; if the source field does not establish first-party collection, pick another finding or drop the first-party wording.
- **Reference ID**: `SEKOIA-DELTA-YYYY-<PROSPECT>-NNN`, series `Technical Brief`.

## Research phase

1. **Select and validate the finding**: `search_cti` then `get_cti_object_by_id` on the campaign or cluster. Record provenance, confidence, first observed date.
2. **Trace the chain**: `get_cti_object_relationships_by_id` with one `target_types` value per call: `["malware"]`, then `["attack-pattern"]`, then `["indicator"]`; combined calls overflow. Read confidence and dates from the nested `relationship`. For each branch record the relationship confidence and whether it is direct from the campaign or chained through another object; the brief states which ("relationship confidence 100, direct from the campaign, not a chain").
3. **Count as CALCULATED RESULTs**: indicators by type (URLs, hashes, IPs), techniques, related objects, with the counting method stated. Never round.
4. **Reports**: `get_cti_object_reports_by_id` for the write-up trail and publication dates (timing is one of the test criteria).
5. **Manifest** rows as you go; this brief lives or dies on lineage.

## Drafting phase (section skeleton)

Write short plain sentences per the protocol step 14; prose carries the meaning, tables carry the detail.

1. Cover block: kicker `Technical Brief · <Month Year>`, title in the "Not Another Feed" register ("Not Another Feed: Testing Incremental Intelligence Value"), subtitle ("One verified Sekoia finding. One question: does it add anything operationally useful?"), SCOPE / SOURCE / DATE / REFERENCE, TLP banner.
2. **The finding**: the campaign in one page, opening with a "Why now:" sentence pair (trigger and evidence age in months). What Sekoia operated and observed, first observed date, the evidence chain as three branches (malware, ATT&CK techniques, indicators) with per-branch relationship confidence and the direct-vs-chained statement, the counts. Figure: the evidence tree (protocol `assets/diagram_tree.svg`, Mermaid flowchart in markdown) with the confidence and direct-or-chained on each edge and the counts on the leaves.
3. **Operational Value**: "What could <prospect> do with this?" in three columns Detect / Hunt / Investigate, each mapped to where it lands in their stack (SIEM, EDR, network, auth) and assessed on Coverage, Context, Effort, Timing, Scale.
4. **The Test**: "You already have intelligence. Does this add anything?" Probe questions as short prompts: Was it already known? When did it become available to you? Same semantic depth? Already reconstructed as connected objects? How much manual work otherwise? Can one finding be assessed across multiple eligible customers?
5. **The two-outcome table**, both outcomes legitimate:
   - `A measurable delta`: a new hunt or detection hypothesis · additional context or pivots · less analytical reconstruction · a useful customer-telemetry gap revealed.
   - `No material delta`: equivalent intelligence already available · no change to an analyst workflow · limited relevance to managed environments · operational cost exceeds the benefit.
   Close with the falsifiable sentence: "If there is no measurable delta, <prospect> should not add Sekoia."
6. **Test criteria**: coverage, timing, context depth, relationships, operational value, analyst effort, each with how to measure it against their existing sources.
7. **Boundaries and redistribution**: the brief is TLP-bound working material; MSSP redistribution of Sekoia content to end customers follows the licensing terms, stated in one line, commercial details out of scope.
8. **Proposed next step**: one bounded working session to run the test on their data.

## Outputs

- `<REFERENCE>.md`, `<REFERENCE>_manifest.md`, `<REFERENCE>.html`, `<REFERENCE>.pdf` (printed from the HTML per the protocol's PDF section) from the protocol skill's `assets/report_template.html` (the two-outcome table uses the generic table block; probe questions use callouts). Every token filled, unused blocks removed.

## QA gate

Run the protocol QA checklist, apply the step 14 writing conventions, then step 15 (the evidence-tree figure with a `Figures` table in the manifest) before filling the HTML. Extra delta checks: the first-party provenance claim is backed by the object's source field; every count has a methodology; the "no material delta" outcome is present and not softened; nothing in the brief asserts what the prospect's stack contains (that is exactly what the test measures).
