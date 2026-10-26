---
name: sol-hunt
description: Turn a natural-language question about Sekoia telemetry into a bounded SOL (Sekoia Operating Language) query, run it with run_sol_query and read the rows back in words. Use when asked "when did this user last log in", "what happened on this host", "how many alerts did this rule raise this week", "list the most urgent alerts", "did this IP, domain or hash appear in our events", "which hosts contacted this indicator", "has Sekoia CTI fired on this threat", "which log sources do we have", or any hunt over events, alerts, cases, assets or rules. Requires the Sekoia MCP server with View query builder data sources and Execute query.
---

# SOL hunt

One question, one bounded query, one answer in words. The language reference is `references/sol_reference.md`; tested queries are in `references/sol_patterns.md`. Read the patterns first and adapt the closest one; read the reference only for an operator, function or datasource the patterns do not cover.

## Rules for every query

1. **Bound the event tables at both ends.** `events` and `eternal_events` need `where timestamp between (<start> .. <end>)`, for example `between (ago(30d) .. now())`. The server rejects a query with only `timestamp > ago(30d)`. `alerts` and `cases` accept `where created_at > ago(7d)`. Default windows: 24 hours for a host, 7 days for a rule, 30 days for a user or an indicator. State the window in the answer.
2. **Filter fast.** `==` and `in [...]` run inside the datastore; `contains`, the `~` case-insensitive variants and `matches regex` are slower. Use them only when the exact value is unknown, then confirm the value and filter with `==`.
3. **Aggregate before listing.** For "how many" or "broken down by", `aggregate count() by <field>` first. List rows only when the user wants examples, with few `select` fields and a `limit`.
4. **One question per query, few queries per answer.** The Query Builder allows 10 runs a minute; stay under five per answer.
5. **Show the query.** Print the query you ran in a code block before reading the result, so the analyst can rerun it in Sekoia.
6. **Read back in words.** A count comes with its window and filter. An empty result reads "no rows for that filter between <start> and <end>", never "it never happened". A result at the `limit`, or at the 10,000-row processing cap, is truncated: say so and offer a narrower window or an aggregate.

## Unknown fields

Field names depend on the intake. Before guessing, run one discovery query on the dialect concerned and read the field names from the row:

```
events
| where timestamp between (ago(1h) .. now())
| where sekoiaio.intake.dialect == '<dialect>'
| limit 1
```

The dialects present come from `aggregate count() by sekoiaio.intake.dialect` (pattern in the reference).

## Conduct

Read-only: when the user asks for an action (close an alert, comment a case), give the step to take in Sekoia. Cite every alert and case short ID. Facts first, then any assessment in a separate sentence. End with what was not checked.

## Degradation

- `run_sol_query` absent: say that `View query builder data sources` and `Execute query` are missing and give the query for the analyst to run in the Sekoia Query Builder.
- Rejected as not time-bounded: add the `between` filter and rerun once.
- `Unknown identifier`: the field does not exist on that datasource; run the discovery query, or check the datasource fields in the reference.
- Timeout or memory error: narrow the window, add a filter, aggregate, then rerun once.
