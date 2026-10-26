---
name: exposure-assessment
description: Produce a Sekoia Intelligence Exposure Assessment, a customer-grade report showing how verified threat behaviour could intersect a sector or a named recipient, built from Sekoia MCP CTI evidence. Use when asked for an "exposure assessment", "are organisations like X exposed to actor/CVE/mechanism Y", a "sector RFI", a "technical RFI for government/telecom/finance", or a customer-facing threat report for a vertical. Outputs a markdown report, an evidence manifest and a self-contained HTML rendering. Requires the Sekoia MCP server connected with View intelligence (and the query builder permissions for optional own-telemetry checks).
---

# CTI exposure assessment

Produce the "Sekoia Intelligence — Exposure Assessment" series: a five-page-class report that turns validated Sekoia evidence into an exposure mechanism, operational validation questions and explicit boundaries. The reference worked example is the Financial Services assessment (device-code authentication and token-based cloud access).

Apply the `evidence-protocol` skill for all research, claim classification, manifest and QA. If it is unavailable: search hits are candidates not claims, retrieve exact objects, always use `target_types`, keep the four retrieval states separate, never combine separately true facts, never merge clusters, state the four boundaries.

## Inputs to settle first

- **Anchor**: the threat (actor, cluster set, campaign, CVE, mechanism) and why now.
- **Scope**: a sector (Financial Services, Government and Public Sector, Telecommunications) or a named recipient. A named recipient changes nothing about evidence rules; it only adds a recipient-relevance check whose negative result is stated plainly.
- **Variant**: standard exposure assessment, or sector technical RFI (adds source arbitration across the names recorded on Sekoia objects and ingested reports, evidence layers and a corroboration timeline).
- **Reference ID**: `SEKOIA-EA-YYYY-<SCOPE>-NNN` for exposure assessments, `SEKOIA-SECTOR-RFI-YYYY-<SCOPE>-NNN` for sector RFIs. Ask for the NNN or mark `[TBD]`.

## Research phase (before any drafting)

1. **Discover** with `search_cti`: the anchor plus its aliases, related campaigns, malware, vulnerabilities. Candidate list only.
2. **Validate** each material object with `get_cti_object_by_id`: ID, type, confidence, first seen, latest relationship date (last seen on indicators only), created and modified kept separate.
3. **Graph**: `get_cti_object_relationships_by_id` with **one `target_types` value per call** (`["malware"]`, then `["tool"]`, then `["attack-pattern"]` for capability; `["identity"]` for sector and geography; `["campaign"]` for activity). A combined call on a well-connected object exceeds response limits. Read confidence and dates from the nested `relationship`, not the related object. For attribution or victimology claims also pull what could weaken them. On truncation, record NOT_RETURNED for that slice and query neighbours separately.
4. **Reports**: `get_cti_object_reports_by_id` on the anchors; note names (FLINT references) and publication dates for the source basis.
5. **Clusters**: if several clusters share the mechanism, keep one evidence set per cluster and assign each a role (best end-to-end mechanism, adds infrastructure evidence, adds distinct implementation) without transferring capabilities.
6. **Mechanism**: reconstruct step by step; mark each step evidence-backed or analytical derivation.
7. **Recipient check** (named recipient only): search the recipient across CTI objects; the expected and publishable result is an explicit absence statement bounded to the search scope.
8. **Tenant check** (optional): only when the recipient is the community of the API key, or when the user asks for it. Apply `../query/references/tenant_check.md`: pattern 1 (alerts already raised by Sekoia CTI on the anchor, infrastructure and malware IDs) and pattern 2 (indicator sweep in the events), bounded, aggregated. Results go to the Operational validation section as rows labelled customer-side evidence, with window and tenant. Not run: record NOT_CHECKED in the manifest and say why (recipient is not the key's tenant, permission missing). For the mechanism's behaviours, the `report-hunt` skill builds the queries.
9. **Manifest**: build the evidence manifest rows as you go, per the protocol reference.

## Drafting phase

Write short plain sentences per the protocol step 14; prose carries the meaning, tables carry the detail. Follow the section skeleton in `references/section_templates.md` exactly (section names, card patterns, table columns, boundary texts). Summary of the skeleton:

1. Cover block: kicker, title (a mechanism sentence, not an actor name), subtitle, SCOPE / SOURCE / DATE / REFERENCE, series line, TLP banner (default `TLP:AMBER`; `TLP:AMBER+STRICT` only when a source used requires it, since STRICT stops the recipient sharing with the vendor or MSSP who runs the checks).
2. EXECUTIVE BRIEF: three prose paragraphs (bold lead sentence stating the finding, then the "Why now:" sentences with the evidence age; facts with sources and dates; two-sentence italic boundary paragraph), then the ANCHOR / FRESHNESS / STATUS strip, then THE VALIDATION QUESTION callout. No cards, no three-column block.
3. WHAT SEKOIA INTELLIGENCE OBSERVED: separately tracked observations on a common timeline, with the observation timeline figure.
4. OBSERVED EXPOSURE MECHANISM: the cluster map figure with one sentence per cluster role, the mechanism chain figure in the house pill style with the model step amber, the dashed "separate, not merged" note where an episode must stay apart, the STEP | STATUS | EVIDENCE table.
5. WHY THIS MATTERS FOR <SCOPE>: two paragraphs translating mechanism to business exposure, then the integrity-model chain of controls the operator should be able to prove.
6. OPERATIONAL VALIDATION: applicability note and dated validation window, then the table VALIDATION QUESTION | OWNER | EVIDENCE TO CHECK (sector RFIs may use TEAM and EVIDENCE TO PRODUCE), questions under 30 words, optional hunting pivot callout.
7. PRIORITIES: P1 / P2 / P3 and the Assessment callout, in both variants.
8. EVIDENCE BASIS: confidence key paragraph, then the table OBJECT/SOURCE | CONFIDENCE | TEMPORAL BASIS | ROLE IN ASSESSMENT (a sector relationship with no date reads "No independent activity date"; counts stay in the manifest), then the 2x2 boundary grid, then the FINAL CUSTOMER QUESTION band.

RFI variant adds, in this order: the mechanism step table carries a SOURCES column with Observed / Corroborated / Hypothesis statuses; a "what the sources establish" section after the mechanism (naming ruling, initial-access accounts, counting bases, actor as recorded); a corroboration timeline annex before the evidence basis. Length budget: exposure assessment about 3,500 words, RFI about 5,000, without Mermaid fences.

After the draft passes the protocol QA checklist, apply the protocol's step 14 writing conventions, then step 15 (figures: Mermaid fence in the markdown, inline SVG from `assets/diagram_*.svg` in the HTML, a `Figures` table in the manifest), then fill the HTML. IDs, confidences, dates and retrieval states stay as retrieved.

## Outputs

Write four files, named by the reference ID:
- `<REFERENCE>.md`: the report.
- `<REFERENCE>_manifest.md`: the evidence manifest.
- `<REFERENCE>.html`: filled from the protocol skill's `assets/report_template.html` (self-contained; every token filled, unused optional blocks removed). If the template is unavailable, produce clean semantic HTML with inline styles.
- `<REFERENCE>.pdf`: printed from the HTML with headless Chrome per the protocol's PDF section, after QA, page count checked.

## QA gate

Run the protocol's QA checklist in full before handing over, including the writing-pass items. State in one line to the user which items, if any, remain as limitations. The report is customer-facing: no internal vault paths, no query dumps, IDs and report names only, no filler prose.
