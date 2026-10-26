# sekoia-soc

Sekoia MCP Server for SOC work in Claude Code: everything on an alert, case summaries and SOL hunting in natural language on your Sekoia community. Read-only.

## Install

```
/plugin marketplace add SEKOIA-IO/claude-marketplace
/plugin install sekoia-soc@sekoia
```

Export `SEKOIA_API_KEY` before starting Claude Code. The key needs `View alerts`, `View cases`, `View events`, `View Rules Catalog`, `View query builder data sources` and `Execute query`; add `View intelligence` to get the CTI profile of the threats behind an alert. A key with fewer permissions still works; the skills say which step they skipped.

## Skills

| Skill | Ask for | Tools |
|---|---|---|
| `alert-details` | "Get alert AL…", "show me everything about alert …", "which host and user are behind alert …" | `get_alert`, `get_rule`, `get_events_from_alert`, `run_sol_query`, `get_cti_object_by_id` |
| `case-summary` | "Summarise case CA…", "timeline of case …", "brief me on this incident" | `get_case`, `get_events_from_alert`, `run_sol_query` |
| `sol-hunt` | "When did this user last log in", "what happened on this host", "how many alerts did this rule raise this week", "did this domain appear in our events" | `run_sol_query` |

`sol-hunt` carries a curated SOL reference (`skills/sol-hunt/references/sol_reference.md`) and tested query patterns; queries on `events` are always bounded with `timestamp between (…)`, which the server requires.

## Example prompts

```
Get everything on alert <short_id>: rule, events, host, user, threats and linked cases.
Summarise case <short_id> as a timeline and a five-line narrative.
How many alerts did the rule "SEKOIA Intelligence Feed" raise in the last 7 days, by threat? Then list the 10 most urgent.
Did any host resolve <domain> in the last 30 days?
Which log sources do we have in Sekoia?
```

See the [SOC how-to guides](https://docs.sekoia.com/integration/mcp/how_to_soc/) and the [SOL documentation](https://docs.sekoia.com/xdr/features/investigate/sol_overview/).
