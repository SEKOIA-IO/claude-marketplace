# Tenant check

How the CTI skills check the Sekoia community of the API key with `run_sol_query`: has Sekoia CTI already fired here, and do the indicators appear in the events. Results are customer-side evidence about that tenant, never Sekoia intelligence. Keep the constraints and the IOC field map in sync with `plugins/sekoia-soc/skills/sol-hunt/references/sol_reference.md`.

## Constraints

- `events` and `eternal_events` must be bounded at both ends: `where timestamp between (ago(30d) .. now())`. A query with only `timestamp > ago(30d)` is rejected. `alerts` accepts `where created_at > ago(30d)`.
- Default window 30 days; state it in the answer.
- 10 runs a minute, 10,000 processed rows per query: aggregate, never list raw events, always `limit`.
- Permissions: `View query builder data sources` and `Execute query`; reading `alerts` also needs alert access on the key.
- The tenant is the community of the API key. Say so; it is not the recipient of a report unless the user says it is.

## Pattern 1: has Sekoia CTI already fired here

The IOC detection of Sekoia raises alerts from the rule `SEKOIA Intelligence Feed` (`detection_type` `CTI`). Each carries `threats.id`, the STIX ID of the CTI object behind the matched indicator, and `rule_pattern`, the indicator itself. The IDs the CTI skills retrieve map straight onto them:

```
alerts
| where created_at > ago(30d)
| where threats.id in ['<intrusion-set--…>', '<malware--…>', '<infrastructure--…>', '<campaign--…>']
| aggregate count() by threats.name, status
| limit 50
```

Then the latest ones:

```
alerts
| where created_at > ago(30d)
| where threats.id in ['<…>']
| select created_at, short_id, rule_name, rule_pattern, status, urgency, entity_name
| order by created_at desc
| limit 20
```

Put in the actor's ID and the IDs of its related infrastructure, malware, tools and campaigns from the relationship slices: alerts usually point at the infrastructure or malware, rarely at the intrusion set. `status` `Rejected` means analysts dismissed the alert; `Pending` means not yet handled. An alert can list several threats, so a count by `threats.name` can show neighbours of the IDs asked.

## Pattern 2: do the indicators appear in the events

```
events
| where timestamp between (ago(30d) .. now())
| where related.ip in ['<ip>'] or related.hosts in ['<domain>'] or related.hash in ['<sha256>']
| aggregate count() by host.name, sekoiaio.intake.dialect
| order by count desc
| limit 50
```

| Indicator | Field |
|---|---|
| IP | `related.ip` |
| Domain | `related.hosts` (DNS query names and URL hosts) |
| Hash | `related.hash` (MD5, SHA-1, SHA-256) |
| URL | `url.original` or `url.full` |

Prefer domains, URLs and hashes. An IP of shared hosting or a CDN matches unrelated traffic. Read IP hits as leads to confirm with the domain or the URL, and say when a count reached the processing cap.

## Pattern 3: first and last hit

```
events
| where timestamp between (ago(30d) .. now())
| where related.hosts in ['<domain>']
| aggregate first_hit = min(timestamp), last_hit = max(timestamp), hits = count() by dns.question.name
| limit 50
```

## Reading the results

- Answer in three labelled parts: what Sekoia CTI records (CTI evidence), which IDs and indicators were checked and where they came from, what the tenant returned over which window (customer-side evidence).
- No rows: "no CTI alert and no event on these N indicators between <start> and <end> in this tenant". Never "not exposed": the indicators checked are a sample, telemetry coverage is partial and retention is finite.
- Rows: name the hosts, users and alert short IDs as found. A hit is a lead for the SOC, not an attribution of the activity to the actor.
- More than a sweep (behaviours, techniques): use the `report-hunt` skill. Free-form hunting: `sekoia-soc:sol-hunt` when that plugin is installed.
