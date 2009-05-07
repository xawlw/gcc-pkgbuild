# $Id$
# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: Allan McRae <allan@archlinux.org>

# toolchain build order: kernel-headers->glibc->binutils->gcc-libs->gcc->binutils->glibc

pkgname=gcc
pkgver=4.4.0
pkgrel=1
#_snapshot=4.4.0-RC-20090414
_libstdcppmanver=20090406
pkgdesc="The GNU Compiler Collection"
arch=('i686' 'x86_64')
license=('GPL' 'LGPL' 'custom')
groups=('base-devel')
url="http://gcc.gnu.org"
depends=('binutils>=2.19' "gcc-libs>=${pkgver}" 'mpfr>=2.3.1' 'cloog-ppl>=0.15.3')
makedepends=('flex')
replaces=('gcc-fortran' 'gcc-objc')
options=('!libtool')
install=gcc.install
source=(ftp://gcc.gnu.org/pub/gcc/releases/gcc-${pkgver}/gcc-{core,g++,fortran,objc}-${pkgver}.tar.bz2
	#ftp://gcc.gnu.org/pub/gcc/snapshots/${_snapshot}/gcc-{core,g++,fortran,objc}-${_snapshot}.tar.bz2
	ftp://ftp.archlinux.org/other/${pkgname}/libstdc++-man.${_libstdcppmanver}.tar.bz2
	gcc_pure64.patch
	gcc-hash-style-both.patch)
md5sums=('c7e65c47fa94541f7f6cd0cf3d9c850b'
         '687cceaed97c4145281b6418c9b16847'
         '68f33643cbece51f9a04844a6c6b94c3'
         '74b40bb4ab4532b34258293daf6b63f9'
         '96438846668ec45e3f3d471edb436967'
         '4030ee1c08dd1e843c0225b772360e76'
         '6fd395bacbd7b6e47c7b74854b478363')

build() {
  if ! locale -a | grep ^de_DE; then
    echo "You need the de_DE locale to build gcc."
    return 1
  fi
  
  cd ${srcdir}/gcc-${pkgver}
  #cd ${srcdir}/gcc-${_snapshot}
  # Don't install libiberty
  sed -i 's/install_to_$(INSTALL_DEST) //' libiberty/Makefile.in

  if [ "${CARCH}" = "x86_64" ]; then
    patch -Np1 -i ../gcc_pure64.patch || return 1
  fi
  patch -Np0 -i ${srcdir}/gcc-hash-style-both.patch || return 1

  echo ${pkgver} > gcc/BASE-VER

  mkdir build
  cd build
  ../configure --prefix=/usr --enable-shared \
      --enable-languages=c,c++,fortran,objc,obj-c++ \
      --enable-threads=posix --mandir=/usr/share/man --infodir=/usr/share/info \
      --enable-__cxa_atexit  --disable-multilib --libdir=/usr/lib \
      --libexecdir=/usr/lib --enable-clocale=gnu --disable-libstdcxx-pch \
      --with-tune=generic
  make || return 1
  make -j1 DESTDIR=${pkgdir} install || return 1
  install -dm755 ${pkgdir}/lib
  ln -sf ../usr/bin/cpp ${pkgdir}/lib/cpp
  ln -sf gcc ${pkgdir}/usr/bin/cc
  ln -sf g++ ${pkgdir}/usr/bin/c++

  # install the libstdc++ man pages
  install -dm755 ${pkgdir}/usr/share/man/man3
  install -m644 ${srcdir}/libstdc++-man.${_libstdcppmanver}/man3/* \
    ${pkgdir}/usr/share/man/man3/
  # deal with conflicts...
  rm -f ${pkgdir}/usr/share/man/man3/{ctime,queue,random,regex,string}.3

  # Remove libraries and translations in gcc-libs
  rm -f ${pkgdir}/usr/lib/lib*
  find ${pkgdir} -name libstdc++.mo -delete

  # Remove fixed includes, either no need for them, or they're not complete
  rm -rf ${pkgdir}/usr/lib/${CHOST}/${pkgver}/include-fixed/*

  # Install Runtime Library Exception
  install -Dm644 ../COPYING.RUNTIME ${pkgdir}/usr/share/licenses/gcc/RUNTIME.LIBRARY.EXCEPTION

  rm -f ${pkgdir}/usr/share/info/dir
  gzip -9 ${pkgdir}/usr/share/info/*
}
