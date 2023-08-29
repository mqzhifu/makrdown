# ck-ar服务器

## 测试机

223.72.141.97

192.168.1.21

## 阿里云\-账号

用户名：seedreality

密码：Qwerty1234\!@\#$

## 阿里云 服务器

```
实例：4核 16GB 通用型 g5系列 IV
I/O 优化实例：I/O 优化实例
系统盘：SSD 云盘/dev/xvda 100GB 模块属性
带宽：3Mbps按固定带宽
CPU：4核
可用区：随机分配
操作系统：CentOS 8.1 64位Linux64位
内存：16GB
地域：华北 3
网络类型：专有网络
体检服务：是
管家服务：是
```

## 后台

https://admin.seedreality.com/

admin

ckarckar666

|key     |value                                        |
|--------|---------------------------------------------|
|生效时间|2022\-06\-15 18:40:01 ~ 2023\-06\-16 00:00:00|
|金额    |¥2,306.88                                    |
|os      |centos8.1                                    |
|外网IP  |8.142.161.156                                |
|内网IP  |172.27.218.143                               |

## SSH

|角色    |用户名|密码            |
|--------|------|----------------|
|普通用户|seedar|~\!@\#$QAZwsxedc|
|普通用户|rsync |~\!@\#qazWSXedc |
|管理员  |root  |CKseedarxr666   |

> 不允许: root 使用 ssh 直接登陆，可用普通账号登陆后再切换root

## 本地文件同步到线上服务器

查看模块下目录\(已被禁止\)

> rsync \-\-port=8877 \-\-list\-only rsync@8.142.161.156::install

客户端需要本地创建自己的密码文件：

touch /etc/rsync.ps.db

> 密码找王东岩要即可

同步文件

> rrsync \-\-port=8877 \-avz \-\-progress /Users/wangdongyan/Desktop/1.jpg xiaoz@8.142.161.156::install \-\-password\-file=/etc/rsync.ps.db

> 上传后，文件所在服务器位置:/install/zip

注：正式服只允许上传，不允许下载

## 目录说明

|统一存储目录说明|位置              |
|----------------|------------------|
|所有日志        |/data/logs/       |
|所有项目日志    |/data/logs/www    |
|所有备份        |/data/bak         |
|所有代码        |/data/www         |
|软件安装包      |/install/zip      |
|软件安装包解包  |/install/src      |
|上传图片        |/data/file\_upload|

## 软件

|                  |版本  |端口           |
|------------------|------|---------------|
|ssh               |      |6655           |
|/soft/apache      |2.4.46|8088           |
|/soft/nginx       |1.18.0|80             |
|/soft/rsync       |      |8877           |
|/soft/etcd        |3.4.14|2372           |
|/etc/supervisord.d|4.2.2 |9988           |
|/soft/redis       |6.0.9 |6375 外网不开放|
|/soft/php         |7.4.12|9000 外网不开放|
|/soft/jdk         |1.8.0 |无端口         |
|/soft/go          |1.16.1|无端口         |
|/soft/tomcat      |8.5.81|8080 外网不开放|
|mysql 阿里云      |8.0   |3306           |

## 域名

|域名           |使用情况|备案  |
|---------------|--------|------|
|seedxr.com     |未使用  |未备案|
|seedxr.net     |未使用  |未备案|
|seedxr.cn      |未使用  |未备案|
|seedreality.com|已使用  |已备案|

## 域名解析

|域名                          |说明                                 |
|------------------------------|-------------------------------------|
|adminapi.seedreality.com      |后台管理系统\-api接口                |
|ossservicebase.seedreality.com|阿里云\-OSS\-文件存储                |
|api.seedreality.com           |基服务:文件管理、配置中心等          |
|static.seedreality.com        |表态资源：html css js 小图片 小文件等|
|godoc.seedreality.com         |基服务代码结构说明                   |
|admin.seedreality.com         |后台管理系统                         |
|aliemail.seedreality.com      |阿里云\-邮件推送服务                 |

## redis

密码:ckckarar

## 项目代码

目录：/data/www

Zgoframe

基础服务 ：文件处理、配置中心等

端口：1111

Zwebuigo

后台管理系统

端口：8888

Zwebuivue

后台管理系统

tomcat\_webapps/platform/

用户/平台

## mysql

配置:

```
RDS规格：2 核 4GB（单机基础版）
数据库类型：MySQL
数据库版本号：8.0
公网流量：按实际使用流量每日扣费
可用区：cn-zhangjiakou-c
系列：基础版
地域：华北 3（张家口）
存储空间：100GB
存储类型：ESSD PL1云盘
网络类型：专有网络
```

|key     |value                                                  |
|--------|-------------------------------------------------------|
|生效时间|2022\-06\-15 18:40:01 ~ 2023\-06\-16 00:00:00          |
|价格    |¥ 520.80                                               |
|外网IP  |rm\-8vb10pi2gz9rma8p8vo.mysql.zhangbei.rds.aliyuncs.com|
|内网IP  |rm\-8vb10pi2gz9rma8p8.mysql.zhangbei.rds.aliyuncs.com  |

mysql用户

