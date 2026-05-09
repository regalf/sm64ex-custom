# Maintainer: Brian Allred brian.d.allred<AT>gmail.com

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

# Default repository settings - can be overridden in customization.cfg
_repo_url="https://github.com/sm64pc/sm64ex.git"
_repo_branch="nightly"

# Source customization.cfg if it exists to allow overriding repo and other settings
[ -f "./customization.cfg" ] && source "./customization.cfg"

source=("${_gitname}::git+${_repo_url}#branch=${_repo_branch}")
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
        echo ""
        source "$_EXT_CONFIG_PATH" && printf " => External configuration file %s will be used to override customization.cfg values." "$_EXT_CONFIG_PATH"
        echo ""
        echo ""
    fi
    
    printf "\n\tWelcome to sm64ex PKGBUILD!"
    printf "\n\tYou'll be presented with some options for customizing the build below."
    printf "\n\t(Unless you fully customized the build via config file)"
    printf "\n\n"

    if [ -z "$_useCache" ]; then
        printf "\tWould you like to cache downloaded resources? [Y/n] "
        read -r CHOICE
        if [ "$CHOICE" = "n" ] || [ "$CHOICE" = "N" ]; then
            _useCache=0
        else
            _useCache=1
        fi
    fi

    if [ "$_useCache" = "1" ]; then
        mkdir -p "$_EXT_CACHE_PATH"

        if [ -z "$_cleanCache" ]; then
            printf "\n\tWould you like to clean the cache? [y/N] "
            read -r CHOICE
            if [ "$CHOICE" = "y" ] || [ "$CHOICE" = "Y" ]; then
                rm -rf "${_EXT_CACHE_PATH:?}"/*
            fi
        fi
    fi

    if [ -z "$_repo_url" ]; then
        printf "\tRepository URL [default: https://github.com/sm64pc/sm64ex.git]: "
        printf "\n\t(Leave empty for default): "
        read -r CHOICE
        if [ -n "$CHOICE" ]; then
            _repo_url="$CHOICE"
            printf "\n\tRepository branch [default: nightly]: "
            read -r CHOICE
            _repo_branch="${CHOICE:-nightly}"
            printf "\n\tNote: Repository changes require setting _repo_url in customization.cfg for future builds.\n"
        fi
        echo ""
    fi

    if [ -z "$_region" ]; then
        printf "\n\tRegion selection:\n\t=> 1. US\n\t2. EU\n\t3. JP\n\tChoice [1-3?]: "
        read -r CHOICE
        if [ "$CHOICE" = "2" ]; then
            _region=eu
            elif [ "$CHOICE" = "3" ]; then
            _region=jp
        else
            _region=us
        fi
        
        echo ""
    fi

    # Copy ROM
    if [ ! -e "$_where"/baserom."$_region".z64 ]; then
        if [ ! -e "$_rom_path" ]; then
            printf "ROM not found. Copy ROM to directory or enter path to ROM: "
            read -r _rom_path
        fi
        
        [ -e "$_where"/baserom."$_region".z64 ] || { cp "$_rom_path" "$_where"/ && mv "$_where"/"$(basename "$_rom_path")" "$_where"/baserom."$_region".z64; }
    fi

    (
        cd "$srcdir/$_gitname" || exit

        printf "\n\tClean build folder? [Y/n] "
        read -r CHOICE
        if [ "$CHOICE" != "n" ] && [ "$CHOICE" != "N" ]; then
            make clean
        fi
        
        printf "\n\tClean extracted assets? [y/N] "
        read -r CHOICE
        if [ "$CHOICE" = "y" ] || [ "$CHOICE" = "Y" ]; then
            make distclean
        fi

        printf "\n\tClean source files? [y/N] "
        read -r CHOICE
        if [ "$CHOICE" = "y" ] || [ "$CHOICE" = "Y" ]; then
            git clean -xfd && git reset --hard
        fi

        cp "$_where"/baserom."$_region".z64 ./

        ./extract_assets.py $_region
    )
    
    # toggleable options vars
    while [ -z "$_skipOptions" ]; do
        printf "\tOption selection:\n\t1. Better camera = %s" "$_bettercamera"
        printf "\n\t2. Build with debug symbols = %s" "$_debug"
        printf "\n\t3. No drawing distance = %s" "$_nodrawingdistance"
        printf "\n\t4. Texture fix = %s" "$_texture_fix"
        printf "\n\t5. Load custom textures (required if selecting a texture pack) = %s" "$_external_data"
        printf "\n\t6. Discord RPC (only works with 64-bit) = %s" "$_discordrpc"
        printf "\n\t7. Windows build = %s" "$_windows_build"
        printf "\n\t8. Enable console mode (Windows only) = %s" "$_windows_console"
        printf "\n\t9. Text-based save files (Experimental) = %s" "$_textsaves"
        printf "\n\t10. Target Web = %s" "$_target_web"
        printf "\n\t11. Finish"
        printf "\n\tToggle [1-10] or Finish [11]: "
        read -r CHOICE
        printf "\n"
        
        if [ "$CHOICE" = "11" ]; then
            _skipOptions=1
            elif [ "$CHOICE" = "1" ]; then
            if [ -z "$_bettercamera" ]; then
                _bettercamera=1
            else
                _bettercamera=$((1-_bettercamera))
            fi
            elif [ "$CHOICE" = "2" ]; then
            if [ -z "$_debug" ]; then
                _debug=1
            else
                _debug=$((1-_debug))
            fi
            elif [ "$CHOICE" = "3" ]; then
            if [ -z "$_nodrawingdistance" ]; then
                _nodrawingdistance=1
            else
                _nodrawingdistance=$((1-_nodrawingdistance))
            fi
            elif [ "$CHOICE" = "4" ]; then
            if [ -z "$_texture_fix" ]; then
                _texture_fix=1
            else
                _texture_fix=$((1-_texture_fix))
            fi
            elif [ "$CHOICE" = "5" ]; then
            if [ -z "$_external_data" ]; then
                _external_data=1
            else
                _external_data=$((1-_external_data))
            fi
            elif [ "$CHOICE" = "6" ]; then
            if [ -z "$_discordrpc" ]; then
                _discordrpc=1
                _target_bits=64
            else
                _discordrpc=$((1-_discordrpc))
                if [ "$_discordrpc" = 1 ]; then
                    _target_bits=64
                else
                    _target_bits=
                fi
            fi
            elif [ "$CHOICE" = "7" ]; then
            if [ -z "$_windows_build" ]; then
                _windows_build=1
            else
                _windows_build=$((1-_windows_build))
                if [ "$_windows_build" = 0 ]; then
                    _windows_console=0
                fi
            fi
            elif [ "$CHOICE" = "8" ]; then
            if [ -z "$_windows_console" ]; then
                _windows_console=1
                _windows_build=1
            else
                _windows_console=$((1-_windows_console))
                if [ "$_windows_console" = 1 ]; then
                    _windows_build=1
                fi
            fi
            elif [ "$CHOICE" = "9" ]; then
            if [ -z "$_textsaves" ]; then
                _textsaves=1
            else
                _textsaves=$((1-_textsaves))
            fi
            elif [ "$CHOICE" = "10" ]; then
            if [ -z "$_target_web" ]; then
                _target_web=1
            else
                _target_web=$((1-_target_web))
            fi
        else
            printf "\tPlease make a valid choice\n"
        fi
    done
    
    # toggleable patches vars
    while [ -z "$_skipPatches" ]; do
        printf "\tPatch selection:\n\t1. 60 FPS = %s" "$_60fps"
        printf "\n\t2. Stay in course after star = %s" "$_no_exit_star"
        printf "\n\t3. Tight controls = %s" "$_tight_controls"
        printf "\n\t4. Captain Toad = %s" "$_captain_toad"
        printf "\n\t5. Return to title from ending = %s" "$_title_return"
        printf "\n\t6. 3D Coins = %s" "$_3d_coin"
        printf "\n\t7. 50 coin 1-UPs = %s" "$_exit_50_coin"
        printf "\n\t8. Mouse support (WIP) = %s" "$_mouse_fix"
        printf "\n\t9. Add exit to title option = %s" "$_title_exit"
        printf "\n\t10. Star select delay = %s" "$_star_delay"
        printf "\n\t11. Time trial mode = %s" "$_time_trial"
        printf "\n\t12. Odyssey moveset = %s" "$_odyssey_moveset"
        printf "\n\t13. Finish"
        printf "\n\tToggle [1-12] or Finish [13]: "
        read -r CHOICE
        printf "\n"
        
        if [ "$CHOICE" = "13" ]; then
            _skipPatches=1
            elif [ "$CHOICE" = "1" ]; then
            if [ -z "$_60fps" ]; then
                _60fps=1
            else
                _60fps=$((1-_60fps))
            fi
            elif [ "$CHOICE" = "2" ]; then
            if [ -z "$_no_exit_star" ]; then
                _no_exit_star=1
            else
                _no_exit_star=$((1-_no_exit_star))
            fi
            elif [ "$CHOICE" = "3" ]; then
            if [ -z "$_tight_controls" ]; then
                _tight_controls=1
            else
                _tight_controls=$((1-_tight_controls))
            fi
            elif [ "$CHOICE" = "4" ]; then
            if [ -z "$_captain_toad" ]; then
                _captain_toad=1
            else
                _captain_toad=$((1-_captain_toad))
            fi
            elif [ "$CHOICE" = "5" ]; then
            if [ -z "$_title_return" ]; then
                _title_return=1
            else
                _title_return=$((1-_title_return))
            fi
            elif [ "$CHOICE" = "6" ]; then
            if [ -z "$_3d_coin" ]; then
                _3d_coin=1
            else
                _3d_coin=$((1-_3d_coin))
            fi
            elif [ "$CHOICE" = "7" ]; then
            if [ -z "$_exit_50_coin" ]; then
                _exit_50_coin=1
            else
                _exit_50_coin=$((1-_exit_50_coin))
            fi
            elif [ "$CHOICE" = "8" ]; then
            if [ -z "$_mouse_fix" ]; then
                _mouse_fix=1
            else
                _mouse_fix=$((1-_mouse_fix))
            fi
            elif [ "$CHOICE" = "9" ]; then
            if [ -z "$_title_exit" ]; then
                _title_exit=1
            else
                _title_exit=$((1-_title_exit))
            fi
            elif [ "$CHOICE" = "10" ]; then
            if [ -z "$_star_delay" ]; then
                _star_delay=1
            else
                _star_delay=$((1-_star_delay))
            fi
            elif [ "$CHOICE" = "11" ]; then
            if [ -z "$_time_trial" ]; then
                _time_trial=1
            else
                _time_trial=$((1-_time_trial))
            fi
            elif [ "$CHOICE" = "12" ]; then
            if [ -z "$_odyssey_moveset" ]; then
                _odyssey_moveset=1
            else
                _odyssey_moveset=$((1-_odyssey_moveset))
            fi
        else
            printf "\tPlease make a valid choice\n"
        fi
    done
    
    (
        cd "$srcdir/$_gitname" || exit
        
        if [ "$_60fps" = "1" ]; then
            git checkout -- enhancements/60fps_ex.patch
            git apply ./enhancements/60fps_ex.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_no_exit_star" = "1" ]; then
            if [ ! -e "./enhancements/nonstop_mode_always_enabled.patch" ]; then
                _download \
                    nonstop_mode_always_enabled.patch \
                    https://sm64pc.info/downloads/patches/nonstop_mode_always_enabled.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/nonstop_mode_always_enabled.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_tight_controls" = "1" ]; then
            if [ ! -e "./enhancements/tight_controls.patch" ]; then
                _download \
                    tight_controls.patch \
                    https://sm64pc.info/downloads/patches/tight_controls.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/tight_controls.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_captain_toad" = "1" ]; then
            if [ ! -e "./enhancements/captain_toad_stars.patch" ]; then
                _download \
                    captain_toad_stars.patch \
                    https://sm64pc.info/downloads/patches/captain_toad_stars.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/captain_toad_stars.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_title_return" = "1" ]; then
            if [ ! -e "./enhancements/leave_game.patch" ]; then
                _download \
                    leave_game.patch \
                    https://sm64pc.info/downloads/patches/leave_game.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/go_back_to_title_from_ending_nightly.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_3d_coin" = "1" ]; then
            if [ ! -e "./enhancements/3d_coin_v2_nightly.patch" ]; then
                _download \
                    3d_coin_v2_nightly.patch \
                    https://sm64pc.info/downloads/patches/3d_coin_v2_nightly.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/3d_coin_v2_nightly.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_exit_50_coin" = "1" ]; then
            if [ ! -e "./enhancements/exit_course_50_coin_fix.patch" ]; then
                _download \
                    exit_course_50_coin_fix.patch \
                    https://sm64pc.info/downloads/patches/exit_course_50_coin_fix.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/exit_course_50_coin_fix.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_title_exit" = "1" ]; then
            if [ ! -e "./enhancements/Added-exit-button.patch" ]; then
                _download \
                    Added-exit-button.patch \
                    https://sm64pc.info/downloads/patches/Added-exit-button.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
                fi
            
            git apply ./enhancements/Added-exit-button.patch --ignore-whitespace --reject || true
        fi
        
        if [ "$_star_delay" = "1" ]; then
            if [ ! -e "./enhancements/increase_delay_on_star_select.patch" ]; then
                _download \
                    increase_delay_on_star_select.patch \
                    https://sm64pc.info/downloads/patches/increase_delay_on_star_select.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/increase_delay_on_star_select.patch --ignore-whitespace --reject || true
        fi

        if [ "$_time_trial" = "1" ]; then
            if [ ! -e "./enhancements/time_trials.2.3.patch" ]; then
                _download \
                    time_trials.2.3.patch \
                    https://sm64pc.info/downloads/patches/time_trials.2.3.patch \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            git apply ./enhancements/time_trials.patch --ignore-whitespace --reject || true
        fi

        if [ "$_odyssey_moveset" = "1" ]; then
            if [ ! -e "./enhancements/smo.1.0.1.patch" ]; then
                _download \
                    odyssey_mario.zip \
                    https://sm64pc.info/downloads/model_pack/odyssey_mario.zip \
                    "./enhancements" \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"

                unzip -o enhancements/odyssey_mario.zip
            fi
            
            git apply ./enhancements/smo.1.0.1.patch --ignore-whitespace --reject || true
        fi
    )
    
    echo ""
    
    # toggleable models vars
    while [ -z "$_skipModels" ]; do
        printf "\tModel selection:\n\t1. HD Bowser = %s" "$_hd_bowser"
        printf "\n\t2. HD Koopa The Quick = %s" "$_hd_koopa"
        printf "\n\t3. HD PEACH = %s" "$_hd_peach"
        printf "\n\t4. Finish"
        printf "\n\tToggle [1-3] or Finish [4]: "
        read -r CHOICE
        printf "\n"
        
        if [ "$CHOICE" = "4" ]; then
            _skipModels=1
            elif [ "$CHOICE" = "1" ]; then
            if [ -z "$_hd_bowser" ]; then
                _hd_bowser=1
            else
                _hd_bowser=$((1-_hd_bowser))
            fi
            elif [ "$CHOICE" = "2" ]; then
            if [ -z "$_hd_koopa" ]; then
                _hd_koopa=1
            else
                _hd_koopa=$((1-_hd_koopa))
            fi
            elif [ "$CHOICE" = "3" ]; then
            if [ -z "$_hd_peach" ]; then
                _hd_peach=1
            else
                _hd_peach=$((1-_hd_peach))
            fi
        else
            printf "\tPlease make a valid choice\n"
        fi
    done
    
    (
        cd "$srcdir/$_gitname" || exit
        
        if [ "$_hd_bowser" = "1" ]; then
            if [ ! -e "hd_bowser.zip" ]; then
                _download \
                    hd_bowser.zip \
                    https://sm64pc.info/downloads/model_pack/hd_bowser.zip \
                    "." \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            unzip -o hd_bowser.zip
        fi
        
        if [ "$_hd_koopa" = "1" ]; then
            if [ ! -e "koopa_the_quick.zip" ]; then
                _download \
                    hd_koopa_the_quick.zip \
                    https://sm64pc.info/downloads/model_pack/hd_koopa_the_quick.zip \
                    "." \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            unzip -o koopa_the_quick.zip
            if ! grep -q '#include "koopa_shell/geo_header.h"' "./actors/common0.h"; then
                sed -i '/#endif/i \
                #include "koopa_shell/geo_header.h"' ./actors/common0.h
            fi
            if ! grep -q '#include "koopa/geo_header.h"' "./actors/group14.h"; then
                sed -i '/#endif/i \
                #include "koopa/geo_header.h"' ./actors/group14.h
            fi
        fi
        
        if [ "$_hd_peach" = "1" ]; then
            if [ ! -e "peach.zip" ]; then
                _download \
                    hd_peach_v2.zip \
                    https://sm64pc.info/downloads/model_pack/hd_peach_v2.zip \
                    "." \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH"
            fi
            
            unzip -o peach.zip
            install -Dm755 peach_hd_textures.zip build/"$_region"_pc/res/peach_hd_textures.zip
            if ! grep -q '#include "peach/geo_header.h"' "./actors/group10.h"; then
                sed -i '/#endif/i \
                #include "peach/geo_header.h"' ./actors/group10.h
	        fi

            _external_data=1
        fi
    )

    echo ""
    
    # vars with options

    if [ -z "$_mario_model" ]; then
        printf "\tMario model selection:\n\t=> 1. Default\n\t2. HD Mario\n\t3. Luigi\n\t4. Hat Kid 2.0\n\t5. Bow Kid 2.0\n\t6. Mawio (OwOify)\n\t7. Odyssey Mario\n\t8. Old School HD Mario\n\t9. Beta Mario\n\tChoice [1-9?]: "
        read -r CHOICE
        if [ "$CHOICE" = "2" ]; then
            _mario_model=hd_mario
            elif [ "$CHOICE" = "3" ]; then
            _mario_model=luigi
            elif [ "$CHOICE" = "4" ]; then
            _mario_model=hat_kid
            elif [ "$CHOICE" = "5" ]; then
            _mario_model=bow_kid
            elif [ "$CHOICE" = "6" ]; then
            _mario_model=mawio
            elif [ "$CHOICE" = "7" ]; then
            _mario_model=odyssey_mario
            elif [ "$CHOICE" = "8" ]; then
            _mario_model=old_school_hd_mario
            elif [ "$CHOICE" = "9" ]; then
            _mario_model=beta_mario
        else
            _mario_model=default
        fi

        echo ""
    fi

    if [ -n "$_mario_model" ] && [ "$_mario_model" != default ]; then
        (
            cd "$srcdir/$_gitname" || exit

            if [ "$_mario_model" = hd_mario ]; then
                if [ ! -e "hd_mario_v2.zip" ]; then
                    _download \
                        hd_mario_v2.zip \
                        https://sm64pc.info/downloads/model_pack/hd_mario_v2.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi

                unzip -o hd_mario_v2.zip
            fi

            if [ "$_mario_model" = luigi ]; then
                if [ ! -e "luigi.zip" ]; then
                    _download \
                        luigi.zip \
                        https://sm64pc.info/downloads/model_pack/luigi.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi

                unzip -o luigi.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = hat_kid ] || [ "$_mario_model" = bow_kid ]; then
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
                if ! grep -q '#include "goomba/geo_header.h"' "./actors/common0.h"; then
                    sed -i '/#endif/i \
                    #include "goomba/geo_header.h"' ./actors/common0.h
                fi
                if ! grep -q '#include "mario_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "mario_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_metal_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i \
                    #include "mario_metal_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_metal_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "mario_metal_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_metal_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i \
                    #include "mario_metal_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "mario_wing_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i \
                    #include "mario_wing_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_wing_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "mario_wing_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_wing_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i \
                    #include "mario_wing_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "mario_winged_metal_cap/model.inc.c"' "./actors/common1.c"; then
                    sed -i '/#endif/i \
                    #include "mario_winged_metal_cap/model.inc.c"' ./actors/common1.c
                fi
                if ! grep -q '#include "mario_winged_metal_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "mario_winged_metal_cap/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "mario_winged_metal_cap/geo.inc.c"' "./actors/common1_geo.c"; then
                    sed -i '/#endif/i \
                    #include "mario_winged_metal_cap/geo.inc.c"' ./actors/common1_geo.c
                fi
                if ! grep -q '#include "star/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "star/geo_header.h"' ./actors/common1.h
                fi
                if ! grep -q '#include "transparent_star/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "transparent_star/geo_header.h"' ./actors/common1.h
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
                        _download \
                            hat_kid.rar \
                             https://sm64pc.info/downloads/model_pack/hat_kid.rar \
                            "." \
                            "$_useCache" \
                            "$_EXT_CACHE_PATH"
                    fi

                    unrar x -o+ ./hat_kid.rar
                    install -Dm755 hat_kid_textures_sounds.zip build/"$_region"_pc/res/hat_kid_textures_sounds.zip
                fi

                if [ "$_mario_model" = bow_kid ]; then
                    if [ ! -e "bow_kid.rar" ]; then
                        _download \
                            bow_kid.rar \
                             https://sm64pc.info/downloads/model_pack/bow_kid.rar \
                            "." \
                            "$_useCache" \
                            "$_EXT_CACHE_PATH"
                    fi

                    unrar x -o+ ./bow_kid.rar
                    install -Dm755 bow_kid_textures_sounds.zip build/"$_region"_pc/res/bow_kid_textures_sounds.zip
                fi
            fi

            if [ "$_mario_model" = mawio ]; then
                if [ ! -e "mawio.zip" ]; then
                    _download \
                        mawio.zip \
                        https://sm64pc.info/downloads/model_pack/mawio.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi

                unzip -o mawio.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = odyssey_mario ]; then
                if [ ! -e "odyssey_mario.zip" ]; then
                    _download \
                        odyssey_mario.zip \
                        https://sm64pc.info/downloads/model_pack/odyssey_mario.zip \
                         "." \
                         "$_useCache" \
                         "$_EXT_CACHE_PATH"
                fi

                unzip -o odyssey_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
                if ! grep -q '#include "mario_cap/geo_header.h"' "./actors/common1.h"; then
                    sed -i '/#endif/i \
                    #include "mario_cap/geo_header.h"' ./actors/common1.h
                fi
            fi

            if [ "$_mario_model" = old_school_hd_mario ]; then
                if [ ! -e "old_school_hd_mario.zip" ]; then
                    _download \
                        old_school_hd_mario.zip \
                        https://sm64pc.info/downloads/model_pack/old_school_hd_mario.zip \
                         "." \
                         "$_useCache" \
                         "$_EXT_CACHE_PATH"
                fi

                unzip -o old_school_hd_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi

            if [ "$_mario_model" = beta_mario ]; then
                if [ ! -e "beta_mario.zip" ]; then
                    _download \
                        beta_mario.zip \
                        https://sm64pc.info/downloads/model_pack/beta_mario.zip \
                         "." \
                         "$_useCache" \
                         "$_EXT_CACHE_PATH"
                fi

                unzip -o beta_mario.zip
                if ! grep -q '#include "mario/geo_header.h"' "./actors/group0.h"; then
                    sed -i '/#endif/i \
                    #include "mario/geo_header.h"' ./actors/group0.h
                fi
            fi
        )    
    fi

    if [ -z "$_texture_pack" ]; then
        printf "\tTexture pack selection:\n\t=> 1. Default\n\t2. MollyMutt's Texture Pack\n\t3. Hypatia's Mario Craft\n\t4. SM64 Redrawn\n\t5. RESRGAN-16xre-upscale\n\t6. RESRGAN N64 Faithful\n\t7. p3st Texture Pack\n\t8. Cleaner Aesthetics\n\t9. OwOify Project (WIP)\n\t10. Minecraft\n\t11. JappaWakka & Admentus HD\n\t12. Beta HUD\n\tChoice [1-12?]: "
        read -r CHOICE
        if [ "$CHOICE" = "2" ]; then
            _texture_pack=mollymutt
            elif [ "$CHOICE" = "3" ]; then
            _texture_pack=hypatia
            elif [ "$CHOICE" = "4" ]; then
            _texture_pack=sm64_redrawn
            elif [ "$CHOICE" = "5" ]; then
            _texture_pack=resrgan_16x
            elif [ "$CHOICE" = "6" ]; then
            _texture_pack=resrgan_n64
            elif [ "$CHOICE" = "7" ]; then
            _texture_pack=p3st
            elif [ "$CHOICE" = "8" ]; then
            _texture_pack=cleaner
            elif [ "$CHOICE" = "9" ]; then
            _texture_pack=owo
            elif [ "$CHOICE" = "10" ]; then
            _texture_pack=minecraft
            elif [ "$CHOICE" = "11" ]; then
            _texture_pack=jappawakka_admentus_hd
            elif [ "$CHOICE" = "12" ]; then
            _texture_pack=beta_hud
        else
            _texture_pack=default
        fi

        echo ""
    fi

    if [ -n "$_texture_pack" ] && [ "$_texture_pack" != default ]; then
        (
            cd "$srcdir/$_gitname" || exit

            if [ "$_texture_pack" = mollymutt ]; then
                if [ ! -e "mollymutt.zip" ]; then
                    _download \
                        mollymutt.zip \
                        https://sm64pc.info/downloads/texture_pack/mollymutt.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi
                install -Dm755 mollymutt.zip build/"$_region"_pc/res/mollymutt.zip
            fi

            if [ "$_texture_pack" = hypatia ]; then
                if [ ! -e "hypatia.zip" ]; then
                    _download \
                        hypatia.zip \
                        https://sm64pc.info/downloads/hypatia.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi
                
                install -Dm755 hypatia.zip build/"$_region"_pc/res/hypatia.zip
            fi

            if [ "$_texture_pack" = sm64_redrawn ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    https://github.com/TechieAndroid/sm64redrawn.git \
                    "" \
                    sm64redrawn \
                    sm64redrawn.zip \
                    gfx
            fi

            if [ "$_texture_pack" = resrgan_16x ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git \
                    "" \
                    resrgan_16x \
                    resrgan.zip \
                    gfx
            fi

            if [ "$_texture_pack" = resrgan_n64 ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    https://github.com/pokeheadroom/RESRGAN-16xre-upscale-HD-texture-pack.git \
                    n64-resrgan-faithful \
                    resrgan_n64 \
                    resrgan_n64.zip \
                    gfx
            fi

            if [ "$_texture_pack" = p3st ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    https://github.com/p3st-textures/p3st-Texture_pack.git \
                    "" \
                    p3st \
                    p3st-textures.zip \
                    gfx \
                    p3st-sound.zip \
                    sound
            fi

            if [ "$_texture_pack" = minecraft ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    git://github.com/Almondatchy3/MCtexturepackSM64 \
                    "" \
                    minecraft \
                    minecraft.zip \
                    gfx
            fi

            if [ "$_texture_pack" = jappawakka_admentus_hd ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    git://github.com/JappaWakka/Mario64HDTexturePack_PC \
                    "" \
                    jappawakka_admentus_hd \
                    jappawakka_admentus_hd.zip \
                    gfx
            fi

            if [ "$_texture_pack" = cleaner ]; then
                _clone \
                    "$_useCache" \
                    "$_EXT_CACHE_PATH" \
                    https://github.com/CrashCrod/Cleaner-Aesthetics.git \
                    "" \
                    cleaner \
                    cleaner.zip \
                    gfx
            fi

            if [ "$_texture_pack" = owo ]; then
                if [ ! -e "owo.zip" ]; then
                    _download \
                        owo.zip \
                        https://sm64pc.info/downloads/texture_pack/owo.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi
                install -Dm755 owo.zip build/"$_region"_pc/res/owo.zip
            fi

            if [ "$_texture_pack" = beta_hud ]; then
                if [ ! -e "beta_hud.zip" ]; then
                    _download \
                        beta_hud.zip \
                        https://sm64pc.info/downloads/texture_pack/beta_hud.zip \
                        "." \
                        "$_useCache" \
                        "$_EXT_CACHE_PATH"
                fi

                unzip -o beta_hud.zip

                install -Dm755 beta_hud.zip build/"$_region"_pc/res/beta_hud.zip
            fi

            _external_data=1
        )
    fi
    
    if [ "$_target_bits" = default ]; then
        _target_bits=
        
        # _target_bits gets set with discordrpc above, but we'll double check here
        elif [ -z "$_target_bits" ] && { [ "$_discordrpc" = "0" ] || [ -z "$_discordrpc" ]; }; then
        printf "\tBitness:\n\t=> 1. Default\n\t2. 64\n\t3. 32\n\tChoice [1-3?]: "
        read -r CHOICE
        if [ "$CHOICE" = "2" ]; then
            _target_bits=64
            elif [ "$CHOICE" = "3" ]; then
            _target_bits=32
        else
            _target_bits=
        fi
        
        echo ""
    fi
    
    # only allow d3d selection if _windows_build = 1
    if [ -z "$_render_api" ]; then
        _output="\tRendering API:\n\t=> 1. OpenGL 2.1\n\t2. OpenGL 1.3 (for older hardware)"
        
        if [ "$_windows_build" = "1" ]; then
            _output="$_output\n\t3. Direct3D 11\n\t4. Direct3D 12\n\tNote that D3D renderers may cause issues with other options\n\tChoice [1-4?]: "
        else
            _output="$_output\n\tChoice [1-2?]: "
        fi
        
        printf "$_output"
        read -r CHOICE
        if [ "$CHOICE" = "2" ]; then
            _render_api=GL_LEGACY
            elif [ "$_windows_build" = "1" ] && [ "$CHOICE" = "3" ]; then
            _render_api=D3D11
            elif [ "$_windows_build" = "1" ] && [ "$CHOICE" = "4" ]; then
            _render_api=D3D12
        else
            _render_api=GL
        fi
        
        echo ""
    fi
    
    # build OPTIONS string
    _OPTIONS="VERSION="$_region
    
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
    
    printf "\n\tBuilding with the following command:\n\tmake %s %s\n\n" "$_OPTIONS" "${MAKEFLAGS:--j$(nproc)}"

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

    # Don't create shortcuts and scripts if target is web
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
        
        install -Dm755 "$_where/sm64ex.desktop" "${pkgdir}/usr/share/applications/sm64ex.desktop"
        install -Dm755 "$_where/SuperMario64.png" "${pkgdir}/usr/share/icons/hicolor/64x64/apps/SuperMario64.png"
    fi
    
    # cleanup
    rm -rf "$_where/win_gl"
    rm -rf "$_where/win"
    rm -rf "$_where/web"
    rm -rf "$_where/region"
    rm -rf "$_where/winescript.sh"
    rm -rf "$_where/sm64ex.sh"
    if [ -e "$_where/sm64ex.desktop.orig" ]; then
        mv "$_where/sm64ex.desktop.orig" "$_where/sm64ex.desktop"
    fi
}
