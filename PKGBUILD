# Maintainer: Jason R. McNeil <jason@jasonrm.net>

_kernel_version=$(pacman -Q linux | awk '{print $2}')

pkgname='zfsonlinux-git'
pkgver=3283_dae3e9ea2
pkgrel=1
license=('CDDL' 'GPL')
pkgdesc='ZFS on Linux is an advanced file system and volume manager which was originally developed for Solaris and is now maintained by the OpenZFS community'
depends=("linux=${_kernel_version}")
makedepends=('git' "linux-headers=${_kernel_version}" "python2" "python")
arch=('x86_64')
url='http://zfsonlinux.org/'
source=('zfs::git+https://github.com/zfsonlinux/zfs.git'
        'zfsonlinux.initcpio.install'
        'zfsonlinux.initcpio.hook')
sha256sums=('SKIP'
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

    echo "${ZFS_REV}_${ZFS_SHA}"
}

build() {
    _kernel_module_version=$(ls -1 /usr/lib/modules/ | grep ${_kernel_version})

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
        --with-linux=/usr/lib/modules/${_kernel_module_version}/build \
        --with-mounthelperdir=/usr/bin \
        --with-udevdir=/lib/udev
    make
}

check() {
    # Verify the version of the kernel we are requiring is the same as the built module is expecting
    NUM_WRONG=$(strings "${srcdir}"/zfs/module/*/*.ko | grep 'vermagic=' | grep -v --count ${_kernel_version} || :)
    if [[ $NUM_WRONG -gt 0 ]]; then
        echo "[CRITICAL] $NUM_WRONG module(s) are built against the wrong kernel"
        exit 1
    fi
}

package() {
    _kernel_module_version=$(ls -1 /usr/lib/modules/ | grep ${_kernel_version})

    # ZFS
    cd ${srcdir}/zfs
    make DESTDIR="${pkgdir}" install
    sed -i "s+${srcdir}++" ${pkgdir}/usr/src/zfs-*/${_kernel_module_version}/Module.symvers

    # take ownership of hostid file
    install -d -m755 "${pkgdir}"/etc
    touch "${pkgdir}"/etc/hostid; chmod 644 "${pkgdir}"/etc/hostid

    install -d -m755 "${pkgdir}"/etc/zfs/initcpio-keys.d

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

