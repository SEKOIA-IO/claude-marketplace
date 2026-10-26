# Exposure assessment section templates

Contents: cover block · executive brief · observed · mechanism · why it matters · operational validation · priorities · evidence basis · RFI variant blocks · figures · tone anchors

The reference layout is the published Salt Typhoon telecommunications RFI: prose executive brief, Anchor / Freshness / Status strip, pill-chain figures, tables, a dashed "separate, not merged" note where two episodes must stay apart, a priorities band and one closing assessment. No card rows, no three-column text blocks.

## Cover block

- Kicker (all caps): `<SCOPE ALL CAPS> EXPOSURE ASSESSMENT · <MONTH YEAR>` or `<SCOPE> TECHNICAL RFI · <MONTH YEAR>`
- Title: a mechanism sentence, never a bare actor name. Example: "How Compromised Network Infrastructure Can Become Long-Term Espionage Exposure"; "When Legitimate Cloud Authentication Becomes Financial-Workflow Exposure".
- Subtitle: mechanism keywords in one line, italic in the series ("Lessons from Salt Typhoon and RedMike: edge-device exploitation, router persistence, GRE tunnelling…").
- Metadata rows: SCOPE (sector or recipient) · SOURCE (always `Sekoia Intelligence`) · DATE · REFERENCE.
- Series line: `Sekoia Intelligence — Exposure Assessment`.
- Classification: `TLP:AMBER — Restricted to Sekoia and authorised recipient-organisation personnel only.` (STRICT only when a source used requires it)
- Brand row above the kicker in HTML: the Sekoia wordmark, from the protocol `assets/brand/`; the markdown cover carries no image.
- No running header above the wordmark: the series line and the TLP banner below the metadata carry that information. Footer: `Sekoia Intelligence` / `<SCOPE> · <REFERENCE>`, no page number (the static footer prints once).

## 1. EXECUTIVE BRIEF

Subhead: one positive sentence stating the core finding ("Configuration changes outlive the patch"). Avoid the "not X, it is Y" shape; one such pivot per section at most.

Three paragraphs, in this order, no cards:

1. **Lead paragraph.** Opens with one bold sentence that states the finding a reader in the sector does not already know ("Sources disagree on how Salt Typhoon enters carrier routers and agree on what it leaves behind"), never a restatement of the anchor's existence. Then a "Why now:" sentence pair: the trigger for this edition and the age of the most recent evidence in months ("The most recent carrier-device evidence is the August 2025 advisory, thirteen months before this edition"). Older lead style for reference: one bold sentence that states the assessment ("Salt Typhoon should be treated as a telecommunications infrastructure espionage case."). The rest of the paragraph says what the exposure is for the scope, in two or three sentences. This is the ASSESSMENT.
2. **Facts paragraph.** Two to five short sentences, in this order: the actor and when Sekoia started tracking it, the one campaign that anchors the mechanism, the confirming ingested report, what was taken. One source per sentence, one date per sentence, dates as month and year. No parenthesis and no ISO date in the brief: confidence values, `first_seen` fields, the latest relationship date and relationship counts go to the strip below and to the Evidence basis table. No inventory sentences: a list of campaigns lives in section 2, not here. This is the FACT paragraph; nothing in it is interpretation. Worked example: "Sekoia has tracked Salt Typhoon since 2019. Three of its thirteen campaigns concern carrier network devices, the most specific being the Cisco IOS XE exploitation wave of December 2024. In February 2025 the Canadian Cyber Centre confirmed three compromised routers at one Canadian operator, with configurations retrieved and a GRE tunnel added. In November 2024 the FBI and CISA stated that call records, some private communications and court-ordered request data had been taken at unnamed US operators." Every source named in the brief must be an object ingested in Sekoia Intelligence, cited in the evidence basis by its Sekoia ID.
3. **Boundary paragraph, italic.** Two sentences at most. Opens "This is a sector exposure-validation brief." and names the one thing not asserted that matters most (no recipient-specific compromise; the exposure model is not an observation). The full boundary set lives in the evidence basis grid; the brief does not repeat it. This is the NOT ASSERTED content.

Then the strip `ANCHOR | FRESHNESS | STATUS`, which carries the metadata the prose leaves out: anchor object with its Sekoia confidence and the aliases that matter ("Salt Typhoon, Sekoia intrusion set, confidence 70; aliases RedMike, Earth Estries"); freshness as the ISO first_seen, the latest relationship date and the latest source date ("first seen 2019-01-01; latest relationship 2026-02-28; advisory 2025-08-27"); last_seen appears only for indicators; status states what is confirmed and what is unresolved ("web UI path confirmed; credential path corroborated; recipient unresolved") or recipient relevance or its absence. In a sector variant with no named recipient the status reads "sector relevance only, no recipient assessed", never "recipient unresolved".

