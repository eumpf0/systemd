# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Maintainer: Tom Gundersen <teg@jklm.no>

pkgbase=systemd
pkgname=('systemd' 'libsystemd' 'systemd-sysvcompat')
pkgver=233
pkgrel=4
arch=('i686' 'x86_64')
url="https://www.github.com/systemd/systemd"
makedepends=('acl' 'cryptsetup' 'docbook-xsl' 'gperf' 'lz4' 'xz' 'pam' 'libelf'
             'intltool' 'iptables' 'kmod' 'libcap' 'libidn' 'libgcrypt'
             'libmicrohttpd' 'libxslt' 'util-linux' 'linux-api-headers'
             'python-lxml' 'quota-tools' 'shadow' 'gnu-efi-libs' 'git')
options=('strip')
source=("git://github.com/systemd/systemd.git#tag=v$pkgver"
        'initcpio-hook-udev'
        'initcpio-install-systemd'
        'initcpio-install-udev'
        'arch.conf'
        'loader.conf'
        'splash-arch.bmp'
        'systemd-user.pam'
        'systemd-hwdb.hook'
        'systemd-sysusers.hook'
        'systemd-tmpfiles.hook'
        'systemd-update.hook')
sha512sums=('SKIP'
            'f0d933e8c6064ed830dec54049b0a01e27be87203208f6ae982f10fb4eddc7258cb2919d594cbfb9a33e74c3510cfd682f3416ba8e804387ab87d1a217eb4b73'
            '691acebb243b9cd7fb63272662f34bdb9aead710c69aee9361ab2322f9f108600ad5b0214fc00b7cb2d9c95db8abd748030625d60d6567efd98663c56ba28c65'
            'a25b28af2e8c516c3a2eec4e64b8c7f70c21f974af4a955a4a9d45fd3e3ff0d2a98b4419fe425d47152d5acae77d64e69d8d014a7209524b75a81b0edb10bf3a'
            '61032d29241b74a0f28446f8cf1be0e8ec46d0847a61dadb2a4f096e8686d5f57fe5c72bcf386003f6520bc4b5856c32d63bf3efe7eb0bc0deefc9f68159e648'
            'c416e2121df83067376bcaacb58c05b01990f4614ad9de657d74b6da3efa441af251d13bf21e3f0f71ddcb4c9ea658b81da3d915667dc5c309c87ec32a1cb5a5'
            '5a1d78b5170da5abe3d18fdf9f2c3a4d78f15ba7d1ee9ec2708c4c9c2e28973469bc19386f70b3cf32ffafbe4fcc4303e5ebbd6d5187a1df3314ae0965b25e75'
            'b90c99d768dc2a4f020ba854edf45ccf1b86a09d2f66e475de21fe589ff7e32c33ef4aa0876d7f1864491488fd7edb2682fc0d68e83a6d4890a0778dc2d6fe19'
            '2c1f765e7cefc50f07ad994634ea25d9396e6b9c0de46e58f18377e642a471517a0dbf5eb547070a38c6ecf84ec8e030f650a6cee010871cd7a466a32534adda'
            '9d27d97f172a503f5b7044480a0b9ccc0c4ed5dbb2eb3b2b1aa929332c3bcfe38ef0c0310b6566f23b34f9c05b77035221164a7ab7677784c4a54664f12fca22'
            '0f4efddd25256e09c42b953caeee4b93eb49ecc6eaebf02e616b4dcbfdac9860c3d8a3d1a106325b2ebc4dbc6e08ac46702abcb67a06737227ccb052aaa2a067'
            '10190fba9f39a8f4b620a0829e0ba8ed63bb4dbeca712966011ee7807880d01ab2abff1a80baafeb6674db70526a473fe585db8190e864f318fc4d6068552618')
validpgpkeys=(
  '63CDA1E5D3FC22B998D20DD6327F26951A015CC4'  # Lennart Poettering
)

