# pokemon-card-data

ポケモンカードゲームのカードデータ（日本語マスター）の公開データセット。
[PTCGL Replayer](https://replay.ptcgtools.com) のスクレイピングパイプラインから生成。

`CardFace`（論理カード）単位で 1 ファイル = `cards/{slug}.yml`。日本語と英語のテキストを併記。

## レイアウト

```
cards/                       1 face = 1 ファイル（フラット、~2000 件）
sets/manifest.yml            JP セット master（released_at + regulation_mark）
index/all.json               全 face の概要 (slug + name + first_set)
index/slugs.txt              全 slug 改行区切り
index/by-regulation.json     regulation_mark → [slug, ...]
index/by-type.json           card_type → [slug, ...]
index/by-energy.json         energy_type → [slug, ...]
meta.yml                     schema_version / generated_at / total_faces
```

## クイックスタート

1 枚取得:

```
curl https://raw.githubusercontent.com/1ulce/pokemon-card-data/main/cards/pikachu-ex.yml
```

一括インデックス:

```
curl https://raw.githubusercontent.com/1ulce/pokemon-card-data/main/index/all.json
```

## カード YAML スキーマ

英語版 [README.md](README.md) の "Card YAML schema" を参照。

## `first_appearance` の決定

各 face の JP printings をこの順で評価:

1. `released_at` が入った `CardSet` を持つ printing があれば、その中で **`released_at` が最古** のものを採用
2. 無ければ、**`pokemon_card_id` が最小** の printing をフォールバック (= pokemon-card.com に最初に登録された個体に近い)。`released_at` と `regulation_mark` は `null` になる
3. JP printing が 1 件も無い face（極めて稀）は `first_appearance: null`

## ライセンス・権利

- データセットのスキーマ・構造・生成テキスト: **CC0 1.0**（`LICENSE-DATA` 参照）
- カード名・効果テキスト・画像 URL・商標は **株式会社ポケモン／The Pokémon Company International／株式会社クリーチャーズ／株式会社ゲームフリーク** の権利物。これは fan-made なコミュニティリソースで、教育・分析目的での利用を想定しています。詳細は `NOTICE.md`。

権利者の方で削除をご希望の場合は issue を立てていただければ対応します。
