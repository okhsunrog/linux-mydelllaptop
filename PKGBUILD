# Maintainer: okhsunrog <me@gornushko.com>
pkgbase=linux-mydelllaptop
pkgdesc="pf-kernel by okhsunrog (personal build for me Dell laptop)"
pkgver=5.12.pf6
_product=$pkgbase
pkgrel=2
arch=(x86_64)
url="https://gitlab.com/post-factum/pf-kernel/-/wikis/README"
license=(GPL2)
makedepends=(
  make gcc bc kmod libelf pahole cpio perl
  xmlto git tar inetutils xz
)
options=('!strip')
_srcname="pf-kernel-v5.12-pf6"
source=(
  "https://gitlab.com/post-factum/pf-kernel/-/archive/v5.12-pf6/pf-kernel-v5.12-pf6.tar.gz"
  config
  flashmtk.patch
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src    
  for src in "${source[@]}"; do    
    src="${src%%::*}"    
    src="${src##*/}"    
    [[ $src = *.patch ]] || continue    
    echo "Applying patch $src..."    
    patch -Np1 < "../$src"    
  done 

  echo "Setting config..."
  cp ../config .config
  make olddefconfig

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make all -j2
}

_package() {
  pkgdesc="The $pkgdesc. Kernel and modules"
  depends=(coreutils kmod initramfs)
  optdepends=('crda: to set the correct wireless channels of your country'
              'linux-firmware: firmware images needed for some devices')

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build and source links
  rm -f "$modulesdir"/{source,build}

}

pkgname=("${_product}")
for _package in "${pkgname[@]}"; do
	local _package_no_git="${_package%-git}"
	local _package_stripped="${_package_no_git#$_product}"
	eval "package_${_package}() {
	$(declare -f "_package${_package_stripped}")
	_package${_package_stripped}
}"
done
