# Maintainer: adewale
# Contributor: mesmer
pkgname=sentinelagent
pkgver=26.2.1.24
pkgrel=4
pkgdesc='SentinelOne endpoint protection agent (requires vendor-provided Linux installer)'
arch=('x86_64')
url='https://www.sentinelone.com/'
license=('LicenseRef-SentinelOne')
depends=('glibc' 'kmod' 'zlib')
makedepends=('libarchive')
options=('!strip')
install=sentinelagent.install
source=("SentinelAgent_linux_x86_64_v${pkgver//./_}.deb::file://SentinelAgent_linux_x86_64_v${pkgver//./_}.deb")
sha256sums=('3ff2e97e591135410abafb90e9297cb550738caadf0dd4119b2013574b29e630')

package() {
  bsdtar -xOf "${srcdir}/SentinelAgent_linux_x86_64_v${pkgver//./_}.deb" data.tar.xz | bsdtar -xf - -C "${pkgdir}" ./opt/sentinelone

  install -Dm644 "${pkgdir}/opt/sentinelone/configuration/sentinelone.service" \
    "${pkgdir}/usr/lib/systemd/system/sentinelone.service"
  install -d "${pkgdir}/usr/bin"
  ln -s /opt/sentinelone/bin/sentinelctl "${pkgdir}/usr/bin/sentinelctl"
}
