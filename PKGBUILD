# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/lvm2
# 						Maintainer: Eric Bélanger <eric@archlinux.org>
# 						Maintainer: Thomas Bächler <thomas@archlinux.org>

pkgbase=lvm2
pkgname=('lvm2' 'device-mapper')
pkgver=2.02.177
pkgrel=6
arch=('x86_64')
url="http://sourceware.org/lvm2/"
license=('GPL2' 'LGPL2.1')
makedepends=('eudev-obarun' 'thin-provisioning-tools')
optdepends=('lvmetad-s6rcserv: rc lvmetad service'
			'lvmetad-s6serv: classic lvmetad service')
groups=('base')
source=(https://mirrors.kernel.org/sourceware/lvm2/releases/LVM2.${pkgver}.tgz
        lvm2_install
        lvm2_hook
        11-dm-initramfs.rules)
sha1sums=('b114b2ef40fca63c6df290a5f1aac54ff3e764aa'
          '1ec6b847745b1b69004422841eb7442de12b4ba8'
          '81fc438356216abdaead0742555e1719e6ff3127'
          'f6a554eea9557c3c236df2943bb6e7e723945c41')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal


build() {

 	local CONFIGUREOPTS=(
		--prefix=/usr
		--sbindir=/usr/bin
		--sysconfdir=/etc
		--localstatedir=/var
        --enable-applib
        --enable-cmdlib
        --enable-dmeventd
        --enable-lvmetad
		--enable-lvmpolld
        --enable-pkgconfig
        --enable-readline
        --enable-udev_rules
        --enable-udev_sync
        --enable-use-lvmetad
        --with-cache=internal
        --with-default-dm-run-dir=/run
        --with-default-locking-dir=/run/lock/lvm
        --with-default-pid-dir=/run
        --with-default-run-dir=/run/lvm
        --with-thin=internal
        --with-udev-prefix=/usr
        --enable-udev-rule-exec-detection
        --with-udevdir=/usr/lib/udev/rules.d
		--with-dmeventd-path=/usr/bin
		--with-lvmetad-pidfile=/run
		--disable-udev-systemd-background-jobs
        --with-systemdsystemunitdir=no
        --with-tmpfilesdir=no
		)
  cp -a LVM2.${pkgver} LVM2-initramfs

  cd LVM2.${pkgver}

  ./configure "${CONFIGUREOPTS[@]}"
  make
  # Build legacy udev rule for initramfs
  cd ../LVM2-initramfs
  ./configure "${CONFIGUREOPTS[@]}"
 
  cd udev
  make 69-dm-lvm-metad.rules
}

package_device-mapper() {
  pkgdesc="Device mapper userspace library and tools"
  url="http://sourceware.org/dm/"
  depends=(glibc)
  provides=('device-mapper=$pkgver')
  
  cd LVM2.${pkgver}
  make DESTDIR="${pkgdir}" install_device-mapper
  # extra udev rule for device-mapper in initramfs
  install -D -m644 "${srcdir}/11-dm-initramfs.rules" "${pkgdir}/usr/lib/initcpio/udev/11-dm-initramfs.rules"
 }

package_lvm2() {
  pkgdesc="Logical Volume Manager 2 utilities"
  depends=('bash' "device-mapper>=${pkgver}" 'readline' 'thin-provisioning-tools')
  conflicts=('lvm' 'mkinitcpio<0.7')
  backup=('etc/lvm/lvm.conf'
    'etc/lvm/lvmlocal.conf')
  options=('!makeflags')
  install=lvm2.install

  cd LVM2.${pkgver}
  make DESTDIR="${pkgdir}" install_lvm2
  # install applib
  make -C liblvm DESTDIR="${pkgdir}" install
  # /etc directories
  install -d "${pkgdir}"/etc/lvm/{archive,backup}
  # mkinitcpio hook
  install -D -m644 "${srcdir}/lvm2_hook" "${pkgdir}/usr/lib/initcpio/hooks/lvm2"
  install -D -m644 "${srcdir}/lvm2_install" "${pkgdir}/usr/lib/initcpio/install/lvm2"
  
  # extra udev rule for lvmetad in non-systemd initramfs
  install -D -m644 "${srcdir}/LVM2-initramfs/udev/69-dm-lvm-metad.rules" "${pkgdir}/usr/lib/initcpio/udev/69-dm-lvm-metad.rules"
}