|角色    |用户名  |密码       |
|--------|--------|-----------|
|管理员  |superman|ckarCKAR789|
|普通用户|seed    |willbeOK618|

> 普通用户，只有DML权限
> 
> 
> 注：mysql直连，必须得加IP白名单

## mysql\-ui\-admin

http://8.142.161.156:8088/pma

## 业务数据库\-说明

|数据库名      |说明        |
|--------------|------------|
|seed          |微\(基\)服务|
|gva           |后台管理系统|
|seed\_platform|用户/平台   |

## superVisor

查看/管理：superVisor 进程

> http://8.142.161.156:9988/
> 
> 
> ckadmin
> 
> 
> ckckarar

## 监控/prometheus

|目录                      |版本  |端口号|
|--------------------------|------|------|
|/soft/nginx\-vst\-export  |0.10.3|9913  |
|/soft/node\_export        |1.0.1 |9100  |
|/soft/etcd                |      |2372  |
|jmx\_prometheus\_javaagent|      |30018 |
|mysqld\_exporter          |      |9104  |
|redis\_exporter           |      |9121  |

从supervior中导出重启失败的项目

```
cd /soft/exporter
./redis_exporter -redis.addr 127.0.0.1:6375  -redis.password ckckarar  -web.listen-address 0.0.0.0:9121  &
./node_exporter --web.listen-address=0.0.0.0:9100 &
./nginx-vts-exporter -nginx.scrape_uri=http://127.0.0.1/ck_nginx_status/format/json -telemetry.address=0.0.0.0:9913 &
./mysqld_exporter/mysqld_exporter --config.my-cnf=/soft/exporter/mysqld_exporter/exporter.conf &
```

metrics

|名称                |metrics                          |grafana                                                                                                         |
|--------------------|---------------------------------|----------------------------------------------------------------------------------------------------------------|
|node\_expoter       |http://8.142.161.156:9100/metrics|http://192.168.1.21:3000/d/rYdddlPWk/node\-exporter\-full?orgId=1&refresh=1m                                    |
|nginx\_vts\_exporter|http://8.142.161.156:9913/metrics|http://192.168.1.21:3000/d/gc73XI37k/nginx\-vts\-stats?orgId=1                                                  |
|mysqld\_expoter     |http://8.142.161.156:9104/metrics|http://192.168.1.21:3000/d/MQWgroiiz/mysql\-overview?orgId=1&refresh=1m                                         |
|redis\_expoter      |http://8.142.161.156:9121/metrics|http://192.168.1.21:3000/d/xDLNRKUWz/redis\-dashboard\-for\-prometheus\-redis\-exporter\-helm\-stable\-redis\-ha|
|etcd                |http://8.142.161.156:2379/metrics|http://192.168.1.21:3000/d/EY1WzHqnz/etcd\-by\-prometheus?orgId=1                                               |
|tomcat              |http://8.142.161.156:30018/      |http://192.168.1.21:3000/d/chanjarster\-jvm\-dashboard/jvm\-dashboard?orgId=1&refresh=10s                       |

grafana\-datashboard\-import\-id

```
node:1860
nginx:2949
mysql:7362
redis:11835
etcd:3070
jmx:8563
```

apache状态查看:

> http://8.142.161.156:8088/ck\-apache\-status

nginx状态查看

> http://8.142.161.156/ck\_nginx\_status

## tomcat

ckadmin

ckckarar

## prometheus

> http://192.168.1.21:9090/

## grafana

> http://192.168.1.21:3000/
> 
> 
> admin
> 
> 
> ckarckar

## https

使用阿里云免费的证书，到期日期：

> 2023\-06\-07

## OSS

外网访问基地址：

```
servicebase.oss-cn-beijing.aliyuncs.com
ossservicebase.seedreality.com
```

目前只有一个bucket：

servicebase

> sdk请自动去阿里官方下载，accessKey找王东岩要

## jdk环境变量 \(/etc/profile\)

```
export JAVA_HOME=/soft/jdk
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=${JAVA_HOME}/jre
```

## 192.168.1.21

算是测试机吧，目前应用到的：

1. CICD 项目部署
2. prometheus 收集指标
3. grafana 可视化指标

## 邮件推送

阿里云：

```
service@aliemail.seedreality.com
accessKeyaccessKeySecret = "GcVCuaZA7KWxV0o7UyzzSXhg9zCQfm"
```

> 密钥找王东岩要，sdk直接去阿里云下载即可

qq企业邮箱：

```
host    = "smtp.exmail.qq.com"
from    = "dongyan.w@seedreality.com"
ps      = "mM010010"
authCode = "2EGdKudfF6KvdosN"
```

> 这里的密码和authCode请自行登陆自己的企业邮箱设定 https://exmail.qq.com/login

邮箱组：

op@seedreality.com

dev@seedreality.com

## 未完成

报警：prometheus \+ alertmanager , 硬盘得加监控报警

CICD UI 可视化

项目metrcs 收集

大文件上传nginx

设定防火墙策略

CDN

日志文件定期清理

rsync ： 文件/目录权限，IP白名单

## 暂时不用

日志收集：filebeat \+ ES

短信\-配置信息

%E6%B5%8B%E8%AF%95%E6%9C%BA%0A\-\-\-\-%0A223.72.141.97%0A192.168.1.21%20%0A%0A%0A%E9%98%BF%E9%87%8C%E4%BA%91\-%E8%B4%A6%E5%8F%B7%0A\-\-\-%0A%E7%94%A8%E6%88%B7%E5%90%8D%EF%BC%9Aseedreality%0A%E5%AF%86%E7%A0%81%EF%BC%9AQwerty1234\!%40%23%24%0A%0A%E9%98%BF%E9%87%8C%E4%BA%91%20%E6%9C%8D%E5%8A%A1%E5%99%A8%0A\-\-\-\-%0A%60%60%60%0A%E5%AE%9E%E4%BE%8B%EF%BC%9A4%E6%A0%B8%2016GB%20%E9%80%9A%E7%94%A8%E5%9E%8B%20g5%E7%B3%BB%E5%88%97%20IV%0AI%2FO%20%E4%BC%98%E5%8C%96%E5%AE%9E%E4%BE%8B%EF%BC%9AI%2FO%20%E4%BC%98%E5%8C%96%E5%AE%9E%E4%BE%8B%0A%E7%B3%BB%E7%BB%9F%E7%9B%98%EF%BC%9ASSD%20%E4%BA%91%E7%9B%98%2Fdev%2Fxvda%20100GB%20%E6%A8%A1%E5%9D%97%E5%B1%9E%E6%80%A7%0A%E5%B8%A6%E5%AE%BD%EF%BC%9A3Mbps%E6%8C%89%E5%9B%BA%E5%AE%9A%E5%B8%A6%E5%AE%BD%0ACPU%EF%BC%9A4%E6%A0%B8%0A%E5%8F%AF%E7%94%A8%E5%8C%BA%EF%BC%9A%E9%9A%8F%E6%9C%BA%E5%88%86%E9%85%8D%0A%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%EF%BC%9ACentOS%208.1%2064%E4%BD%8DLinux64%E4%BD%8D%0A%E5%86%85%E5%AD%98%EF%BC%9A16GB%0A%E5%9C%B0%E5%9F%9F%EF%BC%9A%E5%8D%8E%E5%8C%97%203%0A%E7%BD%91%E7%BB%9C%E7%B1%BB%E5%9E%8B%EF%BC%9A%E4%B8%93%E6%9C%89%E7%BD%91%E7%BB%9C%0A%E4%BD%93%E6%A3%80%E6%9C%8D%E5%8A%A1%EF%BC%9A%E6%98%AF%0A%E7%AE%A1%E5%AE%B6%E6%9C%8D%E5%8A%A1%EF%BC%9A%E6%98%AF%0A%60%60%60%0A%0A%E5%90%8E%E5%8F%B0%0A\-\-\-\-%0Ahttps%3A%2F%2Fadmin.seedreality.com%2F%0Aadmin%0Ackarckar666%0A%0A%0A%7C%20key%20%7C%20value%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%E7%94%9F%E6%95%88%E6%97%B6%E9%97%B4%20%20%7C%202022\-06\-15%2018%3A40%3A01%20~%202023\-06\-16%2000%3A00%3A00%20%7C%0A%7C%20%E9%87%91%E9%A2%9D%20%7C%20%C2%A52%2C306.88%20%7C%0A%7C%20os%20%20%7C%20centos8.1%20%7C%0A%7C%20%E5%A4%96%E7%BD%91IP%20%7C%208.142.161.156%20%7C%0A%7C%20%E5%86%85%E7%BD%91IP%20%7C%20172.27.218.143%20%7C%0A%0ASSH%0A\-\-\-\-%0A%0A%7C%20%E8%A7%92%E8%89%B2%20%7C%E7%94%A8%E6%88%B7%E5%90%8D%20%20%7C%E5%AF%86%E7%A0%81%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%20%7C%20%20seedar%7C%20~\!%40%23%24QAZwsxedc%20%7C%0A%7C%20%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%20%7C%20%20rsync%7C%20~\!%40%23qazWSXedc%20%7C%0A%7C%20%E7%AE%A1%E7%90%86%E5%91%98%20%7C%20%20root%7C%20CKseedarxr666%20%7C%0A%3E%E4%B8%8D%E5%85%81%E8%AE%B8%3A%20root%20%E4%BD%BF%E7%94%A8%20ssh%20%E7%9B%B4%E6%8E%A5%E7%99%BB%E9%99%86%EF%BC%8C%E5%8F%AF%E7%94%A8%E6%99%AE%E9%80%9A%E8%B4%A6%E5%8F%B7%E7%99%BB%E9%99%86%E5%90%8E%E5%86%8D%E5%88%87%E6%8D%A2root%0A%0A%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A5%E5%88%B0%E7%BA%BF%E4%B8%8A%E6%9C%8D%E5%8A%A1%E5%99%A8%0A\-\-\-\-%0A%E6%9F%A5%E7%9C%8B%E6%A8%A1%E5%9D%97%E4%B8%8B%E7%9B%AE%E5%BD%95\(%E5%B7%B2%E8%A2%AB%E7%A6%81%E6%AD%A2\)%0A%3Ersync%20\-\-port%3D8877%20%20\-\-list\-only%20rsync%408.142.161.156%3A%3Ainstall%0A%0A%0A%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%9C%80%E8%A6%81%E6%9C%AC%E5%9C%B0%E5%88%9B%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E5%AF%86%E7%A0%81%E6%96%87%E4%BB%B6%EF%BC%9A%0Atouch%20%2Fetc%2Frsync.ps.db%0A%3E%E5%AF%86%E7%A0%81%E6%89%BE%E7%8E%8B%E4%B8%9C%E5%B2%A9%E8%A6%81%E5%8D%B3%E5%8F%AF%0A%0A%E5%90%8C%E6%AD%A5%E6%96%87%E4%BB%B6%0A%3Errsync%20\-\-port%3D8877%20\-avz%20\-\-progress%20%2FUsers%2Fwangdongyan%2FDesktop%2F1.jpg%20xiaoz%408.142.161.156%3A%3Ainstall%20\-\-password\-file%3D%2Fetc%2Frsync.ps.db%0A%0A%3E%E4%B8%8A%E4%BC%A0%E5%90%8E%EF%BC%8C%E6%96%87%E4%BB%B6%E6%89%80%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%8D%E7%BD%AE%3A%2Finstall%2Fzip%0A%0A%E6%B3%A8%EF%BC%9A%E6%AD%A3%E5%BC%8F%E6%9C%8D%E5%8F%AA%E5%85%81%E8%AE%B8%E4%B8%8A%E4%BC%A0%EF%BC%8C%E4%B8%8D%E5%85%81%E8%AE%B8%E4%B8%8B%E8%BD%BD%0A%0A%0A%0A%E7%9B%AE%E5%BD%95%E8%AF%B4%E6%98%8E%0A\-\-\-%0A%7C%20%E7%BB%9F%E4%B8%80%E5%AD%98%E5%82%A8%E7%9B%AE%E5%BD%95%E8%AF%B4%E6%98%8E%20%7C%E4%BD%8D%E7%BD%AE%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%E6%89%80%E6%9C%89%E6%97%A5%E5%BF%97%20%7C%2Fdata%2Flogs%2F%20%20%7C%0A%7C%20%E6%89%80%E6%9C%89%E9%A1%B9%E7%9B%AE%E6%97%A5%E5%BF%97%20%7C%2Fdata%2Flogs%2Fwww%20%20%7C%0A%7C%20%E6%89%80%E6%9C%89%E5%A4%87%E4%BB%BD%20%7C%20%2Fdata%2Fbak%20%7C%0A%7C%20%E6%89%80%E6%9C%89%E4%BB%A3%E7%A0%81%20%7C%20%2Fdata%2Fwww%20%7C%0A%7C%20%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%8C%85%20%7C%20%2Finstall%2Fzip%20%7C%0A%7C%20%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%8C%85%E8%A7%A3%E5%8C%85%20%7C%20%2Finstall%2Fsrc%20%7C%0A%7C%20%E4%B8%8A%E4%BC%A0%E5%9B%BE%E7%89%87%20%7C%2Fdata%2Ffile\_upload%20%20%7C%0A%0A%0A%E8%BD%AF%E4%BB%B6%0A\-\-\-%0A%7C%20%20%7C%20%E7%89%88%E6%9C%AC%20%7C%E7%AB%AF%E5%8F%A3%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C\-\-\-%20%7C%0A%7C%20ssh%20%7C%20%20%20%7C6655%7C%0A%7C%20%2Fsoft%2Fapache%20%7C%202.4.46%20%20%7C8088%7C%0A%7C%20%2Fsoft%2Fnginx%20%7C1.18.0%20%20%7C80%7C%0A%7C%20%2Fsoft%2Frsync%20%7C%20%20%7C8877%7C%0A%7C%20%2Fsoft%2Fetcd%20%7C%203.4.14%20%20%7C2372%20%7C%0A%7C%20%2Fetc%2Fsupervisord.d%20%7C%204.2.2%20%20%7C9988%7C%0A%7C%20%2Fsoft%2Fredis%20%7C6.0.9%20%20%20%7C%206375%20%E5%A4%96%E7%BD%91%E4%B8%8D%E5%BC%80%E6%94%BE%7C%0A%7C%20%2Fsoft%2Fphp%20%7C%207.4.12%20%7C9000%20%E5%A4%96%E7%BD%91%E4%B8%8D%E5%BC%80%E6%94%BE%7C%0A%7C%20%2Fsoft%2Fjdk%20%7C%201.8.0%20%7C%E6%97%A0%E7%AB%AF%E5%8F%A3%7C%0A%7C%20%2Fsoft%2Fgo%20%7C%201.16.1%20%7C%E6%97%A0%E7%AB%AF%E5%8F%A3%7C%0A%7C%2Fsoft%2Ftomcat%7C8.5.81%20%7C8080%20%E5%A4%96%E7%BD%91%E4%B8%8D%E5%BC%80%E6%94%BE%7C%0A%7Cmysql%20%E9%98%BF%E9%87%8C%E4%BA%91%7C8.0%7C3306%7C%0A%0A%0A%0A%E5%9F%9F%E5%90%8D%0A\-\-\-%0A%0A%7C%20%E5%9F%9F%E5%90%8D%20%7C%20%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5%20%20%7C%E5%A4%87%E6%A1%88%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C\-\-%7C%0A%7C%20seedxr.com%20%7C%20%E6%9C%AA%E4%BD%BF%E7%94%A8%20%7C%E6%9C%AA%E5%A4%87%E6%A1%88%7C%0A%7C%20%09seedxr.net%20%7C%20%E6%9C%AA%E4%BD%BF%E7%94%A8%20%7C%E6%9C%AA%E5%A4%87%E6%A1%88%7C%0A%7C%20seedxr.cn%20%7C%20%E6%9C%AA%E4%BD%BF%E7%94%A8%20%7C%E6%9C%AA%E5%A4%87%E6%A1%88%7C%0A%7C%20seedreality.com%20%7C%20%E5%B7%B2%E4%BD%BF%E7%94%A8%20%7C%E5%B7%B2%E5%A4%87%E6%A1%88%7C%0A%0A%0A%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90%0A\-\-\-%0A%7C%20%E5%9F%9F%E5%90%8D%20%7C%E8%AF%B4%E6%98%8E%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20adminapi.seedreality.com%20%7C%20%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F\-api%E6%8E%A5%E5%8F%A3%20%20%7C%0A%7C%20ossservicebase.seedreality.com%20%7C%20%E9%98%BF%E9%87%8C%E4%BA%91\-OSS\-%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%20%20%7C%0A%7C%20api.seedreality.com%20%7C%20%E5%9F%BA%E6%9C%8D%E5%8A%A1%3A%E6%96%87%E4%BB%B6%E7%AE%A1%E7%90%86%E3%80%81%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E7%AD%89%20%7C%0A%7C%20static.seedreality.com%20%7C%20%E8%A1%A8%E6%80%81%E8%B5%84%E6%BA%90%EF%BC%9Ahtml%20css%20js%20%E5%B0%8F%E5%9B%BE%E7%89%87%20%E5%B0%8F%E6%96%87%E4%BB%B6%E7%AD%89%20%7C%0A%7C%20godoc.seedreality.com%20%7C%20%E5%9F%BA%E6%9C%8D%E5%8A%A1%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84%E8%AF%B4%E6%98%8E%20%7C%0A%7C%20admin.seedreality.com%20%7C%20%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%20%7C%0A%7C%20aliemail.seedreality.com%20%7C%20%E9%98%BF%E9%87%8C%E4%BA%91\-%E9%82%AE%E4%BB%B6%E6%8E%A8%E9%80%81%E6%9C%8D%E5%8A%A1%20%7C%0A%0A%0Aredis%0A\-\-\-%0A%E5%AF%86%E7%A0%81%3Ackckarar%0A%0A%0A%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81%0A\-\-\-\-%0A%E7%9B%AE%E5%BD%95%EF%BC%9A%2Fdata%2Fwww%0A%0AZgoframe%0A%E5%9F%BA%E7%A1%80%E6%9C%8D%E5%8A%A1%20%EF%BC%9A%E6%96%87%E4%BB%B6%E5%A4%84%E7%90%86%E3%80%81%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E7%AD%89%0A%E7%AB%AF%E5%8F%A3%EF%BC%9A1111%0A%0AZwebuigo%0A%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%0A%E7%AB%AF%E5%8F%A3%EF%BC%9A8888%0A%0AZwebuivue%0A%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%0A%0Atomcat\_webapps%2Fplatform%2F%0A%E7%94%A8%E6%88%B7%2F%E5%B9%B3%E5%8F%B0%0A%0A%0Amysql%0A\-\-\-\-\-%0A%E9%85%8D%E7%BD%AE%3A%0A%60%60%60%0ARDS%E8%A7%84%E6%A0%BC%EF%BC%9A2%20%E6%A0%B8%204GB%EF%BC%88%E5%8D%95%E6%9C%BA%E5%9F%BA%E7%A1%80%E7%89%88%EF%BC%89%0A%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B1%BB%E5%9E%8B%EF%BC%9AMySQL%0A%E6%95%B0%E6%8D%AE%E5%BA%93%E7%89%88%E6%9C%AC%E5%8F%B7%EF%BC%9A8.0%0A%E5%85%AC%E7%BD%91%E6%B5%81%E9%87%8F%EF%BC%9A%E6%8C%89%E5%AE%9E%E9%99%85%E4%BD%BF%E7%94%A8%E6%B5%81%E9%87%8F%E6%AF%8F%E6%97%A5%E6%89%A3%E8%B4%B9%0A%E5%8F%AF%E7%94%A8%E5%8C%BA%EF%BC%9Acn\-zhangjiakou\-c%0A%E7%B3%BB%E5%88%97%EF%BC%9A%E5%9F%BA%E7%A1%80%E7%89%88%0A%E5%9C%B0%E5%9F%9F%EF%BC%9A%E5%8D%8E%E5%8C%97%203%EF%BC%88%E5%BC%A0%E5%AE%B6%E5%8F%A3%EF%BC%89%0A%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4%EF%BC%9A100GB%0A%E5%AD%98%E5%82%A8%E7%B1%BB%E5%9E%8B%EF%BC%9AESSD%20PL1%E4%BA%91%E7%9B%98%0A%E7%BD%91%E7%BB%9C%E7%B1%BB%E5%9E%8B%EF%BC%9A%E4%B8%93%E6%9C%89%E7%BD%91%E7%BB%9C%0A%60%60%60%0A%0A%0A%7C%20key%20%7C%20value%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%E7%94%9F%E6%95%88%E6%97%B6%E9%97%B4%20%20%7C%202022\-06\-15%2018%3A40%3A01%20~%202023\-06\-16%2000%3A00%3A00%20%20%7C%0A%7C%20%E4%BB%B7%E6%A0%BC%20%7C%20%C2%A5%20520.80%20%7C%0A%7C%20%E5%A4%96%E7%BD%91IP%20%7C%20rm\-8vb10pi2gz9rma8p8vo.mysql.zhangbei.rds.aliyuncs.com%20%7C%0A%7C%20%E5%86%85%E7%BD%91IP%7C%20rm\-8vb10pi2gz9rma8p8.mysql.zhangbei.rds.aliyuncs.com%20%7C%0A%0A%0Amysql%E7%94%A8%E6%88%B7%0A%0A%7C%E8%A7%92%E8%89%B2%20%20%7C%20%E7%94%A8%E6%88%B7%E5%90%8D%20%7C%E5%AF%86%E7%A0%81%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%E7%AE%A1%E7%90%86%E5%91%98%20%7C%20superman%20%7C%20ckarCKAR789%20%7C%0A%7C%20%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%20%7C%20seed%20%7C%20willbeOK618%20%7C%0A%3E%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%EF%BC%8C%E5%8F%AA%E6%9C%89DML%E6%9D%83%E9%99%90%0A%3E%E6%B3%A8%EF%BC%9Amysql%E7%9B%B4%E8%BF%9E%EF%BC%8C%E5%BF%85%E9%A1%BB%E5%BE%97%E5%8A%A0IP%E7%99%BD%E5%90%8D%E5%8D%95%0A%0Amysql\-ui\-admin%0A\-\-\-%0Ahttp%3A%2F%2F8.142.161.156%3A8088%2Fpma%0A%0A%0A%0A%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%BA%93\-%E8%AF%B4%E6%98%8E%0A\-\-\-%0A%0A%7C%20%E6%95%B0%E6%8D%AE%E5%BA%93%E5%90%8D%20%7C%E8%AF%B4%E6%98%8E%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20seed%20%7C%E5%BE%AE\(%E5%9F%BA\)%E6%9C%8D%E5%8A%A1%20%20%7C%0A%7C%20gva%20%7C%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%20%20%7C%0A%7C%20seed\_platform%20%7C%20%E7%94%A8%E6%88%B7%2F%E5%B9%B3%E5%8F%B0%20%7C%0A%0A%0AsuperVisor%0A\-\-\-\-%0A%E6%9F%A5%E7%9C%8B%2F%E7%AE%A1%E7%90%86%EF%BC%9AsuperVisor%20%E8%BF%9B%E7%A8%8B%0A%3Ehttp%3A%2F%2F8.142.161.156%3A9988%2F%0Ackadmin%0Ackckarar%0A%0A%0A%0A%E7%9B%91%E6%8E%A7%2Fprometheus%0A\-\-\-\-%0A%0A%0A%7C%20%20%E7%9B%AE%E5%BD%95%7C%20%E7%89%88%E6%9C%AC%20%7C%E7%AB%AF%E5%8F%A3%E5%8F%B7%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%2Fsoft%2Fnginx\-vst\-export%20%7C0.10.3%7C9913%7C%0A%7C%20%2Fsoft%2Fnode\_export%20%7C%201.0.1%20%20%7C9100%7C%0A%7C%20%2Fsoft%2Fetcd%20%7C%20%20%7C%202372%20%7C%0A%7C%20jmx\_prometheus\_javaagent%20%7C%20%20%7C%2030018%20%7C%0A%7C%20mysqld\_exporter%20%7C%20%20%7C%209104%20%7C%0A%7C%20redis\_exporter%20%7C%20%20%7C%209121%20%7C%0A%0A%0A%E4%BB%8Esupervior%E4%B8%AD%E5%AF%BC%E5%87%BA%E9%87%8D%E5%90%AF%E5%A4%B1%E8%B4%A5%E7%9A%84%E9%A1%B9%E7%9B%AE%0A%0A%0A%0A%0A%0A%0A%60%60%60%0Acd%20%2Fsoft%2Fexporter%0A.%2Fredis\_exporter%20\-redis.addr%20127.0.0.1%3A6375%20%20\-redis.password%20ckckarar%20%20\-web.listen\-address%200.0.0.0%3A9121%20%20%26%0A.%2Fnode\_exporter%20\-\-web.listen\-address%3D0.0.0.0%3A9100%20%26%0A.%2Fnginx\-vts\-exporter%20\-nginx.scrape\_uri%3Dhttp%3A%2F%2F127.0.0.1%2Fck\_nginx\_status%2Fformat%2Fjson%20\-telemetry.address%3D0.0.0.0%3A9913%20%26%0A.%2Fmysqld\_exporter%2Fmysqld\_exporter%20\-\-config.my\-cnf%3D%2Fsoft%2Fexporter%2Fmysqld\_exporter%2Fexporter.conf%20%26%0A%60%60%60%0A%0Ametrics%0A%0A%7C%20%E5%90%8D%E7%A7%B0%20%7Cmetrics%20%20%7C%20grafana%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20node\_expoter%20%7C%20http%3A%2F%2F8.142.161.156%3A9100%2Fmetrics%20%7C%20http%3A%2F%2F192.168.1.21%3A3000%2Fd%2FrYdddlPWk%2Fnode\-exporter\-full%3ForgId%3D1%26refresh%3D1m%20%7C%0A%7C%20nginx\_vts\_exporter%20%7C%20http%3A%2F%2F8.142.161.156%3A9913%2Fmetrics%20%7C%20http%3A%2F%2F192.168.1.21%3A3000%2Fd%2Fgc73XI37k%2Fnginx\-vts\-stats%3ForgId%3D1%20%7C%0A%7C%20mysqld\_expoter%20%7C%20http%3A%2F%2F8.142.161.156%3A9104%2Fmetrics%20%7C%20http%3A%2F%2F192.168.1.21%3A3000%2Fd%2FMQWgroiiz%2Fmysql\-overview%3ForgId%3D1%26refresh%3D1m%20%7C%0A%7C%20redis\_expoter%20%7C%20http%3A%2F%2F8.142.161.156%3A9121%2Fmetrics%20%7Chttp%3A%2F%2F192.168.1.21%3A3000%2Fd%2FxDLNRKUWz%2Fredis\-dashboard\-for\-prometheus\-redis\-exporter\-helm\-stable\-redis\-ha%20%20%7C%0A%7Cetcd%7Chttp%3A%2F%2F8.142.161.156%3A2379%2Fmetrics%7Chttp%3A%2F%2F192.168.1.21%3A3000%2Fd%2FEY1WzHqnz%2Fetcd\-by\-prometheus%3ForgId%3D1%7C%0A%7Ctomcat%7Chttp%3A%2F%2F8.142.161.156%3A30018%2F%7Chttp%3A%2F%2F192.168.1.21%3A3000%2Fd%2Fchanjarster\-jvm\-dashboard%2Fjvm\-dashboard%3ForgId%3D1%26refresh%3D10s%7C%0A%0Agrafana\-datashboard\-import\-id%0A%60%60%60%0Anode%3A1860%0Anginx%3A2949%0Amysql%3A7362%0Aredis%3A11835%0Aetcd%3A3070%0Ajmx%3A8563%0A%60%60%60%0A%0A%0A%0Aapache%E7%8A%B6%E6%80%81%E6%9F%A5%E7%9C%8B%3A%0A%3Ehttp%3A%2F%2F8.142.161.156%3A8088%2Fck\-apache\-status%0A%0Anginx%E7%8A%B6%E6%80%81%E6%9F%A5%E7%9C%8B%0A%3Ehttp%3A%2F%2F8.142.161.156%2Fck\_nginx\_status%0A%0Atomcat%0A\-\-\-\-%0Ackadmin%0Ackckarar%0A%0A%0Aprometheus%0A\-\-\-\-%0A%3Ehttp%3A%2F%2F192.168.1.21%3A9090%2F%0A%0Agrafana%0A\-\-\-\-\-%0A%3Ehttp%3A%2F%2F192.168.1.21%3A3000%2F%0Aadmin%0Ackarckar%0A%0Ahttps%0A\-\-\-\-%0A%E4%BD%BF%E7%94%A8%E9%98%BF%E9%87%8C%E4%BA%91%E5%85%8D%E8%B4%B9%E7%9A%84%E8%AF%81%E4%B9%A6%EF%BC%8C%E5%88%B0%E6%9C%9F%E6%97%A5%E6%9C%9F%EF%BC%9A%0A%3E2023\-06\-07%0A%0AOSS%0A\-\-\-\-%0A%E5%A4%96%E7%BD%91%E8%AE%BF%E9%97%AE%E5%9F%BA%E5%9C%B0%E5%9D%80%EF%BC%9A%0A%60%60%60%0Aservicebase.oss\-cn\-beijing.aliyuncs.com%0Aossservicebase.seedreality.com%0A%60%60%60%0A%E7%9B%AE%E5%89%8D%E5%8F%AA%E6%9C%89%E4%B8%80%E4%B8%AAbucket%EF%BC%9A%0Aservicebase%0A%0A%3Esdk%E8%AF%B7%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%98%BF%E9%87%8C%E5%AE%98%E6%96%B9%E4%B8%8B%E8%BD%BD%EF%BC%8CaccessKey%E6%89%BE%E7%8E%8B%E4%B8%9C%E5%B2%A9%E8%A6%81%0A%0Ajdk%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%20\(%2Fetc%2Fprofile\)%0A\-\-\-\-%0A%60%60%60%0Aexport%20JAVA\_HOME%3D%2Fsoft%2Fjdk%0Aexport%20PATH%3D%24PATH%3A%24JAVA\_HOME%2Fbin%0Aexport%20CLASSPATH%3D.%3A%24JAVA\_HOME%2Fjre%2Flib%2Frt.jar%3A%24JAVA\_HOME%2Flib%2Fdt.jar%3A%24JAVA\_HOME%2Flib%2Ftools.jar%0Aexport%20JRE\_HOME%3D%24%7BJAVA\_HOME%7D%2Fjre%0A%60%60%60%0A%0A192.168.1.21%0A\-\-\-\-\-%0A%E7%AE%97%E6%98%AF%E6%B5%8B%E8%AF%95%E6%9C%BA%E5%90%A7%EF%BC%8C%E7%9B%AE%E5%89%8D%E5%BA%94%E7%94%A8%E5%88%B0%E7%9A%84%EF%BC%9A%0A1.%20CICD%20%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%0A3.%20prometheus%20%E6%94%B6%E9%9B%86%E6%8C%87%E6%A0%87%0A2.%20grafana%20%E5%8F%AF%E8%A7%86%E5%8C%96%E6%8C%87%E6%A0%87%0A%0A%0A%E9%82%AE%E4%BB%B6%E6%8E%A8%E9%80%81%0A\-\-\-\-\-%0A%0A%E9%98%BF%E9%87%8C%E4%BA%91%EF%BC%9A%0A%60%60%60%0Aservice%40aliemail.seedreality.com%0AaccessKeyId%20%3D%20%22LTAI5tJbjZiWQ9Xn9N2brRFD%22%0AaccessKeySecret%20%3D%20%22GcVCuaZA7KWxV0o7UyzzSXhg9zCQfm%22%0A%60%60%60%0A%3E%E5%AF%86%E9%92%A5%E6%89%BE%E7%8E%8B%E4%B8%9C%E5%B2%A9%E8%A6%81%EF%BC%8Csdk%E7%9B%B4%E6%8E%A5%E5%8E%BB%E9%98%BF%E9%87%8C%E4%BA%91%E4%B8%8B%E8%BD%BD%E5%8D%B3%E5%8F%AF%0A%0Aqq%E4%BC%81%E4%B8%9A%E9%82%AE%E7%AE%B1%EF%BC%9A%0A%60%60%60%0Ahost%20%20%20%20%3D%20%22smtp.exmail.qq.com%22%0Afrom%20%20%20%20%3D%20%22dongyan.w%40seedreality.com%22%0Aps%20%20%20%20%20%20%3D%20%22mM010010%22%0AauthCode%20%3D%20%222EGdKudfF6KvdosN%22%0A%60%60%60%0A%3E%E8%BF%99%E9%87%8C%E7%9A%84%E5%AF%86%E7%A0%81%E5%92%8CauthCode%E8%AF%B7%E8%87%AA%E8%A1%8C%E7%99%BB%E9%99%86%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BC%81%E4%B8%9A%E9%82%AE%E7%AE%B1%E8%AE%BE%E5%AE%9A%20https%3A%2F%2Fexmail.qq.com%2Flogin%0A%0A%E9%82%AE%E7%AE%B1%E7%BB%84%EF%BC%9A%0Aop%40seedreality.com%0Adev%40seedreality.com%0A%0A%0A%E6%9C%AA%E5%AE%8C%E6%88%90%0A\-\-\-%0A%E6%8A%A5%E8%AD%A6%EF%BC%9Aprometheus%20%2B%20alertmanager%20%2C%20%E7%A1%AC%E7%9B%98%E5%BE%97%E5%8A%A0%E7%9B%91%E6%8E%A7%E6%8A%A5%E8%AD%A6%0ACICD%20UI%20%E5%8F%AF%E8%A7%86%E5%8C%96%0A%E9%A1%B9%E7%9B%AEmetrcs%20%E6%94%B6%E9%9B%86%0A%E5%A4%A7%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0nginx%0A%E8%AE%BE%E5%AE%9A%E9%98%B2%E7%81%AB%E5%A2%99%E7%AD%96%E7%95%A5%0ACDN%0A%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E5%AE%9A%E6%9C%9F%E6%B8%85%E7%90%86%0Arsync%20%EF%BC%9A%20%E6%96%87%E4%BB%B6%2F%E7%9B%AE%E5%BD%95%E6%9D%83%E9%99%90%EF%BC%8CIP%E7%99%BD%E5%90%8D%E5%8D%95%0A%0A%E6%9A%82%E6%97%B6%E4%B8%8D%E7%94%A8%0A\-\-\-\-%0A%E6%97%A5%E5%BF%97%E6%94%B6%E9%9B%86%EF%BC%9Afilebeat%20%2B%20ES%0A%E7%9F%AD%E4%BF%A1\-%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF
