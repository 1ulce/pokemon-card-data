# pokemon-card-data

Open dataset of Pokémon TCG card data (Japanese-side master), derived from
[PTCGL Replayer](https://replay.ptcgtools.com)'s scraping pipeline.

Each `CardFace` (logical card) is exported as a single YAML file under
`cards/{slug}.yml`. Both Japanese and English text are included where available.

## Layout

```
cards/                       1 face = 1 file (~2000 entries, flat)
  pikachu-ex.yml
  charizard-ex.yml
  ...
sets/
  manifest.yml               JP set master (released_at + regulation_mark)
index/
  all.json                   bulk index (slug + name + first_set)
  slugs.txt                  all slugs, newline-separated
  by-regulation.json         regulation_mark → [slug, ...]
  by-type.json               card_type → [slug, ...]
  by-energy.json             energy_type → [slug, ...]
meta.yml                     schema_version / generated_at / total_faces
```

## Quick start

Fetch a single card:

```
curl https://raw.githubusercontent.com/1ulce/pokemon-card-data/main/cards/pikachu-ex.yml
```

Fetch the bulk index:

```
curl https://raw.githubusercontent.com/1ulce/pokemon-card-data/main/index/all.json
```

## Card YAML schema

```yaml
schema_version: 2
slug: pikachu-ex                        # ASCII unique key

name:
  ja: ピカチュウex
  en: Pikachu ex
  prefix: null                          # e.g. "Marnie's" possessive prefix

card_type: Pokémon                      # Pokémon | Trainer | Energy
card_subtype: たねポケモン                # subtype label (JP-side text)
energy_type: Lightning
hp: 200
retreat_cost: 1
regulation_mark: H                      # H | I | J | null

meta_tags: [ex]                         # ex / mega / ace_spec / tera ...

evolves_from:
  ja: ピチュー
  en: Pichu

weakness:
  type: Fighting
  value: "×2"
resistance:
  type: null
  value: null

body_text:                              # Pokémon descriptive text
  ja: ...
  en: ...
rules_text:                             # Trainer/Tool rules text
  ja: ...
  en: ...

abilities:
  - position: 1
    name:
      ja: でんきにくいせ
      en: Static Resilience
    effect_text:
      ja: ...
      en: ...

attacks:
  - position: 1
    name:
      ja: 10まんボルト
      en: Thunderbolt
    cost: [Lightning, Lightning, Colorless]
    damage: "120"
    effect_text:
      ja: ...
      en: ...

first_appearance:                       # always present (the first JP printing)
  set_name_ja: ワイルドフォース
  released_at: 2024-01-26               # may be null for promo / fallback cases
  regulation_mark: H                    # may be null
  collector_number: "012/078"
  image_url: https://www.pokemon-card.com/.../pikachu-ex.jpg

printings:                              # schema_version 2: every physical printing (JP then EN)
  - language: ja                        # ja | en
    set_code: SV5K                      # set identifier (unique per language); may be null for JP products without one
    set_mark: SV5K                      # the set mark printed in the card's number box (SV9 / SVN / SV1a / SV-P ...)
    set_name: ワイルドフォース
    collector_number: "012"             # as printed; special rares may exceed the set size (no set total is stored)
    collector_number_unprinted: false   # true = the card has no number; collector_number holds a source ID substitute
    rarity: RR                          # shared vocabulary (JP mark codes): C U R RH PR RR RRR K A AR CHR SR SAR CSR S SSR TR HR UR MA BWR
    rarity_unprinted: false             # true = no rarity mark is printed (deck products, high-class normal slots)
    illustrator: 5ban Graphics          # normalized (one spelling per person); co-illustrators joined with " / "
    illustrator_unprinted: false        # true = no illustrator credit is printed (basic energy)
    regulation_mark: H                  # as printed on this printing (JP reprints keep the first-appearance mark)
    regulation_mark_unprinted: false    # true = printed before regulation marks existed (EN before SWSH, reprints)
    image_url: https://...
```

### Three states of a printing attribute

Each of `rarity` / `illustrator` / `regulation_mark` (and the number) can be in one of three states:

| state | value | `*_unprinted` | meaning |
|---|---|---|---|
| present | non-null | `false` | the attribute as printed on the card |
| unprinted | `null` | `true` | the card physically has no such mark (deck products, basic energy, pre-regulation-mark era) |
| unknown | `null` | `false` | not yet collected from any source |

`collector_number` is never null: when the card has no number, the value is a source ID substitute and
`collector_number_unprinted` is `true`.

Rarity uses one vocabulary for JP and EN so the same rarity compares equal across languages
(JP `RR` = EN "Double Rare", JP `SR` = EN "Ultra Rare", JP `UR` = EN "Secret Rare"/hyper rare,
`RH` = EN-only "Rare Holo"). The source wording is not exported.

### `first_appearance` selection

For each face, the JP printings are scanned in this order:

1. If any printing belongs to a `CardSet` with a known `released_at`, the
   printing whose set has the **oldest `released_at`** is chosen.
2. Otherwise, the printing with the **smallest `pokemon_card_id`** (which is
   roughly the order cards were registered on pokemon-card.com) is used as a
   fallback. In that case `released_at` and `regulation_mark` will be `null`.
3. If a face has no JP printing at all (extremely rare), `first_appearance`
   will be `null`.

This guarantees `first_appearance.set_name_ja` is always present whenever a
face has at least one JP printing.

## Update cadence

Regenerated periodically from the upstream PTCGL Replayer database. See
`meta.yml` for the timestamp of the current snapshot.

## License & rights

- Dataset structure / schema / generated text representations: **CC0 1.0** (see `LICENSE-DATA`).
- Card names, effect text, image URLs, and trademarks are property of
  **The Pokémon Company / The Pokémon Company International / Creatures Inc. /
  GAME FREAK Inc.** This is a fan-made, community resource intended for
  educational and analytical use. See `NOTICE.md` for details.

If you are a rights holder and need this resource taken down, please open an
issue and we'll comply.

## Related

- [PTCGL Replayer](https://replay.ptcgtools.com)
- 日本語 README: [README.ja.md](README.ja.md)
