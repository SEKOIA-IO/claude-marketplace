# Field guide

What the fields returned by the Sekoia MCP threat intelligence tools mean, and how to read them in an answer. Values observed on the live server in September and October 2026.

## Common object fields

| Field | Meaning | How to read it |
|---|---|---|
| `id` | STIX ID, `<type>--<uuid>` | Quote it in full; it is the link target `https://app.sekoia.io/objects/<id>` |
| `type` | STIX type | `intrusion-set`, `threat-actor`, `campaign`, `malware`, `tool`, `attack-pattern`, `vulnerability`, `identity`, `infrastructure`, `report`, `indicator`, observable types such as `ipv4-addr`, `domain-name`, `file` |
| `name`, `aliases` | Name and recorded aliases | "Sekoia records X as an alias of Y" |
| `description` | Analyst text, may quote vendor text | Facts inside it are description statements, weaker than relationships |
| `confidence` | STIX 2.1, 0 to 100 | Read per object or per relationship; 100 confirmed by other sources, 70 probably true, 50 possibly true, 30 doubtful; absent when the field is missing |
| `first_seen` | Start of observed activity | The activity date on objects; absent on some |
| `last_seen` | End of observed activity | Read on indicators only. On other objects it is rarely filled and sometimes a period-end placeholder (2026-12-31 on actors in October 2026); ignore it there and use the latest relationship date (below) |
| `created`, `modified` | Sekoia record dates | Never activity dates; `modified` changes on edits |
| `published` (reports) | Publication date | The date to cite for a report |
| `revoked`, `x_ic_revocation_reason` | Withdrawn object | Use the surviving object the reason names |
| `x_ic_deprecated` | Retired content | Say so if quoted |
| `x_ic_is_in_flint` | Covered by a Sekoia FLINT report | Signals Sekoia-authored coverage |
| `x_ic_is_sector`, `sectors`, `identity_class` | Sector identity marker | The pivot object for sector questions |
| `x_inthreat_uploaded_files` (reports) | Files attached to the report: `file_name`, `mime_type`, `sha256` | The FLINT PDF; the MCP does not return it, the Sekoia API does with `GET /v2/inthreat/objects/<id>/files/<sha256>` (used by `report-hunt`); the marking printed on the PDF can be stricter than `object_marking_refs` |
| `x_inthreat_sources_refs` | Source identities, on objects and on observables | Sekoia has several source identities, all named `Sekoia …` (see Source identities below); the others are vendors or government sources |
| `external_references` | Source URLs | The primary source behind a campaign or report |
| `object_marking_refs` | TLP markings | See below |
| `goals` (intrusion sets, threat actors) | List; mostly the goal vocabulary below, with legacy free text on older records | Match by stem, case-insensitively; quote off-vocabulary values as recorded |
| `objective` (campaigns) | One string; the goal vocabulary on recent records, free text and multi-value strings on older ones | Same as `goals` |
| `primary_motivation`, `secondary_motivations` | STIX 2.1 `attack-motivation-ov`: `organizational-gain`, `ideology`, `notoriety`, `personal-gain` and the rest of that list | On some actors only; why the actor acts, where `goals` says what it does |
| `resource_level` | STIX 2.1 `attack-resource-level-ov`: `individual`, `club`, `contest`, `team`, `organization`, `government` | Quote as recorded |

## Goal vocabulary

`goals` on intrusion sets and `objective` on campaigns share one list of values. Current records use it in lower case; older records predate it. Values in use in October 2026, with the stem that matches them and their legacy spellings:

| Value | Stem | Meaning |
|---|---|---|
| `espionage` | `espion` | Intelligence collection |
| `lucrative` | `lucrativ` | Financial gain: ransomware, theft, fraud, extortion |
| `sabotage` | `sabotag` | Destruction or degradation of systems or data |
| `disruption` | `disrupt` | Temporary denial of service or availability |
| `influence` | `influenc` | Information operations, hack-and-leak, propaganda |
| `reconnaissance` | `reconnais` | Target discovery and access preparation |
| `pre-positioning` | `pre-?position` | Persistent access kept for later effect |

The list is observed, not a published enumeration. Older records can differ:

- Casing and spelling variants: the stem catches them.
- Several values in one string on campaigns ("Sabotage / Espionage"): the stem catches each value.
- Free-text values the stem does not catch ("Financial gain", "Ransomware", "Hacktivism").
- No value at all on some objects: count them and say so.

Report a synonym as recorded ("goal recorded as 'Financial gain'"), never folded silently into a vocabulary value. Grouping it under the nearest value is an assessment, labelled as such.