Then the callout `THE VALIDATION QUESTION`: one sentence a CISO can carry into a meeting, naming the teams and the telemetry.

## 2. WHAT SEKOIA INTELLIGENCE OBSERVED

Subhead: "Separately tracked observations on a common <year> timeline". State: "These are separately tracked observations of comparable tradecraft, not one campaign evolving into another."

Figure: the observation timeline (`diagram_timeline.svg`, Mermaid gantt in markdown) when three or more dated observations exist.

Per cluster or campaign: an entry titled with the campaign name (bold in markdown, `h5` in HTML), its own first-seen date, confidence and evidence in three or four sentences. Any other paragraph label in the report (attribution context, naming, counting bases, anti-double-counting note, limitations) is a plain sentence-case `####` heading or a first sentence, never a bold run-in.

## 3. OBSERVED EXPOSURE MECHANISM

Subhead pattern: "One <surface>, <n> separately tracked clusters".

Figure: the cluster map (`diagram_clusters.svg`) when two or more clusters are tracked, then one sentence per cluster stating its role:
- "EvilTokens: best end-to-end observed mechanism."
- "Kali365: adds infrastructure and behavioural detection evidence."
- "TokenApp: adds a distinct implementation and PRT-based access capability."

Figure: the mechanism chain (`diagram_chain.svg`, pill style, the model step amber dashed). In markdown the same steps appear as the arrow strip and as a Mermaid flowchart. Under the chain, the dashed note `SEPARATE, NOT MERGED` when a second episode must not be joined ("initial-access method for the 2024 US telecom compromise is not publicly confirmed").

Then the STEP | STATUS | EVIDENCE table and the roles paragraph: "These roles are complementary but not interchangeable: no capability is transferred from one cluster to another merely because each can produce <result>. Capabilities remain cluster-specific and must not be inherited."

## 4. WHY THIS MATTERS FOR <SCOPE>

Subhead: "From <access> to <business> exposure". Two paragraphs: the first says why this sector holds the asset the mechanism reaches; the second says which architectures share the structural exposure regardless of whether they were named. Every downstream scenario is an EXPOSURE MODEL and reads as one.

Figure: the integrity model, a pill chain of the controls the operator should be able to prove, last pill highlighted (reuse `diagram_chain.svg`). Caption: "The purpose is to validate exposure and investigation readiness, not to conclude that <actor> has targeted this operator."

Optional table `SURFACE | RELEVANT MECHANISM` listing the persistence or exposure surfaces (RFI variant).

## 5. OPERATIONAL VALIDATION

Subhead: "What <teams> should validate". Open with the applicability note: "Where <technology> is deployed, the validation questions below apply directly; equivalent telemetry applies where architectures differ." Then the validation window as a dated sentence ("The validation window starts in July 2023, the first_seen of the earliest campaign used here"); "the relevant period" is never left undefined.

Table, 5 to 9 rows ordered along the mechanism: `VALIDATION QUESTION` | `OWNER` | `EVIDENCE TO CHECK`. Owners are paired roles (IAM / SOC, IAM / DFIR, SOC / Cloud Security, Messaging Security / DFIR, Fraud / Treasury / DFIR, Network Engineering / SOC). RFI variant uses `VALIDATION QUESTION` | `TEAM` | `EVIDENCE TO PRODUCE`.

Questions are testable, bound to a telemetry source and to the dated window, and under 30 words; port numbers, protocol lists and tool names go in the evidence column, not in the question. Prefer "Were unexpected device-code authentication flows initiated or successfully completed in the window?" over "Monitor suspicious authentication."

## 6. PRIORITIES

Mandatory in both variants. Subhead: "Three actions in order, then the assessment". `P1: <name>` / `P2: <name>` / `P3: <name>`, one paragraph each, each traceable to a mechanism step and to validation rows, closing on an `Assessment:` callout that names the two facts only the recipient can establish.

Optional boxed hunting pivot (one concrete pivot a hunter can run today, named after its cluster).

## 7. EVIDENCE BASIS

Subhead: "What the evidence establishes, and its boundaries".

Open with the confidence key, one paragraph: Sekoia confidence is the STIX 2.1 confidence field set on each object and each relationship separately, 0 to 100, documented against Admiralty credibility (100 confirmed by other sources, 70 probably true, 50 possibly true, 30 doubtful, intermediate values between anchors); read where it sits, never transferred; "absent" when the field is empty. Relationship counts (13 campaign, 18 vulnerability…) live in the manifest, not in the table cells.

