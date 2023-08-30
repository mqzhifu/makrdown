# 测试机

223.72.141.97
192.168.1.21

目前应用到的：

1. CICD 项目部署
2. prometheus 收集指标
3. grafana 可视化指标

# 阿里云-账号

| 名   | 值               |
| ---- | ---------------- |
| name | seedreality      |
| ps   | Qwerty1234\!@\#$ |

# 服务器-配置

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

| key      | value                                         |
| -------- | --------------------------------------------- |
| 生效时间 | 2022\-06\-15 18:40:01 ~ 2023\-06\-16 00:00:00 |
| -------- | --------------------------------------------- |
| 金额     | ¥2,306.88                                     |
| os       | centos8.1                                     |
| 外网 IP  | 8.142.161.156                                 |
| 内网 IP  | 172.27.218.143                                |

# SSH

| 角色     | 用户名 | 密码             |
| -------- | ------ | ---------------- |
| 普通用户 | seedar | ~\!@\#$QAZwsxedc |
| 普通用户 | rsync  | ~\!@\#qazWSXedc  |
| 管理员   | root   | CKseedarxr666    |

> 不允许: root 使用 ssh 直接登陆，可用普通账号登陆后再切换 root

# 本地文件同步到线上服务器

查看模块下目录(已被禁止)

> rsync \-\-port=8877 \-\-list\-only rsync@8.142.161.156::install

客户端需要本地创建自己的密码文件：

touch /etc/rsync.ps.db

> 密码找王东岩要即可

同步文件

> rrsync \-\-port=8877 \-avz \-\-progress /Users/wangdongyan/Desktop/1.jpg xiaoz@8.142.161.156::install \-\-password\-file=/etc/rsync.ps.db

> 上传后，文件所在服务器位置:/install/zip

注：正式服只允许上传，不允许下载

# 目录说明

| 统一存储目录说明 | 位置              |
| ---------------- | ----------------- |
| 所有日志         | /data/logs/       |
| 所有项目日志     | /data/logs/www    |
| 所有备份         | /data/bak         |
| 所有代码         | /data/www         |
| 软件安装包       | /install/zip      |
| 软件安装包解包   | /install/src      |
| 上传图片         | /data/file_upload |

# 软件/端口号

|                    | 版本   | 端口            |
| ------------------ | ------ | --------------- |
| ssh                |        | 6655            |
| /soft/apache       | 2.4.46 | 8088            |
| /soft/nginx        | 1.18.0 | 80              |
| /soft/rsync        |        | 8877            |
| /soft/etcd         | 3.4.14 | 2372            |
| /etc/supervisord.d | 4.2.2  | 9988            |
| /soft/redis        | 6.0.9  | 6375 外网不开放 |
| /soft/php          | 7.4.12 | 9000 外网不开放 |
| /soft/jdk          | 1.8.0  | 无端口          |
| /soft/go           | 1.16.1 | 无端口          |
| /soft/tomcat       | 8.5.81 | 8080 外网不开放 |
| mysql 阿里云       | 8.0    | 3306            |

# 域名

| 域名            | 使用情况 | 备案   |
| --------------- | -------- | ------ |
| seedxr.com      | 未使用   | 未备案 |
| seedxr.net      | 未使用   | 未备案 |
| seedxr.cn       | 未使用   | 未备案 |
| seedreality.com | 已使用   | 已备案 |

# 域名解析

| 域名                           | 说明                                  |
| ------------------------------ | ------------------------------------- |
| adminapi.seedreality.com       | 后台管理系统\-api 接口                |
| ossservicebase.seedreality.com | 阿里云\-OSS\-文件存储                 |
| api.seedreality.com            | 基服务:文件管理、配置中心等           |
| static.seedreality.com         | 表态资源：html css js 小图片 小文件等 |
| godoc.seedreality.com          | 基服务代码结构说明                    |
| admin.seedreality.com          | 后台管理系统                          |
| aliemail.seedreality.com       | 阿里云\-邮件推送服务                  |

# redis

密码:ckckarar

# 项目代码

## Zgoframe

目录：/data/www

基础服务 ：文件处理、配置中心等
端口：1111

## 后台

https://admin.seedreality.com/
admin
ckarckar666

Zwebuigo
后台管理系统
端口：8888
Zwebuivue
后台管理系统

## 用户/平台

tomcat\webapps/platform/

# mysql

## 配置:

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

| key      | value                                                   |
| -------- | ------------------------------------------------------- |
| 生效时间 | 2022\-06\-15 18:40:01 ~ 2023\-06\-16 00:00:00           |
| 价格     | ¥ 520.80                                                |
| 外网 IP  | rm\-8vb10pi2gz9rma8p8vo.mysql.zhangbei.rds.aliyuncs.com |
| 内网 IP  | rm\-8vb10pi2gz9rma8p8.mysql.zhangbei.rds.aliyuncs.com   |

用户

