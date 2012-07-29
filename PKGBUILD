# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Pierre Schmitz <pierre@archlinux.de>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

# Building for x86_64 requires lib32-glibc & lib32-zlib from [multilib]. These
# libraries are linked from the NaCl toolchain, and are only needed during
# build time.

pkgname=chromium
pkgver=20.0.1132.57
pkgrel=2
pkgdesc="The open-source project behind Google Chrome, an attempt at creating a safer, faster, and more stable browser"
arch=('i686' 'x86_64')
url="http://www.chromium.org/"
license=('BSD')
depends=('gtk2' 'dbus-glib' 'nss' 'alsa-lib' 'xdg-utils' 'bzip2' 'libevent'
         'libxss' 'libgcrypt' 'ttf-dejavu' 'desktop-file-utils'
         'hicolor-icon-theme')
makedepends=('python2' 'perl' 'gperf' 'yasm' 'mesa' 'libgnome-keyring'
             'elfutils' 'subversion')
optdepends=('kdebase-kdialog: needed for file dialogs in KDE')
# Needed for the NaCl toolchain
[[ $CARCH == x86_64 ]] && makedepends+=('lib32-zlib')
provides=('chromium-browser')
conflicts=('chromium-browser')
backup=('etc/chromium/default')
install=chromium.install
source=(http://commondatastorage.googleapis.com/chromium-browser-official/$pkgname-$pkgver.tar.bz2
        naclsdk_linux-$pkgver.tar.bz2::http://commondatastorage.googleapis.com/nativeclient-mirror/nacl/nacl_sdk/$pkgver/naclsdk_linux.bz2
        chromium.desktop
        chromium.default
        chromium.sh
        chromium-gcc47.patch
        chromium-20.0.1132.57-bison-2.6-fix.patch
        chromium-20.0.1132.57-glib-2.16-use-siginfo_t.patch
        sqlite-3.7.6.3-fix-out-of-scope-memory-reference.patch)
sha256sums=('1225c6b6306e89c8892bc0d18e94567d6081d777dee9b8c90efd0da9f6f2641b'
            '0137e738cad095aeb03da0e4eb92df4ab8f4693c81d462eb5a413de6ecd875d1'
            '09bfac44104f4ccda4c228053f689c947b3e97da9a4ab6fa34ce061ee83d0322'
            '478340d5760a9bd6c549e19b1b5d1c5b4933ebf5f8cfb2b3e2d70d07443fe232'
            '4999fded897af692f4974f0a3e3bbb215193519918a1fa9b31ed51e74a2dccb9'
            'f607347ba8477d3c8e60eb3803d26f3c9869f77fd49986c60887c59a6aa7d30d'
            'd7aecc17e1eb582fe791c3e5fb2ca3f0efcb9bf5379309c1c27be35be4363bba'
            'c1baf14121502efbc2a31b64029dcafa0e28ca5b71ad0e28a3c6342d18198615'
            'a700aa054800d1b21d84eaba27c38a703dfa023e9226d11a942690c2a0630aff')

build() {
  cd "$srcdir/chromium-$pkgver"

  # Fix build with gcc 4.7 (patch from openSUSE)
  patch -Np2 -i "$srcdir/chromium-gcc47.patch"

  # Fix build with glibc 2.16
  patch -Np1 -i "$srcdir/chromium-20.0.1132.57-glib-2.16-use-siginfo_t.patch"

  # Fix build with bison 2.6 (patch from Alexis Menard)
  # http://crbug.com/138243 / https://bugs.webkit.org/show_bug.cgi?id=92264
  patch -d third_party/WebKit -Np1 -i \
    "$srcdir/chromium-20.0.1132.57-bison-2.6-fix.patch"

  # http://code.google.com/p/chromium/issues/detail?id=109527
  sed -i 's|glib/gutils.h|glib.h|' ui/base/l10n/l10n_util.cc

  # SQLite: Fix a problem in fts3_write.c causing stack memory to be referenced
  # after it is out of scope (http://www.sqlite.org/src/info/f9c4a7c8f4)
  # (http://code.google.com/p/chromium/issues/detail?id=122525)
  patch -i "$srcdir/sqlite-3.7.6.3-fix-out-of-scope-memory-reference.patch" \
    third_party/sqlite/amalgamation/sqlite3.c

  # Use Python 2
  find . -type f -exec sed -i -r \
    -e 's|/usr/bin/python$|&2|g' \
    -e 's|(/usr/bin/python2)\.4$|\1|g' \
    {} +
  # There are still a lot of relative calls which need a workaround
  mkdir "$srcdir/python2-path"
  ln -s /usr/bin/python2 "$srcdir/python2-path/python"
  export PATH="$srcdir/python2-path:$PATH"

  ln -s "$srcdir/pepper_${pkgver%%.*}/toolchain/linux_x86_newlib" \
    native_client/toolchain/linux_x86_newlib

  # CFLAGS are passed through release_extra_cflags below
  export -n CFLAGS CXXFLAGS

  # Silence "identifier 'nullptr' is a keyword in C++11" warnings
  CFLAGS+=' -Wno-c++0x-compat'

  build/gyp_chromium --depth=. \
    -Dwerror= \
    -Dlinux_sandbox_path=/usr/lib/chromium/chromium-sandbox \
    -Dlinux_strip_binary=1 \
    -Dlinux_use_gold_binary=0 \
    -Dlinux_use_gold_flags=0 \
    -Drelease_extra_cflags="$CFLAGS" \
    -Dffmpeg_branding=Chrome \
    -Dproprietary_codecs=1 \
    -Duse_system_bzip2=1 \
    -Duse_system_ffmpeg=0 \
    -Duse_system_libevent=1 \
    -Duse_system_libjpeg=1 \
    -Duse_system_libpng=1 \
    -Duse_system_libxml=0 \
    -Duse_system_ssl=0 \
    -Duse_system_yasm=1 \
    -Duse_system_zlib=1 \
    -Duse_gconf=0 \
    -Ddisable_sse2=1

  make chrome chrome_sandbox BUILDTYPE=Release
}

package() {
  cd "$srcdir/chromium-$pkgver"

  install -D out/Release/chrome "$pkgdir/usr/lib/chromium/chromium"

  install -Dm4755 -o root -g root out/Release/chrome_sandbox \
    "$pkgdir/usr/lib/chromium/chromium-sandbox"

  cp out/Release/{*.pak,libffmpegsumo.so,nacl_helper{,_bootstrap}} \
    out/Release/{libppGoogleNaClPluginChrome.so,nacl_irt_*.nexe} \
    "$pkgdir/usr/lib/chromium/"

  # Allow users to override command-line options
  install -Dm644 "$srcdir/chromium.default" "$pkgdir/etc/chromium/default"

  cp -a out/Release/locales out/Release/resources "$pkgdir/usr/lib/chromium/"

  find "$pkgdir/usr/lib/chromium/" -name '*.d' -type f -delete

  install -Dm644 out/Release/chrome.1 "$pkgdir/usr/share/man/man1/chromium.1"

  install -Dm644 "$srcdir/chromium.desktop" \
    "$pkgdir/usr/share/applications/chromium.desktop"

  for size in 16 22 24 32 48 64 128 256; do
    install -Dm644 "chrome/app/theme/chromium/product_logo_$size.png" \
      "$pkgdir/usr/share/icons/hicolor/${size}x${size}/apps/chromium.png"
  done

  install -D "$srcdir/chromium.sh" "$pkgdir/usr/bin/chromium"

  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/chromium/LICENSE"
}

# vim:set ts=2 sw=2 et:
