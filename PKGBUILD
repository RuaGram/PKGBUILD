# Maintainer: Revincx <revincx233@gmail.com>

pkgname=yukigram-rua
pkgver=6.7.r24651.84219988b2
pkgrel=1
pkgdesc='Yet another unofficial tdesktop client, but forked from yukigram'
arch=('x86_64')
url="https://github.com/Revincx/Yukigram"
license=('GPL3')
provides=('yukigram-desktop')
conflicts=('yukigram-desktop')
depends=(
    'abseil-cpp'
    'ada'
    'boost-libs'
    'ffmpeg'
    'glib2'
    'hicolor-icon-theme'
    'hunspell'
    'kcoreaddons'
    'libavif'
    'libdispatch'
    'libheif'
    'libjxl'
    'libstdc++'
    'libxcomposite'
    'libxdamage'
    'libxrandr'
    'libxtst'
    'lz4'
    'minizip'
    'zlib'
    'glibc'
    'libgcc'
    'openal'
    'openh264'
    'openssl'
    'pipewire'
    'protobuf'
    'qt6-imageformats'
    'qt6-svg'
    'qt6-wayland'
    'rnnoise'
    'xxhash'
)
makedepends=('cmake' 'git' 'ninja' 'python' 'range-v3' 'microsoft-gsl' 'ccache'
             'libtg_owt' 'gobject-introspection' 'boost' 'fmt' 'glib2-devel' 'gperf')
optdepends=('geoclue: geoinformation support'
            'geocode-glib-2: geocoding support'
            'webkit2gtk: embedded browser features'
            'xdg-desktop-portal: desktop integration')

_source="https://github.com/Revincx/Yukigram.git"
_branch="dev"

source=("$pkgname::git+$_source#branch=$_branch"
        "tdesktop-fix-minizip-includes.patch")

_source_tdlib() {

  _pkgsrc_tdlib="telegram-tdlib"
  source+=("$_pkgsrc_tdlib"::"git+https://github.com/tdlib/td.git")
  sha512sums+=('SKIP')
}

sha512sums=('SKIP'
            'SKIP')

pkgver() {
    [[ -z "$HEAD" ]] && HEAD=origin/HEAD
    cd "$pkgname"
    local version="$(grep AppVersionStr Telegram/SourceFiles/core/version.h | head -1 | sed -e 's|.;||' -e 's|constexpr auto AppVersionStr = .||')"
    printf "%s.r%s.%s" "$version" "$(git rev-list --count $HEAD)" "$(git rev-parse --short=10 $HEAD)"
}

_source_tdlib

prepare() {
    cd "$pkgname"
    git reset --hard $HEAD
    export __SOURCE_DIR=$_source_dir

    git submodule update --init --recursive --force
    
    patch -Np1 -d $srcdir/$pkgname/Telegram/lib_base -i $srcdir/tdesktop-fix-minizip-includes.patch
}

bail() {
    echo "$@"
    exit 1
}

validate_api() {
    [[ "$API_ID" =~ ^[1-9][0-9]*$ ]] || bail "API_ID must be a positive number"
    [[ "$API_HASH" =~ ^[0-9a-f]{32}$ ]] || bail "API_HASH must contain 32 hex digits [0-9a-f]"
}

build() {
    validate_api

    CXXFLAGS+=' -ffat-lto-objects'

    echo "Building tde2e..."

    cmake -B "build_tde2e" \
        -S "$_pkgsrc_tdlib" \
        -G Ninja \
        -DCMAKE_BUILD_TYPE=None \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DTD_E2E_ONLY=ON \
        -DBUILD_SHARED_LIBS=OFF \
        -DBUILD_TESTING=OFF \
        -Wno-dev

    cmake --build "build_tde2e"
    DESTDIR="$srcdir/deps" cmake --install "build_tde2e"

    echo "Building yukigram..."
    cmake -B build \
        -S $pkgname \
        -G Ninja \
        -D CMAKE_INSTALL_PREFIX="/usr" \
        -D CMAKE_PREFIX_PATH="$srcdir/deps/usr" \
        -D CMAKE_INTERPROCEDURAL_OPTIMIZATION=OFF \
        -D CMAKE_EXE_LINKER_FLAGS="-Wl,--copy-dt-needed-entries" \
        -D TDESKTOP_API_ID="$API_ID" \
        -D TDESKTOP_API_HASH="$API_HASH" \
        -D DESKTOP_APP_DISABLE_AUTOUPDATE=ON \
        -Wno-dev
    
    cmake --build build --config Release --parallel
}

package() {
    DESTDIR="$pkgdir" cmake --install build
}

# Based on https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=telegram-desktop-userfonts (commit 9ce5fd07)
# fix-lzma.patch took from https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=64gram-desktop (commit 2f6d1aeb)
