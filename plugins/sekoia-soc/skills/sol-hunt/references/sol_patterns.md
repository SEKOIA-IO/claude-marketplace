# SOL patterns

Each pattern ran on the live `run_sol_query` tool in October 2026. Replace the `<placeholders>`, keep the time bound and the `limit`.

## Environment

Which log sources the tenant has, the first query before any behaviour hunt:

```
events
| where timestamp between (ago(1d) .. now())
| aggregate count() by sekoiaio.intake.dialect
| order by count desc
| limit 40
```

## Users

Last successful sign-in of a user (Entra ID and other identity intakes fill `user.email`; others use `user.name`):

```
events
| where timestamp between (ago(30d) .. now())
| where event.category == 'authentication' and action.outcome == 'success'
| where user.email == '<user@domain>'
| select timestamp, event.action, source.ip, sekoiaio.intake.dialect
| order by timestamp desc
| limit 1
```

Failed sign-ins of a user by source IP:

```
events
| where timestamp between (ago(7d) .. now())
| where event.category == 'authentication' and action.outcome == 'failure'
| where user.email == '<user@domain>'
| aggregate count() by source.ip
| order by count desc
| limit 20
```

Device code sign-ins (Entra ID), the flow abused by device code phishing kits:

```
events
| where timestamp between (ago(7d) .. now())
| where sekoiaio.intake.dialect == 'microsoft entra id / azure ad'
| where azuread.properties.authenticationProtocol == 'deviceCode'
| aggregate count() by user.email, azuread.properties.appDisplayName, action.outcome
| order by count desc
| limit 20
```

## Hosts

Activity of a host by category:

```
events
| where timestamp between (ago(1d) .. now())
| where host.name == '<host>'
| aggregate count() by event.category
| order by count desc
| limit 20
```

Latest process and network events on a host:

```
events
| where timestamp between (ago(1d) .. now())
| where host.name == '<host>'
| where event.category in ['process', 'network']
| select timestamp, event.action, process.name, process.parent.name, destination.ip, dns.question.name
| order by timestamp desc
| limit 50
```

## Indicators

Sweep for IPs, domains and hashes; `related.*` covers every ECS field of the type:

```
events
| where timestamp between (ago(30d) .. now())
| where related.ip in ['<ip>', '<ip>'] or related.hosts in ['<domain>'] or related.hash in ['<sha256>']
| aggregate count() by host.name, sekoiaio.intake.dialect
| order by count desc
| limit 50
```

First and last hit per indicator, once the sweep returned rows:

```
events
| where timestamp between (ago(30d) .. now())
| where related.ip in ['<ip>', '<ip>']
| aggregate first_hit = min(timestamp), last_hit = max(timestamp), hits = count() by related.ip
| limit 50
```

## Alerts

Alerts of a rule by status:

```
alerts
| where created_at > ago(7d)
| where rule_name == '<rule name>'
| aggregate count() by status
| order by count desc
```

Most urgent alerts of a rule:

```
alerts
| where created_at > ago(7d)
| where rule_name == '<rule name>'
| select created_at, short_id, status, urgency, entity_name
| order by urgency desc
| limit 10
```

Full row of one alert (entity, timings, cases, assets, threats):

```
alerts
| where short_id == '<AL…>'
| limit 1
```

Alerts raised by Sekoia CTI on given threats (STIX IDs of intrusion sets, malware, tools, infrastructure, campaigns):

```
alerts
| where created_at > ago(30d)
| where threats.id in ['<stix id>', '<stix id>']
| aggregate count() by threats.name, status
| limit 50
```

What the Sekoia CTI feed raised recently, by threat:

```
alerts
| where created_at > ago(30d)
| where rule_name == 'SEKOIA Intelligence Feed'
| aggregate count() by threats.name
| order by count desc
| limit 20
```

## Cases and correlation

Recent cases with their priority (priority is not in the `get_case` payload):

```
cases
| where created_at > ago(30d)
| select short_id, title, priority, created_at
| order by created_at desc
| limit 20
```

Alerts touching a set of hosts, then their details:

```
eternal_events
| where timestamp between (ago(7d) .. now())
| where host.name in ['<host>', '<host>']
| aggregate make_set(alert_short_ids) by host.name
| limit 20
```

```
alerts
| where created_at > ago(7d)
| where short_id in ['<AL…>', '<AL…>']
| select created_at, short_id, rule_name, status, urgency
| order by created_at desc
| limit 50
```

## Detection coverage

Built-in or custom rules matching a behaviour, with their enablement (one instance per community, so several rows per rule are normal):

```
rule_definitions
| where name contains~ '<keyword>'
| lookup rule_instances on uuid == rule_definition_uuid into inst
| aggregate count() by name, source, inst.enabled
| limit 20
```
