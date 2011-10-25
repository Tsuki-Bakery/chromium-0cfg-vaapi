# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Pierre Schmitz <pierre@archlinux.de>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgname=chromium
pkgver=15.0.874.102
pkgrel=1
pkgdesc="The open-source project behind Google Chrome, an attempt at creating a safer, faster, and more stable browser"
arch=('i686' 'x86_64')
url="http://www.chromium.org/"
license=('BSD')
depends=('gtk2' 'dbus-glib' 'nss' 'alsa-lib' 'xdg-utils' 'bzip2' 'libevent'
         'libxss' 'libgcrypt' 'ttf-dejavu' 'desktop-file-utils'
         'hicolor-icon-theme')
makedepends=('python2' 'perl' 'gperf' 'yasm' 'mesa' 'libgnome-keyring')
provides=('chromium-browser')
conflicts=('chromium-browser')
install=chromium.install
source=(http://build.chromium.org/official/chromium-$pkgver.tar.bz2
        chromium.desktop
        chromium.sh
        gcc-4.6.patch
        nacl.gypi)
sha1sums=('d8dd8c5f57099ee879e8bd3ab53419f10339762c'
          '7d97535ec0ed124e95888de84b2c6a3654a27d4a'
          '427ecf06cae11f28f59b1912d659ad5541391682'
          '39999918746524fff30e73dc656754733df5c2c2'
          'df4cee39e1d49e10f9c075f5e6e9db28e8260926')

build() {
  cd "$srcdir/chromium-$pkgver"

  # NaCL build remains faily
  cp "$srcdir/nacl.gypi" chrome/

  # Fix build with gcc 4.6
  # http://code.google.com/p/chromium/issues/detail?id=80071
  patch -Np0 -i "$srcdir/gcc-4.6.patch"

  # Fix build with CUPS 1.5
  sed -i '/#include <cups\/cups.h>/ a #include <cups/ppd.h>' \
    chrome/browser/ui/webui/print_preview_handler.cc

  # Use Python 2
  find . -type f -exec sed -i -r \
    -e 's|/usr/bin/python$|&2|g' \
    -e 's|(/usr/bin/python2)\.4$|\1|g' \
    {} +
  # There are still a lot of relative calls which need a workaround
  mkdir "$srcdir/python2-path"
  ln -s /usr/bin/python2 "$srcdir/python2-path/python"
  export PATH="$srcdir/python2-path:$PATH"

  # We need to disable system_ssl until "next protocol negotiation" support is
  # available in our nss package.
  # (See https://bugzilla.mozilla.org/show_bug.cgi?id=547312)

  build/gyp_chromium -f make build/all.gyp --depth=. \
    -Dno_strict_aliasing=1 \
    -Dwerror= \
    -Dlinux_sandbox_path=/usr/lib/chromium/chromium-sandbox \
    -Dlinux_strip_binary=1 \
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
    -Ddisable_nacl=1 \
    $([[ $CARCH == i686 ]] && echo '-Ddisable_sse2=1')

  make chrome chrome_sandbox BUILDTYPE=Release
}

package() {
  cd "$srcdir/chromium-$pkgver"

  install -D out/Release/chrome ${pkgdir}/usr/lib/chromium/chromium

  install -Dm4755 -o root -g root out/Release/chrome_sandbox \
    "$pkgdir/usr/lib/chromium/chromium-sandbox"

  # Add out/Release/{libppGoogleNaClPluginChrome.so,nacl_irt_x86_*.nexe} again
  # when NaCl build is fixed
  cp out/Release/{{chrome,resources}.pak,libffmpegsumo.so} \
    "$pkgdir/usr/lib/chromium/"

  # These links are only needed when building with system ffmpeg
  #ln -s /usr/lib/libavcodec.so.52 ${pkgdir}/usr/lib/chromium/
  #ln -s /usr/lib/libavformat.so.52 ${pkgdir}/usr/lib/chromium/
  #ln -s /usr/lib/libavutil.so.50 ${pkgdir}/usr/lib/chromium/

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
