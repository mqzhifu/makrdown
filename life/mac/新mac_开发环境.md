# 新mac 开发环境

## 配置自带SHELL

\(如果直接用on my zsh ,以下：不用配置了\)

指令行提示符：

> export PS1="\[\\e\[32m\]\[\\u@\\h \\W\]$\[\\e\[m\] "

添加：ll指令\+字符集

> touch ~/.bash\_profile
> 
> 
> vim ~/.bash\_profile

```
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

export PATH=$PATH:/var/root/go/bin
```

> source ~/.bash\_profile

## 添加：创建文件的小脚本工具

touch /usr/local/bin/create\_file.sh

chmod 744 /usr/local/bin/create\_file.sh

vim /usr/local/bin/create\_file.sh

```
#/bin/bash
#出错即立刻停止，保证调用的高级语言能快速捕捉到
set -e

NEW_FILE=$1
FILE_CONTENT=$2
if [ ! -n "$1" ] ;then
        echo "file name is empty~";
        exit
fi

if [ -f "$NEW_FILE" ];then
        echo "错误：文件已存在，请不要重复创建"
        exit
fi

echo "new file: "$NEW_FILE" , ok. " ;
`touch $NEW_FILE`
#`chown root:wheel $NEW_FILE`
`chmod 744 $NEW_FILE `

if [ -n "$2" ] ;then
        echo "file content: "$FILE_CONTENT
        echo $FILE_CONTENT > $NEW_FILE
fi

```

## 创建目录

mkdir /data/golang

mkdir /data/php

mkdir /data/python

mkdir /data/cplus

mkdir /data/mysql

mkdir \-p /data/logs/golang

mkdir /data/bak

mkdir /data/cicd

mkdir \-p /data/install/zip

mkdir \-p /data/install/src

## sucureCRT

得先破解一下，它的安装包里有说明

然后，创建3个连接

192.168.1.21 （22 seed seed1234）

