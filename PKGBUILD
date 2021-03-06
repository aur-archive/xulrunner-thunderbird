# Maintainer: Brandon Carpenter <hashstat .AT. yahoo .DOT. com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Alexander Baldeck <alexander@archlinux.org>

# This package is nearly a duplicate of the now out-of-date xulrunner package
# in the official *extra* repository.  The main difference is that it builds
# xulrunner from the official Thunderbird source tarballs rather than from
# xulrunner tarballs, which tend to lag behind the Thunderbird releases.

# Use the package for Thunderbird extension development.

pkgname=xulrunner-thunderbird
pkgver=24.2.0
pkgrel=1
pkgdesc="Mozilla Runtime Environment for official Thunderbird package"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
depends=('gtk2' 'mozilla-common' 'nss>=3.14.1' 'libxt' 'libxrender' 'hunspell' 'startup-notification' 'mime-types' 'dbus-glib' 'alsa-lib' 'libevent' 'sqlite>=3.7.4' 'libvpx' 'python2')
makedepends=('zip' 'unzip' 'pkg-config' 'diffutils' 'yasm' 'mesa' 'autoconf2.13')
url="http://wiki.mozilla.org/XUL:Xul_Runner"
source=(ftp://ftp.mozilla.org/pub/mozilla.org/thunderbird/releases/$pkgver/source/thunderbird-$pkgver.source.tar.bz2{,.asc}
        mozconfig
        mozilla-pkgconfig.patch
        shared-libs.patch)
options=('!emptydirs' 'staticlibs')
conflicts=('xulrunner' 'xulrunner-current')
replaces=('xulrunner-oss')
sha256sums=('66474132bd6ebbb8a913c3f4acd4ecc9bec011e4c7ee49475f29558801a905cf'
            'SKIP'
            '3fba82b327f8825ebe93ceaeaea4968d57cf7d700f40bf4457b06d263bcc2e8f'
            '23485d937035648add27a7657f6934dc5b295e886cdb0506eebd02a43d07f269'
            'e2b4a00d14f4ba69c62b3f9ef9908263fbab179ba8004197cbc67edbd916fdf1')

prepare() {
  cd "$srcdir/comm-esr24/mozilla"

  cp "$srcdir/mozconfig" .mozconfig

  #fix libdir/sdkdir - fedora
  patch -Np1 -i "$srcdir/mozilla-pkgconfig.patch"
  patch -Np1 -i "$srcdir/shared-libs.patch"

  # WebRTC build tries to execute "python" and expects Python 2
  # Workaround taken from chromium PKGBUILD
  mkdir -p "$srcdir/python2-path"
  ln -sf /usr/bin/python2 "$srcdir/python2-path/python"

  # configure script misdetects the preprocessor without an optimization level
  # https://bugs.archlinux.org/task/34644
  sed -i '/ac_cpp=/s/$CPPFLAGS/& -O2/' configure
}

build() {
  cd "$srcdir/comm-esr24/mozilla"

  export PATH="$srcdir/python2-path:$PATH"
  export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/xulrunner-$pkgver"
  export PYTHON="/usr/bin/python2"

  make -j1 -f client.mk build MOZ_MAKE_FLAGS="$MAKEFLAGS"
}

package() {
  cd "$srcdir/comm-esr24/mozilla"

  make -j1 -f client.mk DESTDIR="$pkgdir" install

  rm -rf "$pkgdir"/usr/lib/xulrunner-$pkgver/{dictionaries,hyphenation}
  ln -sf /usr/share/hunspell "$pkgdir/usr/lib/xulrunner-$pkgver/dictionaries"
  ln -sf /usr/share/hyphen "$pkgdir/usr/lib/xulrunner-$pkgver/hyphenation"

  # add xulrunner library path to ld.so.conf
  install -d $pkgdir/etc/ld.so.conf.d
  echo "/usr/lib/xulrunner-$pkgver" > $pkgdir/etc/ld.so.conf.d/xulrunner.conf

  chmod +x "${pkgdir}/usr/lib/xulrunner-devel-$pkgver/sdk/bin/xpt.py"
  sed -i 's|!/usr/bin/env python$|!/usr/bin/env python2|' \
    "$pkgdir"/usr/lib/xulrunner-devel-$pkgver/sdk/bin/{xpt,header,typelib,xpidl}.py
}
