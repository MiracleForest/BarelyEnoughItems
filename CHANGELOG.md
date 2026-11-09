# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0-beta.1] - 2026-11-09

First beta of 0.2.0. Please report anything that behaves differently from 0.1.3.

### Added

- Support for **Minecraft 1.26.60** with LeviLamina 26.60.x, which is now the version BEI is installed as by default. Minecraft 1.26.51 with LeviLamina 26.51.x is still supported and gets its own build of every release, so nothing is lost by updating.
  - BEI is compiled against one LeviLamina line at a time, so each release now ships one download per line: `bei-client-win-x64-ll26.60.zip` and `bei-client-win-x64-ll26.51.zip`.

### Changed

- The package is now described by `lip.toml` of format 4 instead of `tooth.json`, so installing it needs [lip](https://github.com/LiteLDev/lip) **0.40.0** or later, or a LeviLauncher that reads it. Older releases stay installable with older lip.
- Each supported LeviLamina line is a variant named after it: `26_60` for 26.60.x and `26_51` for 26.51.x. The default variant, and `client` — the variant earlier releases were installed as — are the newest line, so `lip update` and LeviLauncher keep working and simply move you to 26.60.
  - To stay on Minecraft 1.26.51, install `github.com/MiracleForest/BarelyEnoughItems#26_51`, or pick the `26_51` variant on the mod's page in LeviLauncher.
  - Your settings and bookmarks in `mods/BEI/config/` are kept either way.
- The downloads are now lip mod archives: the files sit at the root of the ZIP instead of inside a `BEI/` folder. Extract one into `mods/BEI/`, not into `mods/`.

### Fixed

- On Minecraft 1.26.60, fuels that the game describes with an item component (coal and charcoal among them) are read from that component, because the call BEI used for them on 1.26.51 is gone from 1.26.60.
- Container slots are now found by asking the game what each of the open screen's collections is, instead of through a lookup table that 1.26.60 no longer offers. Recipe transfer, the cursor stack and creative delete work the same on both versions.

## [0.1.3] - 2026-11-01

### Changed

- The repository moved to [MiracleForest/BarelyEnoughItems](https://github.com/MiracleForest/BarelyEnoughItems). Links point there now.
- The lip package is now `github.com/MiracleForest/BarelyEnoughItems`, and the mod page moves to [levimc.org/mods/github.com/MiracleForest/BarelyEnoughItems](https://levimc.org/mods/github.com/MiracleForest/BarelyEnoughItems/). Installing by the new path needs lip 0.35.0 or later. With lip 0.36.0 or later, `lip update` moves an existing install to the new path automatically. With older lip, run `lip uninstall github.com/LiteLDev/BarelyEnoughItems#client` and then `lip install github.com/MiracleForest/BarelyEnoughItems#client`. Either way your settings in `mods/BEI/config/` are kept. Older versions stay installable at their old path.

## [0.1.2] - 2026-10-24

### Added

- A logo, and screenshots and videos of search, recipes, one-click transfer and bookmarks in the [LeviMC Mods gallery](https://levimc.org/mods/github.com/LiteLDev/BarelyEnoughItems/?tab=gallery).

### Changed

- A shorter README written for players. Notes for developers moved to [CONTRIBUTING.md](CONTRIBUTING.md#development).

## [0.1.1] - 2026-10-21

### Fixed

- `lip install github.com/LiteLDev/BarelyEnoughItems#client` failed after the repository rename, because `tooth.json` still declared the old path `github.com/LiteLDev/bei`.

## [0.1.0] - 2026-10-21

First release.

### Added

- **Item list** next to every container screen, with paging, mouse-wheel scrolling and native item icons (including enchantment glint).
  - It covers every creative inventory item, plus registry items that are missing from it.
  - Items carry their localized names, creative groups and item tags.
- **Search** with JEI syntax: `@namespace`, `#tag`, `$id`, `-exclude`, `a|b` and `"phrase"`.
  - Matching ignores case and accents.
  - Chinese names also match by pinyin, full or initials.
  - `↑` / `↓` recall the last 20 searches.
- **Recipe screen**:
  - category tabs and paging;
  - workstation shortcuts;
  - cycling ingredient alternatives;
  - left click an ingredient for its recipes and right click for its uses;
  - back navigation.
- **Recipe categories**:
  - crafting, including special recipes shown as examples;
  - smelting, blasting, smoking, campfire and soul campfire cooking;
  - fuel;
  - stonecutting;
  - smithing;
  - cartography;
  - brewing;
  - anvil repair;
  - composting;
  - a generic fallback for any other recipe type.
- **Hotkeys**:
  - `R` recipes, `U` uses, `A` bookmark, `O` show/hide the item list, `Backspace` back, `Ctrl` + `F` focus search.
  - They work over BEI panels, over vanilla slots and inside the recipe screen.
  - They can be remapped in *Settings → Keyboard & Mouse*.
- **Recipe transfer** with the `+` button (shift for as many sets as possible) into:
  - the inventory and crafting-table grids;
  - furnaces, blast furnaces and smokers, including fuel;
  - the smithing table.

  Missing ingredients are highlighted.
- **Bookmarks** panel on the left.
- **Cheat mode** for players who may use commands:
  - take items from the list;
  - in creative mode, delete the held item by clicking the list.
- **In-game settings** popup (wrench button next to the search bar). `Ctrl`-click it to toggle cheat mode.
- **Translations**: English and Simplified Chinese.

[Unreleased]: https://github.com/MiracleForest/BarelyEnoughItems/compare/v0.2.0-beta.1...HEAD
[0.2.0-beta.1]: https://github.com/MiracleForest/BarelyEnoughItems/compare/v0.1.3...v0.2.0-beta.1
[0.1.3]: https://github.com/MiracleForest/BarelyEnoughItems/compare/v0.1.2...v0.1.3
[0.1.2]: https://github.com/MiracleForest/BarelyEnoughItems/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/MiracleForest/BarelyEnoughItems/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/MiracleForest/BarelyEnoughItems/releases/tag/v0.1.0