An actor's `goals` cover its whole history; they do not say what a given campaign did. Only the campaign's `objective` is evidence for that. Never carry an actor goal over to one of its campaigns or to one of its sector relationships ("APT28 has an espionage goal and targets Energy" is not "APT28 conducts espionage against Energy").

## Source identities

`x_inthreat_sources_refs` lists identity objects. Sekoia's own production is spread over a dozen of them: the main `Sekoia` identity and one per collection capability. Confirmed on the live server in October 2026:

| Identity | Name |
|---|---|
| `identity--357447d7-9229-4ce1-b7fa-f1b83587048e` | Sekoia |
| `identity--556006db-0d85-4ecb-8845-89d40ae0d40f` | Sekoia C2 Tracker |
| `identity--62295172-3a3a-4433-8663-e018d20c7f6c` | Sekoia Honeypot |
| `identity--9a70c9af-d7cb-49bd-b23c-46dee62f2d91` | Sekoia Malware Watcher |
| `identity--c66b518f-5955-4092-8943-d1ed7b5be767` | Sekoia Dark Web Monitoring |

The table is not exhaustive. Any source ID not listed here: retrieve it with `get_cti_object_by_id`. It is a Sekoia source when its `name` starts with `Sekoia`; Sekoia source identities also carry `x_ic_is_source: true` and `contact_information: tdr@sekoia.io`.

All of them are Sekoia first-party evidence, not ingested third-party reporting. Name the specific source in the answer ("Sekoia C2 Tracker", not just "Sekoia"): it says how the evidence was collected.

## Relationship entries

`get_cti_object_relationships_by_id` returns `{related_object, relationship}` pairs. On the `relationship`:

| Field | Meaning |
|---|---|
| `relationship_type` | `attributed-to`, `uses`, `targets`, `exploits`, `originates-from`, `indicates`, `related-to` |
| `source_ref`, `target_ref` | Direction: a campaign is `attributed-to` an intrusion set; an intrusion set `targets` an identity |
| `confidence` | Confidence of the link itself, independent of both objects |
| `created`, `modified` | Record dates; no activity date exists on a relationship |

A relationship's meaning stays within its direction and type. "Intrusion set targets Telecommunications identity" is sector relevance; it is not a list of victims and it does not apply to every campaign of that intrusion set.

**Latest relationship date.** For an actor, malware family or campaign, the most recent `relationship.created` across the slices fetched says when Sekoia last linked something to the object. It is the freshness signal for Sekoia's knowledge, not an activity date. Pair it with the `first_seen` of the most recent related campaign for activity. On a slice, inline or persisted:

```
jq -r '.result | max_by(.relationship.created) | [.relationship.created[:10], .relationship.relationship_type, .related_object.name, .related_object.type] | @tsv' <file>
```

Name the related object and the slices covered ("latest relationship 2026-09-16, a campaign, across the campaign slice"); a slice not fetched may hold a later one.

## Observables and indicators

`search_observables` returns, per searched value:

| Field | Meaning |
|---|---|
| `found` | Whether the value exists in the database |
| `observable.x_inthreat_sources_refs` | Who reported the value; often Sekoia C2 Tracker (see Source identities) |
| `observable.x_ic_indicated_threats` | Threats currently indicated by a valid indicator on this observable; empty means known with no current indicated threat |
| `observable.created`, `modified` | Record dates |
| indicator `last_seen` | Last time the indicator was observed, when present; the only object type where `last_seen` is read |
| indicator `valid_from`, `valid_until` | Validity period, when an indicator is returned; expired validity is historical context |

## Markings

| ID | Marking |
|---|---|
| `marking-definition--613f2e26-407d-48c7-9eca-b8e91df99dc9` | TLP:WHITE |
| `marking-definition--34098fce-860f-48ae-8e50-ebd3cc5e41da` | TLP:GREEN |
| `marking-definition--f88d31f6-486f-44da-b317-01333bde0b82` | TLP:AMBER |

Any other marking ID: retrieve it with `get_cti_object_by_id` before quoting the object outside the session. When the user intends to share the answer, state the strictest marking among the objects used. For a report, the marking printed on its PDF counts too: FLINT 2026-037 carries TLP:WHITE on the object and TLP:AMBER on the PDF.

## Overflow

On well-connected objects a relationship slice or a report list exceeds the response limit and the client persists the payload to a file (path given in the tool result) instead of returning it. Read it:

```
jq '.result[] | {name: .related_object.name, id: .related_object.id, type: .related_object.type, conf: .relationship.confidence, rel: .relationship.relationship_type, first_seen: .related_object.first_seen}' <file>
```

A recovered slice is complete (VALUE). A slice that could not be recovered is NOT_RETURNED and is never presented as absence.
