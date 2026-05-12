# sm64ex-custom PKGBUILD (modular-experimental)

A modular, non-interactive PKGBUILD for building Super Mario 64 PC Port with extensive
customization options, including support for multiple game forks.

## About

This PKGBUILD allows you to build [sm64ex](https://github.com/sm64pc/sm64ex) and its forks
with numerous enhancements, patches, custom models, and texture packs. It is:
- **Fully non-interactive** — all options via `customization.cfg`
- **Modular** — resources driven by `resources.db`, not hardcoded
- **Multi-fork** — build from any sm64ex fork

Based on [Brian Allred's sm64ex-custom](https://gitlab.com/BrianAllred/sm64ex-custom).

## Features (modular-experimental)

- **Modular PKGBUILD** — loop-based patch/model/texture handling via `customization.cfg`
- **Dynamic game repo** — build from any fork via `_repo_url`/`_repo_branch` in config
- **Smart branch switching** — `prepare()` detects repo/branch changes and re-clones automatically
- **Flexible binary detection** — works with any fork's binary name (`sm64.us.*`, `sm64coopdx`, etc.)
- **Compatible with sm64ex-builder GUI** — full support for the Python builder

## Prerequisites

- Arch Linux or derivative
- Base development tools: `base-devel`, `git`
- ROM file: `baserom.us.z64`, `baserom.eu.z64`, or `baserom.jp.z64`

## Installation

### Quick start

```bash
# Clone this repository
git clone -b modular-experimental https://github.com/regalf/sm64ex-custom.git
cd sm64ex-custom

# Place your ROM in the build directory
cp /path/to/your/baserom.us.z64 ./

# Build and install
makepkg -si
```

### Using customization.cfg

```bash
mkidr -p ~/.config/sm64ex-custom
cp customization.cfg ~/.config/sm64ex-custom/config
```

Edit `customization.cfg` to configure patches, models, texture packs, and build options.
No interactive menus will appear during the build.

### Building from a different fork

Set `_repo_url` and `_repo_branch` in `customization.cfg`:

```bash
_repo_url="https://github.com/yourusername/sm64ex-fork.git"
_repo_branch="your-branch"
```

The PKGBUILD will automatically switch to the configured fork during the build.
Patches and resources from `resources.db` are designed for `sm64pc/sm64ex` and may
not work correctly on other forks.

## customization.cfg Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `_repo_url` | `https://github.com/sm64pc/sm64ex.git` | Game source repository |
| `_repo_branch` | `nightly` | Game source branch |
| `_region` | `us` | Game region (us/eu/jp) |
| `_patches` | `""` | Space-separated patch filenames from sm64pc.info |
| `_hd_models` | `""` | Space-separated model filenames |
| `_mario_model_file` | `default` | Mario model filename |
| `_texture_pack` | `default` | Texture pack name |
| `_target_bits` | `default` | Bitness (default/64/32) |
| `_render_api` | `GL` | Render API (GL/GL_LEGACY/D3D11/D3D12) |
| `_bettercamera` | `""` | Enable better camera |
| `_external_data` | `""` | Load external data (required for texture packs) |
| `_windows_build` | `""` | Cross-compile for Windows |

Full list of build options and available resources is in `customization.cfg` and `resources.db`.

## Why not on AUR?

This package is **not available on AUR** because:
- It requires a **proprietary ROM file** which cannot be distributed
- Assets (models, textures, patches) are downloaded from third-party sources
- Several patches and mods have uncertain licensing status

## License

This PKGBUILD is provided as-is. The sm64ex source code and game assets belong to
their respective owners. This project does not distribute any copyrighted content.
