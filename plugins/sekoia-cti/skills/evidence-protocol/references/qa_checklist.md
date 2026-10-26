# Final QA before release

Walk every item. An unmet item is fixed or surfaced in the report as a limitation; it is never silently accepted.

- [ ] Exact objects were retrieved for every material claim; nothing rests on a search snippet alone
- [ ] Every intelligence claim traces to a Sekoia MCP retrieval; no external document, no URL fetched, no model knowledge; ingested third-party reporting cited by its Sekoia object ID
- [ ] Relevant relationships were inspected, with `target_types` stated
- [ ] Broader or contradictory context was queried where the claim is sensitive (attribution, targeting, victimology)
- [ ] Oversized or failed retrievals were re-queried with a bounded strategy
- [ ] NOT_RETURNED was never interpreted as EMPTY
- [ ] EMPTY was used as an absence statement only where the query was exhaustive, with the scope stated
- [ ] Confidence values are quoted exactly as retrieved and were not transferred across objects or relationships
- [ ] Dates kept their semantics; no `modified` or publication date stands in for activity; relationships without dates say "No independent activity date"
- [ ] No two separately true facts were combined into an unsupported joint claim
- [ ] Actors, campaigns and clusters remain separate; no capability was inherited across clusters
- [ ] FACT and CALCULATED RESULT are separated from ASSESSMENT in the text and in the manifest
- [ ] Every EXPOSURE MODEL is explicitly labeled as a model, not an observed incident
- [ ] Customer targeting was not inferred from relevance; the ACCOUNT boundary is stated
- [ ] Customer technology deployment was not assumed; the TECHNOLOGY boundary is stated
- [ ] Every validation question maps to identifiable telemetry with defined criteria
- [ ] Operational derivations (owners, priorities) are distinguishable from retrieved evidence
- [ ] Unresolved gaps and limitations remain visible in the final text
- [ ] Every material claim has a manifest row with exact IDs
- [ ] TLP marking is present and consistent with the markings of the objects used
- [ ] Series conventions respected: em dashes only in the series line and the TLP banner; status and priority labels use colons; tables stayed tables
- [ ] Facts unchanged by any editing: the sets of IDs, confidence values and dates in the report and manifest are identical before and after
- [ ] Sentences short and plain across the report: one idea each, almost all under 20 words, none over 30 except quoted source wording; paragraphs 2 to 4 sentences
- [ ] Executive brief: no parentheses, no ISO dates, no inventory sentences; every fact in it has a row in the evidence basis table and its metadata sits in the strip
- [ ] Evidence basis: every Sekoia object ID links to its `app.sekoia.io/objects/` page, the visible text is the full ID, relationship IDs are not linked, the account-access line follows the table
- [ ] HTML cover shows the Sekoia wordmark inline with `role="img"` and an `aria-label`, no external image reference
- [ ] Executive brief carries "Why now:" with the trigger and the age in months of the most recent evidence; STATUS in a sector variant reads "sector relevance only, no recipient assessed"
- [ ] Evidence basis opens with the confidence key; relationship counts sit in the manifest, not in table cells
- [ ] Validation window is dated in the applicability note; every validation question is under 30 words with lists in the evidence column
- [ ] Priorities section present with P1 to P3 and the Assessment callout
- [ ] No figure label leftovers: no "hollow", no template token, every gantt milestone labelled with the event name; no two sections cite the same evidence in two tables
- [ ] Word count without Mermaid fences within the series budget (exposure assessment about 3,500, RFI about 5,000)
- [ ] Every figure states one claim in its caption and `aria-label`, and every label, date, confidence or count drawn in it maps to a manifest row listed in the manifest `Figures` table
- [ ] `<REFERENCE>.pdf` printed from the final HTML with headless Chrome, page count recorded, page 1 carries the cover and the start of the brief
- [ ] The HTML stays self-contained: no `<script>`, `<style>`, `<foreignObject>` or external reference inside any SVG; each figure has a `<figcaption>`
