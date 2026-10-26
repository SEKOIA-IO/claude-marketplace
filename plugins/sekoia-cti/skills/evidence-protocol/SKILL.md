---
name: evidence-protocol
description: Analytical safeguards for building intelligence assessments, exposure assessments, RFIs or validation material from Sekoia MCP server data. Apply whenever CTI retrieved through the Sekoia MCP tools (search_cti, get_cti_object_by_id, relationships, reports, search_observables) becomes the basis of a customer-facing or material claim, and whenever a Sekoia CTI report skill is invoked. Covers evidence validation, retrieval states, claim classification, cluster discipline, analytical boundaries, the evidence manifest and release QA. Requires the Sekoia MCP server connected with View intelligence.
---

# Sekoia CTI evidence protocol

Core principle: **the MCP is an evidence-retrieval layer, not a conclusion engine.** Search results support discovery. Material claims rest on retrieved objects and relationships that were inspected and validated before use. These safeguards live at agent level; the server does not provide them.

**Single intelligence source.** Sekoia CTI, retrieved through the MCP tools, is the only intelligence source. Third-party reporting counts only when it is ingested in Sekoia Intelligence as an object: cite it by its Sekoia object ID and read its provenance from `x_inthreat_sources_refs` and `external_references`. Sekoia first-party evidence is anything whose source identity is named `Sekoia …`: the main Sekoia identity and its collection sources (Sekoia C2 Tracker, Sekoia Honeypot, Sekoia Malware Watcher, Sekoia Dark Web Monitoring and the rest); resolve an unknown source ID with `get_cti_object_by_id` and name the specific source. URLs on objects are provenance to cite, never to fetch. Never quote a document from outside the base and never fill a gap from model knowledge: what Sekoia CTI does not hold is a stated gap. Customer telemetry through `run_sol_query` is the one other evidence class and stays labeled customer-side.

## Research workflow

### 1. Discover candidate intelligence

Use `search_cti` and `search_observables` to identify candidate threat actors, intrusion sets, campaigns, malware, infrastructure, indicators, vulnerabilities, reports, identities and relationships. Treat every search hit as candidate evidence. A search hit alone never becomes a material claim.

### 2. Retrieve and validate exact objects

For each object used materially, call `get_cti_object_by_id` and validate where applicable: object ID, type, name, source and provenance, confidence, `first_seen`, `last_seen` (indicators only), `valid_from`, `valid_until`, `created`, `modified`, publication date, and the latest relationship `created` on the slices fetched.

Hard rules:
- **Recent modification does not establish recent threat activity.**
- **High confidence does not mean certainty or current activity.**
- `created`, `modified`, `first_seen`, `last_seen` and publication date are never interchangeable. `last_seen` is read on indicators only: on other objects it is rarely filled and sometimes a period-end placeholder. For an actor, malware or campaign, the latest relationship `created` dates Sekoia's knowledge and the most recent campaign `first_seen` dates activity. When a relationship has no activity date of its own, record "No independent activity date"; never borrow a date from a linked object.
- Confidence belongs to the object or relationship it was read from. It does not transfer to other relationships involving that object.

### 3. Retrieve the graph context

Important claims never rest on an object's description alone. Call `get_cti_object_relationships_by_id` with **one `target_types` value per call**: on a well-connected object, even a three-type call exceeds response limits. Each result entry nests `{related_object, relationship}`; read confidence, `created`, `modified`, `source_ref` and `target_ref` from the **relationship**, not from the related object. This establishes what is connected to what, through which relationship, with which confidence, provenance and temporal basis.

Where the claim is sensitive (attribution, targeting, victimology), also query relationship classes able to **weaken or contradict** it: alternative attribution, other victims, other operators. Retrieving only supporting evidence is cherry-picking.

**Oversized graphs:** if a relationship payload is truncated or too large, do not abandon the object. Use a bounded strategy: filter by relationship type, filter by `target_types`, retrieve in smaller groups, query the most relevant neighbours separately. Filtering is a retrieval strategy, never permission to narrow the analysis to supporting evidence.

Pull the reports behind material objects with `get_cti_object_reports_by_id`. The payload embeds full report content and can exceed response limits on well-covered objects; when truncated, treat completeness as NOT_RETURNED and fall back to the report objects already returned by `search_cti` for names and publication dates.

### 4. Record retrieval states

Every material retrieval ends in exactly one state. Read `references/claim_types.md` for the full definitions.

