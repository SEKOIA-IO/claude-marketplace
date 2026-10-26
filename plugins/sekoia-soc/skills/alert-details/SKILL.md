---
name: alert-details
description: Gather everything Sekoia holds on one alert from its short ID (AL…), the alert record, its rule, the events behind it, the hosts, users, files and network values involved, the linked cases and assets, and the threats with their Sekoia CTI context, and present it as facts without a verdict. Use when asked "get alert AL…", "show me everything about alert …", "details of alert …", "what is in alert …", "which host and user are behind alert …", "what rule raised alert …". Requires the Sekoia MCP server with View alerts, View events and View Rules Catalog; View intelligence and the query builder permissions add the CTI and alert-row steps.
---

# Alert details

Collect, do not judge. The output is the full factual record of one alert so an analyst can decide.

## Steps

1. **Alert**: `get_alert(alert_short_id)`. Keep `title`, `status.name`, `urgency`, `detection_type` (`CTI`, `Sigma`…), `rule` (`uuid`, `name`, `pattern`, `severity`), `details`, `ttps` (threats and ATT&CK techniques, each with `id`, `type`, `name`, `description`), `first_seen_at` and `last_seen_at`. `created_at` is an epoch in seconds; print it as ISO 8601 UTC.
2. **Alert row** (fields `get_alert` does not return), with `run_sol_query`:
   ```
   alerts
   | where short_id == '<AL…>'
   | limit 1
   ```
   Keep `entity_name`, `occurrences`, `cases`, `assets`, `threats` and the `custom_fields.*` that are filled.
3. **Rule**: `get_rule(rule.uuid)` with the UUID from step 1 (it is the rule instance). Keep `description`, `type`, `source` (`SEKOIA` or custom), `effort`, `false_positives`, `references`, `datasources`, `goal`. Say whether the rule is a behavioural detection (Sigma pattern on events), a vendor verdict pass-through (pattern on a vendor alert field) or a CTI match (`type: cti`, pattern is an indicator).
4. **Events**: `get_events_from_alert(alert_short_id, first_seen_at, last_seen_at)` with the alert's own timestamps. Extract and deduplicate: hosts (`host.name`, `host.ip`), users (`user.name`, `user.email`), files and hashes, process and parent with command line, network values (`source.ip`, `destination.ip`, `dns.question.name`, `url.original`), intake dialect, and any vendor verdict or mitigation fields. Show at most three events in full and count the rest.
5. **Threats**: for each entry of `ttps` whose `type` is not `attack-pattern`, give the name, type, STIX ID and the link `https://app.sekoia.io/objects/<id>`; `get_cti_object_by_id` adds aliases and the activity start when `View intelligence` is granted. ATT&CK techniques are listed by name. For a CTI alert, `rule.pattern` is the matched indicator: quote it.
6. **Linked cases and assets**: list the case short IDs from step 2 and offer the `case-summary` skill; list the asset names.

## Output

```
# Alert AL… <title>

**Status** <s> · **Urgency** <u> · **Detection** <type> · **Entity** <name> · **First seen** <UTC> · **Last seen** <UTC> · **Occurrences** <n>

## Rule
<name>, <source>, <behavioural / vendor pass-through / CTI match>. <one-line description>. False positives documented: <text or none>.

## Entities
- Hosts: …  · Users: …  · Files and hashes: …  · Processes: <parent> → <process> `<command line>`  · Network: …

## Events
<count> events from <dialect>; the first three in a compact table.

## Threats and CTI
- <name> (<type>, <STIX ID>, link); matched indicator `<pattern>` …
- ATT&CK: …

## Linked cases and assets
…

## Next pivots
- Case summary for <CA…>; hunt on <host or user> with `sol-hunt`; other alerts on the same threat (`alerts | where threats.id in [...]`).
```

## Rules

- Facts only, each tied to the tool that returned it; no verdict, no severity of your own.
- Cite the alert short ID, the rule UUID and every STIX ID in full.
- Read-only: when asked to change the status or comment, give the step in Sekoia.

## Degradation

- `get_alert` returns nothing: wrong short ID, alert in another community than the API key, or deleted. Say so; do not guess another ID.
- `run_sol_query` missing: skip step 2 and name the permissions (`View query builder data sources`, `Execute query`); cases and assets are then not retrieved.
- `get_rule` missing: `View Rules Catalog`; describe the rule from `rule.name` and `rule.pattern` only.
- `get_events_from_alert` missing: `View events`; list entities from `details` and `rule.pattern` only.
- `get_events_from_alert` empty: say so; the `eternal_events` table may still hold them (`where alert_short_ids == '<AL…>'`, bounded with `between`).
- CTI tools missing: `View intelligence`; list the threat IDs and names from `ttps` without profiles.
