# ro-item-db

Pre-renewal Ragnarok Online item database, built from an
[rAthena](https://github.com/rathena/rathena) snapshot and published as ready-to-use
gzipped JSON bundles.

It powers the item tooltips, "where do I get this" lookups and drop notifications in
[roBrowser](https://github.com/MrAntares/roBrowserLegacy) plugins, but the bundles are
plain JSON — nothing in here is roBrowser-specific.

## Consuming a bundle

Fetch it straight from jsDelivr, pinned to a commit so the URL is immutable and
cached forever:

```
https://cdn.jsdelivr.net/gh/<owner>/ro-item-db@<commit-sha>/bundles/rathena-pre-re.json.gz
```

`raw.githubusercontent.com/<owner>/ro-item-db/<commit-sha>/bundles/…` serves the same
bytes if jsDelivr is unreachable. Both send `Access-Control-Allow-Origin: *`, so a
browser can fetch them directly.

```js
const res  = await fetch(URL)
const json = await new Response(
    res.body.pipeThrough(new DecompressionStream('gzip'))
).text()
const { manifest, items, skillNames, mobNames } = JSON.parse(json)

items[1201]           // { id, name, aegis, type, slots, bonuses, producedBy, consumedBy,
                      //   droppedBy, boughtFrom, unlocks, … }
mobNames[1002]        // "Poring"
skillNames.AL_HEAL    // { id: 28, desc: "Heal" }
```

Every bundle is self-contained: one fetch gives you the whole database, nothing to
merge client-side.

## Bundles

| Bundle | Contents |
| --- | --- |
| `bundles/rathena-pre-re.json.gz` | Stock rAthena pre-renewal, plus the recipes and quests the script parsers drop as orphans. No server-specific content. |
| `bundles/cyro.json.gz` | `rathena-pre-re` plus cyro's own NPCs (Hat Maker). |

[`manifest.json`](manifest.json) at the repo root indexes what each bundle holds
(item counts, recipe counts, the rAthena commit it was built from, build timestamp) so
you can tell what changed without downloading anything.

## Asset list (`rocache/`)

Not part of the item database: `rocache/pre-re.json.gz` is the list of files in a
pre-renewal `data.grf` (path + size per entry), used by roBrowser's Rocache plugin to
download the client assets in bulk. It is hosted here for the same pinned jsDelivr
URL. `rocache/pre-re.meta.json` carries its counts and sha1.

It is generated from a GRF, not from rAthena, by
`robrowser/tools/v3/plugins/rocache/tools/build-asset-list.mjs`.

## Editing the data by hand

Most of the database is derived mechanically from rAthena's `.yml` and `.txt` files.
What lives in this repo is the part that *can't* be — recipes the script parsers can't
see, and server-specific content. Those are the override layers, and they are meant to
be edited by hand (the GitHub web editor is fine):

```
cyro/
  hatmaker.json                  the custom Hat Maker NPC in izlude
rathena-classic/
  headgear-quests.json           stock headgear quest NPCs with no setquest anchor
  lvl4-weapon-quest.json         Level 4 Weapon Quest — driven by a permanent variable
  rms-fixes.json                 corrections from a RateMyServer cross-check
```

Each file is a document with three optional arrays — `items`, `quests`, `produces` —
plus a free-form `_comment`. Keys starting with `_` (`_comment`, `_name`) are
documentation and are ignored by the builder, so use them freely to keep entries
readable:

```jsonc
{
  "_comment": "why this file exists",
  "produces": [
    {
      "category": "hatmaker",
      "product":   { "id": 5086, "qty": 1 },
      "materials": [ { "id": 5024, "qty": 1 }, { "id": 539, "qty": 30 } ],
      "zeny": 0,
      "npc": "Hat Maker", "map": "izlude", "x": 125, "y": 118,
      "_name": "Alarm Mask"
    }
  ]
}
```

[`bundles.json`](bundles.json) maps bundles to the layers they stack. Adding a layer
directory and an entry there is all it takes to publish another server's database.

## Rebuilding

The builder is not in this repo — it lives alongside it in the roBrowser tree, at
`robrowser/rathena/item-db/`. It reads the override layers from here and writes
`bundles/` and `manifest.json` back into it:

```bash
cd robrowser/rathena/item-db
RATHENA_SOURCE=snapshot npm run build
```

`RATHENA_SOURCE=snapshot` builds against the pinned rAthena tarball. Without it the
builder re-fetches `master` from GitHub, which makes the whole database drift instead
of just the entries you edited. Set `ITEM_DB_PUBLISH_DIR` if this repo is not checked
out next to the roBrowser tree.

Publishing is `git push` — there is no upload step. Then point consumers at the new
commit SHA.

## Licence

The data is derived from [rAthena](https://github.com/rathena/rathena) (GPL-3.0);
item and monster names are Gravity's. Published for use with Ragnarok Online private
servers and community tooling.
