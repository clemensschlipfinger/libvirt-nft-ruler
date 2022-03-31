pkgname=libvirt-nft-ruler
pkgver=0.1
pkgrel=1
pkgdesc='Nftables hooks script for libvirtd'
arch=('any')
depends=('libvirt' 'python')
source=('https://github.com/clemensschlipfinger/libvirt-nft-ruler/archive/refs/heads/main.tar.gz')
sha1sums=('SKIP')

package() {
 mkdir -p "${pkgdir}/etc/libvirt/hooks/network.d"
 mkdir -p "${pkgdir}/usr/share/$pkgname"

 cd 'libvirt-nft-ruler-main'
 cp "libvirt-nft-ruler" "${pkgdir}/etc/libvirt/hooks/network.d"

 cp -r "templates" "${pkgdir}/usr/share/$pkgname"
}
