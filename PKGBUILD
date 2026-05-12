# Maintainer: Brian Allred brian.d.allred<AT>gmail.com
# Modular non-interactive version
# Resources are driven by customization.cfg lists, not hardcoded individually.

pkgname=sm64ex-custom-git
pkgrel=1
pkgver=r646.d7ca2c04
pkgdesc='Super Mario 64 PC Port (sm64ex)'
arch=('any')
url='https://github.com/sm64pc/sm64ex'
license=('none')
depends=('sdl2')
makedepends=('zip' 'unzip' 'git' 'unrar' 'wget' 'glew' 'python' 'audiofile')
optdepends=('mingw-w64-crt: Cross-compile for Windows'
            'mingw-w64-gcc: Cross-compile for Windows'
            'mingw-w64-glew: Cross-compile for Windows'
            'mingw-w64-sdl2: Cross-compile for Windows'
            'mingw-w64-headers: Cross-compile for Windows'
            'emscripten: Build for Web')
provides=(sm64ex)

_gitname=sm64ex

source=('git+https://github.com/sm64pc/sm64ex.git#branch=master')
sha256sums=('SKIP')

_where="$PWD"

# ---- Base URLs for sm64pc.info resources ----
_PATCH_URL="https://sm64pc.info/downloads/patches"
_MODEL_URL="https://sm64pc.info/downloads/model_pack"
_TEXTURE_URL="https://sm64pc.info/downloads/texture_pack"

# ---- Git-based texture packs: name -> "url branch checkoutPath zipName subdir" ----
_GIT_TEXTURE_MAP() {
    case $1 in
        sm64_redrawn)
            echo "https://github.com/TechieAndroid/sm64redrawn.git '' sm64redrawn sm64redrawn.zip gfx"
            ;;
        resrgan_16x)
            echo "https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git '' resrgan_16x resrgan.zip gfx"
            ;;
        resrgan_n64)
            echo "https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git n64-resrgan-faithful resrgan_n64 resrgan_n64.zip gfx"
            ;;
        p3st)
            echo "https://github.com/p3st-textures/p3st-Texture_pack.git '' p3st p3st-textures.zip gfx p3st-sound.zip sound"
            ;;
        minecraft)
            echo "git://github.com/Almondatchy3/MCtexturepackSM64 '' minecraft minecraft.zip gfx"
            ;;
        jappawakka_admentus_hd)
            echo "git://github.com/JappaWakka/Mario64HDTexturePack_PC '' jappawakka_admentus_hd jappawakka_admentus_hd.zip gfx"
            ;;
        cleaner)
            echo "https://github.com/CrashCrod/Cleaner-Aesthetics.git '' cleaner cleaner.zip gfx"
            ;;
        *)
            return 1
            ;;
    esac
}

_download() {
    _fileName=$1
    _url=$2
    _destinationPath=$3
    _useCache=$4
    _cachePath=$5

    if [ "$_useCache" = "1" ]; then
        if [ ! -e "$_cachePath/$_fileName" ]; then
            wget -O "$_cachePath/$_fileName" "$_url"
        fi
        cp "$_cachePath/$_fileName" "$_destinationPath/"
    else
        wget -O "$_destinationPath/$_fileName" "$_url"
    fi
}

_clone() {
    _useCache=$1
    _cachePath=$2
    _url=$3
    _branch=$4
    _checkoutPath=$5

    shift; shift; shift; shift; shift

    _wd=$(pwd)

    if [ "$_useCache" = "1" ]; then
        cd "$_cachePath" || exit
    fi

    if [ ! -d "$_checkoutPath" ]; then
        git clone "$_url" "$_checkoutPath"
    fi

    (
        cd "$_checkoutPath" || exit

        if [ -n "$_branch" ]; then
            git checkout "$_branch"
        fi

        git clean -xfd && git reset --hard && git pull
        while (( "$#" )); do
            zip -r "$1" "$2"

            install -Dm755 "$1" "$srcdir/$_gitname"/build/"$_region"_pc/res/"$1"

            shift; shift;
        done
    )

    cd "$_wd" || exit
}

# ---- Extract archive based on extension ----
_extract() {
    _file=$1
    _dest=${2:-.}
    case $_file in
        *.zip) unzip -o "$_file" -d "$_dest" ;;
        *.7z)  7z x -y "$_file" -o"$_dest" ;;
        *.rar) unrar x -o+ "$_file" "$_dest" ;;
        *.tar.gz|*.tar.xz) tar xf "$_file" -C "$_dest" ;;
    esac
}