| 角色     | 用户名   | 密码        |
| -------- | -------- | ----------- |
| 管理员   | superman | ckarCKAR789 |
| 普通用户 | seed     | willbeOK618 |

> 普通用户，只有 DML 权限
> 注：mysql 直连，必须得加 IP 白名单

## mysql-ui-admin

http://8.142.161.156:8088/pma

## 业务数据库-说明

| 数据库名      | 说明         |
| ------------- | ------------ |
| seed          | 微\(基\)服务 |
| gva           | 后台管理系统 |
| seed_platform | 用户/平台    |

# superVisor

查看/管理：superVisor 进程

> http://8.142.161.156:9988/
> ckadmin
> ckckarar

# 监控/prometheus

| 目录                     | 版本   | 端口号 |
| ------------------------ | ------ | ------ |
| /soft/nginx-vst-export   | 0.10.3 | 9913   |
| /soft/node_export        | 1.0.1  | 9100   |
| /soft/etcd               |        | 2372   |
| jmx_prometheus_javaagent |        | 30018  |
| mysqld_exporter          |        | 9104   |
| redis_exporter           |        | 9121   |

从 supervior 中导出重启失败的项目

```
cd /soft/exporter
./redis_exporter -redis.addr 127.0.0.1:6375  -redis.password ckckarar  -web.listen-address 0.0.0.0:9121  &
./node_exporter --web.listen-address=0.0.0.0:9100 &
./nginx-vts-exporter -nginx.scrape_uri=http://127.0.0.1/ck_nginx_status/format/json -telemetry.address=0.0.0.0:9913 &
./mysqld_exporter/mysqld_exporter --config.my-cnf=/soft/exporter/mysqld_exporter/exporter.conf &
```

metrics

| 名称               | metrics                           | grafana                                                                                                          |
| ------------------ | --------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| node_expoter       | http://8.142.161.156:9100/metrics | http://192.168.1.21:3000/d/rYdddlPWk/node\-exporter\-full?orgId=1&refresh=1m                                     |
| nginx_vts_exporter | http://8.142.161.156:9913/metrics | http://192.168.1.21:3000/d/gc73XI37k/nginx\-vts\-stats?orgId=1                                                   |
| mysqld_expoter     | http://8.142.161.156:9104/metrics | http://192.168.1.21:3000/d/MQWgroiiz/mysql\-overview?orgId=1&refresh=1m                                          |
| redis_expoter      | http://8.142.161.156:9121/metrics | http://192.168.1.21:3000/d/xDLNRKUWz/redis\-dashboard\-for\-prometheus\-redis\-exporter\-helm\-stable\-redis\-ha |
| etcd               | http://8.142.161.156:2379/metrics | http://192.168.1.21:3000/d/EY1WzHqnz/etcd\-by\-prometheus?orgId=1                                                |
| tomcat             | http://8.142.161.156:30018/       | http://192.168.1.21:3000/d/chanjarster\-jvm\-dashboard/jvm\-dashboard?orgId=1&refresh=10s                        |

grafana-datashboard-import-id

```
node:1860
nginx:2949
mysql:7362
redis:11835
etcd:3070
jmx:8563
```

apache 状态查看:

> http://8.142.161.156:8088/ck-apache-status

nginx 状态查看

> http://8.142.161.156/ck_nginx\_status

# tomcat

ckadmin

ckckarar

# prometheus

> http://192.168.1.21:9090/

# grafana

> http://192.168.1.21:3000/
> admin
> ckarckar

# https

# OSS

外网访问基地址：

```
servicebase.oss-cn-beijing.aliyuncs.com
ossservicebase.seedreality.com
```

目前只有一个 bucket：

servicebase

> sdk 请自动去阿里官方下载，accessKey 找王东岩要

# jdk 环境变量 (/etc/profile)

```
export JAVA_HOME=/soft/jdk
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=${JAVA_HOME}/jre
```

# 邮件推送

阿里云：

```
service@aliemail.seedreality.com
accessKeyaccessKeySecret = "GcVCuaZA7KWxV0o7UyzzSXhg9zCQfm"
```

> 密钥找王东岩要，sdk 直接去阿里云下载即可

qq 企业邮箱：

```
host    = "smtp.exmail.qq.com"
from    = "dongyan.w@seedreality.com"
ps      = "mM010010"
authCode = "2EGdKudfF6KvdosN"
```

> 这里的密码和 authCode 请自行登陆自己的企业邮箱设定 https://exmail.qq.com/login

邮箱组：

op@seedreality.com

dev@seedreality.com

# 未完成

报警：prometheus \+ alertmanager , 硬盘得加监控报警
CICD UI 可视化
项目 metrcs 收集
大文件上传 nginx
设定防火墙策略
CDN
日志文件定期清理

rsync ： 文件/目录权限，IP 白名单

## 暂时不用

日志收集：filebeat + ES
短信-配置信息