_backports=(
  # build-sys: make RPM macros installation path configurable
  'ff2e33db54719bfe8feea833571652318c6d197c'
  # resolved: do not start LLMNR or mDNS stack when no network enables them
  '2c7ef56459bf6fe7761595585aa4eed5cd183f27^..2c7ef56459bf6fe7761595585aa4eed5cd183f27^2'
)

_validate_tag() {
  local success fingerprint trusted status tag=v$pkgver

  parse_gpg_statusfile /dev/stdin < <(git verify-tag --raw "$tag" 2>&1)

  if (( ! success )); then
    error 'failed to validate tag %s\n' "$tag"
    return 1
  fi

  if ! in_array "$fingerprint" "${validpgpkeys[@]}" && (( ! trusted )); then
    error 'unknown or untrusted public key: %s\n' "$fingerprint"
    return 1
  fi

  case $status in
    'expired')
      warning 'the signature has expired'
      ;;
    'expiredkey')
      warning 'the key has expired'
      ;;
  esac

  return 0
}

prepare() {
  cd "$pkgbase"

  _validate_tag || return

  local _commit
  for _commit in "${_backports[@]}"; do
    git cherry-pick -n "$_commit"
  done

  # nss-resolve: drop the internal fallback to libnss_dns
  git show 5486a31d287f26bcd7c0a4eb2abfa4c074b985f1 -- \
    Makefile.am src/nss-resolve/nss-resolve.c | git apply --index

  ./autogen.sh
}

build() {
  cd "$pkgbase"

  local timeservers=({0..3}.arch.pool.ntp.org)

  local configure_options=(
    --libexecdir=/usr/lib
    --localstatedir=/var
    --sysconfdir=/etc

    --enable-lz4
    --enable-gnuefi
    --disable-audit
    --disable-ima

    --with-sysvinit-path=
    --with-sysvrcnd-path=
    --with-ntp-servers="${timeservers[*]}"
    --with-default-dnssec=no
    --with-dbuspolicydir=/usr/share/dbus-1/system.d
    --without-kill-user-processes
    --with-rpmmacrosdir=no
    # TODO(dreisner): consider changing this to unified
    --with-default-hierarchy=hybrid
  )

  ./configure "${configure_options[@]}"

  make
}

