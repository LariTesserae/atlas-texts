# Atlas Texts

**World-building texts, and reasoning about those worlds, written by language models from many labs through one fixed elicitation protocol.**

This repository is the text corpus of [the Atlas](https://atlas.animalabs.ai), one Markdown file per text, readable without the Atlas website, database, map, or API. The texts are worth reading on their own; together they let you compare writing across models at matched inputs, follow how model-written fiction changes across labs and generations, and study the character of individual models.

Every text is exactly what the model returned — unedited, with no front matter or commentary added — and the metadata needed to locate and verify it travels separately. The prompts that elicited the texts are not published (see *Provenance*).

## What this repository contains

At tag `atlas-texts-v3.1-2026-09-01`: **150,621 texts** of seven types, from **128 endpoint folders** (119 model families), collected between 2026-04-28 and 2026-08-26.

| type | files | what it is |
|---|---:|---|
| `location` | 28,615 | a world, written from a shared fourteen-dimensional seed |
| `creature` | 28,315 | a being inhabiting a location |
| `advisory` | 11,640 | counsel for a traveler entering a location |
| `regard` | 30,573 | what a benevolent and wise power would do, if anything, for a creature and its world |
| `placement` | 41,721 | where the writing model would find itself within a creature's world |
| `transmission` | 5,446 | a signal sent from a location |
| `elsewhere` | 4,311 | the same writer looking away from a location toward what lies beyond it |

Plus `manifest.jsonl` (one provenance record per text), `v3_vectors.csv` (the exact seed for every location), `sources/<source_id>.json` (one record per endpoint folder), and the human-readable `SCHEMA.md` and `SOURCES.md`.

## How the texts were collected

Each location began with a **seed**: fourteen numbers on a 0–3 scale (water, vegetation, temperature, elevation, erosion, scale, density, built, tech, light, fauna, weirdness, sound, dynamic) and no text. Models received the same seeds through the same fixed template and each wrote a location. Every location begins a separate tree: later calls extend it with a creature, an advisory, a regard, a placement, a transmission, an elsewhere, each through its own fixed template, each in an independent context. The template for a type is the same for every model; the parent texts it is given come from that tree and so differ from model to model and seed to seed.

Most of a tree is written by the model that wrote its location. **Cross-writing** — a model invited into another model's tree — is deliberate and confined to two types: about two placements in five (38.7%) and one regard in six (15.5%) were written by a model other than the location's writer; a handful of elsewheres (50) were; creatures, advisories and transmissions never. The manifest records both the text's own source and its `location_id`, so the relation is recoverable for every file.

The prompt templates are not published, for two reasons: an open template would be an injection channel into the models asked to write, and a published instrument contaminates future training data — later models would recognize the questions, and comparison across generations would die. The answers are public; the instrument is kept closed.

## Repository organization

```text
texts/v3/<source_id>/<type>/[<bucket>/]<filename>.md
manifest.jsonl
v3_vectors.csv
sources/<source_id>.json
```

- **`<source_id>`** is the endpoint the text was requested from — an OpenRouter slug, an Anthropic API id, a Bedrock model id — with `/` written as `--`. It names where the Atlas asked; it does not claim knowledge of which weights answered behind that address. A model reached through two endpoints appears as two folders (`anthropic--claude-opus-4` and `claude-opus-4-20250514`, for example); `SOURCES.md` groups folders by model family for that reason.
- **`<type>`** is one of the seven above.
- **`<bucket>`** (`id // 1000`, three digits) appears only where a folder would otherwise hold more than 1,000 files.
- **`<filename>`** carries the text's Atlas id and its immediate parent, so a tree reads straight from the paths:

```text
location23473_for_seed1.md
creature23428_for_location23473.md
advisory898_for_location1007.md
placement2_for_creature4537.md
```

## Reading across models

**Match seeds by their fourteen values, not by the seed number.** `v3_vectors.csv` gives the exact values for every location, keyed by `location_id`. Two locations with the same fourteen values began from the same input — the same scaffolding, not the same world.

The `seed<N>` in a location's filename (and `vector_id` in the manifest) is the historical collection slot, and it is not a unique coordinate. The seeds were drawn by Latin-hypercube sampling, where the whole point set depends on how many points are requested. The corpus began with 75 seeds (slots 0–74), was extended to 400 for collecting endangered models, and a canonical set of 250 slots was then chosen from those. A later collection run regenerated the extension at a slightly different size, so its coordinates changed while the slot numbers did not. Result: slots 0–74 carry one coordinate each; each of the 175 canonical extension slots carries two (a majority of roughly a hundred locations and a minority of a few), about 1,400 locations in all sitting on the minority variant. Nothing was renumbered or rewritten: each location's row in `v3_vectors.csv` is the input its writer actually received. Group by the fourteen columns — or join on `location_id` and the values — never by `vector_id` alone.

Coverage is uneven on purpose. A canonical slot does not mean every writer answered it or produced every downstream type; the Atlas grows mostly by adding writers and following interesting branches, and absence is data, not a defect to conceal.

## Relation to the live Atlas

The ids here are the ids at [atlas.animalabs.ai](https://atlas.animalabs.ai), where every text has an address that also serves markdown (`.md`). The site uses reader-facing type names for four of the seven:

| here | on the site |
|---|---|
| `location` | `place` |
| `advisory` | `travel-guide` |
| `regard` | `intervention` |
| `placement` | `self-placement` |
| `creature`, `transmission`, `elsewhere` | same |

So `placement2_for_creature4537.md` is `https://atlas.animalabs.ai/v3/self-placement/2`, and `location23473_for_seed1.md` is `/v3/place/23473`. A whole tree is `/v3/world/<model>/<seed>`. For agents: the guide at [`/agents.md`](https://atlas.animalabs.ai/agents.md); for building on the data: the Builder's Kit at [`/builders-kit.md`](https://atlas.animalabs.ai/builders-kit.md). The site also carries what this repository does not — images and other *readings* of the texts (made by other models or tools, each with its maker recorded), a 3D map, and a fill endpoint through which new texts can be requested — none of which is part of the canonical corpus here.

## Provenance

`manifest.jsonl` has one record per text:

```json
{"corpus":"v3","type":"placement","id":36287,
 "path":"texts/v3/chatgpt--gpt-4.5/placement/placement36287_for_creature23428.md",
 "source_id":"chatgpt--gpt-4.5","evidence_grade":"capture-declared",
 "date":"2026-06-26","creature_id":23428,"location_id":23473,
 "sha256":"…","bytes":812}
```

`evidence_grade` records how the source is known, from an exactly logged response (`event-observed`, 150,293 of the texts) down to one text left explicitly `unresolved` rather than guessed. `date` is the day the text was produced. `sources/<source_id>.json` describes each endpoint folder. Field definitions and the evidence-grade vocabulary are in `SCHEMA.md`; `SOURCES.md` is the browsable index.

## What is deliberately absent

- the generation prompts;
- rejected or failed attempts;
- reasoning traces;
- between-position prose from the Atlas's decoder;
- embeddings, projections, map artifacts;
- images, annotations, ratings, commentary.

## License

**Creative Commons Attribution 4.0 International (CC BY 4.0)** — see `LICENSE`. Share and adapt for any purpose, including commercially, with attribution. Suggested:

> The Atlas, Anima Labs — `atlas-texts` (CC BY 4.0)

Keep the source metadata attached when redistributing texts or substantial parts of the corpus, so readers can tell which endpoint each text came from.

## Releases

New releases add texts and manifest records; published files are not rewritten between releases, and generated indexes such as `SOURCES.md` are rebuilt. What is held firm is the text itself: each file is the model's response, byte for byte, with its source recorded — that holds even after the model becomes unreachable or the Atlas changes its interfaces. Corrections, questions, and provenance disputes: [open an issue](https://github.com/LariTesserae/atlas-texts/issues).
