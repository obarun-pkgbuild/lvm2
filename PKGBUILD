# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/lvm2
# 						Maintainer: Eric Bélanger <eric@archlinux.org>
# 						Maintainer: Thomas Bächler <thomas@archlinux.org>

pkgbase=lvm2
pkgname=('lvm2' 'device-mapper')
pkgver=2.02.175
pkgrel=2
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
sha1sums=('11f0a866ea2a4be6002de85d50a437e3d8c9935d'
          '664258ebd7dd3459f79fffa464b267db0d7109dd'
          '81fc438356216abdaead0742555e1719e6ff3127'
          'f6a554eea9557c3c236df2943bb6e7e723945c41')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  cd LVM2.${pkgver}

  # enable lvmetad
  sed -i 's|use_lvmetad = 0|use_lvmetad = 1|' conf/example.conf.in
 
}

build() {
	
	cp -a LVM2.${pkgver} LVM2-initramfs
	cd LVM2-initramfs
  ./configure \
		--prefix=/usr \
		--sysconfdir=/etc \
		--localstatedir=/var \
		--sbindir=/usr/bin \
        --enable-pkgconfig \
        --enable-readline \
        --enable-dmeventd \
        --enable-cmdlib \
        --enable-applib \
        --enable-udev_sync \
        --enable-udev_rules \
        --enable-udev-rule-exec-detection \
        --enable-lvmetad \
        --with-thin=internal \
        --with-cache=internal \
		--disable-udev-systemd-background-jobs \
		--with-udev-prefix=/usr \
		--with-udevdir=/usr/lib/udev/rules.d \
        --with-systemdsystemunitdir=no \
        --with-tmpfilesdir=no \
		--with-default-locking-dir=/run/lock/lvm \
		--with-default-pid-dir=/run \
		--with-default-dm-run-dir=/run \
		--with-default-run-dir=/run/lvm \
		--with-lvmetad-pidfile=/run \
		--with-dmeventd-path=/usr/bin \
		
  cd udev
  make 69-dm-lvm-metad.rules
}

package_device-mapper() {
  pkgdesc="Device mapper userspace library and tools"
  url="http://sourceware.org/dm/"
  depends=(glibc)
  provides=('device-mapper=$pkgver')
  cd LVM2-initramfs
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

  cd LVM2-initramfs
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
