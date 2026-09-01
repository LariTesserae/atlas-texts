# Schema

The exact contract for this repository. Everything here is stable: paths, filenames,
manifest fields, and the source vocabulary are append-only promises.

## 1. Canonical text files

For a text with database value `full_text`, the complete file is exactly:

```
full_text.encode("utf-8")
```

No front matter, no attribution, no canary, no prompt, and **no added trailing
newline**. `.gitattributes` disables all normalization below `texts/`, so the bytes
you clone equal the bytes the model returned. Each file's `sha256` and `bytes` in the
manifest are the hash and length of exactly these bytes.

## 2. Path grammar

```
texts/v3/<source_id>/<type>/[<bucket>/]<filename>
```

- **`<type>`** ∈ `location`, `creature`, `advisory`, `regard`, `placement`,
  `transmission`, `elsewhere`. (`regard` includes records called *interventions*
  elsewhere in the project; there is no separate public type for them.)
- **`<bucket>`** = `id // 1000`, zero-padded to three digits, present **only** in a
  `<source_id>/<type>` folder that holds **more than 1,000 files**. Small folders are
  flat. A folder's sharded-or-flat state is fixed at first publication and never
  changes (append-only).
- **`<filename>`** encodes the parent relation (see §3).

## 3. Filenames

Self-describing, so a text carries its position even outside the tree:

| type | filename pattern |
|---|---|
| `location` | `location<id>_for_seed<vector_id>.md` |
| `creature` | `creature<id>_for_location<location_id>.md` |
| `advisory` | `advisory<id>_for_location<location_id>.md` |
| `elsewhere` | `elsewhere<id>_for_location<location_id>.md` |
| `transmission` | `transmission<id>_for_location<location_id>.md` |
| `placement` | `placement<id>_for_creature<creature_id>.md` |
| `regard` | `regard<id>_for_creature<creature_id>.md` |

`<id>` is the item's stable corpus id. The relation ids resolve to other files in the
tree. `seed<vector_id>` preserves the historical vector-slot label recorded during
collection; it is **not** guaranteed to be a unique coordinate. The exact
fourteen-dimensional coordinate for a location is in `v3_vectors.csv`, keyed by the
location's stable id (§6).

## 4. `manifest.jsonl`

One JSON object per line, sorted by `(type, id)`. Fields:

| field | meaning |
|---|---|
| `corpus` | always `"v3"` |
| `type`, `id` | the item's type and stable id |
| `path` | its path under `texts/` |
| `source_id` | the endpoint folder it lives in (§5) |
| `evidence_grade` | how the source is known (§5.1) |
| `date` | the call date, `YYYY-MM-DD` (the day the text was produced) |
| `sha256`, `bytes` | hash and byte length of the file |
| `location_id` / `creature_id` / `vector_id` | structural relations, where the type has them |

A release manifest is immutable once tagged. Generated indexes (`SOURCES.md`,
`catalog/`) may be regenerated.

## 5. Sources

`sources/<source_id>.json` describes one endpoint folder. The **`source_id` is the
requested endpoint**, slugged (`/` → `--`, other separators → `-`). Channel is *not*
in the path: the same model reached through different access paths shares one folder,
because the endpoint — not the access route — is what identifies the writing.

**An endpoint is the model route that was requested, read from our own inference
logs. It is not a claim about which underlying weights answered.** Where a log
recorded only the project's internal model tag rather than a provider API id, the
folder name is that tag verbatim (`endpoint_granularity: "house-tag"`); we do **not**
reconstruct an API id it didn't record. Bedrock ids have their region/inference-profile
prefix stripped. Nothing is ever invented to look more precise than the log.

### 5.1 Evidence grades

Strongest first. Every text has exactly one.

| grade | meaning |
|---|---|
| `event-observed` | exact response bytes joined to a logged inference event |
| `batch-observed` | exact response bytes joined to a preserved batch-output artifact |
| `artifact-observed` | exact response bytes survive in another sourced artifact |
| `capture-declared` | a manual capture surface (e.g. a chat transcript) identifies the source |
| `event-correlated` | a provider event without response bytes identifies the endpoint by timing/session |
| `operation-observed` | an internal operation/fill record names the requested endpoint |
| `route-reconstructed` | the endpoint follows from the historical route and collection context |
| `unresolved` | the evidence does not justify a more specific source |

### 5.2 `source-unresolved`

Exactly one address may be genuinely undecidable between provider channels on the
available evidence; it lives under `texts/v3/source-unresolved/…` and is graded
`unresolved`. It is left open rather than assigned to a plausible-looking route.
Resolving it later would move a path and is therefore a deliberate, documented act,
not an automatic one.

## 6. Seeds — `v3_vectors.csv` and comparing across models

Every `location` was generated from a fourteen-dimensional coordinate. The **exact
coordinate each location received** is published in `v3_vectors.csv`, one row per
published location:

- Header (16 columns): `location_id,vector_id,` then the fourteen dimensions in this
  order — `water, vegetation, temperature, elevation, erosion, scale, density, built,
  tech, light, fauna, weirdness, sound, dynamic`.
- Values are numeric on the public **0–3** scale; the authority is the snapshot's
  `locations.param_vector`, verbatim. Rows are sorted by ascending `location_id`.
- The file carries numbers, dimension names, and the range only — **no** prompt text,
  template, rendered parameter prose, or natural-language level labels.

**`vector_id` is a historical slot label, not a unique coordinate.** The original seed
ids `0–74` have one stable coordinate each. During a later expansion, the 175 added
seed ids were reused for two different coordinate sets across collection runs (a
resampling artifact); about 1,400 locations sit on the minority variant. So two
locations sharing a `vector_id` may carry different coordinates.

To compare across models, **join on `location_id` and match the fourteen values** (or
on both `vector_id` and the values) — never on `vector_id` alone. An analyst wanting a
unique-vector table can `drop_duplicates` over the fourteen columns; the corpus does
not manufacture one canonical winner per `vector_id`. Everything downstream (creatures,
placements, …) inherits its world through the `_for_location` / `_for_creature` links,
and reaches its numeric seed through the location's row in `v3_vectors.csv`.

## 7. Invariants

- One item → exactly one path. No path is reused.
- A published text is never rewritten, renamed, or deleted; changed source bytes open
  a new review, never an overwrite.
- Later releases add new texts and new immutable manifests; generated indexes may
  change.
- Public files and records contain no prompts, system messages, request payloads,
  reasoning, credentials, or private paths.
- Every manifest location has exactly one row in `v3_vectors.csv`.
- A published location's numeric seed row is immutable; a changed coordinate for an
  already-published location is a provenance failure, not a routine regeneration.
- `vector_id` is a historical label and is not required to identify one unique
  coordinate across all v3 records.
