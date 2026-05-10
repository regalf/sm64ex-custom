# sm64ex-custom PKGBUILD

A customized PKGBUILD for building Super Mario 64 PC Port with extensive customization options.

## About

This PKGBUILD allows you to build the [sm64ex](https://github.com/sm64pc/sm64ex) port with numerous enhancements, patches, custom models, and texture packs. It's based on Brian Allred's original work from [sm64ex-custom](https://gitlab.com/BrianAllred/sm64ex-custom).

## Changes Made

This version includes several fixes and improvements over the original:

### Fixed Issues
- **Updated all Discord CDN links** (`cdn.discordapp.com`) to permanent links on `sm64pc.info` for:
  - All patch files (60fps, tight controls, captain toad, etc.)
  - Model packs (HD Mario, Luigi, Bowser, Peach, etc.)
  - Texture packs (mollymutt, owo, beta hud, etc.)
- **Fixed syntax error** with `OPTIONS` variable (renamed to `_OPTIONS` to avoid conflict with makepkg's read-only variable)
- **Fixed missing parameters** in `_download()` function calls (added `$_useCache` and `$_EXT_CACHE_PATH`)
- **Fixed file extraction** commands (`.rar` → `.zip` where appropriate)
- **Fixed inconsistent filenames** in download checks and extraction commands

### Why Links Were Updated
The original PKGBUILD used Discord CDN links which are:
- **Not permanent** - files can become unavailable if messages are deleted
- **Unreliable** - Discord servers can be deleted or archived
- **Not suitable for packaging** - external dependencies should use stable hosting

All links now point to `sm64pc.info` which hosts these files permanently.

## Why not on AUR?

This package is **not available on AUR** because:
- It requires a **proprietary ROM file** (`baserom.{us,eu,jp}.z64`) which cannot be distributed
- Many assets (models, textures, patches) are downloaded from third-party sources during build
- The interactive build process doesn't fit AUR's non-interactive build philosophy
- Several patches and mods have uncertain licensing status

## Prerequisites

- Arch Linux or derivative
- Base development tools: `base-devel`, `git`
- ROM file: `baserom.us.z64`, `baserom.eu.z64`, or `baserom.jp.z64`

## Installation

### Quick start

```bash
# Clone this repository
git clone https://github.com/regalf/sm64ex-custom.git
cd sm64ex-custom

# Place your ROM in the build directory
cp /path/to/your/baserom.us.z64 ./

# Build and install
makepkg -si
```

### Using customization.cfg

Create a config file at `~/.config/sm64ex-custom/config` to pre-configure build options:

```bash
mkdir -p ~/.config/sm64ex-custom
cp customization.cfg ~/.config/sm64ex-custom/config
```

Edit the config file to set your preferences:

```bash
# Example customization.cfg
EXT_CONFIG_PATH=~/.config/sm64ex-custom/config
EXT_CACHE_PATH=~/.config/sm64ex-custom/cache/
_useCache=1

# Build options
_bettercamera=1
_debug=0
_texture_fix=1
_external_data=1

# Patches
_60fps=1
_no_exit_star=1
_tight_controls=1

# Mario model (default, hd_mario, luigi, hat_kid, bow_kid, mawio, odyssey_mario, old_school_hd_mario, beta_mario)
_mario_model=hd_mario

# Texture pack (default, mollymutt, hypatia, sm64_redrawn, resrgan_16x, resrgan_n64, p3st, cleaner, owo, minecraft, jappawakka_admentus_hd, beta_hud)
_texture_pack=mollymutt

# Region (us, eu, jp)
_region=us
```

## Build Options

### Available Patches
- 60 FPS
- Better camera
- Tight controls
- Captain Toad stars
- 3D coins
- Exit course after 50 coins
- Discord RPC
- Time trials
- Odyssey moveset
- And many more!

### Mario Models
- Default, HD Mario, Luigi, Hat Kid, Bow Kid, Mawio, Odyssey Mario, Old School HD, Beta Mario

### Texture Packs
- MollyMutt's, Hypatia's, SM64 Redrawn, RESRGAN upscales, p3st, Cleaner, OwOify, Minecraft, JappaWakka & Admentus HD, Beta HUD

## Original Repository

This work is based on [Brian Allred's sm64ex-custom](https://gitlab.com/BrianAllred/sm64ex-custom).

## License

This PKGBUILD is provided as-is. The sm64ex source code and game assets belong to their respective owners. This project does not distribute any copyrighted content.
