# SOL reference

Curated from the Sekoia documentation (Investigate > Sekoia Operating Language) and checked against the live `run_sol_query` tool in October 2026. Where the documentation and the tool disagree, the tool wins and the difference is noted. Keep the IOC field map and the constraints in sync with `plugins/sekoia-cti/skills/query/references/tenant_check.md`.

## Constraints of run_sol_query

- `events` and `eternal_events` must be bounded at both ends: `where timestamp between (ago(30d) .. now())`. A query with only `timestamp > ago(30d)` is rejected before it runs, although the documentation's own examples use that form.
- `alerts`, `cases` and the other tables accept a one-sided filter: `where created_at > ago(7d)`.
- Limits: 10,000 processed rows per query (partial result beyond), 10 runs a minute, 10-minute timeout, 3 GB of memory per query. `set internal_fetch_limit = 100000;` raises the fetch cap; do not use it by default.
- The tool returns `{query, run_uuid, results}`; an error returns the server message (`Unknown identifier '<x>'`, not time-bounded).

## Query structure

```
datasource
| operator
| operator
```

Statements before the query end with `;`: `let start = ago(24h);`, `set internal_fetch_limit = 100000;`. Comments start with `//`. Lists use brackets: `in ['a', 'b']`.

## Operators

| Operator | Syntax | Notes |
|---|---|---|
| `where` | `\| where <condition>` | `and`, `or`, parentheses; chained `where` lines act as `and`; null checks `!= null`, `== null` |
| `between` | `where timestamp between (<start> .. <end>)` | Required on event tables |
| `select` | `\| select a, b, c = a + b` | Alias `project`; can create columns |
| `extend` | `\| extend c = <expr>` | Appends a computed column; not pushed down |
| `distinct` | `\| distinct host.name, user.name` | |
| `aggregate` | `\| aggregate count() by field` | Named: `aggregate total = count() by field`; several functions and keys allowed; default column names such as `count`, `count_distinct_source.ip` |
| `order by` | `\| order by count desc` | Default order is descending; several keys allowed |
| `limit` | `\| limit 100` | Always on listings |
| `top` | `\| top 10 by count desc` | |
| `count` | `\| count` | |
| `join` | `\| inner join <table> on left_field == right_field` | Types `inner` (default), `left`, `leftanti`, `rightanti`; right table becomes an object named after it (`intake.name`), rename with `into x`; not pushed down |
| `lookup` | `\| lookup <table> on left == right into x` | Always a left lookup; prefer it over `join` for small right tables; `on cidr_match($left.source.ip, $right.cidr)` |
| `let` | `let ids = alerts \| where … \| select short_id;` then `where x in ids` | Variables and nested queries |
| `render` | `\| render barchart with (x=…, y=…)` | Charts in the UI; not useful through the MCP |

## Comparison and string operators

- `==`, `!=`, `<`, `<=`, `>`, `>=`; `=~` is case-insensitive equality.
- `in [..]` (case-sensitive), `in~ [..]` (case-insensitive); negation `not x in [..]`.
- `contains`, `startswith`, `endswith`, each with a `~` case-insensitive variant.
- `matches regex '<re>'`: restricted syntax; `(?:…)`, `(?P<name>…)`, `(?i)`, lookarounds and backreferences return no rows without an error; case-insensitive modifiers do not work on `events`.
- No `has` operator.

Speed, fastest first: `==` and `!=`, `in`, `in~`, `startswith`, `startswith~`, `contains`, `endswith`, `contains~`, `endswith~`, `matches regex`.

## Functions

- Time: `now()`, `now(-2d)`, `ago(30d)` (suffixes `d h m s`), `bin(timestamp, 1h)`, `year()`, `month()`, `week()`, `format_datetime(t, '%Y-%m-%d')`.
- Aggregation: `count()`, `count_distinct(f)`, `countif(<predicate>)`, `sum`, `min`, `max`, `avg`, `make_set(f)`.
- String: `tolower`, `toupper`, `extract(<regex>, <group>, <source>)`, `replace_regex`, `split(s, delim[, index])`, `base64_decode`, `base64_encode`; raw strings `@'C:\path'`.
- Network: `cidr_match(source.ip, ['198.51.100.0/24'])` (IPv4, pushed down).
- Other: `iff(cond, a, b)`, `coalesce(a, b)`, `round`, `toint`, `tostring`, `parse_json`, `toscalar`.

## Pushdown (performance)

Pushed down: `count`, `limit`, time functions, simple `where` comparisons and `cidr_match`, `select`, `distinct`, `order by`, `top`, the first `aggregate`. Not pushed down: `extend`, `join`, `lookup`, `countif`, row functions (`extract`, `iff`, `tolower`, `coalesce`). Filter and aggregate before any join.

## Datasources

