# Maintainer: Brian Allred brian.d.allred<AT>gmail.com
# Simplified non-interactive version

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

source=('git+https://github.com/sm64pc/sm64ex.git#branch=nightly')
sha256sums=('SKIP')

_where="$PWD"

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

_configure_options() {
    source "$_where"/customization.cfg

    if [ -e "$_EXT_CONFIG_PATH" ]; then
        source "$_EXT_CONFIG_PATH"
    fi

    # Defaults
    _EXT_CACHE_PATH=${_EXT_CACHE_PATH:-~/.cache/sm64ex-custom}
    _EXT_CONFIG_PATH=${_EXT_CONFIG_PATH:-~/.config/sm64ex-custom/config}

    # Cache
    _useCache=${_useCache:-0}
    if [ "$_useCache" = "1" ]; then
        mkdir -p "$_EXT_CACHE_PATH"
    fi
    if [ "$_cleanCache" = "1" ] && [ -n "$_EXT_CACHE_PATH" ]; then
        rm -rf "${_EXT_CACHE_PATH}"/*
    fi

    # Region
    _region=${_region:-us}

    # ROM
    if [ ! -e "$_where"/baserom."$_region".z64 ] && [ -n "$_rom_path" ]; then
        cp "$_rom_path" "$_where"/ && mv "$_where"/"$(basename "$_rom_path")" "$_where"/baserom."$_region".z64
    fi

    (
        cd "$srcdir/$_gitname" || exit

        make clean

        cp "$_where"/baserom."$_region".z64 ./
        ./extract_assets.py $_region
    )

    # ---- Options (from config, no prompts) ----
    # bettercamera, debug, nodrawingdistance, texture_fix, external_data
    # discordrpc, windows_build, windows_console, textsaves, target_web

    # Discord RPC forces 64-bit
    if [ "$_discordrpc" = "1" ]; then
        _target_bits=64
    fi

    # Windows console forces windows build
    if [ "$_windows_console" = "1" ]; then
        _windows_build=1
    fi

    # ---- Apply patches ----
    (
        cd "$srcdir/$_gitname" || exit

        if [ "$_60fps" = "1" ]; then
            git checkout -- enhancements/60fps_ex.patch
            git apply ./enhancements/60fps_ex.patch --ignore-whitespace --reject || true
        fi

        if [ "$_no_exit_star" = "1" ]; then
            if [ ! -e "./enhancements/nonstop_mode_always_enabled.patch" ]; then
                _download nonstop_mode_always_enabled.patch \
                    https://sm64pc.info/downloads/patches/nonstop_mode_always_enabled.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/nonstop_mode_always_enabled.patch --ignore-whitespace --reject || true
        fi

        if [ "$_tight_controls" = "1" ]; then
            if [ ! -e "./enhancements/tight_controls.patch" ]; then
                _download tight_controls.patch \
                    https://sm64pc.info/downloads/patches/tight_controls.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/tight_controls.patch --ignore-whitespace --reject || true
        fi

        if [ "$_captain_toad" = "1" ]; then
            if [ ! -e "./enhancements/captain_toad_stars.patch" ]; then
                _download captain_toad_stars.patch \
                    https://sm64pc.info/downloads/patches/captain_toad_stars.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/captain_toad_stars.patch --ignore-whitespace --reject || true
        fi

        if [ "$_title_return" = "1" ]; then
            if [ ! -e "./enhancements/leave_game.patch" ]; then
                _download leave_game.patch \
                    https://sm64pc.info/downloads/patches/leave_game.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/go_back_to_title_from_ending_nightly.patch --ignore-whitespace --reject || true
        fi

        if [ "$_3d_coin" = "1" ]; then
            if [ ! -e "./enhancements/3d_coin_v2_nightly.patch" ]; then
                _download 3d_coin_v2_nightly.patch \
                    https://sm64pc.info/downloads/patches/3d_coin_v2_nightly.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/3d_coin_v2_nightly.patch --ignore-whitespace --reject || true
        fi

        if [ "$_exit_50_coin" = "1" ]; then
            if [ ! -e "./enhancements/exit_course_50_coin_fix.patch" ]; then
                _download exit_course_50_coin_fix.patch \
                    https://sm64pc.info/downloads/patches/exit_course_50_coin_fix.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/exit_course_50_coin_fix.patch --ignore-whitespace --reject || true
        fi

        if [ "$_title_exit" = "1" ]; then
            if [ ! -e "./enhancements/Added-exit-button.patch" ]; then
                _download Added-exit-button.patch \
                    https://sm64pc.info/downloads/patches/Added-exit-button.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/Added-exit-button.patch --ignore-whitespace --reject || true
        fi

        if [ "$_star_delay" = "1" ]; then
            if [ ! -e "./enhancements/increase_delay_on_star_select.patch" ]; then
                _download increase_delay_on_star_select.patch \
                    https://sm64pc.info/downloads/patches/increase_delay_on_star_select.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/increase_delay_on_star_select.patch --ignore-whitespace --reject || true
        fi

        if [ "$_time_trial" = "1" ]; then
            if [ ! -e "./enhancements/time_trials.2.3.patch" ]; then
                _download time_trials.2.3.patch \
                    https://sm64pc.info/downloads/patches/time_trials.2.3.patch \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
            fi
            git apply ./enhancements/time_trials.patch --ignore-whitespace --reject || true
        fi

        if [ "$_odyssey_moveset" = "1" ]; then
            if [ ! -e "./enhancements/smo.1.0.1.patch" ]; then
                _download odyssey_mario.zip \
                    https://sm64pc.info/downloads/model_pack/odyssey_mario.zip \
                    "./enhancements" "$_useCache" "$_EXT_CACHE_PATH"
                unzip -o enhancements/odyssey_mario.zip
            fi
            git apply ./enhancements/smo.1.0.1.patch --ignore-whitespace --reject || true
        fi
    )

    # ---- HD Models ----
    (
        cd "$srcdir/$_gitname" || exit

        if [ "$_hd_bowser" = "1" ]; then
            if [ ! -e "hd_bowser.zip" ]; then
                _download hd_bowser.zip \
                    https://sm64pc.info/downloads/model_pack/hd_bowser.zip \
                    "." "$_useCache" "$_EXT_CACHE_PATH"
            fi
            unzip -o hd_bowser.zip
        fi

        if [ "$_hd_koopa" = "1" ]; then
            if [ ! -e "koopa_the_quick.zip" ]; then
                _download hd_koopa_the_quick.zip \
                    https://sm64pc.info/downloads/model_pack/hd_koopa_the_quick.zip \
                    "." "$_useCache" "$_EXT_CACHE_PATH"
            fi
            unzip -o koopa_the_quick.zip
            if ! grep -q '#include "koopa_shell/geo_header.h"' "./actors/common0.h"; then
                sed -i '/#endif/i #include "koopa_shell/geo_header.h"' ./actors/common0.h
            fi
            if ! grep -q '#include "koopa/geo_header.h"' "./actors/group14.h"; then
                sed -i '/#endif/i #include "koopa/geo_header.h"' ./actors/group14.h
            fi
        fi

        if [ "$_hd_peach" = "1" ]; then
            if [ ! -e "peach.zip" ]; then
                _download hd_peach_v2.zip \
                    https://sm64pc.info/downloads/model_pack/hd_peach_v2.zip \
                    "." "$_useCache" "$_EXT_CACHE_PATH"
            fi
            unzip -o peach.zip
            install -Dm755 peach_hd_textures.zip build/"$_region"_pc/res/peach_hd_textures.zip
            if ! grep -q '#include "peach/geo_header.h"' "./actors/group10.h"; then
                sed -i '/#endif/i #include "peach/geo_header.h"' ./actors/group10.h
            fi
            _external_data=1
        fi
    )

    # Mario Model
    _mario_model=${_mario_model:-default}

    if [ "$_mario_model" != default ]; then
        (
            cd "$srcdir/$_gitname" || exit

            if [ "$_mario_model" = hd_mario ]; then
                if [ ! -e "hd_mario_v2.zip" ]; then
                    _download hd_mario_v2.zip \
                        https://sm64pc.info/downloads/model_pack/hd_mario_v2.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o hd_mario_v2.zip
            fi

            if [ "$_mario_model" = luigi ]; then
                if [ ! -e "luigi.zip" ]; then
                    _download luigi.zip \
                        https://sm64pc.info/downloads/model_pack/luigi.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o luigi.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = hat_kid ] || [ "$_mario_model" = bow_kid ]; then
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
                if ! grep -q '#include "goomba/geo_header.h"' "./actors/common0.h"; then
                    sed -i '/#endif/i #include "goomba/geo_header.h"' ./actors/common0.h
                fi
                if ! grep -q '#include "mario_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "mario_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_metal_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i #include "mario_metal_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_metal_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "mario_metal_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_metal_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i #include "mario_metal_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "mario_wing_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i #include "mario_wing_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_wing_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "mario_wing_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_wing_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i #include "mario_wing_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "mario_winged_metal_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i #include "mario_winged_metal_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_winged_metal_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "mario_winged_metal_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_winged_metal_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i #include "mario_winged_metal_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "star/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "star/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "transparent_star/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "transparent_star/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q 'mario_metal_cap' "Makefile.split"; then
                    sed -i 's/mario_cap*/mario_cap mario_metal_cap/g' Makefile.split
                fi
                if ! grep -q 'mario_wing_cap' "Makefile.split"; then
                    sed -i 's/mario_metal_cap*/mario_metal_cap mario_wing_cap/g' Makefile.split
                fi
                if ! grep -q 'mario_winged_metal_cap' "Makefile.split"; then
                    sed -i 's/mario_wing_cap*/mario_wing_cap mario_winged_metal_cap/g' Makefile.split
                fi

                if [ "$_mario_model" = hat_kid ]; then
                    if [ ! -e "hat_kid.rar" ]; then
                        _download hat_kid.rar \
                            https://sm64pc.info/downloads/model_pack/hat_kid.rar \
                            "." "$_useCache" "$_EXT_CACHE_PATH"
                    fi
                    unrar x -o+ ./hat_kid.rar
                    install -Dm755 hat_kid_textures_sounds.zip build/"$_region"_pc/res/hat_kid_textures_sounds.zip
                fi

                if [ "$_mario_model" = bow_kid ]; then
                    if [ ! -e "bow_kid.rar" ]; then
                        _download bow_kid.rar \
                            https://sm64pc.info/downloads/model_pack/bow_kid.rar \
                            "." "$_useCache" "$_EXT_CACHE_PATH"
                    fi
                    unrar x -o+ ./bow_kid.rar
                    install -Dm755 bow_kid_textures_sounds.zip build/"$_region"_pc/res/bow_kid_textures_sounds.zip
                fi
            fi

            if [ "$_mario_model" = mawio ]; then
                if [ ! -e "mawio.zip" ]; then
                    _download mawio.zip \
                        https://sm64pc.info/downloads/model_pack/mawio.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o mawio.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = odyssey_mario ]; then
                if [ ! -e "odyssey_mario.zip" ]; then
                    _download odyssey_mario.zip \
                        https://sm64pc.info/downloads/model_pack/odyssey_mario.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o odyssey_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
                if ! grep -q '#include "mario_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i #include "mario_cap/geo_header.h"' ./actors/common1.h
                fi
            fi

            if [ "$_mario_model" = old_school_hd_mario ]; then
                if [ ! -e "old_school_hd_mario.zip" ]; then
                    _download old_school_hd_mario.zip \
                        https://sm64pc.info/downloads/model_pack/old_school_hd_mario.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o old_school_hd_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = beta_mario ]; then
                if [ ! -e "beta_mario.zip" ]; then
                    _download beta_mario.zip \
                        https://sm64pc.info/downloads/model_pack/beta_mario.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o beta_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi
        )
    fi

    # Texture Pack
    _texture_pack=${_texture_pack:-default}

    if [ "$_texture_pack" != default ]; then
        (
            cd "$srcdir/$_gitname" || exit

            if [ "$_texture_pack" = mollymutt ]; then
                if [ ! -e "mollymutt.zip" ]; then
                    _download mollymutt.zip \
                        https://sm64pc.info/downloads/texture_pack/mollymutt.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                install -Dm755 mollymutt.zip build/"$_region"_pc/res/mollymutt.zip
            fi

            if [ "$_texture_pack" = hypatia ]; then
                if [ ! -e "hypatia.zip" ]; then
                    _download hypatia.zip \
                        https://sm64pc.info/downloads/hypatia.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                install -Dm755 hypatia.zip build/"$_region"_pc/res/hypatia.zip
            fi

            if [ "$_texture_pack" = sm64_redrawn ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    https://github.com/TechieAndroid/sm64redrawn.git "" \
                    sm64redrawn sm64redrawn.zip gfx
            fi

            if [ "$_texture_pack" = resrgan_16x ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git "" \
                    resrgan_16x resrgan.zip gfx
            fi

            if [ "$_texture_pack" = resrgan_n64 ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git n64-resrgan-faithful \
                    resrgan_n64 resrgan_n64.zip gfx
            fi

            if [ "$_texture_pack" = p3st ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    https://github.com/p3st-textures/p3st-Texture_pack.git "" \
                    p3st p3st-textures.zip gfx p3st-sound.zip sound
            fi

            if [ "$_texture_pack" = minecraft ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    git://github.com/Almondatchy3/MCtexturepackSM64 "" \
                    minecraft minecraft.zip gfx
            fi

            if [ "$_texture_pack" = jappawakka_admentus_hd ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    git://github.com/JappaWakka/Mario64HDTexturePack_PC "" \
                    jappawakka_admentus_hd jappawakka_admentus_hd.zip gfx
            fi

            if [ "$_texture_pack" = cleaner ]; then
                _clone "$_useCache" "$_EXT_CACHE_PATH" \
                    https://github.com/CrashCrod/Cleaner-Aesthetics.git "" \
                    cleaner cleaner.zip gfx
            fi

            if [ "$_texture_pack" = owo ]; then
                if [ ! -e "owo.zip" ]; then
                    _download owo.zip \
                        https://sm64pc.info/downloads/texture_pack/owo.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                install -Dm755 owo.zip build/"$_region"_pc/res/owo.zip
            fi

            if [ "$_texture_pack" = beta_hud ]; then
                if [ ! -e "beta_hud.zip" ]; then
                    _download beta_hud.zip \
                        https://sm64pc.info/downloads/texture_pack/beta_hud.zip \
                        "." "$_useCache" "$_EXT_CACHE_PATH"
                fi
                unzip -o beta_hud.zip
                install -Dm755 beta_hud.zip build/"$_region"_pc/res/beta_hud.zip
            fi

            _external_data=1
        )
    fi

    # Bitness
    if [ "$_target_bits" = default ]; then
        _target_bits=
    fi

    # Render API
    _render_api=${_render_api:-GL}

    # Build OPTIONS string
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

    if [ "$_target" = "pc" ]; then
        if [ -e "$_where/win" ] || [ -e "$_where/win_gl" ]; then
            install -Dm755 "${pkgdir}/opt/sm64ex/sm64.${_region}".* "${pkgdir}/opt/sm64ex/sm64ex.exe"
        else
            install -Dm755 "${pkgdir}/opt/sm64ex/sm64.${_region}".* "${pkgdir}/opt/sm64ex/sm64ex"
        fi

        rm "${pkgdir}/opt/sm64ex/sm64.${_region}".*

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
