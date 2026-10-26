---
name: threat-horizon-assessment
description: Produce a Sekoia Intelligence Threat Horizon Assessment, a forward-looking customer report assessing how a class of threats (for example AI-enabled attacks) is evolving and what to expect in 3 to 12 months, with confidence levels, confirm/invalidate criteria and explicit intelligence gaps, built from Sekoia MCP CTI evidence. Use when asked for a "threat horizon", "how will X threats evolve", "forecast for customer Y", "AI threat assessment", "strategic threat outlook". Outputs a markdown report, an evidence manifest and a self-contained HTML rendering. Requires the Sekoia MCP server connected with View intelligence.
---

# CTI threat horizon assessment

Produce the UOB-style horizon report: a small number of verified mechanisms, what materially changed, a falsifiable 3 to 12 month forecast and the gaps. The value is discipline: every forecast carries the evidence anchor behind it and the observation that would invalidate it.

Apply the `evidence-protocol` skill throughout. If unavailable: search hits are candidates not claims, retrieve exact objects, keep dates and confidence semantics, separate observed from emerging, never combine separately true facts, state the recipient boundary.

## Inputs to settle first

- **Theme**: the threat class under assessment (AI-enabled operations, identity attacks, edge-device espionage…).
- **Recipient and scope**: named customer plus the teams addressed (CTI, SOC, fraud, identity, cloud security, third-party risk), sector and geography.
- **Window**: the evidence window (typically the last 6 months) and the forecast horizon (3 to 12 months).
- **Reference ID**: `SEKOIA-<RECIPIENT>-<THEME>-YYYY-NNN` (worked example: `SEKOIA-UOB-AIHORIZON-2026-001`).

## Research phase

1. **Candidate mechanisms**: `search_cti` across the theme; shortlist 3 to 5 **independent** mechanisms, each with its own evidence anchor (a campaign, intrusion set or report), not five retellings of one campaign.
2. **Validate** each anchor with `get_cti_object_by_id`; record first seen, the latest relationship date, created and modified separately. Freshness = the most recent campaign `first_seen` (activity) and the latest relationship date (Sekoia's knowledge); `last_seen` only on indicators; never the modification date.
3. **Graph**: relationships with **one `target_types` value per call** (`["malware"]`, `["attack-pattern"]`, `["identity"]` for sector and geography); combined calls overflow on well-connected objects. Read confidence and dates from the nested `relationship`. Pull contradicting context on any attribution or targeting statement.
4. **Reports**: `get_cti_object_reports_by_id` per anchor for sourcing and publication dates.
5. **Recipient relevance check**: search the recipient name and its sector and geography across CTI objects. The publishable outcome is usually the explicit negative: "No verified campaign currently links <recipient> to the activity reviewed", bounded to the search scope. Sector or geography relationships are relevance, never targeting of the recipient.
6. **Observed vs emerging**: split mechanisms into observed (validated activity) and emerging signals held to "a different evidentiary standard", labeled as such.
7. **Tenant baseline** (optional): only when the recipient is the community of the API key, or when the user asks. Pattern 1 of `../query/references/tenant_check.md` on the anchor IDs gives the current baseline of Sekoia CTI alerts for "What <recipient> Should Watch"; label it customer-side evidence with window and tenant. Not run: NOT_CHECKED in the manifest.
8. **Manifest** rows as you go.

## Drafting phase (section skeleton)

Write short plain sentences per the protocol step 14; prose carries the meaning, tables carry the detail.

1. Cover block: kicker `<THEME ALL CAPS> · TECHNICAL ASSESSMENT · <MONTH YEAR>`, title, subtitle ("<N> verified mechanisms by which <theme> is entering real attacker operations, assessed against Sekoia Intelligence"), Scope (recipient teams), reference, evidence window and forecast horizon, TLP banner.
2. **Executive Summary**: what changed in one paragraph, what did not, the recipient-relevance statement. Per the protocol step 14 conventions, detailed metadata goes to the Anchor / Freshness / Status triples and the tables; the summary stays plain prose. Carries a "Why now:" sentence pair with the trigger and the age in months of the most recent evidence.
3. **<N> Material Changes**: one numbered subsection per mechanism, titled by function ("1. Create trust", "2. Compromise and monetise identity"). Each carries:
   - an Anchor / Freshness / Status triple (Status states recipient relevance or its absence: "No <recipient>-specific evidence: sector/geography relevance only")
   - what materially changed, evidence-cited
   - an arrow chain strip of the mechanism (`Lure → step → step → outcome`); when the chain has four or more steps, also the pill-chain figure (protocol `assets/diagram_chain.svg`, Mermaid flowchart in markdown)
4. **Emerging Attack Surface**: the signals that do not meet the observed bar, explicitly flagged as held to a different evidentiary standard.
5. **Why This Matters for <recipient> and <geography>**: sector translation; every downstream scenario labeled EXPOSURE MODEL.
6. **3 to 12 Month Horizon**: the horizon strip figure (protocol `assets/diagram_horizon.svg`: evidence window, today, 3 and 12 month marks, one diamond per forecast with its confidence), then the forecast table, columns exactly `Forecast` | `Confidence` | `Would confirm / invalidate`. Confidence scale in words: Low, Low-Medium, Medium, Medium-High, High. Every row has a falsifiable criterion in both directions ("two quiet quarters would weaken; a full year with none would invalidate"). Add a `Strategic trigger` callout for the one event that would change the whole picture.
7. **What <recipient> Should Watch**: observable early indicators tied to telemetry.
8. **What <recipient> Should Do**: action table `#` | `PRIORITY` | `ACTION`, about 5 rows, each traceable to a mechanism.
9. **Intelligence Gaps and Next Collection Priorities**: the NOT_RETURNED and NOT_CHECKED items, stated as gaps.
10. **Methodology**: two paragraphs citing the evidence protocol and opening with the confidence key (search as discovery, object validation, claim types, retrieval states, boundaries).

## Outputs

- `<REFERENCE>.md`, `<REFERENCE>_manifest.md`, `<REFERENCE>.html`, `<REFERENCE>.pdf` (printed from the HTML per the protocol's PDF section) filled from the protocol skill's `assets/report_template.html` (forecast table uses the generic table block; chain strips use the chain block). Every token filled, unused blocks removed.

## QA gate

Run the protocol QA checklist, apply the step 14 writing conventions, then step 15 (figures with a `Figures` table in the manifest) before filling the HTML. Extra horizon checks: every forecast row has an invalidation criterion; no forecast rests on a `modified` date; observed and emerging never mix in one section; the recipient-relevance statement appears in the executive summary and is bounded to its search scope.
