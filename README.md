# BlockStateUpgradeSchema
(Mostly) auto-generated schemas for upgrading blockstates found in older Minecraft: Bedrock worlds

## Contents
### `nbt_upgrade_schema/*.json`
These schemas describe how to upgrade blockstate NBT from one version to the next. The structure of the schema is described in `nbt_upgrade_schema_schema.json`. An example implementation can be seen [in PocketMine-MP 5.8.2](https://github.com/pmmp/PocketMine-MP/blob/5.8.2/src/data/bedrock/block/upgrade/BlockStateUpgrader.php).

#### Notes
- Mojang don't always bump the format version when making backwards-incompatible changes. A prominent example of this is in the [`0131_1.18.20.27_beta_to_1.18.30.json`](/nbt_upgrade_schema/0131_1.18.20.27_beta_to_1.18.30.json).
- `remappedStates` requires different treatment than other rules:
  - It **must have highest priority**
  - If a blockstate matches a remap rule, **do not apply any other transformations** (the newly mapped state will be correct for the new version).
  - `oldState` acts as a **search criteria, not an exact match**.
    - Because of this, the remapped state rules must be tested in the order of most criteria to least criteria (this is the order they are provided in by the JSON).
    - For example, the transformation of `minecraft:concrete` with `silver` colour is different from other colours, as seen in [`0201_1.20.0.23_beta_to_1.20.10.24_beta.json`](/nbt_upgrade_schema/0201_1.20.0.23_beta_to_1.20.10.24_beta.json#L69-L88). If matches were tested in the wrong order, `silver` concrete would be transformed incorrectly.
- With the exception of `remappedStates`, modifications can be applied in any order, e.g. `renamedIds` can be applied before or after `renamedProperties`.
  - To facilitate this, `addedProperties`, `renamedProperties`, `removedProperties` and `remappedPropertyValues` always use the old blockID and old property names for indexing.

### `block_legacy_id_map.json`
This JSON file contains a mapping of string ID -> legacy ID for all blocks known up 1.16.0.

Technically, you'd only need everything up to 1.2.13, but the excess might be handy in some cases.

### `id_meta_to_nbt/*.bin`
These binary files contain mappings of all known valid ID/meta combinations to their corresponding blockstate NBTs for that version.
For example, `1.12.0.bin` allows you to convert 1.12.0 ID/meta into 1.12.0 NBTs.

#### Usage
If you only plan to upgrade worlds to the latest version (1.20.1 at the time of writing), you will only need the 1.12.0 file, which will suffice for upgrading all older blocks (and blockitems) to 1.12 and newer.

However, if you want to upgrade to 1.9, 1.10 or 1.11, you may need the 1.9.0 file in order to upgrade saved blockitems, as 1.9 started to save blockitems using blockstate NBT on disk. The 1.12.0 file won't be suitable for this case, as it contains the NBT blockstates appropriate for 1.12, which will not work on earlier versions.

If you previously used the now-deleted `1.12.0_to_1.18.10_blockstate_map.bin`, you can replace it directly with the `1.12.0.bin`. You will need to upgrade the states within as before.

#### Schema
The file is structured as described below.

- unsigned varint32 - Number of entries
  - unsigned varint32 - block string ID length
  - byte[] - block string ID
  - unsigned varint32 - Number of meta -> blockstate pairings
    - unsigned varint32 - Meta value
    - TAG_Compound (standard little-endian) - NBT blockstate (as of that file's version) corresponding to the ID and meta pair

## Generating NBT upgrade schemas for new versions

First, you need to get a `.bin` mapping file, which you can obtain using the current version of BDS + [pmmp/mapping mod](https://github.com/pmmp/mapping). It requires that you place the palette for the previous version in `input_files/old_block_palettes`.

The output file will be placed in `mapping_files/old_palette_mappings`. This file is then provided as the input for the [schema generator script](https://github.com/pmmp/PocketMine-MP/blob/5.8.2/tools/generate-blockstate-upgrade-schema.php), which produces the JSON schemas like the ones you see in this repo.

Currently the code needed for this is baked into an experimental branch of PocketMine-MP; it's planned to separate it into its own library in the future.

## Background

Since Minecraft Bedrock doesn't auto-upgrade terrain or inventories unless they've been loaded during a game session, any software that supports Minecraft Bedrock worlds has to accommodate all of the old blockstates all the way back to the first versions that used them, and many backwards-incompatible changes were made since then.
If they do not, they may randomly fail to load chunks in older worlds that work just fine in latest Bedrock.

Other projects such as [CloudburstMC/BlockStateUpdater](https://github.com/CloudburstMC/BlockStateUpdater) attempted to address this by writing library code to deal with the problem; however, this approach comes with several problems:
- it is non-portable (can't be used by a non-JVM language)
- it relies on manual analysis of the Minecraft server binary
- it is created manually by humans, which is naturally an error-prone process
- it may not include all changes due to the game's own code not accounting for all changes (i.e. Mojang themselves sometimes miss things or don't bother to account for things, such as the addition of `item_frame_photo_bit`)

This project aims to address the problem by providing schemas describing how to upgrade blockstates without binding the information to any particular language. These schemas are developed for future use in [PocketMine-MP](https://github.com/pmmp/PocketMine-MP).

It has the following advantages which make it desirable:
- the schemas are JSON - any language and any implementation can parse and use them
- the quality of the mappings is guaranteed to be at least as good as Bedrock's own, due to being generated[^1] from information created by BDS itself using [pmmp/mapping](https://github.com/pmmp/mapping) and [pmmp/BlockPaletteArchive](https://github.com/pmmp/BlockPaletteArchive)
- it includes changes which are not obvious from analysing the game code - e.g. properties such as `item_frame_photo_bit` were added without a version bump, and the game itself does not have an upgrader to add it
- tiny footprint - the schemas can be added to a binary with almost no noticeable increase in size after compression

[^1]: Amended by hand in some cases, due to errors in Bedrock's own blockstate upgrader
