# BlockStateUpgradeSchema
(Mostly) auto-generated schemas for upgrading blockstates found in older Minecraft: Bedrock worlds

## Background

Since Minecraft Bedrock doesn't auto-upgrade terrain or inventories unless they've been loaded during a game session, any software that supports Minecraft Bedrock worlds has to accommodate all of the old blockstates all the way back to the first versions that used them, and many backwards-incompatible changes were made since then.
If they do not, they may randomly fail to load chunks in older worlds that work just fine in latest Bedrock.

Other projects such as [CloudburstMC/BlockStateUpdater](https://github.com/CloudburstMC/BlockStateUpdater) attempted to address this by writing library code to deal with the problem; however, this approach comes with several problems:
- it is non-portable (can't be used by a non-JVM language)
- it relies on manual analysis of the Minecraft server binary
- it is created manually by humans, which is naturally error-prone
- it may not include all changes due to the game's own code not accounting for all changes (i.e. Mojang themselves sometimes miss things or don't bother to account for things, such as the addition of `item_frame_photo_bit`)

This project aims to address the problem by providing schemas describing how to upgrade blockstates without binding the information to any particular language. These schemas are developed for future use in [PocketMine-MP](https://github.com/pmmp/PocketMine-MP).

It has the following advantages which make it desirable:
- the schemas are JSON - any language and any implementation can parse and use them
- the quality of the mappings is guaranteed to be at least as good as Bedrock's own, due to being generated[^1] from information created by BDS itself using [pmmp/mapping](https://github.com/pmmp/mapping) and [pmmp/BlockPaletteArchive](https://github.com/pmmp/BlockPaletteArchive)
- it includes changes which are not obvious from analysing the game code - e.g. properties such as `item_frame_photo_bit` were added without a version bump, and the game itself does not have an upgrader to add it
- tiny footprint - the schemas can be added to a binary with almost no noticeable increase in size after compression

[^1]: Amended by hand in some cases, due to errors in Bedrock's own blockstate upgrader
