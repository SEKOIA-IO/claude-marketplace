# Hunt fields

Field names for turning a report's procedures into SOL queries, by telemetry family. Fields marked verified were used in queries that ran on the live `run_sol_query` tool; the others come from the Sekoia SOL documentation. Field presence depends on the intake: when a family or a field is not listed, run the discovery query first.

## Query shape

Every hunt query on `events`:

```
events
| where timestamp between (ago(<window>) .. now())
| where <family filter>
| where <procedure filter>
| aggregate count() by <host or user>, sekoiaio.intake.dialect
| order by count desc
| limit 20
```

- Exact strings from the report (a full User-Agent, a URL path, a binary name): `==` or `in [...]`, the fastest filters.
- Process and file names: `in~ [...]`, case-insensitive.
- Fragments of a command line or a URL: `contains~`, slower, so keep the window short.
- Where and when something first appeared: `aggregate first_seen = min(timestamp), hits = count() by host.name`.

## Discovery

The log sources present, then the fields of one source:

```
events
| where timestamp between (ago(1d) .. now())
| aggregate count() by sekoiaio.intake.dialect
| order by count desc
| limit 40
```

```
events
| where timestamp between (ago(1h) .. now())
| where sekoiaio.intake.dialect == '<dialect>'
| limit 1
```

## Common to all families

| Field | Notes |
|---|---|
| `timestamp` | Verified; bound it with `between` |
| `sekoiaio.intake.dialect` | Verified; the log source, lower case (`microsoft entra id / azure ad`, `microsoft 365 / office 365`, `crowdstrike falcon telemetry`, `sekoia.io endpoint agent`, `zscaler internet access`…) |
| `host.name`, `user.name`, `user.email` | Verified |
| `event.category` | Verified; array (`authentication`, `process`, `network`, `file`, `web`…); `==` matches one value |
| `event.action`, `action.outcome` | Verified; `action.outcome` is `success` or `failure` |

## Endpoint process (EDR, endpoint agent)

| Field | Notes |
|---|---|
| `process.name`, `process.parent.name` | Verified; parent → child chains |
| `process.command_line` | Verified; arguments, encoded commands, URLs |
| `process.executable`, `process.hash.sha256` | Documented |
| `file.name`, `file.path`, `file.hash.sha256` | Documented |

## Identity provider sign-ins (Microsoft Entra ID)

Dialect `microsoft entra id / azure ad`, `event.category == 'authentication'`.

| Field | Notes |
|---|---|
| `azuread.properties.authenticationProtocol` | Verified; `deviceCode` for the device code flow |
| `azuread.properties.appDisplayName` | Verified; the application signed in to |
| `user_agent.original` | Verified; match a User-Agent quoted in a report with `==` |
| `source.ip`, `user.email`, `action.outcome` | Verified |
| `azuread.properties.riskLevelDuringSignIn`, `azuread.properties.conditionalAccessStatus` | Seen in sign-in events |

Audit events (consent grants, application registrations) are a separate log stream that not every tenant ingests; discover the `event.action` values before hunting them.

## Microsoft 365 audit

Dialect `microsoft 365 / office 365`.

| Field | Notes |
|---|---|
| `event.action` | Verified; operation names as Microsoft logs them (`UpdateInboxRules`, `New-InboxRule`, `Set-InboxRule`…) |
| `user.email` | Verified |

## Proxy, DNS and firewall

| Field | Notes |
|---|---|
| `url.original`, `url.full` | Verified; full URL with the query string |
| `url.domain`, `url.path` | Verified |
| `dns.question.name` | Verified |
| `related.hosts` | Verified; DNS names and URL hosts in one field |
| `related.ip`, `related.hash` | Verified; IPs and hashes anywhere in the event |
| `destination.ip`, `source.ip` | Documented |

## Worked examples

Device code sign-ins, from a report on a device code phishing kit:

```
events
| where timestamp between (ago(7d) .. now())
| where sekoiaio.intake.dialect == 'microsoft entra id / azure ad'
| where azuread.properties.authenticationProtocol == 'deviceCode'
| aggregate count() by user.email, azuread.properties.appDisplayName, action.outcome
| order by count desc
| limit 20
```

Delivery URL with a query string quoted in a report:

```
events
| where timestamp between (ago(7d) .. now())
| where url.original contains '<query string from the report>'
| aggregate count() by url.domain, host.name, sekoiaio.intake.dialect
| limit 20
```

Script host spawning an installer (a parent → child chain quoted in a report):

```
events
| where timestamp between (ago(7d) .. now())
| where process.parent.name in~ ['powershell.exe', 'pwsh.exe']
| where process.name in~ ['msiexec.exe']
| aggregate count() by host.name, sekoiaio.intake.dialect
| order by count desc
| limit 20
```

## Reading results

A hit is a lead to triage with the SOC, not attribution to the report's threat. An empty result means no match for that hypothesis in that window and that telemetry, not absence of the threat; check first that the source exists in the tenant (discovery) and that the field is filled on that dialect.
