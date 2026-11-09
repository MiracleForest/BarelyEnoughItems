# Contributing to BarelyEnoughItems

Thanks for helping! Bug reports, ideas and pull requests are all welcome.

## Reporting bugs and requesting features

Use the [issue templates](https://github.com/MiracleForest/BarelyEnoughItems/issues/new/choose). For bugs, include:

- the BEI version;
- the LeviLamina version;
- the Minecraft version;
- steps to reproduce;
- the relevant part of `logs/latest.log` from your game folder.

## Development setup

See [Development](#development) below. [AGENTS.md](AGENTS.md) is the developer guide. It covers the layout of the code, the contracts between modules, threading and lifetime rules, and how to read the LeviLamina SDK headers. Please read it before changing anything under `src/`.

## Pull requests

1. Fork the repository and create a branch from `main`.
2. Keep each pull request focused on one change. Describe what it does and how you tested it.
3. Follow the existing style: C++20, the provided `.clang-format` and `.editorconfig`, English in code and comments.
4. User-facing text goes through `app::tr` with entries in **both** `assets/lang/en_US/` and `assets/lang/zh_CN/`.
5. Logic in `src/core` must stay free of game headers and come with unit tests in `tests/`:
   `xmake build -y bei-tests; xmake run bei-tests`.
6. If you change something visible in game, run the relevant end-to-end scenario from [tools/e2e](tools/e2e/README.md) and check its screenshots. Better still, add a scenario.
7. CI must pass before a pull request is merged.

Commit messages use the imperative mood ("Add anvil category", not "Added …"). Update [CHANGELOG.md](CHANGELOG.md) under *Unreleased* for user-visible changes.

## Development

### Building

You need Windows x64, [xmake](https://xmake.io), Visual Studio Build Tools with MSVC 14.4x or newer, and LLVM (`clang-cl`, on `PATH`).

BEI is compiled against one LeviLamina line at a time, chosen with `--levilamina`. Each supported line has a lip variant of the same name and a lock file in `locks/`, which pins the packages of that line; copy it to `xmake-requires.lock` (which is generated, and not tracked) before configuring:

| Line | Variant | Lock file | Configure with |
| --- | --- | --- | --- |
| LeviLamina 26.60.x (Minecraft 1.26.60) | `26_60` | `locks/26_60.lock` | `--levilamina=26.60.5` (the default) |
| LeviLamina 26.51.x (Minecraft 1.26.51) | `26_51` | `locks/26_51.lock` | `--levilamina=26.51.7` |

```powershell
git config --global core.longpaths true
Copy-Item locks/26_60.lock xmake-requires.lock
xmake f -a x64 -m release -p windows --target_type=client --levilamina=26.60.5 -y  # add --vs=2026 for Visual Studio 18
xmake -y                                                                            # produces bin/BEI/
```

Copy `bin/BEI/` into the game's `mods/` folder (LeviLauncher) to try it.

The build defines `BEI_LL_VERSION` as the line it is building for (`2651`, `2660`). Where a LeviLamina API differs between lines, guard both paths with it rather than dropping the older one, and keep the guard as small as possible:

```cpp
#if BEI_LL_VERSION >= 2660
    // 26.60 and later
#else
    // 26.51
#endif
```

To add a line, add its row above: a `locks/<variant>.lock` (configure once with the new `--levilamina` and copy the generated `xmake-requires.lock`), a variant in `lip.toml`, and a matrix entry in `.github/workflows/build.yml` and `.github/workflows/release.yml`.

### Releasing

The version is written in three places and the release workflow refuses a tag that disagrees with
any of them:

1. `lip.toml` — `version`;
2. `manifest.json` — `version`, what the mod reports in game;
3. `CHANGELOG.md` — a `## [<version>] - <date>` section, which becomes the release notes.

Then tag the commit `v<version>` and push the tag. The workflow builds every line, runs the unit
tests, packs one ZIP per line and creates the release; a version with a `-` in it, such as
`0.2.0-beta.1`, is published as a prerelease.

### Tests

- Unit tests for the pure logic in `src/core`: `xmake build -y bei-tests; xmake run bei-tests`.
- End-to-end tests drive the real game through LeviLauncher and take screenshots: see [tools/e2e](tools/e2e/README.md).
  `pwsh tools/e2e/run.ps1 -Scenario smoke` is the quickest check.

### Showcase media

The pictures and videos in the README and on the [LeviMC Mods gallery](https://levimc.org/mods/github.com/MiracleForest/BarelyEnoughItems/?tab=gallery)
live in [`docs/showcase/`](docs/showcase/) with the manifest `showcase.json`. They are reproducible:

```powershell
pwsh tools/e2e/run.ps1 -Scenario showcase      # stages the scenes, writes tools/e2e/out/showcase/
pwsh tools/e2e/showcase-encode.ps1             # crops/encodes them into docs/showcase/
```

### How it works

BEI runs entirely on the client. It reads the recipes the server already sends to every client, so it
works in single-player and on any server without server-side installation. Icons, panels and text are
drawn with the game's own item renderer and UI textures. The code layout, module contracts and the
rules for working with Bedrock internals are in [AGENTS.md](AGENTS.md).

## License

By contributing you agree that your contributions are licensed under the [LGPL-3.0-or-later](COPYING.LESSER), the license of this project.