Table `OBJECT / SOURCE` | `CONFIDENCE` | `TEMPORAL BASIS` | `ROLE IN ASSESSMENT`, one row per material object and report. Confidence exactly as retrieved; a sector or geography relationship with no date reads `No independent activity date`. RFIs with many ingested sources may use `OBJECT / SOURCE` | `TYPE` | `CONF.` | `HOW USED` and mark unscored ingested sources `n/a`; every row stays a Sekoia object.

Every Sekoia object ID in this table (intrusion set, campaign, vulnerability, malware, tool, identity, report) is a link to its platform page, `https://app.sekoia.io/objects/<id>`, with the full ID as the visible text so the row stays usable on paper. Relationship IDs are never linked, the manifest keeps bare IDs, and no other section prints IDs. Markdown form `[<id>](https://app.sekoia.io/objects/<id>)`; HTML form `<a href="https://app.sekoia.io/objects/<id>"><code><id></code></a>`. One line follows the table: "Object links open in the Sekoia platform and require a Sekoia account with CTI access."

Boundary grid, 2x2, adapt the nouns, keep the shape:
- `ACCOUNT BOUNDARY`: "No recipient-specific targeting or compromise is established, and this remains a sector exposure assessment rather than an account-evidenced incident."
- `TECHNOLOGY BOUNDARY`: "<Technology> deployment is not assumed for the recipient; equivalent telemetry applies where architectures differ."
- `CLUSTER BOUNDARY`: "<Clusters> remain distinct; comparable mechanics do not establish shared operators or inherited capabilities."
- `EXPOSURE-MODEL BOUNDARY`: "The <workflow> progression is an analytical validation model, not an observed recipient incident, and implies no independent activity date."

Limitations as a short bullet list under a plain `####` label.

Close on the `FINAL CUSTOMER QUESTION` band: one sentence a CISO can carry into a meeting.

## RFI variant blocks

- **What the sources establish** (after the mechanism, never before it): the naming ruling when ingested sources use different names (aliases on the Sekoia objects, names in ingested reports; pick one neutral term, state the mapping), the initial-access accounts as one bullet per source with a ruling, the counting bases, then an `Actor as recorded` table `FIELD` | `AS RECORDED` with rows Attribution, Sector and geography, Tooling (device side and host side kept apart), Confirmed NOT, Analytical boundary. No naming-map figure; the prose carries it.
- **Evidence status in the step table**: the mechanism step table gains a `SOURCES` column and a status legend line above it (Observed: Sekoia first-party evidence or an ingested report describes the artefact on a real device; Corroborated: two or more ingested sources agree and at least one did not observe it directly; Hypothesis: no source describes it). Cells read `Observed`, `Corroborated`, `Hypothesis`, never the long form repeated per row. Sources are short clauses, each opening with the source name and date. There is no separate evidence-layers table and no source-layers figure.
- **Corroboration timeline** `DATE` | `DOCUMENTED ACTIVITY`, placed as an annex before the evidence basis, cells under 15 words, rows limited to dates cited elsewhere, with an anti-double-counting note when metrics come from different counting bases. Figure: the two-lane timeline, observation windows above, ingested report publications below; every milestone label is the event name.
- **Section order for the RFI**: 1 executive brief, 2 observed exposure mechanism, 3 what the sources establish, 4 why this matters, 5 operational validation, 6 priorities, 7 corroboration timeline, 8 evidence basis.
- **Length budget**: exposure assessment about 3,500 words, RFI about 5,000 words, both counted without Mermaid fences; tables and the evidence basis included.

## Figures

See the protocol's `references/diagrams.md` for the archetypes, the two lanes (Mermaid in markdown, inline SVG in HTML) and the geometry. Default set for an exposure assessment: observation timeline (section 2), cluster map when two or more clusters (section 3), mechanism chain (section 3), integrity model (section 4). RFI adds the naming map (source arbitration), source layers (evidence layers) and the two-lane corroboration timeline. Skip any figure a sentence says faster.

## Tone anchors (match this register)

- "The mechanism does not depend on a counterfeit Microsoft sign-in page: the user completes a genuine device-authorisation workflow, so the investigative question shifts from credential theft to whether the resulting tokens and cloud activity are attacker-controlled."
- Sober, technical, no fear language, no superlatives. Verdicts are questions the customer can test, not warnings.
- Section openers are one or two short sentences that state the finding. No scene-setting, no transition words, no closing summary.
- British-leaning spelling appears in the series (authorisation, behaviour); keep whichever the anchor documents use, consistently.
