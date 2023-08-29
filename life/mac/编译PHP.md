# 编译PHP

## openssl

brew install openssl@1.1

export OPENSSL\_CFLAGS="\-I/usr/local/Cellar/openssl@1.1/1.1.1t/include"

export OPENSSL\_LIBS="\-L/usr/local/Cellar/openssl@1.1/1.1.1t/lib"

export PKG\_CONFIG\_PATH=$PKG\_CONFIG\_PATH:/usr/local/lib/pkgconfig

export PATH=/usr/local/Cellar/openssl@1.1/1.1.1t/bin:$PATH

export LDFLAGS=\-L/usr/local/Cellar/openssl@1.1/1.1.1t/lib

## oniguruma

wget wget https://github.com/kkos/oniguruma/archive/v6.9.4.tar.gz \-O oniguruma\-6.9.4.tar.gz

tar \-zxvf oniguruma\-6.9.4.tar.gz

brew install autoconf

brew install libiconv

## 开始编译

./configure prefix=/opt/soft/php \-\-with\-config\-file\-path=/opt/soft/php/etc \-\-with\-curl \-\-with\-mhash \-with\-gettext \-\-with\-pear \-\-with\-xmlrpc \-\-enable\-soap \-\-enable\-fpm \-\-with\-zlib \-\-enable\-sockets \-\-enable\-mbstring \-\-with\-mysqli=shared,mysqlnd \-\-enable\-pdo \-\-with\-pdo\-mysql=shared,mysqlnd \-\-enable\-exif \-\-enable\-bcmath \-\-enable\-debug \-\-enable\-pcntl \-\-with\-iconv=/usr/local/opt/libiconv \-\-with\-openssl\-dir=/usr/local/Cellar/openssl@1.1/1.1.1t \-\-with\-openssl

%0Aopenssl%0A\-\-\-\-\-\-%0Abrew%20install%20openssl%401.1%0Aexport%20OPENSSL\_CFLAGS%3D%22\-I%2Fusr%2Flocal%2FCellar%2Fopenssl%401.1%2F1.1.1t%2Finclude%22%0Aexport%20OPENSSL\_LIBS%3D%22\-L%2Fusr%2Flocal%2FCellar%2Fopenssl%401.1%2F1.1.1t%2Flib%22%0Aexport%20PKG\_CONFIG\_PATH%3D%24PKG\_CONFIG\_PATH%3A%2Fusr%2Flocal%2Flib%2Fpkgconfig%0A%0A%0Aexport%20PATH%3D%2Fusr%2Flocal%2FCellar%2Fopenssl%401.1%2F1.1.1t%2Fbin%3A%24PATH%0Aexport%20LDFLAGS%3D\-L%2Fusr%2Flocal%2FCellar%2Fopenssl%401.1%2F1.1.1t%2Flib%0A%0Aoniguruma%0A\-\-\-\-%0Awget%20wget%20https%3A%2F%2Fgithub.com%2Fkkos%2Foniguruma%2Farchive%2Fv6.9.4.tar.gz%20\-O%20oniguruma\-6.9.4.tar.gz%0Atar%20\-zxvf%20oniguruma\-6.9.4.tar.gz%0Abrew%20install%20autoconf%0Abrew%20install%20libiconv%0A%0A%20%0A%E5%BC%80%E5%A7%8B%E7%BC%96%E8%AF%91%0A\-\-\-\-%0A.%2Fconfigure%20prefix%3D%2Fopt%2Fsoft%2Fphp%20\-\-with\-config\-file\-path%3D%2Fopt%2Fsoft%2Fphp%2Fetc%20\-\-with\-curl%20\-\-with\-mhash%20\-with\-gettext%20\-\-with\-pear%20\-\-with\-xmlrpc%20\-\-enable\-soap%20\-\-enable\-fpm%20%20\-\-with\-zlib%20\-\-enable\-sockets%20\-\-enable\-mbstring%20%20\-\-with\-mysqli%3Dshared%2Cmysqlnd%20\-\-enable\-pdo%20\-\-with\-pdo\-mysql%3Dshared%2Cmysqlnd%20\-\-enable\-exif%20\-\-enable\-bcmath%20\-\-enable\-debug%20\-\-enable\-pcntl%20\-\-with\-iconv%3D%2Fusr%2Flocal%2Fopt%2Flibiconv%20\-\-with\-openssl\-dir%3D%2Fusr%2Flocal%2FCellar%2Fopenssl%401.1%2F1.1.1t%20%20\-\-with\-openssl%0A%0A%0A
