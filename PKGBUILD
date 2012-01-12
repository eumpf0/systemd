# Maintainer: Dave Reisner <dreisner@archlinux.org>

pkgname=systemd
pkgver=38
pkgrel=1
pkgdesc="Session and Startup manager"
arch=('i686' 'x86_64')
url="http://www.freedesktop.org/wiki/Software/systemd"
license=('GPL2')
depends=('dbus-core' 'kbd' 'libcap' 'util-linux>=2.19' 'udev>=172' 'xz')
makedepends=('acl' 'gperf' 'cryptsetup' 'intltool' 'linux-api-headers')
optdepends=('cryptsetup: required for encrypted block devices'
            'dbus-python: systemd-analyze'
            'initscripts: legacy support for hostname and vconsole setup'
            'initscripts-systemd: native boot and initialization scripts'
            'python2-cairo: systemd-analyze'
            'systemd-arch-units: collection of native unit files for Arch daemon/init scripts')
groups=('systemd')
options=('!libtool')
backup=(etc/dbus-1/system.d/org.freedesktop.systemd1.conf
        etc/dbus-1/system.d/org.freedesktop.hostname1.conf
        etc/dbus-1/system.d/org.freedesktop.login1.conf
        etc/dbus-1/system.d/org.freedesktop.locale1.conf
        etc/dbus-1/system.d/org.freedesktop.timedate1.conf
        etc/systemd/system.conf
        etc/systemd/user.conf
        etc/systemd/systemd-logind.conf)
install=systemd.install
source=("http://www.freedesktop.org/software/$pkgname/$pkgname-$pkgver.tar.xz"
        "os-release")
md5sums=('68c66dce5a28c0efd7c210af5d11efed'
         '752636def0db3c03f121f8b4f44a63cd')

build() {
  cd "$pkgname-$pkgver"

  # Don't unset locale in getty
  # https://bugzilla.redhat.com/show_bug.cgi?id=663900
  sed -i -e '/^Environ.*LANG/s/^/#/' \
         -e '/^ExecStart/s/agetty/& -8/' units/getty@.service.m4


  ./configure --sysconfdir=/etc \
              --libexecdir=/usr/lib \
              --libdir=/usr/lib \
              --localstatedir=/var \
              --with-rootprefix= \
              --with-rootlibdir=/lib

  make

  # fix .so links in manpages
  sed -i 's|\.so halt\.8|.so man8/systemd.halt.8|' man/{halt,poweroff}.8
  sed -i 's|\.so systemd\.1|.so man1/systemd.1|' man/init.1
}

package() {
  cd "$pkgname-$pkgver"

  make DESTDIR="$pkgdir" install

  install -Dm644 "$srcdir/os-release" "$pkgdir/etc/os-release"
  printf "d /run/console 755 root root\n" > "$pkgdir/usr/lib/tmpfiles.d/console.conf"

  # fix systemd-analyze for python2
  sed -i '1s/python$/python2/' "$pkgdir/usr/bin/systemd-analyze"

  # rename man pages to avoid conflicts with sysvinit and initscripts
  cd "$pkgdir/usr/share/man"

  manpages=(man8/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8
            man5/{hostname,{vconsole,locale}.conf}.5)

  for manpage in "${manpages[@]}"; do
    IFS='/' read section page <<< "$manpage"
    mv "$manpage" "$section/systemd.$page"
  done
}

# vim: ft=sh syn=sh et
