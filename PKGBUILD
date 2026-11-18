# Maintainer: hiruocha <hiruocha[at]outlook[dot]com>

pkgname=linux-marble
pkgver=7.2.8.r1469610.c9f87aa
pkgrel=1
pkgdesc="Linux for Xiaomi POCO F5 / Redmi Note 12 Turbo"
url="https://github.com/iLtanaka/linux-marble"
arch=('aarch64')
license=('GPL-2.0-only')
options=('!debug' '!strip')

depends=('coreutils' 'initramfs' 'kmod')
optdepends=('linux-firmware: firmware images needed for some devices'
            'linux-firmware-xiaomi-marble: firmware images needed for marble')

makedepends=('bc' 'clang' 'cpio' 'initramfs' 'kmod' 'libelf' 'llvm' 'lld'
             'openssl' 'pahole' 'perl' 'python' 'xxhash' 'zstd')

_commit=c9f87aac17d306786d1d15f67d8bdd226770e27a

source=("$pkgname-$_commit.tar.gz::$url/archive/$_commit.tar.gz"
        "config::https://raw.githubusercontent.com/iLtanaka/pmaports/7ed5ca661e40b381290dba4bb4c41c7312d41925/device/testing/linux-xiaomi-marble/config-xiaomi-marble.aarch64")
sha256sums=('f5df18b728f5ad23b57dc420197ec8b99f466ca48c061db49eaa62c5905e96e8'
            '47955958344f0073b9924afb1f3952e1f62300b4daab40b90f2827460c364375')

export KBUILD_BUILD_HOST=archlinuxarm
export KBUILD_BUILD_USER=$pkgname
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd "$srcdir/$pkgname-$_commit"

  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgname#linux}" > localversion.20-pkgname

  cp "$srcdir/config" .config

  make LLVM=1 olddefconfig

  make LLVM=1 -s kernelrelease > version
}

build() {
  cd "$srcdir/$pkgname-$_commit"

  make LLVM=1 Image modules
  make LLVM=1 qcom/sm8475-xiaomi-marble.dtb
}

package() {
  cd "$srcdir/$pkgname-$_commit"
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  install -Dm644 arch/arm64/boot/Image "$modulesdir/vmlinuz"
  echo "$pkgname" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  install -Dm644 arch/arm64/boot/dts/qcom/sm8475-xiaomi-marble.dtb \
    "$pkgdir/boot/dtbs/qcom/sm8475-xiaomi-marble.dtb"

  make LLVM=1 INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/bin/true modules_install
  rm "$modulesdir/build"

  install -Dm644 COPYING "$pkgdir/usr/share/licenses/$pkgname/COPYING"
}
