# $Id$
# Maintainer: Christophe Robin <c.robin@smartphp.org>
# Based on php package from Pierre Schmitz <pierre@archlinux.de>

pkgname=php-debug
_sourcename=php
pkgver=5.2.8
pkgrel=1
_suhosinver=${pkgver}-0.9.6.3
pkgdesc='A high-level scripting language (DEBUG VERSION)'
arch=('i686' 'x86_64')
license=('PHP')
provides=('php')
conflicts=('php')
url='http://www.php.net'
backup=('etc/php/php.ini')
options=(!strip)
depends=('glibc' 'readline' 'ncurses' 'libxml2' 'pcre')
makedepends=('apache' 'imap' 'postgresql-libs' 'mysql' 'libldap' 
             'libpng' 'libjpeg' 'sqlite3' 'unixodbc' 'net-snmp' 'mhash' 'gmp'
             'libmcrypt' 'tidyhtml' 'aspell' 'libtool' 'freetype2' 'libjpeg' 
             'curl' 'libxslt' 'pam' 'openssl' 'bzip2' 'gdbm' 'db')
optdepends=('bzip2: bz2'
            'curl: curl'
            'gdbm: dba'
            'libpng: gd'
            'libjpeg: gd'
            'freetype2: gd'
            'pam: imap'
            'libldap: ldap'
            'libmcrypt: mcrypt'
            'libtool: mcrypt'
            'libmysqlclient: mysql/mysqli/pdo_mysql'
            'unixodbc: odbc/pdo_odbc'
            'openssl: openssl'
            'postgresql-libs: pgsql/pdo_pgsql'
            'aspell: pspell'
            'net-snmp: snmp'
            'sqlite3: pdo_sqlite'
            'tidyhtml: tidy'
            'libxslt: xsl'
            'mhash: mhash'
            'gmp: gmp'
            )
source=("http://www.php.net/distributions/${_sourcename}-${pkgver}.tar.bz2" \
        "http://download.suhosin.org/suhosin-patch-${_suhosinver}.patch.gz" \
        'php.ini' 'apache.conf' 'db-configure.patch')
md5sums=('8760a833cf10433d3e72271ab0d0eccf'
         'd455c3dd5b652046dbac2951a58f64fa'
         'f131ec28cb079f604588c7ffefae5f62'
         '96ca078be6729b665be8a865535a97bf'
         '74e5ce5a02488ec91b1c59f539e42936')

build() {
	phpconfig="--prefix=/usr \
	--sysconfdir=/etc/php \
	--with-layout=GNU \
	--with-config-file-path=/etc/php \
	--with-config-file-scan-dir=/etc/php/conf.d \
	--enable-inline-optimization \
	--enable-debug \
	--disable-rpath \
	--disable-static \
	--enable-shared \
	--mandir=/usr/share/man \
	"

	phpextensions="--with-openssl=shared \
	--with-zlib=shared \
	--enable-bcmath=shared \
	--with-bz2=shared \
	--enable-calendar=shared \
	--with-curl=shared \
	--enable-dba=shared \
	--without-db2 \
	--without-db3 \
	--with-db4=shared \
	--with-gdbm=shared \
	--enable-dbase=shared \
	--enable-exif=shared \
	--enable-ftp=shared \
	--with-gd=shared \
	--enable-gd-native-ttf \
	--with-jpeg-dir=shared,/usr \
	--with-png-dir=shared,/usr \
	--with-gettext=shared \
	--with-imap=shared \
	--with-imap-ssl=shared \
	--with-ldap=shared \
	--enable-mbstring=shared \
	--with-mcrypt=shared \
	--with-mysql=shared \
	--with-mysql-sock=/tmp/mysql.sock \
	--with-mysql=shared \
	--with-mysqli=shared \
	--with-ncurses=shared \
	--with-unixODBC=shared,/usr \
	--enable-pdo=shared \
	--with-pdo-mysql=shared \
	--with-pdo-sqlite=shared,/usr \
	--with-pdo-odbc=shared,unixODBC,/usr \
	--with-pdo-pgsql=shared \
	--with-sqlite=shared \
	--enable-sqlite-utf8 \
	--with-pgsql=shared \
	--enable-shmop=shared \
	--with-snmp=shared \
	--enable-soap=shared \
	--enable-sysvmsg=shared \
	--enable-sysvsem=shared \
	--enable-sysvshm=shared \
	--with-tidy=shared \
	--with-xsl=shared \
	--enable-zip=shared \
	--enable-posix=shared \
	--enable-sockets=shared \
	--enable-xml \
	--with-ttf=shared \
	--enable-session=shared \
	--with-regex=php \
	--with-pcre-regex=/usr \
	--enable-mbstring=all \
	--enable-mbregex \
	--enable-json=shared \
	--with-iconv=shared \
	--with-xmlrpc=shared \
	--with-pspell=shared \
	--with-freetype-dir=shared,/usr \
	--with-mime-magic=shared \
	--with-gmp=shared \
	--with-mhash=shared \
	"

	PEAR_INSTALLDIR=/usr/share/pear
	export PEAR_INSTALLDIR

	cd ${srcdir}/${_sourcename}-${pkgver}

	# avoid linking against old db version
	patch -p0 -i ${srcdir}/db-configure.patch || return 1

	# apply suhosin patch
	patch -p1 -i ${srcdir}/suhosin-patch-${_suhosinver}.patch || return 1

	# cli
	./configure ${phpconfig} \
		--disable-cgi \
		--with-readline \
		--enable-pcntl \
		--with-pear=/usr/share/pear \
		${phpextensions} || return 1
	make || return 1
	# make test
	make INSTALL_ROOT=${pkgdir} install || return 1

	# cleanup
	rm -f ${pkgdir}`${pkgdir}/usr/bin/php-config --extension-dir`/*.a
	# install php.ini
	install -D -m644 ${srcdir}/php.ini ${pkgdir}/etc/php/php.ini
	install -d -m755 ${pkgdir}/etc/php/conf.d/

	# cgi and fcgi
	./configure ${phpconfig} \
		--enable-fastcgi \
		--enable-cgi \
		--enable-discard-path \
		--enable-force-cgi-redirect \
		--disable-cli \
		${phpextensions} || return 1
	make || return 1
	install -D -m755 sapi/cgi/php-cgi ${pkgdir}/usr/bin/php-cgi || return 1

	# mod_php
	./configure ${phpconfig} \
		--with-apxs2 \
		--disable-cli \
		${phpextensions} || return 1
	make || return 1
	install -D -m644 libs/libphp5.so ${pkgdir}/usr/lib/httpd/modules/libphp5.so || return 1
	install -D -m644 ${srcdir}/apache.conf ${pkgdir}/etc/httpd/conf/extra/php5_module.conf || return 1
}
