# Changelog

## 0.1.0

Initial release.

- `sekoia-cti`: Sekoia MCP Server connection and the skills `query`, `evidence-protocol`, `exposure-assessment`, `threat-horizon-assessment`, `intelligence-delta`, `report-hunt`.
  - `report-hunt`: hunt packs derived from FLINT reports (existing CTI alerts, telemetry coverage, built-in rules, behaviour queries).
  - Tenant check in `query` (recipe 3) with alerts raised by Sekoia CTI and indicator sweeps; optional tenant check in `exposure-assessment` and `threat-horizon-assessment`. SOL queries on `events` are bounded with `timestamp between (…)`, which the server requires.
- `sekoia-soc`: Sekoia MCP Server connection and the skills `alert-details`, `case-summary`, `sol-hunt`, with a curated SOL reference and tested query patterns.
