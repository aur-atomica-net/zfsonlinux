# Maintainer: Jason R. McNeil <jason@jasonrm.net>

_kernel_version=$(pacman -Q linux | awk '{print $2}')
_kernel_module_version=$(pacman -Ql linux | grep -m 1 -oE '[0-9]+\.[0-9]+\.[0-9]+-[0-9]+')

pkgname='zfsonlinux-git'
pkgver=2805.954_b33d668dd.9df9692
pkgrel=1
license=('CDDL' 'GPL')
pkgdesc='An implementation of OpenZFS designed to work in a Linux environment'
depends=("linux=${_kernel_version}")
makedepends=('git' "linux-headers=${_kernel_version}")
arch=('x86_64')
url='http://zfsonlinux.org/'
source=('zfs::git+https://github.com/zfsonlinux/zfs.git'
        'spl::git+https://github.com/zfsonlinux/spl.git'
        'zfsonlinux.initcpio.install'
        'zfsonlinux.initcpio.hook')
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP')
provides=('zfs')
conflicts=('zfsonlinux' 'zfs-git' 'spl-git' 'zfs-utils-git' 'spl-utils-git')
backup=('etc/hostid' 'etc/sudoers.d/zfs')
install=zfsonlinux.install

pkgver() {
    cd ${srcdir}/zfs
    ZFS_REV=$(git rev-list --count --first-parent HEAD)
    ZFS_SHA=$(git rev-parse --short --verify HEAD)

    cd ${srcdir}/spl
    SPL_REV=$(git rev-list --count --first-parent HEAD)
    SPL_SHA=$(git rev-parse --short --verify HEAD)

    echo "${ZFS_REV}.${SPL_REV}_${ZFS_SHA}.${SPL_SHA}"
}

build() {
    # SPL
    cd ${srcdir}/spl
    ./autogen.sh
    ./configure \
        --cache-file=../spl.config.cache \
        --libdir=/usr/lib \
        --prefix=/usr \
        --sbindir=/usr/bin \
        --with-linux=/usr/lib/modules/${_kernel_module_version}-ARCH/build
    make

    # ZFS
    cd ${srcdir}/zfs
    ./autogen.sh
    ./configure \
        --cache-file=../zfs.config.cache \
        --datadir=/usr/share \
        --includedir=/usr/include \
        --libdir=/usr/lib \
        --libexecdir=/usr/lib/zfs \
        --prefix=/usr \
        --sbindir=/usr/bin \
        --sysconfdir=/etc \
        --with-spl=${srcdir}/spl \
        --with-linux=/usr/lib/modules/${_kernel_module_version}-ARCH/build \
        --with-mounthelperdir=/usr/bin \
        --with-udevdir=/lib/udev
    make
}

package() {
    # SPL
    cd ${srcdir}/spl
    make DESTDIR="${pkgdir}" install
    sed -i "s+${srcdir}++" ${pkgdir}/usr/src/spl-*/${_kernel_module_version}-ARCH/Module.symvers

    # ZFS
    cd ${srcdir}/zfs
    make DESTDIR="${pkgdir}" install
    sed -i "s+${srcdir}++" ${pkgdir}/usr/src/zfs-*/${_kernel_module_version}-ARCH/Module.symvers

    # take ownership of hostid file
    install -d -m755 "${pkgdir}"/etc
    touch "${pkgdir}"/etc/hostid; chmod 644 "${pkgdir}"/etc/hostid

    # initcpio support
    install -D -m644 "${srcdir}"/zfsonlinux.initcpio.hook "${pkgdir}"/usr/lib/initcpio/hooks/zfs
    install -D -m644 "${srcdir}"/zfsonlinux.initcpio.install "${pkgdir}"/usr/lib/initcpio/install/zfs

    # dracut support (untested)
    sed -i "s+dracut_install /usr/lib/gcc/.*+dracut_install /usr/lib/libgcc_s\.so\.1+" ${pkgdir}/usr/lib/dracut/modules.d/90zfs/module-setup.sh

    cd "${srcdir}/zfs/contrib/bash_completion.d"
    make DESTDIR="${pkgdir}" install

    # Remove unused
    rm -r "${pkgdir}"/etc/init.d "${pkgdir}"/etc/zfs/zfs-functions "${pkgdir}"/etc/default "${pkgdir}"/usr/share/initramfs-tools

    # Move /lib to /usr/lib and cleanup
    cp -r "${pkgdir}"/{lib,usr}
    rm -r "${pkgdir}"/lib
}
