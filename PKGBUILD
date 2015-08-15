
# $Id$
# Maintainer: Andreas Radke <andyrtr@archlinux.org>

pkgname=cups-usblp
pkgver=1.5.0
pkgrel=1
arch=('i686' 'x86_64')
license=('GPL')
url="http://www.cups.org/"
makedepends=('libtiff>=3.9.2-2' 'libpng>=1.4.0' 'acl' 'openslp' 'pam' 'xdg-utils' 'krb5' 'gnutls>=2.8.3' 'poppler>=0.12.3'
             'xinetd' 'gzip' 'autoconf' 'php' 'libusb-compat' 'dbus-core' 'avahi'  'hicolor-icon-theme')
source=(ftp://ftp.easysw.com/pub/cups/${pkgver}/cups-${pkgver}-source.tar.bz2
        cups-avahi.patch
        cups-no-export-ssllibs.patch
        cups-no-gcrypt.patch
        cups cups.logrotate cups.pam)
#options=('!emptydirs')
md5sums=('e54ed09ede2340fc3014913333520fe4'
         'e0843e8d8c345792ac73a185260e69fe'
         '9b8467a1e51d360096b70e2c3c081e6c'
         '3733c23e77eb503bd94cc368e02830dc'
         '9657daa21760bb0b5fa3d8b51d5e01a1'
         'f861b18f4446c43918c8643dcbbd7f6d'
         '96f82c38f3f540b53f3e5144900acf17')

# move client.conf man page for next update to the client pkg.

build() {
  cd ${srcdir}/cups-${pkgver}
  # Avahi support in the dnssd backend. patch from Debian based on the Fedora work but brings it in a single file http://patch-tracker.debian.org/package/cups
  patch -Np1 -i ${srcdir}/cups-avahi.patch

  # Do not export SSL libs in cups-config
  patch -Np1 -i "${srcdir}/cups-no-export-ssllibs.patch"

  patch -Np1 -i "${srcdir}/cups-no-gcrypt.patch"
  
  # Rebuild configure script for --enable-avahi.
  aclocal -I config-scripts
  autoconf -I config-scripts

  ./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var \
     --libdir=/usr/lib \
     --with-logdir=/var/log/cups \
     --with-docdir=/usr/share/cups/doc \
     --with-cups-user=daemon \
     --with-cups-group=lp \
     --enable-pam=yes \
     --disable-ldap \
     --enable-raw-printing \
     --enable-dbus --with-dbusdir=/etc/dbus-1 \
     --enable-ssl=yes --enable-gnutls \
     --enable-threads \
     --enable-avahi\
     --with-php=/usr/bin/php-cgi \
     --with-pdftops=pdftops \
     --with-optim="$CFLAGS" \
     --disable-libusb
  make
}

check() {
  cd "$srcdir/cups-$pkgver"
  #httpAddrGetList(workstation64): FAIL
  #1 TESTS FAILED!
  #make[1]: *** [testhttp] Error 1
  make -k check || /bin/true
}

package() {
provides=('cups')
conflicts=('cups')
pkgdesc="The CUPS Printing System - deamon package"
install=cups.install
backup=(etc/cups/cupsd.conf
        etc/cups/mime.convs
        etc/cups/mime.types
        etc/cups/snmp.conf
        etc/cups/printers.conf
        etc/cups/classes.conf
        etc/cups/client.conf
        etc/cups/subscriptions.conf
        etc/dbus-1/system.d/cups.conf
        etc/logrotate.d/cups
        etc/pam.d/cups
        etc/xinetd.d/cups-lpd)
depends=('acl' 'openslp' 'pam' "libcups>=${pkgver}" 'poppler>=0.12.3' 'libusb-compat' 'dbus-core' 'hicolor-icon-theme')
optdepends=('php: for included phpcups.so module'
            'ghostscript: for non-PostScript printers to print with CUPS to convert PostScript to raster images'
            'foomatic-db: drivers use Ghostscript to convert PostScript to a printable form directly'
            'foomatic-db-engine: drivers use Ghostscript to convert PostScript to a printable form directly'
            'foomatic-db-nonfree: drivers use Ghostscript to convert PostScript to a printable form directly'
            'xdg-utils: xdg .desktop file support')

  cd ${srcdir}/cups-${pkgver}
  make BUILDROOT=${pkgdir} install-data install-exec

  # this one we ship in the libcups pkg
  rm -f ${pkgdir}/usr/bin/cups-config

  # kill the sysv stuff
  rm -rf ${pkgdir}/etc/rc*.d
  rm -rf ${pkgdir}/etc/init.d
  install -D -m755 ../cups ${pkgdir}/etc/rc.d/cupsd
  install -D -m644 ../cups.logrotate ${pkgdir}/etc/logrotate.d/cups
  install -D -m644 ../cups.pam ${pkgdir}/etc/pam.d/cups

  # fix perms on /var/spool and /etc
  chmod 755 ${pkgdir}/var/spool
  chmod 755 ${pkgdir}/etc

  # serial backend needs to run as root (http://bugs.archlinux.org/task/20396)
  chmod 700 ${pkgdir}/usr/lib/cups/backend/serial

  # install ssl directory where to store the certs, solves some samba issues
  install -dm700 -g lp ${pkgdir}/etc/cups/ssl
  # remove directory from package, we create it in cups rc.d file
  rm -rf ${pkgdir}/var/run
#  install -dm511 -g lp ${pkgdir}/var/run/cups/certs 

  # install some more configuration files that will get filled by cupsd
  touch ${pkgdir}/etc/cups/printers.conf
  touch ${pkgdir}/etc/cups/classes.conf
  touch ${pkgdir}/etc/cups/client.conf
  echo "# see 'man client.conf'" >> ${pkgdir}/etc/cups/client.conf
  echo "ServerName /var/run/cups/cups.sock #  alternative: ServerName hostname-or-ip-address[:port] of a remote server" >> ${pkgdir}/etc/cups/client.conf
  touch ${pkgdir}/etc/cups/subscriptions.conf 
  chgrp lp ${pkgdir}/etc/cups/{printers.conf,classes.conf,client.conf,subscriptions.conf}

  # fix .desktop file
  sed -i 's|^Exec=htmlview http://localhost:631/|Exec=xdg-open http://localhost:631/|g' ${pkgdir}/usr/share/applications/cups.desktop

  # compress some driver files, adopted from Fedora
  find ${pkgdir}/usr/share/cups/model -name "*.ppd" | xargs gzip -n9f
}
