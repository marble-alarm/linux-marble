# Maintainer: hiruocha <hiruocha[at]outlook[dot]com>
# shellcheck disable=SC2086,SC2128,SC2155,SC2295

pkgbase=linux-marble
pkgname=("$pkgbase" "$pkgbase-headers")
pkgver=7.2.8.r1469610.c9f87aa
pkgrel=1
pkgdesc="Linux for Xiaomi POCO F5 / Redmi Note 12 Turbo"
url="https://github.com/iLtanaka/linux-marble"
arch=('aarch64')
license=('GPL-2.0-only')
options=('!debug' '!strip')

makedepends=('bc' 'clang' 'cpio' 'initramfs' 'kmod' 'libelf' 'llvm' 'lld'
             'openssl' 'pahole' 'perl' 'python' 'xxhash' 'zstd')

_commit=c9f87aac17d306786d1d15f67d8bdd226770e27a

source=("$pkgbase-$_commit.tar.gz::$url/archive/$_commit.tar.gz"
        "config::https://raw.githubusercontent.com/iLtanaka/pmaports/7ed5ca661e40b381290dba4bb4c41c7312d41925/device/testing/linux-xiaomi-marble/config-xiaomi-marble.aarch64")
sha256sums=('f5df18b728f5ad23b57dc420197ec8b99f466ca48c061db49eaa62c5905e96e8'
            '47955958344f0073b9924afb1f3952e1f62300b4daab40b90f2827460c364375')

export KBUILD_BUILD_HOST=archlinuxarm
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd "$srcdir/$pkgbase-$_commit"

  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  cp "$srcdir/config" .config

  make LLVM=1 olddefconfig

  make LLVM=1 -s kernelrelease > version
}

build() {
  cd "$srcdir/$pkgbase-$_commit"

  make LLVM=1 Image modules
  make LLVM=1 qcom/sm8475-xiaomi-marble.dtb
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=('coreutils' 'initramfs' 'kmod')
  optdepends=('linux-firmware-xiaomi-marble: firmware for wifi, bluetooth, modem, gpu, sensors'
              "$pkgbase-headers: headers and scripts for building modules")

  cd "$srcdir/$pkgbase-$_commit"
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"
  provides=("KERNEL-VERSION-ALARM=$kernver")

  install -Dm644 arch/arm64/boot/Image "$modulesdir/vmlinuz"
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  install -Dm644 arch/arm64/boot/dts/qcom/sm8475-xiaomi-marble.dtb \
    "$pkgdir/boot/dtbs/qcom/sm8475-xiaomi-marble.dtb"

  make LLVM=1 INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/bin/true modules_install
  rm "$modulesdir/build"

  install -Dm644 COPYING "$pkgdir/usr/share/licenses/$pkgname/COPYING"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=('binutils' 'glibc' 'libelf' 'libgcc' 'openssl' 'pahole' 'xxhash'
           'zlib' 'zstd')
  provides=('LINUX-HEADERS')

  cd "$srcdir/$pkgbase-$_commit"
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  if [[ $(scripts/config -s CONFIG_OBJTOOL) = y ]]; then
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool
  fi

  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  local _a
  for _a in "$builddir"/arch/*/; do
    [[ $_a = */arm64/ || $_a = */arm/ ]] && continue
    rm -r "$_a"
  done

  rm -r "$builddir/Documentation"
  find -L "$builddir" -type l -delete
  find "$builddir" -type f -name '*.o' -delete

  local _f
  while read -rd '' _f; do
    case "$(file -Sib "$_f")" in
      application/x-sharedlib\;*|application/x-pie-executable\;*)
        strip $STRIP_SHARED "$_f" ;;
      application/x-archive\;*)
        strip $STRIP_STATIC "$_f" ;;
      application/x-executable\;*)
        strip $STRIP_BINARIES "$_f" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  strip $STRIP_STATIC "$builddir/vmlinux"

  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

for _p in "${pkgname[@]}"; do
  eval "package_${_p}() {
    _package${_p#$pkgbase}
  }"
done
