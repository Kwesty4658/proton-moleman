# Maintainer: Conor Power <conor-power@themolehole.org>
pkgname="proton-moleman"
pkgdesc="proton-cachyos with valve's bleeding-edge changes applied"
url="https://github.com/CachyOS/proton-cachyos.git"
license=('custom')
arch=(x86_64 x86_64_v3)
pkgrel=1
_srctag=11.0-20260521
pkgver=${_srctag//-/.}
makedepends=(git ccache podman)
provides=(proton)
options=(ccache pestrip lto !buildflags !staticlibs !emptydirs !debug)
source=("git+$url#tag=cachyos-$_srctag-slr")

_vkd3dcommit='e02b9b1'

prepare() {
    [ ! -d build ] && mkdir build
    cd proton-cachyos

    git submodule update --init --recursive --jobs $(nproc)

    cd FEX               && git checkout main          && cd ..
    cd dxvk              && git checkout master        && cd ..
    cd dxvk-nvapi        && git checkout master        && cd ..
    cd vkd3d-proton      && git checkout master        && cd ..
    cd vkd3d ; git fetch && git checkout $_vkd3dcommit && cd ..

    cd wine
    git config user.email "pkgbuild@local" && git config user.name "john pkgbuild" && git config set advice.mergeConflict false

    git remote remove valve 2>/dev/null || true
    git remote add valve https://github.com/ValveSoftware/wine && git fetch valve bleeding-edge
    git rebase --reapply-cherry-picks valve/bleeding-edge -X theirs || {
        git rebase --skip
    }
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
sha256sums=('0aadc763ae826cc2455c630654ae9f8ae89bdf5d1f848a0c5c1a4ccb94b5a4ee')
