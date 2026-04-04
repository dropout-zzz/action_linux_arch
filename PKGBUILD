# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>

pkgname=linux-zen
pkgver=6.19.10.zen1
pkgrel=1
pkgdesc='The Linux ZEN kernel and modules'
url='https://github.com/zen-kernel/zen-kernel'
arch=(
  x86_64
)
license=(GPL-2.0-only)
depends=(
  coreutils
  initramfs
  kmod
)
makedepends=(
  bc
  cpio
  gettext
  libelf
  pahole
  perl
  python
  tar
  xz
)
optdepends=(
  'linux-firmware: firmware images needed for some devices'
  'scx-scheds: to use sched-ext schedulers'
  'wireless-regdb: to set the correct wireless channels of your country'
)
provides=(
  KSMBD-MODULE
  NTSYNC-MODULE
  VHBA-MODULE
  VIRTUALBOX-GUEST-MODULES
  WIREGUARD-MODULE
)
replaces=(
)
options=(
  !debug
  !strip
)
_srcname=linux-${pkgver%.*}
_srctag=v${pkgver%.*}-${pkgver##*.}
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  $url/releases/download/$_srctag/linux-$_srctag.patch.zst{,.sig}
)
source_x86_64=(config.x86_64)
validpgpkeys=(
  ABAF11C65A2970B130ABE3C479BE3E4300411886  # Linus Torvalds
  647F28654894E3BD457199BE38DBBDC86092693E  # Greg Kroah-Hartman
  83BC8889351B5DEBBB68416EB8AC08600F108CDF  # Jan Alexander Steffens (heftig)
)
b2sums=('f91cdd0b8727ce50ba286b321e85f394ff7e2eff27cde6e6e85e8abfc627b2b069bb5679d890dd33384123c866dc3c1325ad83953a4d08d4db669150cd88cccd'
        'SKIP'
        '2d3391509cf78a85d62c5184793e0257810c683a3e7e7253d5c16b63ca3cf4795e91af6ed879b61fe5c05f5b42d52cd93344c77d633efac6c28ad657edbb3cb0'
        'SKIP')
b2sums_x86_64=('462962f6995dfe702a5de08296619aea9b2091930892e32f517d385984a46ecf6bdc4d2261e0896837e7235d4da1dd643e3ed4a759f2d2398d647f0f0913a7b9')

# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('466d441a0ea5e04b7023618b7b201bfd60effab225f806fd41ce663484395a1c'
            'SKIP'
            '20add8776bad2699b1b9f804557471238a1b38504ca7d4e2d70ad0b6c5fc89c0'
            'SKIP')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgname
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgname#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ../config.$CARCH .config
  make olddefconfig
  diff -u ../config.$CARCH .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgname version $(<version)"
}

build() {
  cd $_srcname
  make all
}

package() {
  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgname" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  ZSTD_CLEVEL=19 make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  # remove build link
  rm "$modulesdir"/build
}

# vim:set ts=8 sts=2 sw=2 et:
