# rsync

## rsync

```
touch /soft/rsync/rsyncd.conf  
vim /soft/rsync/rsyncd.conf
```

```
port = 8877
log file = /data/logs/rsync/rsyncd.log
pid file = /var/rsyncd.pid
max connections = 5
strict modes = no

[install]
uid = rsync
gid = rsync
path = /install/zip
read only = no
write only = no   
hosts allow = *
hosts deny = 10.5.3.77
```

服务端\-启动

> rsync \-\-daemon \-\-config=/soft/rsync/rsyncd.conf

设置文件夹权限：

> chown \-R rsync:rsync /install

阿里云开启8877端口，本机开始同步

客户端\-测试，读取下文件列表：

> rsync \-\-port=8877 \-\-list\-only rsync@8.142.161.156::install

客户端\-测试:

> rsync \-\-port=8877 \-avz \-\-progress /data/new\_centos\_install/\* rsync@59.110.167.206::install

修改配置文件，客户端不允许查看列表：

> write only = yes

开始使用自定义用户验证：

修改配置文件:

```
strict modes = no
auth users = xiaoz
secrets file=/soft/rsync/ps.db
```

创建自定义用户密码文件：

```
touch /soft/rsync/ps.db
install:123456
chmod 600 /soft/rsync/ps.db
```

客户端设置密码：

```
touch /etc/rsync.ps.db
1234 > /etc/rsync.ps.db
chmod 600 /etc/rsync.ps.db
```

> cd /install/zip

rsync \-\-port=8877 \-avz \-\-progress /Users/wangdongyan/Desktop/1.jpg xiaoz@8.142.161.156::install \-\-password\-file=/etc/rsync.ps.db

```
#从远端拉取数据到本地
rsync -auzvP --progress  rsync@118.244.192.100::install /install/src     
rsync -auzvP --progress  --delete rsync@118.244.192.100::tools /tools     
#本地数据同步到远端

--delete:保持同步数据一样，如果多的文件直接DEL
--password-file=rsync.password：LINUX用户登陆 
 
nohup `rsync -auzvP --progress rsync@114.112.64.130::www /www`  > rsync.log &

service xinetd restart
```

#### 分分分分分分隔

被同步的机器：

groupadd rsync;

useradd \-g rsync rsync ;

passwd rsync;test123

su \- rsync;测试一下

同步机器：

mkdir /etc/rsync

touch /etc/rsync/rsyncd.conf

vim rsyncd.conf

```

port = 8877
log file = /data/logs/rsync/rsyncd.log
pid file = /var/rsyncd.pid

#是否检查口令文件的权限
strict modes =yes
port = 873
log file = /etc/rsync/rsyncd.log
pid file = /etc/rsync/rsyncd.pid

#模块名
[install]
max connections = 5
uid = rsync
gid = rsync
path = /root/test_rsync
hosts allow = *
hosts deny = 10.5.3.77
secrets file = /etc/rsync.ps
#####################################################
touch /etc/rsync/rsync.ps
rsync:test123

rsync --daemon --config=/etc/rsyncd.conf
ps aux|grep rsync

从服务端同步数据：backup为服务端的模块名
rsync -auzvP --progress rsync@192.168.0.1::tool /tool
rsync -auzvP --progress rsync@192.168.0.1::www /www

```

rsync%0A\-\-\-\-%0A%60%60%60%0Atouch%20%2Fsoft%2Frsync%2Frsyncd.conf%20%20%0Avim%20%2Fsoft%2Frsync%2Frsyncd.conf%0A%60%60%60%0A%60%60%60%0Aport%20%3D%208877%0Alog%20file%20%3D%20%2Fdata%2Flogs%2Frsync%2Frsyncd.log%0Apid%20file%20%3D%20%2Fvar%2Frsyncd.pid%0Amax%20connections%20%3D%205%0Astrict%20modes%20%3D%20no%0A%0A%5Binstall%5D%0Auid%20%3D%20rsync%0Agid%20%3D%20rsync%0Apath%20%3D%20%2Finstall%2Fzip%0Aread%20only%20%3D%20no%0Awrite%20only%20%3D%20no%20%20%20%0Ahosts%20allow%20%3D%20\*%0Ahosts%20deny%20%3D%2010.5.3.77%0A%60%60%60%0A%0A%E6%9C%8D%E5%8A%A1%E7%AB%AF\-%E5%90%AF%E5%8A%A8%0A%3Ersync%20\-\-daemon%20%20\-\-config%3D%2Fsoft%2Frsync%2Frsyncd.conf%20%0A%0A%E8%AE%BE%E7%BD%AE%E6%96%87%E4%BB%B6%E5%A4%B9%E6%9D%83%E9%99%90%EF%BC%9A%0A%3Echown%20\-R%20rsync%3Arsync%20%2Finstall%0A%0A%E9%98%BF%E9%87%8C%E4%BA%91%E5%BC%80%E5%90%AF8877%E7%AB%AF%E5%8F%A3%EF%BC%8C%E6%9C%AC%E6%9C%BA%E5%BC%80%E5%A7%8B%E5%90%8C%E6%AD%A5%20%0A%0A%E5%AE%A2%E6%88%B7%E7%AB%AF\-%E6%B5%8B%E8%AF%95%EF%BC%8C%E8%AF%BB%E5%8F%96%E4%B8%8B%E6%96%87%E4%BB%B6%E5%88%97%E8%A1%A8%EF%BC%9A%0A%3Ersync%20\-\-port%3D8877%20\-\-list\-only%20rsync%408.142.161.156%3A%3Ainstall%0A%0A%E5%AE%A2%E6%88%B7%E7%AB%AF\-%E6%B5%8B%E8%AF%95%3A%0A%3Ersync%20\-\-port%3D8877%20\-avz%20\-\-progress%20%2Fdata%2Fnew\_centos\_install%2F\*%20rsync%4059.110.167.206%3A%3Ainstall%0A%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%8D%E5%85%81%E8%AE%B8%E6%9F%A5%E7%9C%8B%E5%88%97%E8%A1%A8%EF%BC%9A%0A%3Ewrite%20only%20%3D%20yes%0A%0A%E5%BC%80%E5%A7%8B%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E7%94%A8%E6%88%B7%E9%AA%8C%E8%AF%81%EF%BC%9A%0A%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%3A%0A%60%60%60%0Astrict%20modes%20%3D%20no%0Aauth%20users%20%3D%20xiaoz%0Asecrets%20file%3D%2Fsoft%2Frsync%2Fps.db%0A%60%60%60%0A%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89%E7%94%A8%E6%88%B7%E5%AF%86%E7%A0%81%E6%96%87%E4%BB%B6%EF%BC%9A%0A%60%60%60%0Atouch%20%2Fsoft%2Frsync%2Fps.db%0Ainstall%3A123456%0Achmod%20600%20%2Fsoft%2Frsync%2Fps.db%0A%60%60%60%0A%0A%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AE%BE%E7%BD%AE%E5%AF%86%E7%A0%81%EF%BC%9A%0A%60%60%60%0Atouch%20%2Fetc%2Frsync.ps.db%0A1234%20%3E%20%2Fetc%2Frsync.ps.db%0Achmod%20600%20%2Fetc%2Frsync.ps.db%0A%60%60%60%0A%0A%3Ecd%20%2Finstall%2Fzip%20%20%0A%0Arsync%20\-\-port%3D8877%20\-avz%20\-\-progress%20%2FUsers%2Fwangdongyan%2FDesktop%2F1.jpg%20xiaoz%408.142.161.156%3A%3Ainstall%20\-\-password\-file%3D%2Fetc%2Frsync.ps.db%0A%0A%60%60%60%0A%23%E4%BB%8E%E8%BF%9C%E7%AB%AF%E6%8B%89%E5%8F%96%E6%95%B0%E6%8D%AE%E5%88%B0%E6%9C%AC%E5%9C%B0%0Arsync%20\-auzvP%20\-\-progress%20%20rsync%40118.244.192.100%3A%3Ainstall%20%2Finstall%2Fsrc%20%20%20%20%20%0Arsync%20\-auzvP%20\-\-progress%20%20\-\-delete%20rsync%40118.244.192.100%3A%3Atools%20%2Ftools%20%20%20%20%20%0A%23%E6%9C%AC%E5%9C%B0%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5%E5%88%B0%E8%BF%9C%E7%AB%AF%0A%0A\-\-delete%3A%E4%BF%9D%E6%8C%81%E5%90%8C%E6%AD%A5%E6%95%B0%E6%8D%AE%E4%B8%80%E6%A0%B7%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A4%9A%E7%9A%84%E6%96%87%E4%BB%B6%E7%9B%B4%E6%8E%A5DEL%0A\-\-password\-file%3Drsync.password%EF%BC%9ALINUX%E7%94%A8%E6%88%B7%E7%99%BB%E9%99%86%20%0A%20%0Anohup%20%60rsync%20\-auzvP%20\-\-progress%20rsync%40114.112.64.130%3A%3Awww%20%2Fwww%60%20%20%3E%20rsync.log%20%26%0A%0Aservice%20xinetd%20restart%0A%60%60%60%0A%0A%0A%23%23%23%23%20%20%E5%88%86%E5%88%86%E5%88%86%E5%88%86%E5%88%86%E5%88%86%E9%9A%94%0A%0A%E8%A2%AB%E5%90%8C%E6%AD%A5%E7%9A%84%E6%9C%BA%E5%99%A8%EF%BC%9A%0Agroupadd%20rsync%3B%0Auseradd%20\-g%20rsync%20rsync%20%3B%0Apasswd%20rsync%3Btest123%0Asu%20\-%20rsync%3B%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%E5%90%8C%E6%AD%A5%E6%9C%BA%E5%99%A8%EF%BC%9A%0Amkdir%20%2Fetc%2Frsync%0Atouch%20%2Fetc%2Frsync%2Frsyncd.conf%0Avim%20rsyncd.conf%0A%60%60%60%0A%0Aport%20%3D%208877%0Alog%20file%20%3D%20%2Fdata%2Flogs%2Frsync%2Frsyncd.log%0Apid%20file%20%3D%20%2Fvar%2Frsyncd.pid%0A%0A%23%E6%98%AF%E5%90%A6%E6%A3%80%E6%9F%A5%E5%8F%A3%E4%BB%A4%E6%96%87%E4%BB%B6%E7%9A%84%E6%9D%83%E9%99%90%0Astrict%20modes%20%3Dyes%0Aport%20%3D%20873%0Alog%20file%20%3D%20%2Fetc%2Frsync%2Frsyncd.log%0Apid%20file%20%3D%20%2Fetc%2Frsync%2Frsyncd.pid%0A%0A%23%E6%A8%A1%E5%9D%97%E5%90%8D%0A%5Binstall%5D%0Amax%20connections%20%3D%205%0Auid%20%3D%20rsync%0Agid%20%3D%20rsync%0Apath%20%3D%20%2Froot%2Ftest\_rsync%0Ahosts%20allow%20%3D%20\*%0Ahosts%20deny%20%3D%2010.5.3.77%0Asecrets%20file%20%3D%20%2Fetc%2Frsync.ps%0A%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%0Atouch%20%2Fetc%2Frsync%2Frsync.ps%0Arsync%3Atest123%0A%0Arsync%20\-\-daemon%20\-\-config%3D%2Fetc%2Frsyncd.conf%0Aps%20aux%7Cgrep%20rsync%0A%0A%E4%BB%8E%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%90%8C%E6%AD%A5%E6%95%B0%E6%8D%AE%EF%BC%9Abackup%E4%B8%BA%E6%9C%8D%E5%8A%A1%E7%AB%AF%E7%9A%84%E6%A8%A1%E5%9D%97%E5%90%8D%0Arsync%20\-auzvP%20\-\-progress%20rsync%40192.168.0.1%3A%3Atool%20%2Ftool%0Arsync%20\-auzvP%20\-\-progress%20rsync%40192.168.0.1%3A%3Awww%20%2Fwww%0A%0A%0A%60%60%60%0A%0A
