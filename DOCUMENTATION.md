# sm64ex-custom — Documentation

## Overview

This repository contains custom PKGBUILDs for building
[sm64ex](https://github.com/sm64pc/sm64ex) (Super Mario 64 PC Port) with
patches, HD models, texture packs, and build options. Based on the work of
[Brian Allred](https://gitlab.com/BrianAllred/sm64ex-custom).

Repository: `https://github.com/regalf/sm64ex-custom` (private)

---

## Branches

### `master` — Original interactive PKGBUILD

The main branch. Contains the classic PKGBUILD with interactive menus
that guide the user step by step through build configuration.

- Shows `printf`/`read` menus to choose:
  - Region (US/EU/JP)
  - Build options (10 toggles)
  - Patches (12 toggles)
  - HD models (3 toggles)
  - Mario model (dropdown)
  - Texture pack (dropdown)
  - Bitness
  - Render API
- If pre-configured via `customization.cfg` or `~/.config/sm64ex-custom/config`,
  skips the corresponding menus.

**When to use:** Interactive manual builds where you want to pick
options on the fly.

```
git checkout master
makepkg -si
```

### `pkgbuild-no-interactive` — Non-interactive PKGBUILD

Streamlined version (~680 lines vs ~1226) with **zero interactive menus**.
All options are read exclusively from `customization.cfg`.

- Removed: all `while`/`printf`/`read` loops
- Kept: all download, patch, build, and package logic
- If a variable is unset, defaults are used:
  - `_region=us`
  - `_mario_model=default`
  - `_texture_pack=default`
  - `_render_api=GL`
  - `_useCache=0`

**When to use:** Automated builds, CI scripts, or when you want to
configure everything upfront without prompts.

```
git checkout pkgbuild-no-interactive
# Edit customization.cfg...
makepkg -si
```

### `modular-experimental` — Modular PKGBUILD with resources.db

Complete refactoring: instead of hardcoded variables and if/else blocks
for each resource, it uses **lists** in `customization.cfg` driven by an
external database (`resources.db`).

**What changed:**

```
# Old (master / pkgbuild-no-interactive):
_60fps=1
_tight_controls=1
_hd_bowser=1

# New (modular-experimental):
_patches="60fps_ex.patch tight_controls.patch"
_hd_models="hd_bowser.zip hd_koopa_the_quick.zip"
```

**Benefits:**
- Adding a patch/model/texture requires editing only `resources.db`,
  not the PKGBUILD itself
- `resources.db` contains name, URL, and type for each resource
- The Python builder can parse `resources.db` and generate the UI
  dynamically

**Structure:**

```
pkgbuild/
├── PKGBUILD                    ← Modular (519 lines)
├── customization.cfg            ← List-based format
├── resources.db                 ← Patch/model/texture database
├── README.md
├── sm64ex.desktop
└── SuperMario64.png
```

**When to use:** This branch is the **backend** of the Sm64exBuilder
GUI — the builder handles `resources.db` updates and config generation
automatically. **Not recommended for manual use without the builder.**

> **Note:** This branch is designed to be driven exclusively by the
> builder. Manual `makepkg -si` is not advised.

```
git checkout modular-experimental
# Use the builder GUI instead
```

---

## How the PKGBUILD works

### General structure

All branches share the same basic architecture:

| Section | Purpose |
|---------|---------|
| `_download()` | Downloads resources with/without cache |
| `_clone()` | Clones git repos for texture packs |
| `pkgver()` | Computes dynamic version from git |
| `prepare()` | Configures options, applies patches, installs models/textures |
| `build()` | Runs `make` with selected options |
| `package()` | Installs binary, launcher script, .desktop file, icon |

### Key variables

```bash
pkgname=sm64ex-custom-git
depends=('sdl2')
makedepends=('zip' 'unzip' 'git' 'unrar' 'wget' 'glew' 'python' 'audiofile')
source=('git+https://github.com/sm64pc/sm64ex.git#branch=nightly')
```

### Execution flow

1. `makepkg` sources the PKGBUILD and downloads the git source
2. `pkgver()` computes `r{commit-count}.{short-hash}`
3. `prepare()` loads `customization.cfg` + external config, applies patches,
   downloads models and textures
4. `build()` runs `make _OPTIONS -j$(nproc)`
5. `package()` copies files to `/opt/sm64ex` and creates symlinks/shortcuts

---

## Configuration

### customization.cfg

All options are set in this file (in the PKGBUILD directory).

**Basic options:**

```bash
_region=us                    # us, eu, jp
_rom_path=/path/to/baserom.us.z64
_useCache=1                   # Enable download cache
_cleanCache=                  # Set to 1 to clear cache
```

**Build options:**

```bash
_bettercamera=1
_debug=0
_nodrawingdistance=0
_texture_fix=1
_external_data=1              # Required for texture packs
_discordrpc=1
_windows_build=0
_windows_console=0
_textsaves=0
_target_web=0
```

**Available patches:**

```bash
_60fps=1
_no_exit_star=0
_tight_controls=1
_captain_toad=0
_title_return=0
_3d_coin=0
_exit_50_coin=0
_mouse_fix=0
_title_exit=0
_star_delay=0
_time_trial=0
_odyssey_moveset=0
```

**Models and textures:**

```bash
_hd_bowser=1
_hd_koopa=1
_hd_peach=0

_mario_model=hd_mario         # default, hd_mario, luigi, hat_kid, bow_kid,
                              # mawio, odyssey_mario, old_school_hd_mario, beta_mario
_texture_pack=mollymutt       # default, mollymutt, hypatia, sm64_redrawn,
                              # resrgan_16x, resrgan_n64, p3st, cleaner,
                              # owo, minecraft, jappawakka_admentus_hd, beta_hud
```

**Bitness and Render:**

```bash
_target_bits=                 # default (system), 64, 32
_render_api=GL                # GL, GL_LEGACY, D3D11, D3D12
```

### External config

If `~/.config/sm64ex-custom/config` exists, it is loaded after
`customization.cfg` and can override its values.

```bash
_EXT_CONFIG_PATH=~/.config/sm64ex-custom/config
_EXT_CACHE_PATH=~/.config/sm64ex-custom/cache/
```

---

## Non-interactive mode

On the `pkgbuild-no-interactive` and `modular-experimental` branches,
set these variables in `customization.cfg` to skip all prompts:

```bash
_skipOptions=1
_skipPatches=1
_skipModels=1
```

On the `master` branch these variables are only respected if the
PKGBUILD has been configured to use them (see `_configure_options()`).

---

## Builder GUI (wxPython)

The graphical builder is now in a **separate repository**: [`github.com/regalf/sm64ex-builder`](https://github.com/regalf/sm64ex-builder)

### Features (v2.0.0)

- 5-tab notebook GUI (General, Build Options, Patches, Models & Textures, Config)
- Parses `resources.db` dynamically — no hardcoded resource lists
- Search bars for patches and extra models
- Generates `customization.cfg` and launches `makepkg -si` in a background thread
- **Cross-platform**: runs on Linux natively and on **Windows via MSYS2**
- **Output Directory** — automatically copies built game files (binary + `res/`) to a folder of your choice
- **Clean up after build** — removes `src/`, `pkg/`, and package files
- **Install Dependencies** button — installs all required packages (pacman on Linux / pacman inside MSYS2 on Windows)
- Live output console with real-time build logging
- `--reset` flag to clear all settings (`~/.config/sm64ex-builder/`)
- Backend: uses the `modular-experimental` branch of this repo

### Downloads

Pre-built binaries for each release:
- **Linux**: standalone ELF (`sm64ex-builder`)
- **Windows**: standalone `.exe` (`sm64ex-builder.exe`)

### Usage

```bash
# From source
python3 sm64ex-builder.py

# Standalone binary
./sm64ex-builder

# Reset configuration
./sm64ex-builder --reset
```

### Requirements per platform

**Linux (Arch):**
```
python-wxpython  git  base-devel
```

**Windows (MSYS2/Mingw64):**
```
mingw-w64-x86_64-python  mingw-w64-x86_64-wxPython  mingw-w64-x86_64-SDL2
mingw-w64-x86_64-glew    mingw-w64-x86_64-toolchain  base-devel  git
```

### Build from source (standalone binary)

```bash
# Linux
pyinstaller --onefile --name sm64ex-builder sm64ex-builder.py

# Windows (in MSYS2)
pyinstaller --onefile --windowed --name "sm64ex-builder" sm64ex-builder.py
```

---

## Known / resolved issues

### Discord links updated

All `cdn.discordapp.com` links have been replaced with permanent links
on `sm64pc.info` (patches, models, textures).

### Various fixes

- `OPTIONS` renamed to `_OPTIONS` (conflict with makepkg's read-only
  variable)
- Missing parameters added to `_download()` calls (`$_useCache`,
  `$_EXT_CACHE_PATH`)
- `.rar` files converted to `.zip`
- `${_EXT_CACHE_PATH:?}` crashed when unset — added defaults and `-n`
  check

### Why not on AUR

- Requires a proprietary ROM (`baserom.*.z64`) — cannot be distributed
- Third-party assets (patches, models, textures)
- Interactive build process (not AUR-compatible)
- Uncertain licensing on third-party patches/models

---

## Prerequisites

- Arch Linux or derivative
- `base-devel` (includes makepkg)
- Original ROM: `baserom.us.z64`, `baserom.eu.z64` or `baserom.jp.z64`

## Quick start

```bash
git clone https://github.com/regalf/sm64ex-custom.git
cd sm64ex-custom

# Place your ROM
cp /path/to/baserom.us.z64 ./

# Interactive build (master)
makepkg -si

# Non-interactive build
git checkout pkgbuild-no-interactive
# Edit customization.cfg...
makepkg -si

# GUI builder (see Builder GUI section below)
# Clone https://github.com/regalf/sm64ex-builder and run:
python3 sm64ex-builder.py
```
