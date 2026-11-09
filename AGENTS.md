# AGENTS.md

BEI (Barely Enough Items) is a JEI-style item list and recipe viewer for Minecraft Bedrock,
built as a **client** mod on LeviLamina, with xmake and clang-cl on Windows x64.

One source tree supports several LeviLamina lines, one build each. A line is a lip variant of the
same name, and `--levilamina` picks the one being built:

| Line | Minecraft | Variant | Built against | Lock file |
| --- | --- | --- | --- | --- |
| 26.60.x | 1.26.60 | `26_60`, and the default | `--levilamina=26.60.5` (the default) | `locks/26_60.lock` |
| 26.51.x | 1.26.51 | `26_51` | `--levilamina=26.51.7` | `locks/26_51.lock` |

## Layout and ownership

| Path              | What it is                                                                 |
| ----------------- | -------------------------------------------------------------------------- |
| `src/core/`       | Pure C++ logic: model, recipe index, search, layout, state. **No `mc/` or `ll/` includes.** Built as the static lib `bei-core` and unit-tested on the host. |
| `src/game/`       | Adapters from game objects to the core model (`ItemCatalog`, recipe providers, `RecipeManager`). |
| `src/ui/`         | Rendering. `gfx/Painter` wraps the game UI renderer; `UiRoot` hosts `IUiLayer`s; `overlay/`, `recipe/`, `widgets/`. |
| `src/input/`      | `InputRouter`: game input to BEI (mouse, keys, hotkeys, vanilla slot hover). |
| `src/actions/`    | Recipe transfer and cheat-mode give.                                      |
| `src/app/`        | `Controller` (navigation, bookmarks, config, hover), `I18n`.               |
| `src/mod/`        | Entry point. `BEI.cpp` is the only place that calls module `install()`s.   |
| `assets/lang/<locale>/*.json` | Translation fragments, merged into `bin/BEI/lang/<locale>.json` at build time. Each feature owns its own fragment file; a duplicate key fails the build. |
| `tests/`          | GoogleTest suites for `src/core` (target `bei-tests`).                    |
| `tools/e2e/`      | PowerShell harness that drives the real game (see `tools/e2e/README.md`).  |

The public headers in these directories are **contracts** between features:
`core/model/*`, `core/recipe/RecipeIndex.h`, `core/search/SearchEngine.h`, `core/state/*`,
`game/items/ItemCatalog.h`, `game/recipes/IRecipeProvider.h`, `game/recipes/RecipeManager.h`,
`ui/UiLayer.h`, `ui/UiRoot.h`, `ui/gfx/Painter.h`, `ui/overlay/ItemListPanel.h`,
`ui/recipe/RecipeScreen.h`, `ui/widgets/*.h`, `input/InputRouter.h`, `actions/Actions.h`,
`app/Controller.h`.
- Extend them compatibly: add members, don't change or remove existing ones.
- Register new things through the registries: `BEI_REGISTER_RECIPE_PROVIDER`, `BEI_REGISTER_UI_LAYER`.
- Don't edit `src/mod/BEI.cpp` unless you add a new module that needs `install()`.

## Build and test

New tools are on the persistent PATH but not in already-open shells. In a fresh pwsh, refresh first:
`$env:Path=[Environment]::GetEnvironmentVariable('Path','Machine')+';'+[Environment]::GetEnvironmentVariable('Path','User')`

```powershell
Copy-Item locks/26_60.lock xmake-requires.lock                # the line you are building
xmake f -a x64 -m release -p windows --target_type=client --levilamina=26.60.5 -y  # config is sticky
xmake -y                                                      # builds bin/BEI/{BEI.dll,manifest.json,lang/}
xmake build -y bei-tests; xmake run bei-tests                 # unit tests (src/core only)
pwsh tools/e2e/run.ps1 -Scenario smoke                        # real game, see tools/e2e/README.md
```