8.142.161.156\(6655 seedar ~\!@\#$QAZwsxedc \)

8.142.177.235\(\)

## golang

可以用brew install golang ，但是它这个版本略低，还是去官网下载吧

> https://go.dev/dl/go1.18.4.darwin\-amd64.pkg
> 
> 
> https://go.dev/dl/go1.18.4.darwin\-amd64.tar.gz

一个是自动安装包，一个是需要手动安装的包

我用的是安装包，完成后，会自动添加到env\_path 中

如果是tar包，也可以自己添加，vim /etc/profile

> export PATH=$PATH:/var/root/go/bin

自动安装包的目录位置：

> /usr/local/go

安装包的环境变量位置:

> /etc/paths.d

配置下go环境变量：

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

go env 查看下

测试下：

```
cd /data/www/golang/zgoframe
git clonet git@github.com:mqzhifu/zgoframe.git
go mod tidy
go get -u github.com/swaggo/swag/cmd/swag@v1.7.9
go install github.com/swaggo/swag/cmd/swag@v1.7.9
go run main.go
```

## goland

激活：

```
0.0.0.0 account.jetbrains.com
idea.lanyus.com
```

改下目录权限

> chown \-R wangdongyan:\_www /data/www/golang

需要配置下：GO的路径、编译执行参数等

```
偏好设置->Go->Go moudles->environment中添加：
GOPROXY=https://goproxy.cn;GO111MODULE=on

偏好设置->Go->Go Root ，/usr/local/go

```

设置好后，到项目中的.mod 文件中，下载试试

引入配置文件

添加插件：

toml

vue

## redis

创建目录

```
mkdir /soft/redis
mkdir /soft/redis/run
mkdir /soft/redis/data
```

> brew install pkg\-config

brew与item好像共用zsh，brew因为是非root使用，而item2我每次是root进去，这两用户的权限冲突

> sudo chown \-R $\(whoami\) /usr/local/share/doc /usr/local/share/man /usr/local/share/man/man1 /usr/local/share/zsh /usr/local/share/zsh/site\-functions

之后root现变一下权限就行了

> chown \-R $\(whoami\) /usr/local/share/zsh

```
wget https://download.redis.io/redis-stable.tar.gz
tar -zxvf redis-stable.tar.gz
cd redis-stable
vim README.MD
make 
make install    #默认是安装在:/usr/local/bin下
make install PREFIX=/soft/redis #这里也可以加上前缀，所有的redis指令就会被编译到指定目录
```

启动，测试一下

> /soft/redis/bin/redis\-server

```
cp /data/install/src/redis-stable/redis.conf /soft/redis
vim /soft/redis/redis.conf
```

配置主要修改：

```
port 6370
daemonize yes
requirepass 1234567890
pidfile /soft/redis/run/redis_6379.pid  
dir /soft/redis/data  
logfile "/soft/redis/log/redis.log"  
```

启动，测试一下

> /soft/redis/bin/redis\-server /soft/redis/redis.conf

用cli 测试一下

> /soft/redis/bin/redis\-cli \-p 6370

## NGINX

发现还有个鬼东西

xcode\-select \-\-install

好像安了这个鬼东西后，能用GCC

然而 ，失败，上APP STORE 发现 X\-CODE可以随便安装，管他呢，先安一个，然而 11G ，真可怕 ！先下着吧，处理其它 的吧

## apache

> 下载官方包再编译安装失败，用mac系统里自带的吧

vim /etc/apache2/httpd.conf

添加如下：

```
DocumentRoot "/data/www"
<Directory "/data/www">
```

apache \-k restart

127.0.0.1

试了下OK

apache整合PHP

找如下代码，打开注释，加载PHP模块

> LoadModule php7\_module libexec/apache2/libphp7.so

测试一下是否成功

cd /data/www

touch a.php

chmod 777 a.php

vim a.php

```
<?php
phpinfo();
```

> 127.0.0.1/info.php

虚拟目录

```
<VirtualHost *:80>
ServerName local.test.com
DocumentRoot /data/www
</VirtualHost>

```

增加本地HOST映射

vim /etc/hosts

> 127.0.0.1 local.static.com local.api.com local.test.com local.admin.com local.agent.com local.mqadmin.com local.house.com

配置URL重定向

```
vim /etc/apache2/httpd.conf
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
apachectl -k restart

AllowOverride None
修改
AllowOverride All
```

## mysql

UI安装法

> https://dev.mysql.com/doc/mysql\-osx\-excerpt/8.0/en/osx\-installation\-pkg.html

下载包

> https://dev.mysql.com/downloads/mysql/

```
tar -zxvf mysql
mv mysql /soft/mysql

mkdir /data/mysql
mkdir /data/mysql/master_3306
mkdir /data/mysql/init_db
chown -R _mysql:_mysql /data/mysql
chown -R _mysql:_mysql /soft/mysql

cd /soft/mysql/bin
./mysqld --initialize --user=_mysql --basedir=/soft/mysql --datadir=/data/mysql/init_db

cp /data/mysql/init_db/* /data/mysql/master_3306
chown -R _mysql:_mysql /data/mysql/master_3306

```

配置文件

> touch /etc/my.cnf
> 
> 
> vim /etc/my.cnf

启动:

> /soft/mysql/bin/mysqld\_safe &

登陆：

> mysql \-uroot \-p 临时密码

新版本的密码加密格式变了，会导致MYSQL客户端连不上，改回到旧版的

> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql\_native\_password;

修改下新的密码

> alter user 'root'@'localhost' identified by 'mqzhifu';

mysql快速关闭的工具

```
create_file.sh /usr/local/bin/stop_mysql.sh '/soft/mysql/bin/mysqladmin -uroot -pmqzhifu shutdown'
```

mysql快速启动的工具

```
create_file.sh /usr/local/bin/start_mysql.sh '/soft/mysql/bin/mysqld_safe &'
```

mysql快速登陆的工具

```
create_file.sh /usr/local/bin/login_mysql.sh '/soft/mysql/bin/mysql -uroot -pmqzhifu seed_test'
```

## php\-redis扩展

```
wget http://pecl.php.net/get/redis-5.3.2.tgz  
tar redis-5.3.2.tgz  
cd ../src_bag/redis-5.3.2
```

phpize

发现少autoconf

brew autoconf install

重新phpize

报错：/usr/include/php/main/php.h

一直报这个错误，实在没办法了，试下安装xcode指令行工具

xcode\-select \-\-install

不能安装该软件，因为当前无法从软件更新服务器获得

这个狗东西，安装不了，只能去网站下载了

> https://developer.apple.com/download/more/

改一下系统头库

```
cd /usr
sudo ln -s /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/usr/include /usr/include
```

失败,换个方法尝试

./confiere ，不加 php\-config

OK了

make & make install

php扩展默认生成目录

> /usr/lib/php/extensions/no\-debug\-non\-zts\-20180731/

添加php redis扩展

```
chmod 744 /etc/php.ini
vim /etc/php.ini
extension=redis.so
apachectl -k restart
```

试一下，是否成功

> local.test.com/info.php

OK,算是MAC下,第一次安装php扩展，好麻烦

## zip扩展

cd /install/tar\_bag

wget http://pecl.php.net/get/zip\-1.19.1.tgz

tar \-zxvf zip\-1.19.1.tgz

mv zip\-1.19.1 ../src\_bag/

cd ../src\_bag/zip\-1.19.1/

phpize

Please reinstall the libzip distribution

wget https://libzip.org/download/libzip\-1.5.2.tar.gz

没速度 ，NND，先不浪费 时间 了，修改PHP，跳过这个扩展

## JDK

先到官网下一个mac 版的JDK 包 dmg

一路下一步安装完成

```
cd ~
touch .bash_profile
vim
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-15.0.1.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPAT

source ~/.bash_profile
```

## jmeter

http://jmeter.apache.org/download\_jmeter.cgi

```
vim ~/.bash_profile
export JMETER_HOME=/Users/wangdongyan/Downloads/apache-jmeter-5.3
export PATH=$PATH:$JMETER_HOME/bin
```

source ~/.bash\_profile

%E9%85%8D%E7%BD%AE%E8%87%AA%E5%B8%A6SHELL%0A\-\-\-\-\-\-%0A\(%E5%A6%82%E6%9E%9C%E7%9B%B4%E6%8E%A5%E7%94%A8on%20my%20zsh%20%2C%E4%BB%A5%E4%B8%8B%EF%BC%9A%E4%B8%8D%E7%94%A8%E9%85%8D%E7%BD%AE%E4%BA%86\)%0A%0A%E6%8C%87%E4%BB%A4%E8%A1%8C%E6%8F%90%E7%A4%BA%E7%AC%A6%EF%BC%9A%0A%3Eexport%20PS1%3D%22%5C%5B%5Ce%5B32m%5C%5D%5B%5Cu%40%5Ch%20%5CW%5D%24%5C%5B%5Ce%5Bm%5C%5D%20%22%0A%0A%E6%B7%BB%E5%8A%A0%EF%BC%9All%E6%8C%87%E4%BB%A4%2B%E5%AD%97%E7%AC%A6%E9%9B%86%0A%3Etouch%20~%2F.bash\_profile%0A%3Evim%20~%2F.bash\_profile%0A%60%60%60%0Aexport%20LANG%3D%22zh\_CN.UTF\-8%22%0Aexport%20LC\_ALL%3D%22zh\_CN.UTF\-8%22%0A%0Aalias%20ll%3D'ls%20\-alF'%0Aalias%20la%3D'ls%20\-A'%0Aalias%20l%3D'ls%20\-CF'%0A%0Aexport%20PATH%3D%24PATH%3A%2Fvar%2Froot%2Fgo%2Fbin%0A%60%60%60%0A%0A%3Esource%20~%2F.bash\_profile%0A%0A%E6%B7%BB%E5%8A%A0%EF%BC%9A%E5%88%9B%E5%BB%BA%E6%96%87%E4%BB%B6%E7%9A%84%E5%B0%8F%E8%84%9A%E6%9C%AC%E5%B7%A5%E5%85%B7%0A\-\-\-\-%0Atouch%20%2Fusr%2Flocal%2Fbin%2Fcreate\_file.sh%0Achmod%20744%20%2Fusr%2Flocal%2Fbin%2Fcreate\_file.sh%0Avim%20%2Fusr%2Flocal%2Fbin%2Fcreate\_file.sh%0A%60%60%60%0A%23%2Fbin%2Fbash%0A%23%E5%87%BA%E9%94%99%E5%8D%B3%E7%AB%8B%E5%88%BB%E5%81%9C%E6%AD%A2%EF%BC%8C%E4%BF%9D%E8%AF%81%E8%B0%83%E7%94%A8%E7%9A%84%E9%AB%98%E7%BA%A7%E8%AF%AD%E8%A8%80%E8%83%BD%E5%BF%AB%E9%80%9F%E6%8D%95%E6%8D%89%E5%88%B0%0Aset%20\-e%0A%0ANEW\_FILE%3D%241%0AFILE\_CONTENT%3D%242%0Aif%20%5B%20\!%20\-n%20%22%241%22%20%5D%20%3Bthen%0A%20%20%20%20%20%20%20%20echo%20%22file%20name%20is%20empty~%22%3B%0A%20%20%20%20%20%20%20%20exit%0Afi%0A%0Aif%20%5B%20\-f%20%22%24NEW\_FILE%22%20%5D%3Bthen%0A%20%20%20%20%20%20%20%20echo%20%22%E9%94%99%E8%AF%AF%EF%BC%9A%E6%96%87%E4%BB%B6%E5%B7%B2%E5%AD%98%E5%9C%A8%EF%BC%8C%E8%AF%B7%E4%B8%8D%E8%A6%81%E9%87%8D%E5%A4%8D%E5%88%9B%E5%BB%BA%22%0A%20%20%20%20%20%20%20%20exit%0Afi%0A%0Aecho%20%22new%20file%3A%20%22%24NEW\_FILE%22%20%2C%20ok.%20%22%20%3B%0A%60touch%20%24NEW\_FILE%60%0A%23%60chown%20root%3Awheel%20%24NEW\_FILE%60%0A%60chmod%20744%20%24NEW\_FILE%20%60%0A%0Aif%20%5B%20\-n%20%22%242%22%20%5D%20%3Bthen%0A%20%20%20%20%20%20%20%20echo%20%22file%20content%3A%20%22%24FILE\_CONTENT%0A%20%20%20%20%20%20%20%20echo%20%24FILE\_CONTENT%20%3E%20%24NEW\_FILE%0Afi%0A%0A%60%60%60%0A%0A%E5%88%9B%E5%BB%BA%E7%9B%AE%E5%BD%95%0A\-\-\-\-%0Amkdir%20%2Fdata%2Fgolang%0Amkdir%20%2Fdata%2Fphp%0Amkdir%20%2Fdata%2Fpython%0Amkdir%20%2Fdata%2Fcplus%0A%0Amkdir%20%2Fdata%2Fmysql%0Amkdir%20\-p%20%2Fdata%2Flogs%2Fgolang%0Amkdir%20%2Fdata%2Fbak%0Amkdir%20%2Fdata%2Fcicd%0A%0Amkdir%20\-p%20%2Fdata%2Finstall%2Fzip%0Amkdir%20\-p%20%2Fdata%2Finstall%2Fsrc%0A%0A%0AsucureCRT%0A\-\-\-\-%0A%E5%BE%97%E5%85%88%E7%A0%B4%E8%A7%A3%E4%B8%80%E4%B8%8B%EF%BC%8C%E5%AE%83%E7%9A%84%E5%AE%89%E8%A3%85%E5%8C%85%E9%87%8C%E6%9C%89%E8%AF%B4%E6%98%8E%0A%E7%84%B6%E5%90%8E%EF%BC%8C%E5%88%9B%E5%BB%BA3%E4%B8%AA%E8%BF%9E%E6%8E%A5%0A192.168.1.21%20%EF%BC%8822%20seed%20seed1234%EF%BC%89%0A8.142.161.156\(6655%20seedar%20~\!%40%23%24QAZwsxedc%20\)%0A8.142.177.235\(22%20root\)%0A%0Agolang%0A\-\-\-\-%0A%E5%8F%AF%E4%BB%A5%E7%94%A8brew%20install%20golang%20%EF%BC%8C%E4%BD%86%E6%98%AF%E5%AE%83%E8%BF%99%E4%B8%AA%E7%89%88%E6%9C%AC%E7%95%A5%E4%BD%8E%EF%BC%8C%E8%BF%98%E6%98%AF%E5%8E%BB%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E5%90%A7%0A%3Ehttps%3A%2F%2Fgo.dev%2Fdl%2Fgo1.18.4.darwin\-amd64.pkg%0A%3Ehttps%3A%2F%2Fgo.dev%2Fdl%2Fgo1.18.4.darwin\-amd64.tar.gz%0A%0A%E4%B8%80%E4%B8%AA%E6%98%AF%E8%87%AA%E5%8A%A8%E5%AE%89%E8%A3%85%E5%8C%85%EF%BC%8C%E4%B8%80%E4%B8%AA%E6%98%AF%E9%9C%80%E8%A6%81%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85%E7%9A%84%E5%8C%85%0A%E6%88%91%E7%94%A8%E7%9A%84%E6%98%AF%E5%AE%89%E8%A3%85%E5%8C%85%EF%BC%8C%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%8A%A8%E6%B7%BB%E5%8A%A0%E5%88%B0env\_path%20%E4%B8%AD%0A%E5%A6%82%E6%9E%9C%E6%98%AFtar%E5%8C%85%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%B7%B1%E6%B7%BB%E5%8A%A0%EF%BC%8Cvim%20%2Fetc%2Fprofile%0A%3Eexport%20PATH%3D%24PATH%3A%2Fvar%2Froot%2Fgo%2Fbin%0A%0A%E8%87%AA%E5%8A%A8%E5%AE%89%E8%A3%85%E5%8C%85%E7%9A%84%E7%9B%AE%E5%BD%95%E4%BD%8D%E7%BD%AE%EF%BC%9A%0A%3E%2Fusr%2Flocal%2Fgo%0A%0A%E5%AE%89%E8%A3%85%E5%8C%85%E7%9A%84%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E4%BD%8D%E7%BD%AE%3A%0A%3E%2Fetc%2Fpaths.d%0A%0A%E9%85%8D%E7%BD%AE%E4%B8%8Bgo%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%EF%BC%9A%0A%0A%60%60%60%0Ago%20env%20\-w%20GO111MODULE%3Don%0Ago%20env%20\-w%20GOPROXY%3Dhttps%3A%2F%2Fgoproxy.cn%2Cdirect%0A%60%60%60%0A%0Ago%20env%20%E6%9F%A5%E7%9C%8B%E4%B8%8B%0A%0A%E6%B5%8B%E8%AF%95%E4%B8%8B%EF%BC%9A%0A%60%60%60%0Acd%20%2Fdata%2Fwww%2Fgolang%2Fzgoframe%0Agit%20clonet%20git%40github.com%3Amqzhifu%2Fzgoframe.git%0Ago%20mod%20tidy%0Ago%20get%20\-u%20github.com%2Fswaggo%2Fswag%2Fcmd%2Fswag%40v1.7.9%0Ago%20install%20github.com%2Fswaggo%2Fswag%2Fcmd%2Fswag%40v1.7.9%0Ago%20run%20main.go%0A%60%60%60%0A%0Agoland%0A\-\-\-%0A%E6%BF%80%E6%B4%BB%EF%BC%9A%0A%60%60%60%0A0.0.0.0%20account.jetbrains.com%0Aidea.lanyus.com%0A%60%60%60%0A%0A%E6%94%B9%E4%B8%8B%E7%9B%AE%E5%BD%95%E6%9D%83%E9%99%90%0A%3Echown%20\-R%20wangdongyan%3A\_www%20%2Fdata%2Fwww%2Fgolang%0A%0A%0A%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AE%E4%B8%8B%EF%BC%9AGO%E7%9A%84%E8%B7%AF%E5%BE%84%E3%80%81%E7%BC%96%E8%AF%91%E6%89%A7%E8%A1%8C%E5%8F%82%E6%95%B0%E7%AD%89%0A%60%60%60%0A%E5%81%8F%E5%A5%BD%E8%AE%BE%E7%BD%AE\-%3EGo\-%3EGo%20moudles\-%3Eenvironment%E4%B8%AD%E6%B7%BB%E5%8A%A0%EF%BC%9A%0AGOPROXY%3Dhttps%3A%2F%2Fgoproxy.cn%3BGO111MODULE%3Don%0A%0A%E5%81%8F%E5%A5%BD%E8%AE%BE%E7%BD%AE\-%3EGo\-%3EGo%20Root%20%EF%BC%8C%2Fusr%2Flocal%2Fgo%0A%0A%60%60%60%0A%E8%AE%BE%E7%BD%AE%E5%A5%BD%E5%90%8E%EF%BC%8C%E5%88%B0%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84.mod%20%E6%96%87%E4%BB%B6%E4%B8%AD%EF%BC%8C%E4%B8%8B%E8%BD%BD%E8%AF%95%E8%AF%95%0A%0A%E5%BC%95%E5%85%A5%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%20%20%0A%0A%E6%B7%BB%E5%8A%A0%E6%8F%92%E4%BB%B6%EF%BC%9A%0Atoml%0Avue%0A%0A%0A%0A%0A%0Aredis%0A\-\-\-\-\-\-%0A%E5%88%9B%E5%BB%BA%E7%9B%AE%E5%BD%95%0A%60%60%60%0Amkdir%20%2Fsoft%2Fredis%0Amkdir%20%2Fsoft%2Fredis%2Frun%0Amkdir%20%2Fsoft%2Fredis%2Fdata%0A%60%60%60%0A%0A%3Ebrew%20install%20pkg\-config%0A%0Abrew%E4%B8%8Eitem%E5%A5%BD%E5%83%8F%E5%85%B1%E7%94%A8zsh%EF%BC%8Cbrew%E5%9B%A0%E4%B8%BA%E6%98%AF%E9%9D%9Eroot%E4%BD%BF%E7%94%A8%EF%BC%8C%E8%80%8Citem2%E6%88%91%E6%AF%8F%E6%AC%A1%E6%98%AFroot%E8%BF%9B%E5%8E%BB%EF%BC%8C%E8%BF%99%E4%B8%A4%E7%94%A8%E6%88%B7%E7%9A%84%E6%9D%83%E9%99%90%E5%86%B2%E7%AA%81%0A%3Esudo%20chown%20\-R%20%24\(whoami\)%20%2Fusr%2Flocal%2Fshare%2Fdoc%20%2Fusr%2Flocal%2Fshare%2Fman%20%2Fusr%2Flocal%2Fshare%2Fman%2Fman1%20%2Fusr%2Flocal%2Fshare%2Fzsh%20%2Fusr%2Flocal%2Fshare%2Fzsh%2Fsite\-functions%0A%0A%E4%B9%8B%E5%90%8Eroot%E7%8E%B0%E5%8F%98%E4%B8%80%E4%B8%8B%E6%9D%83%E9%99%90%E5%B0%B1%E8%A1%8C%E4%BA%86%0A%3Echown%20\-R%20%24\(whoami\)%20%2Fusr%2Flocal%2Fshare%2Fzsh%0A%0A%60%60%60%0Awget%20https%3A%2F%2Fdownload.redis.io%2Fredis\-stable.tar.gz%0Atar%20\-zxvf%20redis\-stable.tar.gz%0Acd%20redis\-stable%0Avim%20README.MD%0Amake%20%0Amake%20install%20%20%20%20%23%E9%BB%98%E8%AE%A4%E6%98%AF%E5%AE%89%E8%A3%85%E5%9C%A8%3A%2Fusr%2Flocal%2Fbin%E4%B8%8B%0Amake%20install%20PREFIX%3D%2Fsoft%2Fredis%20%23%E8%BF%99%E9%87%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E5%8A%A0%E4%B8%8A%E5%89%8D%E7%BC%80%EF%BC%8C%E6%89%80%E6%9C%89%E7%9A%84redis%E6%8C%87%E4%BB%A4%E5%B0%B1%E4%BC%9A%E8%A2%AB%E7%BC%96%E8%AF%91%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9B%AE%E5%BD%95%0A%60%60%60%0A%E5%90%AF%E5%8A%A8%EF%BC%8C%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%3E%2Fsoft%2Fredis%2Fbin%2Fredis\-server%0A%0A%60%60%60%0Acp%20%2Fdata%2Finstall%2Fsrc%2Fredis\-stable%2Fredis.conf%20%2Fsoft%2Fredis%0Avim%20%2Fsoft%2Fredis%2Fredis.conf%0A%60%60%60%0A%E9%85%8D%E7%BD%AE%E4%B8%BB%E8%A6%81%E4%BF%AE%E6%94%B9%EF%BC%9A%0A%60%60%60%0Aport%206370%0Adaemonize%20yes%0Arequirepass%201234567890%0Apidfile%20%2Fsoft%2Fredis%2Frun%2Fredis\_6379.pid%20%20%0Adir%20%2Fsoft%2Fredis%2Fdata%20%20%0Alogfile%20%22%2Fsoft%2Fredis%2Flog%2Fredis.log%22%20%20%0A%60%60%60%0A%0A%E5%90%AF%E5%8A%A8%EF%BC%8C%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%3E%2Fsoft%2Fredis%2Fbin%2Fredis\-server%20%2Fsoft%2Fredis%2Fredis.conf%0A%0A%E7%94%A8cli%20%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%3E%2Fsoft%2Fredis%2Fbin%2Fredis\-cli%20\-p%206370%0A%0ANGINX%0A\-\-\-\-\-\-\-\-\-\-\-%0A%E5%8F%91%E7%8E%B0%E8%BF%98%E6%9C%89%E4%B8%AA%E9%AC%BC%E4%B8%9C%E8%A5%BF%20%20%0Axcode\-select%20\-\-install%20%20%20%0A%E5%A5%BD%E5%83%8F%E5%AE%89%E4%BA%86%E8%BF%99%E4%B8%AA%E9%AC%BC%E4%B8%9C%E8%A5%BF%E5%90%8E%EF%BC%8C%E8%83%BD%E7%94%A8GCC%20%20%0A%E7%84%B6%E8%80%8C%20%EF%BC%8C%E5%A4%B1%E8%B4%A5%EF%BC%8C%E4%B8%8AAPP%20STORE%20%E5%8F%91%E7%8E%B0%20%20%20X\-CODE%E5%8F%AF%E4%BB%A5%E9%9A%8F%E4%BE%BF%E5%AE%89%E8%A3%85%EF%BC%8C%E7%AE%A1%E4%BB%96%E5%91%A2%EF%BC%8C%E5%85%88%E5%AE%89%E4%B8%80%E4%B8%AA%EF%BC%8C%E7%84%B6%E8%80%8C%2011G%20%EF%BC%8C%E7%9C%9F%E5%8F%AF%E6%80%95%20%EF%BC%81%E5%85%88%E4%B8%8B%E7%9D%80%E5%90%A7%EF%BC%8C%E5%A4%84%E7%90%86%E5%85%B6%E5%AE%83%20%E7%9A%84%E5%90%A7%20%20%0A%0A%0Aapache%0A\-\-\-\-\-%0A%3E%E4%B8%8B%E8%BD%BD%E5%AE%98%E6%96%B9%E5%8C%85%E5%86%8D%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E5%A4%B1%E8%B4%A5%EF%BC%8C%E7%94%A8mac%E7%B3%BB%E7%BB%9F%E9%87%8C%E8%87%AA%E5%B8%A6%E7%9A%84%E5%90%A7%0A%0Avim%20%2Fetc%2Fapache2%2Fhttpd.conf%20%20%0A%E6%B7%BB%E5%8A%A0%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%60%60%60%0ADocumentRoot%20%22%2Fdata%2Fwww%22%0A%3CDirectory%20%22%2Fdata%2Fwww%22%3E%0A%60%60%60%0A%0Aapache%20\-k%20restart%20%20%0A127.0.0.1%20%20%0A%E8%AF%95%E4%BA%86%E4%B8%8BOK%20%20%0A%0Aapache%E6%95%B4%E5%90%88PHP%20%20%0A%E6%89%BE%E5%A6%82%E4%B8%8B%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%89%93%E5%BC%80%E6%B3%A8%E9%87%8A%EF%BC%8C%E5%8A%A0%E8%BD%BDPHP%E6%A8%A1%E5%9D%97%20%20%0A%3ELoadModule%20php7\_module%20libexec%2Fapache2%2Flibphp7.so%0A%0A%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%E6%98%AF%E5%90%A6%E6%88%90%E5%8A%9F%20%20%0A%0Acd%20%2Fdata%2Fwww%0Atouch%20a.php%0Achmod%20777%20a.php%0Avim%20a.php%0A%60%60%60%0A%3C%3Fphp%0Aphpinfo\(\)%3B%0A%60%60%60%0A%0A%3E127.0.0.1%2Finfo.php%0A%0A%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%95%0A%60%60%60%0A%3CVirtualHost%20\*%3A80%3E%0AServerName%20local.test.com%0ADocumentRoot%20%2Fdata%2Fwww%0A%3C%2FVirtualHost%3E%0A%0A%60%60%60%0A%E5%A2%9E%E5%8A%A0%E6%9C%AC%E5%9C%B0HOST%E6%98%A0%E5%B0%84%20%20%0Avim%20%2Fetc%2Fhosts%0A%3E127.0.0.1%20local.static.com%20local.api.com%20local.test.com%20local.admin.com%20local.agent.com%20local.mqadmin.com%20local.house.com%0A%0A%E9%85%8D%E7%BD%AEURL%E9%87%8D%E5%AE%9A%E5%90%91%0A%60%60%60%0Avim%20%2Fetc%2Fapache2%2Fhttpd.conf%0ALoadModule%20rewrite\_module%20libexec%2Fapache2%2Fmod\_rewrite.so%0Aapachectl%20\-k%20restart%0A%0AAllowOverride%20None%0A%E4%BF%AE%E6%94%B9%0AAllowOverride%20All%0A%60%60%60%0A%0Amysql%0A\-\-\-%0AUI%E5%AE%89%E8%A3%85%E6%B3%95%0A%0A%3Ehttps%3A%2F%2Fdev.mysql.com%2Fdoc%2Fmysql\-osx\-excerpt%2F8.0%2Fen%2Fosx\-installation\-pkg.html%0A%0A%E4%B8%8B%E8%BD%BD%E5%8C%85%20%20%0A%3Ehttps%3A%2F%2Fdev.mysql.com%2Fdownloads%2Fmysql%2F%0A%60%60%60%0Atar%20\-zxvf%20mysql%0Amv%20mysql%20%2Fsoft%2Fmysql%0A%0Amkdir%20%2Fdata%2Fmysql%0Amkdir%20%2Fdata%2Fmysql%2Fmaster\_3306%0Amkdir%20%2Fdata%2Fmysql%2Finit\_db%0Achown%20\-R%20\_mysql%3A\_mysql%20%2Fdata%2Fmysql%0Achown%20\-R%20\_mysql%3A\_mysql%20%2Fsoft%2Fmysql%0A%0Acd%20%2Fsoft%2Fmysql%2Fbin%0A.%2Fmysqld%20\-\-initialize%20\-\-user%3D\_mysql%20\-\-basedir%3D%2Fsoft%2Fmysql%20\-\-datadir%3D%2Fdata%2Fmysql%2Finit\_db%0A%0Acp%20%2Fdata%2Fmysql%2Finit\_db%2F\*%20%2Fdata%2Fmysql%2Fmaster\_3306%0Achown%20\-R%20\_mysql%3A\_mysql%20%2Fdata%2Fmysql%2Fmaster\_3306%0A%0A%0A%60%60%60%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%20%0A%3Etouch%20%2Fetc%2Fmy.cnf%20%20%0Avim%20%2Fetc%2Fmy.cnf%20%20%0A%0A%E5%90%AF%E5%8A%A8%3A%0A%3E%2Fsoft%2Fmysql%2Fbin%2Fmysqld\_safe%20%26%0A%0A%E7%99%BB%E9%99%86%EF%BC%9A%0A%3Emysql%20\-uroot%20\-p%20%E4%B8%B4%E6%97%B6%E5%AF%86%E7%A0%81%0A%0A%E6%96%B0%E7%89%88%E6%9C%AC%E7%9A%84%E5%AF%86%E7%A0%81%E5%8A%A0%E5%AF%86%E6%A0%BC%E5%BC%8F%E5%8F%98%E4%BA%86%EF%BC%8C%E4%BC%9A%E5%AF%BC%E8%87%B4MYSQL%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BF%9E%E4%B8%8D%E4%B8%8A%EF%BC%8C%E6%94%B9%E5%9B%9E%E5%88%B0%E6%97%A7%E7%89%88%E7%9A%84%0A%3EALTER%20USER%20'root'%40'localhost'%20IDENTIFIED%20WITH%20mysql\_native\_password%3B%0A%0A%E4%BF%AE%E6%94%B9%E4%B8%8B%E6%96%B0%E7%9A%84%E5%AF%86%E7%A0%81%0A%3Ealter%20user%20'root'%40'localhost'%20identified%20by%20'mqzhifu'%3B%0A%0Amysql%E5%BF%AB%E9%80%9F%E5%85%B3%E9%97%AD%E7%9A%84%E5%B7%A5%E5%85%B7%0A%60%60%60%0Acreate\_file.sh%20%2Fusr%2Flocal%2Fbin%2Fstop\_mysql.sh%20'%2Fsoft%2Fmysql%2Fbin%2Fmysqladmin%20\-uroot%20\-pmqzhifu%20shutdown'%0A%60%60%60%0Amysql%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8%E7%9A%84%E5%B7%A5%E5%85%B7%0A%60%60%60%0Acreate\_file.sh%20%2Fusr%2Flocal%2Fbin%2Fstart\_mysql.sh%20'%2Fsoft%2Fmysql%2Fbin%2Fmysqld\_safe%20%26'%0A%60%60%60%0Amysql%E5%BF%AB%E9%80%9F%E7%99%BB%E9%99%86%E7%9A%84%E5%B7%A5%E5%85%B7%0A%60%60%60%0Acreate\_file.sh%20%2Fusr%2Flocal%2Fbin%2Flogin\_mysql.sh%20'%2Fsoft%2Fmysql%2Fbin%2Fmysql%20\-uroot%20\-pmqzhifu%20seed\_test'%0A%60%60%60%0A%0A%0A%0A%0A%0A%0Aphp\-redis%E6%89%A9%E5%B1%95%0A\-\-\-%0A%60%60%60%0Awget%20http%3A%2F%2Fpecl.php.net%2Fget%2Fredis\-5.3.2.tgz%20%20%0Atar%20redis\-5.3.2.tgz%20%20%0Acd%20..%2Fsrc\_bag%2Fredis\-5.3.2%0A%60%60%60%0A%0Aphpize%20%20%0A%E5%8F%91%E7%8E%B0%E5%B0%91autoconf%20%0A%0Abrew%20autoconf%20install%20%20%0A%0A%E9%87%8D%E6%96%B0phpize%20%20%0A%0A%E6%8A%A5%E9%94%99%EF%BC%9A%2Fusr%2Finclude%2Fphp%2Fmain%2Fphp.h%20%20%0A%0A%E4%B8%80%E7%9B%B4%E6%8A%A5%E8%BF%99%E4%B8%AA%E9%94%99%E8%AF%AF%EF%BC%8C%E5%AE%9E%E5%9C%A8%E6%B2%A1%E5%8A%9E%E6%B3%95%E4%BA%86%EF%BC%8C%E8%AF%95%E4%B8%8B%E5%AE%89%E8%A3%85xcode%E6%8C%87%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7%20%20%0Axcode\-select%20\-\-install%20%20%0A%E4%B8%8D%E8%83%BD%E5%AE%89%E8%A3%85%E8%AF%A5%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%BD%93%E5%89%8D%E6%97%A0%E6%B3%95%E4%BB%8E%E8%BD%AF%E4%BB%B6%E6%9B%B4%E6%96%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%8E%B7%E5%BE%97%20%20%0A%E8%BF%99%E4%B8%AA%E7%8B%97%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%AE%89%E8%A3%85%E4%B8%8D%E4%BA%86%EF%BC%8C%E5%8F%AA%E8%83%BD%E5%8E%BB%E7%BD%91%E7%AB%99%E4%B8%8B%E8%BD%BD%E4%BA%86%20%20%0A%3Ehttps%3A%2F%2Fdeveloper.apple.com%2Fdownload%2Fmore%2F%20%20%0A%0A%E6%94%B9%E4%B8%80%E4%B8%8B%E7%B3%BB%E7%BB%9F%E5%A4%B4%E5%BA%93%0A%60%60%60%0Acd%20%2Fusr%0Asudo%20ln%20\-s%20%2FApplications%2FXcode.app%2FContents%2FDeveloper%2FPlatforms%2FMacOSX.platform%2FDeveloper%2FSDKs%2FMacOSX10.15.sdk%2Fusr%2Finclude%20%2Fusr%2Finclude%0A%60%60%60%0A%0A%E5%A4%B1%E8%B4%A5%2C%E6%8D%A2%E4%B8%AA%E6%96%B9%E6%B3%95%E5%B0%9D%E8%AF%95%20%20%0A.%2Fconfiere%20%20%EF%BC%8C%E4%B8%8D%E5%8A%A0%20php\-config%0AOK%E4%BA%86%0Amake%20%20%26%20make%20install%20%20%0A%0Aphp%E6%89%A9%E5%B1%95%E9%BB%98%E8%AE%A4%E7%94%9F%E6%88%90%E7%9B%AE%E5%BD%95%20%20%0A%3E%2Fusr%2Flib%2Fphp%2Fextensions%2Fno\-debug\-non\-zts\-20180731%2F%0A%0A%E6%B7%BB%E5%8A%A0php%20redis%E6%89%A9%E5%B1%95%20%20%0A%60%60%60%0Achmod%20744%20%2Fetc%2Fphp.ini%0Avim%20%2Fetc%2Fphp.ini%0Aextension%3Dredis.so%0Aapachectl%20\-k%20restart%0A%60%60%60%0A%0A%E8%AF%95%E4%B8%80%E4%B8%8B%EF%BC%8C%E6%98%AF%E5%90%A6%E6%88%90%E5%8A%9F%20%20%0A%3Elocal.test.com%2Finfo.php%0A%0AOK%2C%E7%AE%97%E6%98%AFMAC%E4%B8%8B%2C%E7%AC%AC%E4%B8%80%E6%AC%A1%E5%AE%89%E8%A3%85php%E6%89%A9%E5%B1%95%EF%BC%8C%E5%A5%BD%E9%BA%BB%E7%83%A6%20%0A%0Azip%E6%89%A9%E5%B1%95%0A\-\-\-\-%0Acd%20%2Finstall%2Ftar\_bag%0Awget%20http%3A%2F%2Fpecl.php.net%2Fget%2Fzip\-1.19.1.tgz%0A%0Atar%20\-zxvf%20zip\-1.19.1.tgz%0Amv%20zip\-1.19.1%20..%2Fsrc\_bag%2F%0Acd%20..%2Fsrc\_bag%2Fzip\-1.19.1%2F%0Aphpize%0A%0A%0APlease%20reinstall%20the%20libzip%20distribution%0Awget%20https%3A%2F%2Flibzip.org%2Fdownload%2Flibzip\-1.5.2.tar.gz%0A%E6%B2%A1%E9%80%9F%E5%BA%A6%20%EF%BC%8CNND%EF%BC%8C%E5%85%88%E4%B8%8D%E6%B5%AA%E8%B4%B9%20%E6%97%B6%E9%97%B4%20%E4%BA%86%EF%BC%8C%E4%BF%AE%E6%94%B9PHP%EF%BC%8C%E8%B7%B3%E8%BF%87%E8%BF%99%E4%B8%AA%E6%89%A9%E5%B1%95%0A%0AJDK%0A\-\-\-\-\-%0A%E5%85%88%E5%88%B0%E5%AE%98%E7%BD%91%E4%B8%8B%E4%B8%80%E4%B8%AAmac%20%E7%89%88%E7%9A%84JDK%20%E5%8C%85%20%20dmg%0A%E4%B8%80%E8%B7%AF%E4%B8%8B%E4%B8%80%E6%AD%A5%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90%20%0A%60%60%60%0Acd%20~%0Atouch%20.bash\_profile%0Avim%0AJAVA\_HOME%3D%2FLibrary%2FJava%2FJavaVirtualMachines%2Fjdk\-15.0.1.jdk%2FContents%2FHome%0APATH%3D%24JAVA\_HOME%2Fbin%3A%24PATH%3A.%0ACLASSPATH%3D%24JAVA\_HOME%2Flib%2Ftools.jar%3A%24JAVA\_HOME%2Flib%2Fdt.jar%3A.%0Aexport%20JAVA\_HOME%0Aexport%20PATH%0Aexport%20CLASSPAT%0A%0Asource%20~%2F.bash\_profile%0A%60%60%60%0A%0Ajmeter%0A\-\-\-\-\-%0Ahttp%3A%2F%2Fjmeter.apache.org%2Fdownload\_jmeter.cgi%0A%60%60%60%0Avim%20~%2F.bash\_profile%0Aexport%20JMETER\_HOME%3D%2FUsers%2Fwangdongyan%2FDownloads%2Fapache\-jmeter\-5.3%0Aexport%20PATH%3D%24PATH%3A%24JMETER\_HOME%2Fbin%0A%60%60%60%0Asource%20~%2F.bash\_profile%20%20%0A%0A%0A%0A%0A%0A