| State | Meaning |
|---|---|
| VALUE | Usable data returned |
| EMPTY | Query completed, nothing matched the exact scope. Supports an absence statement only if the query was exhaustive for that claim |
| NOT_RETURNED | Failed, timed out, truncated or inconclusive. **Not evidence of absence.** Retry bounded or use another strategy; if still inconclusive the claim stays unresolved |
| NOT_CHECKED | Never queried |

The three non-VALUE states are analytically different and are never silently conflated.

### 5. Classify every material claim

Before drafting, assign each important statement one type: **FACT**, **CALCULATED RESULT**, **ASSESSMENT**, **EXPOSURE MODEL** or **NOT ASSERTED**. Definitions and examples in `references/claim_types.md`. FACT and CALCULATED RESULT stay visually and verbally separate from interpretation; EXPOSURE MODEL is always labeled as a model; NOT ASSERTED items are written out, not silently dropped.

### 6. Never combine separately true facts into an unverified claim

Two supported statements do not support their combination. "Actor X relates to Financial Services" plus "Actor X relates to Italy" does not establish "Actor X targets Italian financial institutions". A TTP of an actor was not necessarily used against every victim of that actor. Before any combined statement, confirm the evidence supports the combination itself.

### 7. Keep clusters, campaigns and attribution separate

Technical convergence does not establish common attribution. Three clusters doing token theft justify "analytical convergence around token-based cloud access", never shared operators or a common campaign, unless a retrieved relationship establishes it. Each cluster keeps its own objects, relationships, confidence, timeline, attribution status and capabilities. No capability is inherited across clusters because the technical result is similar.

### 8. Reconstruct the evidenced mechanism

Build the mechanism step by step (for example: phishing lure → device-code authentication → token acquisition → cloud API access → mailbox access). Each step is either directly supported by validated evidence or explicitly marked as an analytical derivation.

### 9. Convert the mechanism into customer validation questions

For each meaningful step ask: what customer-side evidence could confirm or refute this activity? Questions must be tied to an identifiable telemetry source, bounded to the relevant technology and answerable with defined criteria. "Were unexpected device-code authentication flows completed during the period?" beats "Monitor suspicious authentication."

Keep the distinction explicit: **Sekoia evidence establishes the threat mechanism; customer telemetry establishes whether it intersected the customer's environment.** The latter is not Sekoia evidence. When the report also checks the reader's own Sekoia community with `run_sol_query`, run it only when that community is the recipient, bound queries on `events` with `timestamp between (…)` (the server rejects a lower bound alone), aggregate, `limit`, and label the results customer-side evidence with window and tenant. Turning a described behaviour into a hunt query is detection engineering, not intelligence: label such queries hunt hypotheses, never claims about the threat.

### 10. Map questions to telemetry and an indicative owner

| Mechanism | Indicative owner |
|---|---|
| Authentication | IAM / SOC |
| Token activity | IAM / DFIR |
| Cloud or API activity | SOC / Cloud Security |
| Mailbox activity | Messaging Security / DFIR |
| Endpoint behaviour | EDR / DFIR |
| Financial workflow | Fraud / Treasury / IR |

These are operational mappings, not intelligence facts; ownership is confirmed with the customer.

### 11. Apply the four boundaries

Every customer-facing assessment states at least:
- **ACCOUNT BOUNDARY**: no recipient-specific targeting or compromise is asserted without recipient-specific evidence.
- **TECHNOLOGY BOUNDARY**: deployment of the technology is not assumed; the validation applies where it is deployed.
- **CLUSTER BOUNDARY**: comparable tradecraft does not establish common attribution or shared operators.
- **EXPOSURE-MODEL BOUNDARY**: an unobserved downstream scenario is an analytical exposure model, not an observed incident.

### 12. Maintain the evidence manifest

One row per material claim, template and examples in `references/evidence_manifest.md`. The confidence-key source and any campaign list quoted as a count get rows too. The chain must hold: report claim → manifest row → exact Sekoia object or relationship ID.

### 13. Final QA before release

Walk `references/qa_checklist.md` in full. Any unresolved item stays visible in the report as a limitation; missing or failed retrievals are never converted into plausible knowledge or into evidence of absence.

### 14. Writing before rendering

Length budget, counted without Mermaid fences: exposure assessment about 3,500 words, sector RFI about 5,000, horizon assessment about 3,500, delta brief about 2,500. Evidence tables count; when the budget is exceeded, merge tables that cite the same evidence twice before cutting facts.

Write the report as a human analyst writes for a customer: plain, sober, factual, no filler. The reader is busy; the report must read fast.

Short and plain, in every prose block:

