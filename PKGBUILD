# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Maintainer: Tom Gundersen <teg@jklm.no>

pkgbase=systemd
pkgname=('systemd' 'systemd-sysvcompat')
pkgver=208
pkgrel=1
arch=('i686' 'x86_64')
url="http://www.freedesktop.org/wiki/Software/systemd"
makedepends=('acl' 'cryptsetup' 'dbus-core' 'docbook-xsl' 'gobject-introspection' 'gperf'
             'gtk-doc' 'intltool' 'kmod' 'libcap' 'libgcrypt'  'libmicrohttpd' 'libxslt'
             'linux-api-headers' 'pam' 'python' 'quota-tools' 'xz')
options=('!libtool' 'strip' 'debug')
source=("http://www.freedesktop.org/software/$pkgname/$pkgname-$pkgver.tar.xz"
        'initcpio-hook-udev'
        'initcpio-install-systemd'
        'initcpio-install-udev')
md5sums=('6b30239cbea4cb2c832625f1012dbe03'
         '29245f7a240bfba66e2b1783b63b6b40'
         '8b68b0218a3897d4d37a6ccf47914774'
         'bde43090d4ac0ef048e3eaee8202a407')

prepare() {
  cd "$pkgname-$pkgver"
}

build() {
  cd "$pkgname-$pkgver"

  CFLAGS+=' -g'

  ./configure \
      --libexecdir=/usr/lib \
      --localstatedir=/var \
      --sysconfdir=/etc \
      --enable-introspection \
      --enable-gtk-doc \
      --disable-audit \
      --disable-ima \
      --with-sysvinit-path= \
      --with-sysvrcnd-path= \
      --with-firmware-path="/usr/lib/firmware/updates:/usr/lib/firmware"

  make
}

check() {
  # two tests fail due to running under nspawn
  make -C "$pkgname-$pkgver" check || true
}

package_systemd() {
  pkgdesc="system and service manager"
  license=('GPL2' 'LGPL2.1' 'MIT')
  depends=('acl' 'bash' 'dbus-core' 'glib2' 'kbd' 'kmod' 'hwids' 'libcap' 'libgcrypt'
           'pam' 'util-linux' 'xz')
  provides=("libsystemd=$pkgver" 'nss-myhostname' "systemd-tools=$pkgver" "udev=$pkgver"
            'libgudev-1.0.so' 'libsystemd-daemon.so' 'libsystemd-id128.so'
            'libsystemd-journal.so' 'libsystemd-login.so' 'libudev.so')
  replaces=('libsystemd' 'nss-myhostname' 'systemd-tools' 'udev')
  conflicts=('libsystemd' 'nss-myhostname' 'systemd-tools' 'udev')
  optdepends=('cryptsetup: required for encrypted block devices'
              'libmicrohttpd: systemd-journal-gatewayd'
              'quota-tools: kernel-level quota management'
              'python: systemd library bindings'
              'systemd-sysvcompat: symlink package to provide sysvinit binaries')
  backup=(etc/dbus-1/system.d/org.freedesktop.systemd1.conf
          etc/dbus-1/system.d/org.freedesktop.hostname1.conf
          etc/dbus-1/system.d/org.freedesktop.login1.conf
          etc/dbus-1/system.d/org.freedesktop.locale1.conf
          etc/dbus-1/system.d/org.freedesktop.machine1.conf
          etc/dbus-1/system.d/org.freedesktop.timedate1.conf
          etc/systemd/bootchart.conf
          etc/systemd/journald.conf
          etc/systemd/logind.conf
          etc/systemd/system.conf
          etc/systemd/user.conf
          etc/udev/udev.conf)
  install="systemd.install"

  make -C "$pkgname-$pkgver" DESTDIR="$pkgdir" install

  printf "d /run/console 0755 root root\n" > "$pkgdir/usr/lib/tmpfiles.d/console.conf"

  # fix .so links in manpage stubs
  find "$pkgdir/usr/share/man" -type f -name '*.[[:digit:]]' \
      -exec sed -ri '1s|^\.so (.*)\.([0-9]+)|.so man\2/\1.\2|' {} +

  # don't write units to /etc by default -- we'll enable this on post_install
  # as a sane default
  rm "$pkgdir/etc/systemd/system/getty.target.wants/getty@tty1.service"
  rmdir "$pkgdir/etc/systemd/system/getty.target.wants"

  # get rid of RPM macros
  rm -r "$pkgdir/usr/lib/rpm"

  # add back tmpfiles.d/legacy.conf
  install -m644 "systemd-$pkgver/tmpfiles.d/legacy.conf" "$pkgdir/usr/lib/tmpfiles.d"

  # Replace dialout/tape/cdrom group in rules with uucp/storage/optical group
  sed -i 's#GROUP="dialout"#GROUP="uucp"#g;
          s#GROUP="tape"#GROUP="storage"#g;
          s#GROUP="cdrom"#GROUP="optical"#g' "$pkgdir"/usr/lib/udev/rules.d/*.rules

  # add mkinitcpio hooks
  install -Dm644 "$srcdir/initcpio-install-systemd" "$pkgdir/usr/lib/initcpio/install/systemd"
  install -Dm644 "$srcdir/initcpio-install-udev" "$pkgdir/usr/lib/initcpio/install/udev"
  install -Dm644 "$srcdir/initcpio-hook-udev" "$pkgdir/usr/lib/initcpio/hooks/udev"

  # ensure proper permissions for /var/log/journal
  chown root:systemd-journal "$pkgdir/var/log/journal"
  chmod 2755 "$pkgdir/var/log/journal"

  ### split out manpages for sysvcompat
  rm -rf "$srcdir/_sysvcompat"
  install -dm755 "$srcdir"/_sysvcompat/usr/share/man/man8/
  mv "$pkgdir"/usr/share/man/man8/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8 \
     "$srcdir"/_sysvcompat/usr/share/man/man8

  # include MIT license, since it's technically custom
  install -Dm755 "$srcdir/$pkgname-$pkgver/LICENSE.MIT" \
      "$pkgdir/usr/share/licenses/systemd/LICENSE.MIT"
}

package_systemd-sysvcompat() {
  pkgdesc="sysvinit compat for systemd"
  license=('GPL2')
  groups=('base')
  conflicts=('sysvinit')
  depends=('sysvinit-tools' 'systemd')

  mv "$srcdir/_sysvcompat"/* "$pkgdir"

  install -dm755 "$pkgdir/usr/bin"
  for tool in runlevel reboot shutdown poweroff halt telinit; do
    ln -s 'systemctl' "$pkgdir/usr/bin/$tool"
  done

  ln -s '../lib/systemd/systemd' "$pkgdir/usr/bin/init"
}

# vim: ft=sh syn=sh et
