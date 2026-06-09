# Maintainer: Conor Power <conor-power@themolehole.org>
pkgname='proton-moleman'
pkgdesc='proton-cachyos built with wow64 and without tts, libpcap, sarek, and d7vk'
url='https://github.com/CachyOS/proton-cachyos.git'
license=('custom')
arch=(x86_64 x86_64_v3)
pkgrel=3
pkgver=cachyos.11.0.20260521.slr.r0.g16dba00
makedepends=(git ccache podman)
provides=(proton)
options=(ccache pestrip lto !buildflags !staticlibs !emptydirs !debug)
_srctag='cachyos-11.0-20260521-slr'
source=("git+$url#tag=$_srctag")
sha256sums=('0aadc763ae826cc2455c630654ae9f8ae89bdf5d1f848a0c5c1a4ccb94b5a4ee')

prepare() {
    [ ! -d build ] && mkdir build
    cd proton-cachyos
    git submodule update --init --recursive --jobs $(nproc)
}

build() {
    cd build
    ../proton-cachyos/configure.sh  \
        --build-name="$pkgname"     \
        --enable-ccache             \
        --enable-wow64              \
        --without-tts               \
        --without-libpcap           \
        --without-sarek             \
        --without-d7vk
    time (make dist)
}

package() {
    cd build

    # The build system is failing to do this itself for some reason.
    echo `date '+%s'` `GIT_DIR=../proton-cachyos/.git git describe --tags` > dist/version

    local _compat="$pkgdir/usr/share/steam/compatibilitytools.d"

    # Install Proton
    mkdir -p "$_compat/$pkgname"
    rsync --delete -arx dist/* "$_compat/$pkgname"
    # Install Licenses
    mkdir -p "$pkgdir/usr/share/licenses/$pkgname"
    mv "$_compat/$pkgname"/{PATENTS.AV1,LICENSE{,.OFL}} \
        "$pkgdir/usr/share/licenses/$pkgname"
}

pkgver() {
    cd proton-cachyos
    git describe --long --tags --abbrev=7 | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}