# ---- Post-processing for specific models ----
# Each case runs additional steps needed by certain model packs.
_model_post_process() {
    _model_file=$1
    case $_model_file in
        hd_koopa_the_quick.zip)
            if ! grep -q '#include "koopa_shell/geo_header.h"' "./actors/common0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "koopa_shell/geo_header.h"' ./actors/common0.h
            fi
            if ! grep -q '#include "koopa/geo_header.h"' "./actors/group14.h" 2>/dev/null; then
                sed -i '/#endif/i #include "koopa/geo_header.h"' ./actors/group14.h
            fi
            ;;
        hd_peach_v2.zip)
            install -Dm755 peach_hd_textures.zip build/"$_region"_pc/res/peach_hd_textures.zip 2>/dev/null || true
            if ! grep -q '#include "peach/geo_header.h"' "./actors/group10.h" 2>/dev/null; then
                sed -i '/#endif/i #include "peach/geo_header.h"' ./actors/group10.h
            fi
            _external_data=1
            ;;
        hat_kid.rar|hat_kid_v2.1.zip)
            install -Dm755 hat_kid_textures_sounds.zip build/"$_region"_pc/res/hat_kid_textures_sounds.zip 2>/dev/null || true
            _add_cap_includes
            ;;
        bow_kid.rar|bow_kid_v2.1.zip)
            install -Dm755 bow_kid_textures_sounds.zip build/"$_region"_pc/res/bow_kid_textures_sounds.zip 2>/dev/null || true
            _add_cap_includes
            ;;
        odyssey_mario.zip)
            if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
            fi
            if ! grep -q '#include "mario_cap/geo_header.h"' "./actors/common1.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario_cap/geo_header.h"' ./actors/common1.h
            fi
            ;;
        luigi.zip)
            if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
            fi
            ;;
        mawio.zip)
            if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
            fi
            ;;
        old_school_hd_mario.zip)
            if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
            fi
            ;;
        beta_mario.zip)
            if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
            fi
            ;;
    esac
}

_add_cap_includes() {
    for _f in \
        'mario/geo_header.h:actors/group0.h' \
        'goomba/geo_header.h:actors/common0.h' \
        'mario_cap/geo_header.h:actors/common1.h' \
        'mario_metal_cap/model.inc.c:actors/common1.c' \
        'mario_metal_cap/geo_header.h:actors/common1.h' \
        'mario_metal_cap/geo.inc.c:actors/common1_geo.c' \
        'mario_wing_cap/model.inc.c:actors/common1.c' \
        'mario_wing_cap/geo_header.h:actors/common1.h' \
        'mario_wing_cap/geo.inc.c:actors/common1_geo.c' \
        'mario_winged_metal_cap/model.inc.c:actors/common1.c' \
        'mario_winged_metal_cap/geo_header.h:actors/common1.h' \
        'mario_winged_metal_cap/geo.inc.c:actors/common1_geo.c' \
        'star/geo_header.h:actors/common1.h' \
        'transparent_star/geo_header.h:actors/common1.h'; do
        _include="${_f%%:*}"
        _file="${_f##*:}"
        if ! grep -q "#include \"$_include\"" "$_file" 2>/dev/null; then
            sed -i '/#endif/i #include "'"$_include"'"' "$_file"
        fi
    done
    for _pattern in \
        'mario_metal_cap:Makefile.split:mario_cap*:mario_cap mario_metal_cap' \
        'mario_wing_cap:Makefile.split:mario_metal_cap*:mario_metal_cap mario_wing_cap' \
        'mario_winged_metal_cap:Makefile.split:mario_wing_cap*:mario_wing_cap mario_winged_metal_cap'; do
        _cap="${_pattern%%:*}"
        _rest="${_pattern#*:}"
        _mkfile="${_rest%%:*}"
        _rest2="${_rest#*:}"
        _old="${_rest2%%:*}"
        _new="${_rest2##*:}"
        if ! grep -q "$_cap" "$_mkfile" 2>/dev/null; then
            sed -i "s/$_old/$_new/g" "$_mkfile"
        fi
    done
}