| Table | Time field | Key fields |
|---|---|---|
| `events` | `timestamp` | ECS fields plus `sekoiaio.*` (intake, entity, assets, tags); hot retention only |
| `eternal_events` | `timestamp` | Same schema; events attached to alerts and cases, kept permanently; `alert_short_ids` |
| `alerts` | `created_at` | `uuid`, `short_id`, `title`, `rule_name`, `rule_pattern`, `rule_instance_uuid`, `detection_type` (`CTI`, `Sigma`…), `alert_type_category`, `alert_type_value`, `status` (`Pending`, `Ongoing`, `Closed`, `Rejected`), `urgency`, `entity_name`, `first_seen_at`, `last_seen_at`, `occurrences`, `time_to_detect`, `time_to_acknowledge`, `time_to_respond`, `time_to_resolve`, `assignee`, `cases`, `assets`, `threats.id`, `threats.name`, `threats.type`, `custom_fields.*` |
| `cases` | `created_at` | `uuid`, `short_id`, `title`, `description`, `priority`, `first_seen_at`, `last_seen_at`, `custom_fields` |
| `assets` | `created_at` | `uuid`, `name`, `type` (host, account, network), `category`, `criticality`, `atoms`, `tags.tag` |
| `rule_definitions` | `created_at` | `uuid`, `name`, `type` (`sigma`, `cti`…), `source` (`SEKOIA` or custom) |
| `rule_instances` | `created_at` | `rule_definition_uuid`, `community_uuid`, `enabled` |
| `intakes`, `entities`, `communities` | | Names and UUIDs for joins |
| `event_telemetry` | `bucket_start_date` | Ingestion volume per intake |

Live notes: on `alerts`, `short_id` works (the documentation also writes `short_ID`); `threats.id` and `threats.name` work but `select threats` fails with `Unknown identifier`. On CTI alerts (`rule_name == 'SEKOIA Intelligence Feed'`, `detection_type == 'CTI'`), `threats.id` is the STIX ID of the Sekoia CTI object and `rule_pattern` the matched indicator. On other alerts `threats.name` holds ATT&CK technique names. An alert can carry several threats.

## Event fields by entity

Field presence depends on the intake; confirm with a discovery query on the dialect.

| Entity | Fields |
|---|---|
| Time and source | `timestamp`, `event.created`, `sekoiaio.intake.dialect`, `sekoiaio.intake.uuid`, `sekoiaio.entity.uuid` |
| Event | `event.category` (array: `authentication`, `network`, `process`, `file`, `web`, `intrusion_detection`…), `event.action`, `event.outcome`, `action.outcome`, `event.code` |
| Host | `host.name`, `host.hostname`, `host.ip`, `host.os.type`, `agent.id` |
| User | `user.name`, `user.email`, `user.full_name`, `user.id` |
| Network | `source.ip`, `destination.ip`, `destination.port`, `client.ip`, `server.ip`, `source.geo.country_iso_code` |
| DNS and web | `dns.question.name`, `dns.question.registered_domain`, `url.domain`, `url.registered_domain`, `url.original`, `url.full`, `url.path`, `user_agent.original` |
| Process | `process.name`, `process.command_line`, `process.executable`, `process.parent.name`, `process.hash.sha256` |
| File | `file.name`, `file.path`, `file.hash.md5`, `file.hash.sha1`, `file.hash.sha256` |
| Assets | `sekoiaio.any_asset.name`, `sekoiaio.any_asset.uuid` |

### IOC sweep fields

The `related.*` fields collect every value of a type found anywhere in the event, so one filter covers all the ECS fields:

| Indicator | Field | Notes |
|---|---|---|
| IP | `related.ip in [...]` | |
| Domain | `related.hosts in [...]` | Holds DNS query names and URL hosts (`www.x.com` and `x.com`) plus host names |
| Hash | `related.hash in [...]` | Array of MD5, SHA-1 and SHA-256; membership works |
| User | `related.user in [...]` | |
| URL | `url.original in [...]` or `url.full in [...]` | No `related` field |

The IOC detection of Sekoia checks: IPs in `client.ip`, `destination.ip`, `host.ip`, `server.ip`, `source.ip`; domains in `dns.question.name`, `url.domain`, `destination.domain`, `source.domain`, `client.domain`, `server.domain` and their `registered_domain` variants and TLS certificate names; URLs in `url.full`, `url.original`; hashes in `file.hash.*`, `dll.hash.*`, `process.hash.*`, `email.attachments.file.hash.*`.

### Microsoft Entra ID sign-ins

Dialect `microsoft entra id / azure ad`: `event.category == 'authentication'`, `action.outcome`, `user.email`, `source.ip`, `azuread.properties.appDisplayName`, `azuread.properties.authenticationProtocol` (`none`, `deviceCode`…), `azuread.properties.authenticationRequirement`, `azuread.properties.conditionalAccessStatus`, `azuread.properties.riskLevelDuringSignIn`, `user_agent.original`.

## Source documentation

Raw markdown, fetch only when this reference does not cover the need:

- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_overview.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_getting_started.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_best_practices.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_how_to_guides.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_datasets.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_query_examples.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_ref_datasources.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_ref_operators.md
- https://raw.githubusercontent.com/SEKOIA-IO/documentation/main/docs/xdr/features/investigate/sol_ref_functions.md

Published pages: https://docs.sekoia.com/xdr/features/investigate/sol_overview/
