---
name: case-summary
description: Summarise a Sekoia case from its short ID (CA…) as an ordered timeline of its alerts with the hosts, users and rules involved, plus a five-line narrative. Use when asked "summarise case CA…", "timeline of case …", "what happened in case …", "brief me on this incident", "which assets are involved in the case". Requires the Sekoia MCP server with View cases and View events; View alerts, View Rules Catalog and the query builder permissions add the optional steps.
---

# Case summary

Turn a case into something the next shift, a manager or a customer reads in two minutes: what happened, in what order, on which assets, and where the case stands.

## Steps

1. **Case**: `get_case(case_short_id, include_events=false)`. The payload holds `title`, `description`, `status` (a plain string such as `closed`), `created_at`, `updated_at` and `alerts`; each alert carries `short_id`, `title`, `urgency`, `status.name`, `rule` (`uuid`, `name`, `pattern`), `detection_type`, `ttps`, `first_seen_at`, `last_seen_at`. Do not call `get_alert` for each alert; the case payload already has them.
2. **Priority**: not in the `get_case` payload. Read it with `run_sol_query`:
   ```
   cases
   | where short_id == '<CA…>'
   | select priority, first_seen_at, last_seen_at
   | limit 1
   ```
3. **Description tokens**: AI-written descriptions contain `{asset:<uuid>}`, `{rule:<uuid>}` and `{ttp:attack-pattern--…}`. Replace each with a name from the same payload (the asset name usually follows the token in parentheses or quotes; rule names come from the alerts' `rule`; technique names from `ttps`). If no name is available, drop the token. Never print a raw token.
4. **Entities**: hosts and users are not in the alert entries. Call `get_events_from_alert` on two or three anchor alerts (the first, the most urgent, the last) with their own `first_seen_at` and `last_seen_at`; keep `host.name`, `user.name` or `user.email`, and the key object (`process.name`, `file.name`, `destination.ip`, `url.original`). Deduplicate.
5. **Timeline**: order alerts by `first_seen_at` (ISO 8601; alert `created_at` in this payload is an epoch in seconds, do not use it). Group alerts of the same minute on the same host into one line. Mark gaps of several hours.
6. **Narrative**: at most five lines, past tense, facts only; then one line of assessment, labelled, with confidence in words.

## Output

```
# Case CA… <title>

**Status** <s> · **Priority** <p> · **Created** <UTC> · **Alerts** <n> · **Hosts** <n> · **Users** <n>

## Timeline (UTC)
| Time | Alert | Rule | Status | Host | User | What |
|---|---|---|---|---|---|---|

## Entities
- Hosts: <name> (<n alerts>, first seen <time>) …
- Users: …

## Narrative
<up to five lines>

## Assessment
<one line, labelled as assessment>

## Not verified
- …

## Next action in Sekoia
- …
```

## Large cases

Beyond about fifteen alerts: summarise by rule and host with counts, give the first three and the last three alerts in full, and call `get_events_from_alert` on three anchors only.

## Rules

Cite every case and alert short ID. Facts and assessment never in the same sentence. Read-only: for a status change, assignment or comment, give the step in Sekoia. For one alert in depth, offer the `alert-details` skill; for hunting around a host or user, `sol-hunt`.

## Degradation

- `get_case` returns nothing: wrong short ID, case in another community than the API key, or deleted. Do not guess another ID.
- `run_sol_query` missing: priority is "not retrieved"; name `View query builder data sources` and `Execute query`.
- `get_events_from_alert` missing: `View events`; report hosts and users as not retrieved.
- `get_events_from_alert` empty for an anchor: try the next one, then report hosts and users as not retrieved.
