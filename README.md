# BlockStateUpgradeSchema
(Mostly) auto-generated schemas for upgrading blockstates found in older Minecraft: Bedrock worlds

## Contents
### `nbt_upgrade_schema/*.json`
These schemas describe how to upgrade blockstate NBT from one version to the next. The structure of the schema is described in `nbt_upgrade_schema_schema.json`. An example implementation can be seen [in PocketMine-MP 5.0.0-ALPHA2](https://github.com/pmmp/PocketMine-MP/blob/ccb3c3cb05e6eee8afa15d7837e256a446244fe7/src/data/bedrock/block/upgrade/BlockStateUpgrader.php#L33).

#### Gotchas
- Mojang don't always bump the format version when making backwards-incompatible changes. A prominent example of this is in the [`0131_1.18.20.27_beta_to_1.18.30.json`](/nbt_upgrade_schema/0131_1.18.20.27_beta_to_1.18.30.json).
- `remappedPropertyValues` always uses the old property name, if the names were changed.

### `block_legacy_id_map.json`
This JSON file contains a mapping of string ID -> legacy ID for all blocks known up 1.16.0.

Technically, you'd only need everything up to 1.2.13, but the excess might be handy in some cases.

### `1.12.0_to_1.18.10_blockstate_map.bin`
This binary file contains a mapping of all known valid 1.12.0 block ID/meta combinations to their corresponding blockstate NBTs as of 1.18.10.

#### Schema
The file is structured as described below.

- unsigned varint32 - Number of entries
  - unsigned varint32 - 1.12 block string ID length
  - byte[] - 1.12 block string ID
  - unsigned varint32 - Number of meta -> blockstate pairings
    - unsigned varint32 - Meta value
    - TAG_Compound (standard little-endian) - 1.18.10 NBT blockstate corresponding to the current ID and meta pair from 1.12.

## Generating NBT upgrade schemas for new versions

First, you need to get a `.bin` mapping file, which you can obtain using the current version of BDS + [pmmp/mapping mod](https://github.com/pmmp/mapping). It requires that you place the palette for the previous version in `input_files/old_block_palettes`.

The output file will be placed in `mapping_files/old_palette_mappings`. This file is then provided as the input for the [schema generator script](https://github.com/pmmp/PocketMine-MP/blob/e98cf39b47c6c37619cae32d2d2596b08f4d938f/tools/generate-blockstate-upgrade-schema.php), which produces the JSON schemas like the ones you see in this repo.

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
