# sekoia-cti

Sekoia MCP Server for threat intelligence in Claude Code. Three layers: a conversational lookup skill for everyday CTI questions, a hunt skill that turns a report into queries on your own telemetry, and three report skills that turn Sekoia evidence into customer-grade documents under an explicit evidence protocol.

## Install

```
/plugin marketplace add SEKOIA-IO/claude-marketplace
/plugin install sekoia-cti@sekoia
```

Export `SEKOIA_API_KEY` before starting Claude Code. The key needs `View intelligence`. Add `View query builder data sources` and `Execute query` for the tenant check and `report-hunt`, which query your own Sekoia community with SOL.
## Skills

| Skill | Ask for | Output |
|---|---|---|
| `query` | Any CTI lookup: "is this IP known", "what does Sekoia know about actor X", "which actors target sector Y", "which actors exploit CVE-…", "are we exposed to Z" | An answer in a few sentences with IDs, dates, confidence and platform links |
| `report-hunt` | "Build a hunt pack from FLINT 2026-037", "can we see these techniques in our logs" | Hunt pack: alerts already raised by Sekoia CTI, telemetry coverage, built-in rules, bounded SOL queries per behaviour |
| `evidence-protocol` | Applied by the report skills: claim types, retrieval states, cluster discipline, boundaries, evidence manifest, writing pass, figures, rendering | The rules and the HTML template |
| `exposure-assessment` | "Exposure assessment on actor X for sector Y", "sector technical RFI for telecoms" | Markdown report, evidence manifest, HTML, PDF |
| `threat-horizon-assessment` | "How will AI-enabled attacks evolve for customer Y over 12 months" | Markdown report, evidence manifest, HTML, PDF |
| `intelligence-delta` | "Prove the delta for MSSP X", the "not another feed" objection | Markdown brief, evidence manifest, HTML, PDF |

## Example prompts

Lookups:

```
Is 203.0.113.42 known to Sekoia CTI?
What does Sekoia know about RedMike?
Which actors does Sekoia link to CVE-2023-20198?
Are we exposed to TeamTNT? Summarise the actor, list its malware and tools, then check our events for the related indicators over the last 30 days.
Has Sekoia CTI on APT28 or its infrastructure fired in our tenant over the last 30 days?
```

Hunting:

```
Build a hunt pack from FLINT 2026-011 on EvilTokens: which behaviours can our telemetry see, which built-in rules cover them, and the SOL queries for the rest.
```

Reports:

```
Produce a Sekoia Intelligence Exposure Assessment. Anchor: TeamTNT and cloud cryptojacking against exposed Docker and Kubernetes. Scope: cloud-native SaaS providers. Reference SEKOIA-EA-2026-CLOUDNATIVE-001.
Build a telecommunications technical RFI on Salt Typhoon edge-device exploitation and router persistence.
```

## How the report skills work

Every material claim is traced to an exact Sekoia object or relationship ID and classified (fact, calculated result, assessment, exposure model, not asserted). Retrieval states stay separate (value, empty, not returned, not checked). Clusters are never merged, dates are never borrowed between objects, and four boundaries are stated in every report. The evidence manifest ships next to the report and makes the chain auditable.

The PDF is printed from the finished HTML with a local Chrome or Chromium (Playwright's Chromium, the Google Chrome application, or `chromium` on the PATH). Without one, the markdown, manifest and HTML are still produced and the skill says the PDF was not.

See the [threat intelligence how-to guides](https://docs.sekoia.com/integration/mcp/how_to_cti/) and the [MCP tools reference](https://docs.sekoia.com/integration/mcp/tools_reference/).
