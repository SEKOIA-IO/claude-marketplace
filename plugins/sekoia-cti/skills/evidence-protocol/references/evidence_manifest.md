# Evidence manifest

One row per material claim. The manifest ships next to the report (`<reference>_manifest.md`) and makes the chain auditable: report claim → manifest row → exact Sekoia object or relationship. Used for QA, regression testing, hallucination detection and reproducibility.

## Template

| Field | Purpose |
|---|---|
| Claim ID | Unique internal reference (C-01, C-02…) |
| Claim text | The statement as it appears in the analysis |
| Claim type | FACT / CALCULATED RESULT / ASSESSMENT / EXPOSURE MODEL / NOT ASSERTED |
| Supporting object IDs | Exact Sekoia STIX IDs or UUIDs |
| Supporting relationship IDs | Exact relationship IDs where applicable |
| Source | Sekoia (first-party or ingested, source named) / customer |
| Confidence | Exact value retrieved (never rounded or invented) |
| Temporal basis | Exact field and timestamp used (`first_seen 2026-02-…`), or "No independent activity date" |
| Retrieval status | VALUE / EMPTY / NOT_RETURNED / NOT_CHECKED |
| Derivation | None / analytical / operational |
| Boundary | Account / Technology / Cluster / Exposure, when one applies |
| Limitation | Remaining evidence gap, if any |

## Example rows

| Claim ID | Claim text | Claim type | Supporting IDs | Source | Confidence | Temporal basis | Retrieval status | Derivation | Boundary | Limitation |
|---|---|---|---|---|---|---|---|---|---|---|
| C-01 | Sekoia tracks EvilTokens as a campaign weaponising device-code authentication tokens | FACT | campaign--<uuid> | Sekoia | 100 | first_seen 2026-02 | VALUE | None | | |
| C-02 | The campaign has a relationship to the Financial Services sector | FACT | relationship--<uuid> | Sekoia | 100 | No independent activity date | VALUE | None | Cluster | Sector relevance only; not evidence of recent sector activity |
| C-03 | Equivalent access could expose high-value communications at a financial institution | ASSESSMENT | derived from C-01, C-02 | Sekoia | n/a | n/a | n/a | Analytical | Account, Exposure | No recipient-specific evidence |
| C-04 | No verified campaign currently links the recipient to the reviewed activity | NOT ASSERTED (absence stated) | search scope: recipient name across CTI objects | Sekoia | n/a | Query date | EMPTY | None | Account | Absence claim bounded to the exact search scope |

Rules:
- A row with Retrieval status NOT_RETURNED cannot carry claim type FACT.
- Every EXPOSURE MODEL row records the boundary "Exposure".
- Every NOT ASSERTED row states what was deliberately not claimed and why.
- IDs are copied exactly from tool results, never reconstructed from memory.