`xmake-requires.lock` is generated, not tracked: copy the lock of the line you are building over it
before configuring. Switching lines means another `Copy-Item` and another `xmake f`.

`git config --global core.longpaths true` is required, because the LeviLamina package has very long header paths.

## Bedrock / LeviLamina rules (learned the hard way)

**Where the SDK headers are**
- `%LOCALAPPDATA%\.xmake\packages\l\levilamina\<version>\<hash>\include\{ll,mc}`, one tree per
  version you have built against.
- The same source is in the lip cache: `%LOCALAPPDATA%\lip\cache\git+https...LeviLamina.git#v<version>\{src,src-client}`.
- Check these before guessing any name or signature, and diff the trees of two lines before
  assuming an API is the same on both.

**Supporting more than one line**
- The build defines `BEI_LL_VERSION` as the line it is building for: `2651`, `2660`. Guard a
  difference with it, keep the guard as small as possible, and never drop the older path:
  ```cpp
  #if BEI_LL_VERSION >= 2660
  ...
  #else
  ...
  #endif
  ```
- Prefer one unguarded path when the same API works on both lines; that is usually possible, and a
  guard is the last resort.
- A warning `'X' is deprecated: This API is not available` means `X` became `MCNAPI` on that line.
  It still links, but it is not there to call: find another way instead of silencing the warning.
- Every line must build warning-free before a release.

**What you may call**
- `MCAPI`, `MCFOLD` and `MCTAPI` functions and unmarked `virtual` functions are callable.
- **Never call `MCNAPI`.** For example, all of `ItemStackNetManagerClient` is unavailable.
- `::ll::TypedStorage<...> mX` members are readable layout fields: use `mX.get()` or `*mX`, or implicit conversion.
- Hook a virtual function through its `$name` thunk, e.g. `LL_TYPE_INSTANCE_HOOK(H, ll::memory::HookPriority::Normal, Cls, &Cls::$method, Ret, Args...)`. Install with `H::hook()` and remove with `H::unhook()` in your module's install/uninstall.

**Threads**
- UI rendering (`ll::event::AfterUIRenderEvent`) and level ticks run on the client thread.
- `KeyInputEvent` and `MouseInputEvent` arrive on the **window thread**. Decide synchronously whether to `cancel()`, then do the real work via `ll::thread::ClientThreadExecutor::getDefault().execute(...)`.
- Never touch game state from input callbacks.

**Lifetimes**
- Never keep raw game pointers (screens, controllers, items, level) across frames.
- `ItemCatalog` owns cached `ItemStack`s and clears them on level exit.
- Restore vanilla state on screen leave, level exit and focus loss.

**Rendering order**
- In the UI pass the order matters: rectangles → `flushImages`, then item icons, then text → `flushText`. `ui::Painter` does this for you; call `painter.flush()` between z bands.
- Coordinates are GUI units. Mouse pixels × `GuiData::mInvGuiScale` = GUI units.

**Names and screens**
- Item ids are full names (`minecraft:oak_planks`). `ItemStackBase::getTypeName()` returns them.
- Screen names come from `ClientInstance::getScreenName()`: `inventory_screen`, `crafting_screen`, `hud_screen`, …
- Container screens report `ScreenController::_isContainerScreen()`.

**Policy**
- Lamium (github.com/amatouhake/Lamium) has no license. Read it for ideas only and never copy code from it.

## Conventions

- C++20, `.clang-format` (LLVM-based, 4 spaces, 120 columns). Namespaces follow directories: `bei::core`, `bei::game`, `bei::ui`, `bei::input`, `bei::actions`, `bei::app`.
- English in code and comments. User-facing strings go through `app::tr("bei.some.key", "English fallback")` and the lang fragments (`en_US` and `zh_CN` both required).
- Log with `bei::log()` (`mod/Log.h`). Keep per-frame logging out; log on state changes.
- Every `src/core` change comes with gtest coverage in `tests/`.