_configure_options() {
    # ---- Load config ----
    [ -f "$_where/customization.cfg" ] && source "$_where/customization.cfg"
    [ -f "$_EXT_CONFIG_PATH" ] && source "$_EXT_CONFIG_PATH"

    # ---- Defaults ----
    _EXT_CACHE_PATH=${_EXT_CACHE_PATH:-~/.cache/sm64ex-custom}
    _EXT_CONFIG_PATH=${_EXT_CONFIG_PATH:-~/.config/sm64ex-custom/config}
    _useCache=${_useCache:-0}
    _region=${_region:-us}
    if [ "$_useCache" = "1" ]; then
        mkdir -p "$_EXT_CACHE_PATH"
    fi
    if [ "$_cleanCache" = "1" ] && [ -n "$_EXT_CACHE_PATH" ]; then
        rm -rf "${_EXT_CACHE_PATH}"/*
    fi

    # ---- ROM ----
    if [ ! -e "$_where"/baserom."$_region".z64 ] && [ -n "$_rom_path" ]; then
        cp "$_rom_path" "$_where"/ && mv "$_where"/"$(basename "$_rom_path")" "$_where"/baserom."$_region".z64
    fi

    pushd "$srcdir/$_gitname" >/dev/null || exit
    make clean
    cp "$_where"/baserom."$_region".z64 ./
    ./extract_assets.py $_region
    popd >/dev/null

    # ---- Build options (unchanged, these are make flags) ----
    if [ "$_discordrpc" = "1" ]; then
        _target_bits=64
    fi
    if [ "$_windows_console" = "1" ]; then
        _windows_build=1
    fi

    # ---- Generic patch loop ----
    pushd "$srcdir/$_gitname" >/dev/null || exit

    # 60fps: already in repo as enhancements/60fps_ex.patch
    if echo "$_patches" | grep -qw "60fps_ex.patch"; then
        git checkout -- enhancements/60fps_ex.patch 2>/dev/null || true
        git apply ./enhancements/60fps_ex.patch --ignore-whitespace --reject || true
    fi

    for _patch in $_patches; do
        [ "$_patch" = "60fps_ex.patch" ] && continue
        _download "$_patch" "$_PATCH_URL/$_patch" \
            "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
        git apply "./enhancements/$_patch" --ignore-whitespace --reject || true
    done
    popd >/dev/null

    # ---- Generic HD model loop ----
    pushd "$srcdir/$_gitname" >/dev/null || exit
    for _model in $_hd_models; do
        _download "$_model" "$_MODEL_URL/$_model" \
            "." "$_useCache" "$_EXT_CACHE_PATH"
        _extract "$_model"
        _model_post_process "$_model"
    done
    popd >/dev/null

    # ---- Mario model (filename-based) ----
    _mario_model_file=${_mario_model_file:-default}
    if [ "$_mario_model_file" != "default" ] && [ -n "$_mario_model_file" ]; then
        pushd "$srcdir/$_gitname" >/dev/null || exit

        _download "$_mario_model_file" "$_MODEL_URL/$_mario_model_file" \
            "." "$_useCache" "$_EXT_CACHE_PATH"
        _extract "$_mario_model_file"
        _model_post_process "$_mario_model_file"

        # Most Mario models need mario geo_header include
        case $_mario_model_file in
            hd_bowser.zip|hd_koopa_the_quick.zip|hd_peach_v2.zip) ;;
            *)
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h" 2>/dev/null; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
                ;;
        esac
        popd >/dev/null
    fi

    # ---- Texture pack ----
    _texture_pack=${_texture_pack:-default}
    _git_texture_packs=${_git_texture_packs:-}

    if [ "$_texture_pack" != "default" ] && [ -n "$_texture_pack" ]; then
        pushd "$srcdir/$_gitname" >/dev/null || exit

        # Check if it's a git-based texture pack
        _git_info=$(_GIT_TEXTURE_MAP "$_texture_pack" 2>/dev/null) && {
            eval set -- $_git_info
            _clone "$_useCache" "$_EXT_CACHE_PATH" "$1" "$2" "$3" "$4" "$5"
        } || {
            _download "$_texture_pack" "$_TEXTURE_URL/$_texture_pack" \
                "." "$_useCache" "$_EXT_CACHE_PATH"
            case $_texture_pack in
                *.zip)
                    if unzip -l "$_texture_pack" 2>/dev/null | grep -qi "gfx\|actors\|texture"; then
                        _extract "$_texture_pack"
                    fi
                    install -Dm755 "$_texture_pack" build/"$_region"_pc/res/"$_texture_pack"
                    ;;
                *.pak)
                    install -Dm755 "$_texture_pack" build/"$_region"_pc/res/"$_texture_pack"
                    ;;
            esac
        }
        _external_data=1
        popd >/dev/null
    fi

    # Additional git-based texture packs
    if [ -n "$_git_texture_packs" ]; then
        pushd "$srcdir/$_gitname" >/dev/null || exit
        for _gtp in $_git_texture_packs; do
            _git_info=$(_GIT_TEXTURE_MAP "$_gtp" 2>/dev/null) || {
                echo "Warning: unknown git texture pack '$_gtp', skipping"
                continue
            }
            eval set -- $_git_info
            _clone "$_useCache" "$_EXT_CACHE_PATH" "$1" "$2" "$3" "$4" "$5"
        done
        _external_data=1
        popd >/dev/null
    fi

    # ---- Bitness ----
    if [ "$_target_bits" = "default" ]; then
        _target_bits=
    fi

    _render_api=${_render_api:-GL}

    # ---- Build _OPTIONS string ----
    _OPTIONS="VERSION=$_region"
    if [ -n "$_target_bits" ]; then
        _OPTIONS="$_OPTIONS TARGET_BITS=$_target_bits"
    fi
    if [ -n "$_bettercamera" ]; then
        _OPTIONS="$_OPTIONS BETTERCAMERA=$_bettercamera"
    fi
    if [ -n "$_debug" ]; then
        _OPTIONS="$_OPTIONS DEBUG=$_debug"
    fi
    if [ -n "$_nodrawingdistance" ]; then
        _OPTIONS="$_OPTIONS NODRAWINGDISTANCE=$_nodrawingdistance"
    fi
    if [ -n "$_texture_fix" ]; then
        _OPTIONS="$_OPTIONS TEXTURE_FIX=$_texture_fix"
    fi
    if [ -n "$_external_data" ]; then
        _OPTIONS="$_OPTIONS EXTERNAL_DATA=$_external_data"
    fi
    if [ -n "$_discordrpc" ]; then
        _OPTIONS="$_OPTIONS DISCORDRPC=$_discordrpc"
    fi
    if [ -n "$_target_web" ]; then
        _OPTIONS="$_OPTIONS TARGET_WEB=$_target_web"
    fi
    if [ "$_windows_build" = "1" ]; then
        _OPTIONS="$_OPTIONS WINDOWS_BUILD=$_windows_build"
        if [ "$_target_bits" = "64" ]; then
            _OPTIONS="$_OPTIONS CROSS=x86_64-w64-mingw32- CC=x86_64-w64-mingw32-gcc CXX=x86_64-w64-mingw32-g++"
        elif [ "$_target_bits" = "32" ]; then
            _OPTIONS="$_OPTIONS CROSS=i686-w64-mingw32- CC=i686-w64-mingw32-gcc CXX=i686-w64-mingw32-g++"
        fi
    fi
    if [ "$_windows_console" = "1" ]; then
        _OPTIONS="$_OPTIONS WINDOWS_CONSOLE=$_windows_console"
    fi
    if [ -n "$_textsaves" ]; then
        _OPTIONS="$_OPTIONS TEXTSAVES=$_textsaves"
    fi
    if [ -n "$_render_api" ]; then
        _OPTIONS="$_OPTIONS RENDER_API=$_render_api"
    fi
}

_wine_script() {
    cat << 'EOF' > "$_where"/winescript.sh
#!/bin/sh
(
cd /opt/sm64ex
wine sm64ex.exe "$@"
)
EOF
}

_start_script() {
    cat << 'EOF' > "$_where"/sm64ex.sh
#!/bin/sh
(
cd /opt/sm64ex
./sm64ex "$@"
)
EOF
}

pkgver() {
    cd "$srcdir/$_gitname"
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
    # Load _repo_url/_repo_branch early, before _configure_options
    [ -f "$_where/customization.cfg" ] && source "$_where/customization.cfg"
    _repo_url="${_repo_url:-https://github.com/sm64pc/sm64ex.git}"
    _repo_branch="${_repo_branch:-nightly}"

    # ---- Dynamic game repo: switch URL/branch now, before _configure_options ----
    if [ -d "$srcdir/$_gitname/.git" ]; then
        _existing_url=$(git -C "$srcdir/$_gitname" remote get-url origin 2>/dev/null || true)
        if [ "$_existing_url" != "$_repo_url" ]; then
            echo "WARNING: Game repository changed" >&2
            echo "  cached: $_existing_url" >&2
            echo "  config: $_repo_url ($_repo_branch)" >&2
            echo "Re-cloning from configured URL..." >&2
            cd "$srcdir"
            rm -rf "$_gitname"
            git clone --branch "$_repo_branch" --single-branch "$_repo_url" "$_gitname"
        else
            _current_branch=$(git -C "$srcdir/$_gitname" rev-parse --abbrev-ref HEAD 2>/dev/null || true)
            if [ "$_current_branch" != "$_repo_branch" ]; then
                echo "Switching to branch $_repo_branch..." >&2
                cd "$srcdir/$_gitname"
                git fetch origin "$_repo_branch" 2>/dev/null || true
                git checkout "$_repo_branch" 2>/dev/null || true
            fi
        fi
    fi

    _configure_options
    cd "$srcdir/$_gitname"

    if [ "$_windows_build" = "1" ]; then
        if [ "$_render_api" = "GL" ] || [ "$_render_api" = "GL_LEGACY" ]; then
            sed -i 's/-lglew32/-lglew32.dll/g' Makefile
            if [ "$_target_bits" = "64" ]; then
                echo "x86_64" > "$_where/win_gl"
            elif [ "$_target_bits" = "32" ]; then
                echo "i686" > "$_where/win_gl"
            else
                echo "$CARCH" > "$_where/win_gl"
            fi
        else
            touch "$_where/win"
        fi
        _wine_script
    fi

    if [ "$_target_web" = "1" ]; then
        source /etc/profile.d/emscripten.sh
        sed -i 's/TOTAL_MEMORY=20MB/TOTAL_MEMORY=30MB/g' Makefile
        touch "$_where/web"
    fi

    echo "$_region" > "$_where/region"
}

build() {
    cd "$srcdir/$_gitname"
    make $_OPTIONS ${MAKEFLAGS:--j$(nproc)}
}

package() {
    _region=$(cat "$_where/region")

    if [ -e "$_where/web" ]; then
        _target=web
    else
        _target=pc
    fi

    mkdir -p "${pkgdir}/opt"
    cp -r "$srcdir/$_gitname/build/${_region}_${_target}" "${pkgdir}/opt/sm64ex"

    echo "Build output files:" >&2
    ls -la "${pkgdir}/opt/sm64ex/" 2>&1 || echo "(empty)" >&2

    if [ "$_target" = "pc" ]; then
        # Find the built binary (name varies by render API: sm64.us.f3dex2e, sm64.us.GL, etc.)
        _sm64_exe=$(ls "${pkgdir}/opt/sm64ex/sm64.${_region}".* 2>/dev/null | head -1)
        if [ -z "$_sm64_exe" ]; then
            echo "WARNING: No sm64 binary found in build output! Build may have failed." >&2
            ls -la "${pkgdir}/opt/sm64ex/" >&2
            exit 1
        fi
        if [ -e "$_where/win" ] || [ -e "$_where/win_gl" ]; then
            install -Dm755 "$_sm64_exe" "${pkgdir}/opt/sm64ex/sm64ex.exe"
        else
            install -Dm755 "$_sm64_exe" "${pkgdir}/opt/sm64ex/sm64ex"
        fi

        rm -f "${pkgdir}/opt/sm64ex/sm64.${_region}".*

        mkdir -p "${pkgdir}/usr/bin"

        if [ -e "$_where/winescript.sh" ]; then
            install -Dm755 "$_where/winescript.sh" "${pkgdir}/usr/bin/sm64ex"
        else
            _start_script
            install -Dm755 "$_where/sm64ex.sh" "${pkgdir}/usr/bin/sm64ex"
        fi

        ln -sf "/usr/bin/sm64ex" "${pkgdir}/usr/bin/sm64pc"

        if [ -e "$_where/win_gl" ]; then
            _arch=$(cat "$_where/win_gl")
            install -Dm755 "/usr/$_arch-w64-mingw32/bin/glew32.dll" "${pkgdir}/opt/sm64ex/glew32.dll"
        fi

        if [ "$_install_desktop" = "1" ]; then
            install -Dm755 "$_where/sm64ex.desktop" "${pkgdir}/usr/share/applications/sm64ex.desktop"
            install -Dm755 "$_where/SuperMario64.png" "${pkgdir}/usr/share/icons/hicolor/64x64/apps/SuperMario64.png"
        fi
    fi

    rm -rf "$_where/win_gl" "$_where/win" "$_where/web" "$_where/region" \
           "$_where/winescript.sh" "$_where/sm64ex.sh"
    if [ -e "$_where/sm64ex.desktop.orig" ]; then
        mv "$_where/sm64ex.desktop.orig" "$_where/sm64ex.desktop"
    fi
}
