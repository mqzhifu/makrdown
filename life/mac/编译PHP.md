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