- One idea per sentence. Most sentences under 20 words, none over 30 except quoted source wording.
- Plain words over formal ones. Active voice unless the actor is unknown. Verbs over nominalisations outside quoted terms.
- Paragraphs of 2 to 4 sentences, one topic each. The first sentence of every section and paragraph states the point.
- Prose says what the evidence means; tables, strips and figures carry the detail. Never restate in prose what a table on the same page lists.
- Say each thing once. Later sections reference it instead of repeating it.
- No filler openers, no transition words, no closing summaries.

Series conventions that hold in every report, manifest and HTML:

- The two fixed series strings keep their em dash: the series line ("Sekoia Intelligence — Exposure Assessment") and the TLP banner sentence. They reproduce the published PDFs. No other em dash in the report, manifest or HTML.
- Quoted source wording, object descriptions, technique names and report titles are not rewritten.
- IDs, confidence values, dates, retrieval states, claim types and counts are never changed by editing. After any editing pass, the set of IDs, confidences and dates in the report and manifest must equal the set before it.
- Detailed metadata lives in the Anchor / Freshness / Status strip and the evidence tables; the executive brief stays readable prose.
- Tables stay tables. Status and priority labels use a colon: `Observed: technically confirmed`, `P1: <name>`. Card titles keep the series all-caps form.
- Section subheads are sentence case and state the finding.

### 15. Figures

Choose figures from `references/diagrams.md`: a figure earns its place when it shows a mechanism, a separation or a timeline the reader would otherwise assemble from prose; a sentence beats a three-step chain, a single cluster or a two-date timeline. Each figure is drawn twice from one step, label or date list: a Mermaid fence in the markdown, an inline SVG filled from the matching `assets/diagram_*.svg` template in the HTML. Every label, date, confidence and count in a figure maps to a manifest row, and the manifest carries a `Figures` table (figure, claim, rows drawn on). The caption states the claim in one sentence; the SVG carries the same claim in `aria-label`. No script, style or external reference inside any SVG.

## Rendering

Reports rendered as HTML use `assets/report_template.html`: one self-contained file, Sekoia Intelligence series design, `{{TOKEN}}` placeholders documented inside the template. Fill every token; delete optional blocks that do not apply rather than leaving placeholders. The template embeds the Sekoia brand fonts (Inter, Source Serif 4, Dosis) as base64 in its first style block: never modify that block, and edit the template with python string operations rather than reading it whole. Figures come from `assets/diagram_*.svg` (see step 15), pasted inline as `<figure>` blocks. The opening section is an executive brief in prose (lead sentence, facts, italic boundary paragraph, Anchor / Freshness / Status strip, validation question); there are no card rows or three-column blocks in the series. Default marking is TLP:AMBER; TLP:AMBER+STRICT only when a source used requires it. Every brief carries a "Why now:" sentence pair with the age of the most recent evidence, and every evidence basis opens with the confidence key (STIX 2.1 confidence, Admiralty anchors 100 / 70 / 50 / 30, read where it sits). Sekoia object IDs in the evidence basis table link to `https://app.sekoia.io/objects/<id>` with the full ID as visible text, followed by the line "Object links open in the Sekoia platform and require a Sekoia account with CTI access."; relationship IDs and manifest IDs stay plain. The cover carries the Sekoia wordmark from `assets/brand/`, inlined as SVG.

### PDF

Every report ships as `<REFERENCE>.pdf` next to the markdown, manifest and HTML; the PDF is printed from the finished HTML, never authored separately. Command, run after the HTML passes QA:

```
"<chrome>" --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf="<dir>/<REFERENCE>.pdf" "file://<dir>/<REFERENCE>.html"
```

`<chrome>` is any Chrome or Chromium binary: a Playwright Chromium under `~/Library/Caches/ms-playwright` (macOS) or `~/.cache/ms-playwright` (Linux), the Google Chrome application (`/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` on macOS), or `google-chrome` or `chromium` on the PATH. The template's `@page` rule sets A4 with 16 mm by 14 mm margins and lets sections flow; figures, callouts, the strip, the priorities band, the boundary grid and table rows stay unbroken and table headers repeat. A report prints at 8 to 14 pages. Check the page count and open pages 1 and 2 before handing over; the first page holds the cover and the start of the executive brief, never the cover alone. If no Chrome binary is available, say so and deliver the three other files; never substitute a PDF made from the markdown.

## Degradation

- `View intelligence` missing: no CTI claims at all; say so.
- `run_sol_query` missing: skip customer-telemetry cross-checks, mark them NOT_CHECKED.
- No Chrome binary: deliver markdown, manifest and HTML, state that the PDF was not produced and give the print command.