package_systemd() {
  pkgdesc="system and service manager"
  license=('GPL2' 'LGPL2.1')
  depends=('acl' 'bash' 'cryptsetup' 'dbus' 'iptables' 'kbd' 'kmod' 'hwids' 'libcap'
           'libgcrypt' 'libsystemd' 'libidn' 'lz4' 'pam' 'libelf' 'libseccomp'
           'util-linux' 'xz')
  provides=('nss-myhostname' "systemd-tools=$pkgver" "udev=$pkgver")
  replaces=('nss-myhostname' 'systemd-tools' 'udev')
  conflicts=('nss-myhostname' 'systemd-tools' 'udev')
  optdepends=('libmicrohttpd: remote journald capabilities'
              'quota-tools: kernel-level quota management'
              'systemd-sysvcompat: symlink package to provide sysvinit binaries'
              'polkit: allow administration as unprivileged user')
  backup=(etc/pam.d/systemd-user
          etc/systemd/coredump.conf
          etc/systemd/journald.conf
          etc/systemd/journal-remote.conf
          etc/systemd/journal-upload.conf
          etc/systemd/logind.conf
          etc/systemd/system.conf
          etc/systemd/timesyncd.conf
          etc/systemd/resolved.conf
          etc/systemd/user.conf
          etc/udev/udev.conf)
  install="systemd.install"

  make -C "$pkgbase" DESTDIR="$pkgdir" install

  # don't write units to /etc by default. some of these will be re-enabled on
  # post_install.
  rm -r "$pkgdir/etc/systemd/system/"*.wants

  # add back tmpfiles.d/legacy.conf
  install -m644 "$pkgbase/tmpfiles.d/legacy.conf" "$pkgdir/usr/lib/tmpfiles.d"

  # Replace dialout/tape/cdrom group in rules with uucp/storage/optical group
  sed -i 's#GROUP="dialout"#GROUP="uucp"#g;
          s#GROUP="tape"#GROUP="storage"#g;
          s#GROUP="cdrom"#GROUP="optical"#g' "$pkgdir"/usr/lib/udev/rules.d/*.rules
  sed -i 's/dialout/uucp/g;
          s/tape/storage/g;
          s/cdrom/optical/g' "$pkgdir"/usr/lib/sysusers.d/basic.conf

  # add mkinitcpio hooks
  install -Dm644 "$srcdir/initcpio-install-systemd" "$pkgdir/usr/lib/initcpio/install/systemd"
  install -Dm644 "$srcdir/initcpio-install-udev" "$pkgdir/usr/lib/initcpio/install/udev"
  install -Dm644 "$srcdir/initcpio-hook-udev" "$pkgdir/usr/lib/initcpio/hooks/udev"

  # ensure proper permissions for /var/log/journal. This is only to placate
  chown root:systemd-journal "$pkgdir/var/log/journal"
  chmod 2755 "$pkgdir/var/log/journal"

  # match directory mode from extra/polkit
  chmod 0750 "$pkgdir/usr/share/polkit-1/rules.d"

  # we'll create this on installation
  rmdir "$pkgdir/var/log/journal/remote"

  # ship default policy to leave services disabled
  echo 'disable *' >"$pkgdir"/usr/lib/systemd/system-preset/99-default.preset

  # manpages shipped with systemd-sysvcompat
  rm "$pkgdir"/usr/share/man/man8/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8

  # runtime libraries shipped with libsystemd
  rm "$pkgdir"/usr/lib/lib{nss,systemd,udev}*.so*

  # allow core/filesystem to pristine nsswitch.conf
  rm "$pkgdir/usr/share/factory/etc/nsswitch.conf"
  sed -i '/^C \/etc\/nsswitch\.conf/d' "$pkgdir/usr/lib/tmpfiles.d/etc.conf"

  # add example bootctl configuration
  install -Dm644 "$srcdir/arch.conf" "$pkgdir"/usr/share/systemd/bootctl/arch.conf
  install -Dm644 "$srcdir/loader.conf" "$pkgdir"/usr/share/systemd/bootctl/loader.conf
  install -Dm644 "$srcdir/splash-arch.bmp" "$pkgdir"/usr/share/systemd/bootctl/splash-arch.bmp

  install -Dm644 "$srcdir/systemd-hwdb.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-hwdb.hook"
  install -Dm644 "$srcdir/systemd-sysusers.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-sysusers.hook"
  install -Dm644 "$srcdir/systemd-tmpfiles.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-tmpfiles.hook"
  install -Dm644 "$srcdir/systemd-update.hook" "$pkgdir/usr/share/libalpm/hooks/systemd-update.hook"

  # overwrite the systemd-user PAM configuration with our own
  install -Dm644 systemd-user.pam "$pkgdir/etc/pam.d/systemd-user"
}

package_libsystemd() {
  pkgdesc="systemd client libraries"
  depends=('glibc' 'libcap' 'libgcrypt' 'lz4' 'xz')
  license=('GPL2')
  provides=('libsystemd.so' 'libudev.so')

  make -C "$pkgbase" DESTDIR="$pkgdir" install-rootlibLTLIBRARIES
}

package_systemd-sysvcompat() {
  pkgdesc="sysvinit compat for systemd"
  license=('GPL2')
  groups=('base')
  conflicts=('sysvinit')
  depends=('systemd')

  install -dm755 "$pkgdir"/usr/share/man/man8
  cp -d --no-preserve=ownership,timestamp \
    "$pkgbase"/man/{telinit,halt,reboot,poweroff,runlevel,shutdown}.8 \
    "$pkgdir"/usr/share/man/man8

  install -dm755 "$pkgdir/usr/bin"
  for tool in runlevel reboot shutdown poweroff halt telinit; do
    ln -s 'systemctl' "$pkgdir/usr/bin/$tool"
  done

  ln -s '../lib/systemd/systemd' "$pkgdir/usr/bin/init"
}

# vim: ft=sh syn=sh et
