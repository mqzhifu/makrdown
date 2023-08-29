# gateway-APISIX

## 初衷/目的

寻找一个开源软件：对外的公共网关

## 原因

因为linkerd没有限量 黑名单等功能，或者说linkerd就不是对外公共网关，他的侧重点更多是服务间的治理。因此，还得找一个API公共网关，看了下主流的：kong kyt APISIX Zuul

Zuul首先淘汰，JAVA体系里的，1是重 2是尝试捆绑JAVA

kyt：GOLANG编写，但是高级功能收费

kong:绑定postgresql DB,不支持GRPC WEEBSOCKET MQTT,对服务发现也不太友好

最后意外发现了个：APISIX

## 需求

1. 服务发现与治理,consul
2. 负载均衡
3. 熔断
4. 限制，速度 请求数 并发
5. 黑名单\(ip/uid\)
6. 支持http https grpc websocket
7. 验证，JWT OAUTH
8. 链路追踪 ，zipKin
9. 可视化后台管理工具，同时支持HTTP管理
10. 日志，prometheus
11. 兼容旧代码，代码侵入

## 核心需求

1. 防攻击
2. 服务发现与治理

## APISIX

官方文档

> https://apisix.apache.org/zh/docs/apisix/getting\-started

中文文档

> https://www.bookstack.cn/read/apache\-apisix\-1.5\-zh/a2c94bb1dcdc1a83.md

英文文档

> https://apisix.apache.org/docs/apisix/discovery/consul\_kv

其实它有点是KONG的升级版，底层依然是：NGINX\+OPENRESTY，只是放弃了postgreSql，换成了ETCD，同时优化了下结构，代码更少，对一些主流的软件支持略友好，且扩展自由度更高

> 具说性能比KONG至少高2倍以上.

这个软件好像是2019.6月才开源，稳定性对比kong 应该略差些，毕竟，KONG运行了好多年。不过，2021年的我们，再用版本已经到了2.11 应该免去了这个麻烦。

## 普通安装

官方文档用的是docker，因为我是测试，就换了普通的yum方式

先看一下依赖包：open\-restry ,etcd

> 仅限 centos7

折腾一圈下来，遇到各种问题，最大问题是：open\-restry 兼容性

etcd也得有修改点参数：

> enable\-v2: true \#如果用V2版本，使用HTTP的方式，得打开，不然404
> 
> 
> enable\-grpc\-gateway: true

正式安装 apisix

官方用的是yum，3方是下包

> wget https://downloads.apache.org/apisix/2.11.0/apache\-apisix\-2.11.0\-src.tgz

tar \-zxvf

mv

添加lua依赖库

> make deps
> 
> 
> LUAROCKS\_SERVER=https://luarocks.cn make deps

报错，得安装 luarocks

> yum install \-y luarocks lua\-devel lualdap

重装执行，开始下载了。。。时间还挺长.....

报错，得安装ldap

> yum update

> yum \-y install openldap compat\-openldap openldap\-clients openldap\-servers openldap\-servers\-sql openldap\-devel

好了....

执行，lua http 库存有问题，放弃了，改用docker吧

## 再次尝试

> yum \-y install yum\-utils

openrestry

> yum install \-y https://repos.apiseven.com/packages/centos/apache\-apisix\-repo\-1.0\-1.noarch.rpm

> yum\-config\-manager \-\-add\-repo https://repos.apiseven.com/packages/centos/apache\-apisix.repo

> sudo yum install apisix

终于成功了。。。。

对比了下他的几个依赖包

```
apisix-base                    x86_64           1.19.3.2.2-0.el7                 release              35 M
 openresty-openssl111           x86_64           1.1.1l-1.el7                     openresty           1.5 M
 openresty-pcre                 x86_64           8.44-1.el7                       openresty           164 k
 openresty-zlib                 x86_64           1.2.11-3.el7.centos              openresty            54 k

```

默认安装在/usr/local下

apisix start

apisix test

apisix init

apisix init\_etcd

\#优雅停止

apisix quit

\#强制停止

apisix stop

瞬间启动成功... 真不知道前天折腾到3点图啥...

本机再安装一个nginx 80 和 apache 8088 用于测试

先创建一个Upstream

```
curl http://127.0.0.1:9080/apisix/admin/upstreams/10 -X PUT -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -d '
{
  "type": "roundrobin",
  "nodes": {
    "127.0.0.1:80": 1,
    "127.0.0.1:8088": 1
  }
}'
```

查看

> curl http://127.0.0.1:9080/apisix/admin/upstreams \-H "X\-API\-KEY: edd1c9f034335f136f87ad84b625c8f1"

创建router并绑定upstream

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "uri": "/apisix/",
  "host": "service1",
  "upstream_id": "10"
}'
```

验证

> curl \-i \-X GET "http://127.0.0.1:9080/apisix/" \-H "Host: service1"

做下基础验证的测试

创建一个consumer

```
curl http://127.0.0.1:9080/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "username": "service1_user",
    "plugins": {
        "key-auth": {
            "key": "auth-one"
        }
    }
}'
```

修改路由

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
    "plugins": {
        "key-auth": {
            "header": "ser_key"
        }
    },
  "uri": "/apisix/",
  "host": "service1",
  "upstream_id": "10"
}'
```

> curl \-i \-X GET "http://127.0.0.1:9080/apisix/" \-H "Host: service1" \-H "ser\_key:auth\-one"

还有：jwt base\-auth 不测了，有点鸡肋，主要浪费网关计算时间

## docker安装

将 Apache APISIX 的 Docker 镜像下载到本地

> git clone https://github.com/apache/apisix\-docker.git

将当前的目录切换到 apisix\-docker/example 路径下

> cd apisix\-docker/example

运行 docker\-compose 命令，安装 Apache APISIX

> docker\-compose \-p docker\-apisix up \-d

还是官方文档，docker 安装靠谱，分分钟搞定了...

看了下它的编排文件，里面包含的东西还挺多

1. nginx\(两个\) 9081 9082
2. apisix 9080 9091 9443 9092
3. prometheus 9090
4. etcd 2379
5. grafana 3000
6. apisix\-dashboard 9000

停止

> > docker\-compose \-p docker\-apisix stop

测试一下

> curl "http://127.0.0.1:9080/apisix/admin/services/" \-H 'X\-API\-KEY: edd1c9f034335f136f87ad84b625c8f1'

看一下dashboard

> http://8.142.177.235:9000/

## 初步测试

创建一个upstreams

```
curl "http://127.0.0.1:9080/apisix/admin/upstreams/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "httpbin.org:80": 1
  }
}'
```

创建一个router,并绑定到上面的upstreams上

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "uri": "/get",
  "host": "httpbin.org",
  "upstream_id": "1"
}'
```

验证

> curl \-i \-X GET "http://127.0.0.1:9080/get?foo1=bar1&foo2=bar2" \-H "Host: httpbin.org"

## 数据流转

req\-\>etcd\-\>router\-\>plugin\-\>filter\-\>filter\-plugin\-\>upstream

\-\>plugin\-\>real\-service

核心：router upstream plguins

我们看实际就是通过plugins将router upstream串联起，最后找到后端服务器

## router

功能：通过 req信息，找到，router，再 找到upstream

先根据用户的请求信息，如：header uri ，去ETCD里匹配，找到一个router，然后根据该条router的配置信息，执行插件，最后找到一个upstream

plugin

|key                 |desc                              |type    |
|--------------------|----------------------------------|--------|
|echo                |                                  |        |
|grpc\-transcode     |                                  |        |
|jwt                 |                                  |验证    |
|key\-auth           |                                  |验证    |
|basic\-auth         |                                  |验证    |
|cros                |跨域                              |        |
|ip\-restriction     |                                  |安全    |
|ua\-restriction     |过滤UA的，可以过滤掉一些爬虫      |安全    |
|referer\-restriction|                                  |安全    |
|limit\-count        |根据时间窗口内，限制请求次数      |安全    |
|limit\-req          |根据时间窗口内，请求频率          |        |
|limit\-conn         |根据时间窗口内，限制连接数\(并发\)|        |
|ip\-restriction     |IP 黑/白名单                      |        |
|api\-breaker        |熔断                              |        |
|traffic\-split      |分流                              |流控    |
|request\-id         |                                  |        |
|prometheus          |开放HTTP接口等待对方来抓metrics   |监控统计|
|zipkin              |链路追踪，得配合request\-id       |监控统计|
|log\-rotate         |对本地磁盘日志文件动态切割        |日志    |
|tcp\-logger         |将日志主动推送3方                 |日志    |
|traffic\-split      |分流                              |流控    |

关于安全，可以将几个合起来使用：

单个节点，并发不能超10个，每秒最多60次请求，10秒最多500个请求，且UA正常，不在IP黑名单里，且key\-auth正常

基本上能过滤掉所有初级的黑客的DDOS攻击了，另外就是粒度能小到UID 就完美了（不过网关层也不应该关注UID）

验证插件：

略有点特殊，他得绑定一个consumer，并不是应用层理解的，JWT转出UID，所以，它的作用更像是：不允许所有请求都可以访问网关，算是个初级防御吧。综合看下来：key\-auth更适合些

插件

“remote\_addr”

“server\_addr”

“http\_x\_real\_ip”

“http\_x\_forwarded\_for”

对于IP做安全有个问题：

他是挂在ROUTER上的，那么，如果一个服务后面挂了4台机器，等于这个限流是直接针对4台机器的了。。。

limit\-req

限制请求速率

1秒内只可以访问一次，如果大于1 小于3会被延迟，如果大于3直接reject

同样也是基于IP，跟上面有相同问题

api\-breaker：配合check\-health，错误次数，熔断若干时间，最后达到 阀值直接断了，但是不确定check\-health的方式，是针对一组服务的所有机器，还是某个机器。

traffic\-split：可以做灰度发布，但还没研究明白

## upstream

一个服务器配置\(ip:port\)集合表，或者理解为：一组服务的集群

负载均衡也是在这里做的，目前支持如下几种

|key        |dsec                                          |
|-----------|----------------------------------------------|
|roundrobin |轮询，带权重                                  |
|chash      |一致性哈希                                    |
|ewma       |选择延迟最小的节点                            |
|least\_conn|选择 \(active\_conn \+ 1\) / weight 最小的节点|
|balancer   |用户自定义的                                  |

> 感觉也够用

1. 轮询，带权重
2. 一致性哈希

服务发现

Eureka / Consul

discovery\_type

health\-check

健康检查

有点鸡肋，如果启用服务发现，这功能没啥用。。。

另外，只有有请求后health\-check才会触发

retries

scheme：http https grpc

echo

## 服务发现与治理

Eureka consul dns

dns:忽略，负载策略单一，部署成本高

Eureka:忽略，JAVA的东西

consul:主要分析这个

## 总结

感觉网关还是挺有用的，间接反映以前对网关认识不足，之前很多在应用层做的事情完全都可以抽离出来放在网关层统一处理。

如果对apisix做个简短定义： 就是个 nginx 升级版.

## etcd 开启grpc getway proxy

> ./etcd grpc\-proxy start \-\-endpoints=http://127.0.0.1:2378 \-\-listen\-addr=127.0.0.1:2379

%E5%88%9D%E8%A1%B7%2F%E7%9B%AE%E7%9A%84%0A\-\-\-\-\-%0A%E5%AF%BB%E6%89%BE%E4%B8%80%E4%B8%AA%E5%BC%80%E6%BA%90%E8%BD%AF%E4%BB%B6%EF%BC%9A%E5%AF%B9%E5%A4%96%E7%9A%84%E5%85%AC%E5%85%B1%E7%BD%91%E5%85%B3%0A%0A%0A%E5%8E%9F%E5%9B%A0%0A\-\-\-\-%0A%E5%9B%A0%E4%B8%BAlinkerd%E6%B2%A1%E6%9C%89%E9%99%90%E9%87%8F%20%E9%BB%91%E5%90%8D%E5%8D%95%E7%AD%89%E5%8A%9F%E8%83%BD%EF%BC%8C%E6%88%96%E8%80%85%E8%AF%B4linkerd%E5%B0%B1%E4%B8%8D%E6%98%AF%E5%AF%B9%E5%A4%96%E5%85%AC%E5%85%B1%E7%BD%91%E5%85%B3%EF%BC%8C%E4%BB%96%E7%9A%84%E4%BE%A7%E9%87%8D%E7%82%B9%E6%9B%B4%E5%A4%9A%E6%98%AF%E6%9C%8D%E5%8A%A1%E9%97%B4%E7%9A%84%E6%B2%BB%E7%90%86%E3%80%82%E5%9B%A0%E6%AD%A4%EF%BC%8C%E8%BF%98%E5%BE%97%E6%89%BE%E4%B8%80%E4%B8%AAAPI%E5%85%AC%E5%85%B1%E7%BD%91%E5%85%B3%EF%BC%8C%E7%9C%8B%E4%BA%86%E4%B8%8B%E4%B8%BB%E6%B5%81%E7%9A%84%EF%BC%9Akong%20kyt%20APISIX%20Zuul%0A%0AZuul%E9%A6%96%E5%85%88%E6%B7%98%E6%B1%B0%EF%BC%8CJAVA%E4%BD%93%E7%B3%BB%E9%87%8C%E7%9A%84%EF%BC%8C1%E6%98%AF%E9%87%8D%202%E6%98%AF%E5%B0%9D%E8%AF%95%E6%8D%86%E7%BB%91JAVA%0Akyt%EF%BC%9AGOLANG%E7%BC%96%E5%86%99%EF%BC%8C%E4%BD%86%E6%98%AF%E9%AB%98%E7%BA%A7%E5%8A%9F%E8%83%BD%E6%94%B6%E8%B4%B9%0Akong%3A%E7%BB%91%E5%AE%9Apostgresql%20DB%2C%E4%B8%8D%E6%94%AF%E6%8C%81GRPC%20WEEBSOCKET%20MQTT%2C%E5%AF%B9%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E4%B9%9F%E4%B8%8D%E5%A4%AA%E5%8F%8B%E5%A5%BD%0A%0A%E6%9C%80%E5%90%8E%E6%84%8F%E5%A4%96%E5%8F%91%E7%8E%B0%E4%BA%86%E4%B8%AA%EF%BC%9AAPISIX%0A%0A%0A%E9%9C%80%E6%B1%82%0A\-\-\-\-%0A1.%20%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E4%B8%8E%E6%B2%BB%E7%90%86%2Cconsul%0A2.%20%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%0A3.%20%E7%86%94%E6%96%AD%0A4.%20%E9%99%90%E5%88%B6%EF%BC%8C%E9%80%9F%E5%BA%A6%20%E8%AF%B7%E6%B1%82%E6%95%B0%20%E5%B9%B6%E5%8F%91%0A4.%20%E9%BB%91%E5%90%8D%E5%8D%95\(ip%2Fuid\)%0A5.%20%E6%94%AF%E6%8C%81http%20https%20grpc%20websocket%0A6.%20%E9%AA%8C%E8%AF%81%EF%BC%8CJWT%20OAUTH%0A7.%20%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%20%EF%BC%8CzipKin%0A8.%20%E5%8F%AF%E8%A7%86%E5%8C%96%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%EF%BC%8C%E5%90%8C%E6%97%B6%E6%94%AF%E6%8C%81HTTP%E7%AE%A1%E7%90%86%0A9.%20%E6%97%A5%E5%BF%97%EF%BC%8Cprometheus%0A10.%20%E5%85%BC%E5%AE%B9%E6%97%A7%E4%BB%A3%E7%A0%81%EF%BC%8C%E4%BB%A3%E7%A0%81%E4%BE%B5%E5%85%A5%0A%0A%E6%A0%B8%E5%BF%83%E9%9C%80%E6%B1%82%0A\-\-\-\-%0A1.%20%E9%98%B2%E6%94%BB%E5%87%BB%0A2.%20%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E4%B8%8E%E6%B2%BB%E7%90%86%0A%0AAPISIX%0A\-\-\-\-%0A%0A%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%0A%3Ehttps%3A%2F%2Fapisix.apache.org%2Fzh%2Fdocs%2Fapisix%2Fgetting\-started%0A%0A%E4%B8%AD%E6%96%87%E6%96%87%E6%A1%A3%0A%3Ehttps%3A%2F%2Fwww.bookstack.cn%2Fread%2Fapache\-apisix\-1.5\-zh%2Fa2c94bb1dcdc1a83.md%0A%0A%E8%8B%B1%E6%96%87%E6%96%87%E6%A1%A3%0A%3Ehttps%3A%2F%2Fapisix.apache.org%2Fdocs%2Fapisix%2Fdiscovery%2Fconsul\_kv%0A%0A%E5%85%B6%E5%AE%9E%E5%AE%83%E6%9C%89%E7%82%B9%E6%98%AFKONG%E7%9A%84%E5%8D%87%E7%BA%A7%E7%89%88%EF%BC%8C%E5%BA%95%E5%B1%82%E4%BE%9D%E7%84%B6%E6%98%AF%EF%BC%9ANGINX%2BOPENRESTY%EF%BC%8C%E5%8F%AA%E6%98%AF%E6%94%BE%E5%BC%83%E4%BA%86postgreSql%EF%BC%8C%E6%8D%A2%E6%88%90%E4%BA%86ETCD%EF%BC%8C%E5%90%8C%E6%97%B6%E4%BC%98%E5%8C%96%E4%BA%86%E4%B8%8B%E7%BB%93%E6%9E%84%EF%BC%8C%E4%BB%A3%E7%A0%81%E6%9B%B4%E5%B0%91%EF%BC%8C%E5%AF%B9%E4%B8%80%E4%BA%9B%E4%B8%BB%E6%B5%81%E7%9A%84%E8%BD%AF%E4%BB%B6%E6%94%AF%E6%8C%81%E7%95%A5%E5%8F%8B%E5%A5%BD%EF%BC%8C%E4%B8%94%E6%89%A9%E5%B1%95%E8%87%AA%E7%94%B1%E5%BA%A6%E6%9B%B4%E9%AB%98%0A%0A%3E%E5%85%B7%E8%AF%B4%E6%80%A7%E8%83%BD%E6%AF%94KONG%E8%87%B3%E5%B0%91%E9%AB%982%E5%80%8D%E4%BB%A5%E4%B8%8A.%0A%0A%E8%BF%99%E4%B8%AA%E8%BD%AF%E4%BB%B6%E5%A5%BD%E5%83%8F%E6%98%AF2019.6%E6%9C%88%E6%89%8D%E5%BC%80%E6%BA%90%EF%BC%8C%E7%A8%B3%E5%AE%9A%E6%80%A7%E5%AF%B9%E6%AF%94kong%20%E5%BA%94%E8%AF%A5%E7%95%A5%E5%B7%AE%E4%BA%9B%EF%BC%8C%E6%AF%95%E7%AB%9F%EF%BC%8CKONG%E8%BF%90%E8%A1%8C%E4%BA%86%E5%A5%BD%E5%A4%9A%E5%B9%B4%E3%80%82%E4%B8%8D%E8%BF%87%EF%BC%8C2021%E5%B9%B4%E7%9A%84%E6%88%91%E4%BB%AC%EF%BC%8C%E5%86%8D%E7%94%A8%E7%89%88%E6%9C%AC%E5%B7%B2%E7%BB%8F%E5%88%B0%E4%BA%862.11%20%E5%BA%94%E8%AF%A5%E5%85%8D%E5%8E%BB%E4%BA%86%E8%BF%99%E4%B8%AA%E9%BA%BB%E7%83%A6%E3%80%82%0A%0A%E6%99%AE%E9%80%9A%E5%AE%89%E8%A3%85%0A\-\-\-\-%0A%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E7%94%A8%E7%9A%84%E6%98%AFdocker%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%88%91%E6%98%AF%E6%B5%8B%E8%AF%95%EF%BC%8C%E5%B0%B1%E6%8D%A2%E4%BA%86%E6%99%AE%E9%80%9A%E7%9A%84yum%E6%96%B9%E5%BC%8F%0A%0A%E5%85%88%E7%9C%8B%E4%B8%80%E4%B8%8B%E4%BE%9D%E8%B5%96%E5%8C%85%EF%BC%9Aopen\-restry%20%2Cetcd%0A%0A%3E%E4%BB%85%E9%99%90%20centos7%0A%0A%E6%8A%98%E8%85%BE%E4%B8%80%E5%9C%88%E4%B8%8B%E6%9D%A5%EF%BC%8C%E9%81%87%E5%88%B0%E5%90%84%E7%A7%8D%E9%97%AE%E9%A2%98%EF%BC%8C%E6%9C%80%E5%A4%A7%E9%97%AE%E9%A2%98%E6%98%AF%EF%BC%9Aopen\-restry%20%E5%85%BC%E5%AE%B9%E6%80%A7%0Aetcd%E4%B9%9F%E5%BE%97%E6%9C%89%E4%BF%AE%E6%94%B9%E7%82%B9%E5%8F%82%E6%95%B0%EF%BC%9A%0A%0A%3Eenable\-v2%3A%20true%20%23%E5%A6%82%E6%9E%9C%E7%94%A8V2%E7%89%88%E6%9C%AC%EF%BC%8C%E4%BD%BF%E7%94%A8HTTP%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C%E5%BE%97%E6%89%93%E5%BC%80%EF%BC%8C%E4%B8%8D%E7%84%B6404%0A%3Eenable\-grpc\-gateway%3A%20true%0A%0A%E6%AD%A3%E5%BC%8F%E5%AE%89%E8%A3%85%20apisix%0A%0A%E5%AE%98%E6%96%B9%E7%94%A8%E7%9A%84%E6%98%AFyum%EF%BC%8C3%E6%96%B9%E6%98%AF%E4%B8%8B%E5%8C%85%0A%3Ewget%20https%3A%2F%2Fdownloads.apache.org%2Fapisix%2F2.11.0%2Fapache\-apisix\-2.11.0\-src.tgz%0A%0Atar%20\-zxvf%20%0Amv%0A%0A%E6%B7%BB%E5%8A%A0lua%E4%BE%9D%E8%B5%96%E5%BA%93%0A%3Emake%20deps%0A%3ELUAROCKS\_SERVER%3Dhttps%3A%2F%2Fluarocks.cn%20make%20deps%0A%0A%E6%8A%A5%E9%94%99%EF%BC%8C%E5%BE%97%E5%AE%89%E8%A3%85%20luarocks%0A%3Eyum%20install%20\-y%20luarocks%20lua\-devel%20lualdap%0A%0A%E9%87%8D%E8%A3%85%E6%89%A7%E8%A1%8C%EF%BC%8C%E5%BC%80%E5%A7%8B%E4%B8%8B%E8%BD%BD%E4%BA%86%E3%80%82%E3%80%82%E3%80%82%E6%97%B6%E9%97%B4%E8%BF%98%E6%8C%BA%E9%95%BF.....%0A%0A%E6%8A%A5%E9%94%99%EF%BC%8C%E5%BE%97%E5%AE%89%E8%A3%85ldap%0A%3Eyum%20update%0A%0A%3Eyum%20\-y%20install%20openldap%20compat\-openldap%20openldap\-clients%20openldap\-servers%20openldap\-servers\-sql%20openldap\-devel%0A%0A%E5%A5%BD%E4%BA%86....%0A%0A%E6%89%A7%E8%A1%8C%EF%BC%8Clua%20http%20%E5%BA%93%E5%AD%98%E6%9C%89%E9%97%AE%E9%A2%98%EF%BC%8C%E6%94%BE%E5%BC%83%E4%BA%86%EF%BC%8C%E6%94%B9%E7%94%A8docker%E5%90%A7%0A%0A%0A%E5%86%8D%E6%AC%A1%E5%B0%9D%E8%AF%95%0A\-\-%0A%3Eyum%20\-y%20install%20yum\-utils%0A%0Aopenrestry%0A%3Eyum%20install%20\-y%20https%3A%2F%2Frepos.apiseven.com%2Fpackages%2Fcentos%2Fapache\-apisix\-repo\-1.0\-1.noarch.rpm%0A%0A%3Eyum\-config\-manager%20\-\-add\-repo%20https%3A%2F%2Frepos.apiseven.com%2Fpackages%2Fcentos%2Fapache\-apisix.repo%0A%0A%0A%3Esudo%20yum%20install%20apisix%0A%0A%E7%BB%88%E4%BA%8E%E6%88%90%E5%8A%9F%E4%BA%86%E3%80%82%E3%80%82%E3%80%82%E3%80%82%0A%0A%E5%AF%B9%E6%AF%94%E4%BA%86%E4%B8%8B%E4%BB%96%E7%9A%84%E5%87%A0%E4%B8%AA%E4%BE%9D%E8%B5%96%E5%8C%85%0A%0A%60%60%60%0Aapisix\-base%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20x86\_64%20%20%20%20%20%20%20%20%20%20%201.19.3.2.2\-0.el7%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20release%20%20%20%20%20%20%20%20%20%20%20%20%20%2035%20M%0A%20openresty\-openssl111%20%20%20%20%20%20%20%20%20%20%20x86\_64%20%20%20%20%20%20%20%20%20%20%201.1.1l\-1.el7%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20openresty%20%20%20%20%20%20%20%20%20%20%201.5%20M%0A%20openresty\-pcre%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20x86\_64%20%20%20%20%20%20%20%20%20%20%208.44\-1.el7%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20openresty%20%20%20%20%20%20%20%20%20%20%20164%20k%0A%20openresty\-zlib%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20x86\_64%20%20%20%20%20%20%20%20%20%20%201.2.11\-3.el7.centos%20%20%20%20%20%20%20%20%20%20%20%20%20%20openresty%20%20%20%20%20%20%20%20%20%20%20%2054%20k%0A%0A%60%60%60%0A%0A%E9%BB%98%E8%AE%A4%E5%AE%89%E8%A3%85%E5%9C%A8%2Fusr%2Flocal%E4%B8%8B%0A%0Aapisix%20start%0Aapisix%20test%0Aapisix%20init%0Aapisix%20init\_etcd%0A%23%E4%BC%98%E9%9B%85%E5%81%9C%E6%AD%A2%0Aapisix%20quit%0A%23%E5%BC%BA%E5%88%B6%E5%81%9C%E6%AD%A2%0Aapisix%20stop%20%0A%0A%0A%E7%9E%AC%E9%97%B4%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F...%20%20%E7%9C%9F%E4%B8%8D%E7%9F%A5%E9%81%93%E5%89%8D%E5%A4%A9%E6%8A%98%E8%85%BE%E5%88%B03%E7%82%B9%E5%9B%BE%E5%95%A5...%0A%0A%E6%9C%AC%E6%9C%BA%E5%86%8D%E5%AE%89%E8%A3%85%E4%B8%80%E4%B8%AAnginx%2080%20%E5%92%8C%20apache%208088%20%E7%94%A8%E4%BA%8E%E6%B5%8B%E8%AF%95%0A%0A%E5%85%88%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAUpstream%0A%60%60%60%0Acurl%20http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Fupstreams%2F10%20\-X%20PUT%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%20\-d%20'%0A%7B%0A%20%20%22type%22%3A%20%22roundrobin%22%2C%0A%20%20%22nodes%22%3A%20%7B%0A%20%20%20%20%22127.0.0.1%3A80%22%3A%201%2C%0A%20%20%20%20%22127.0.0.1%3A8088%22%3A%201%0A%20%20%7D%0A%7D'%0A%60%60%60%0A%E6%9F%A5%E7%9C%8B%20%0A%3Ecurl%20%20http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Fupstreams%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%0A%0A%E5%88%9B%E5%BB%BArouter%E5%B9%B6%E7%BB%91%E5%AE%9Aupstream%0A%60%60%60%0Acurl%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Froutes%2F1%22%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%20\-X%20PUT%20\-d%20'%0A%7B%0A%20%20%22uri%22%3A%20%22%2Fapisix%2F%22%2C%0A%20%20%22host%22%3A%20%22service1%22%2C%0A%20%20%22upstream\_id%22%3A%20%2210%22%0A%7D'%0A%60%60%60%0A%E9%AA%8C%E8%AF%81%0A%3Ecurl%20\-i%20\-X%20GET%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2F%22%20\-H%20%22Host%3A%20service1%22%0A%0A%0A%E5%81%9A%E4%B8%8B%E5%9F%BA%E7%A1%80%E9%AA%8C%E8%AF%81%E7%9A%84%E6%B5%8B%E8%AF%95%20%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAconsumer%0A%60%60%60%0Acurl%20http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Fconsumers%20\-H%20'X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1'%20\-X%20PUT%20\-d%20'%0A%7B%0A%20%20%20%20%22username%22%3A%20%22service1\_user%22%2C%0A%20%20%20%20%22plugins%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22key\-auth%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22key%22%3A%20%22auth\-one%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D'%0A%60%60%60%0A%0A%E4%BF%AE%E6%94%B9%E8%B7%AF%E7%94%B1%0A%60%60%60%0Acurl%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Froutes%2F1%22%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%20\-X%20PUT%20\-d%20'%0A%7B%0A%20%20%20%20%22plugins%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22key\-auth%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22header%22%3A%20%22ser\_key%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%2C%0A%20%20%22uri%22%3A%20%22%2Fapisix%2F%22%2C%0A%20%20%22host%22%3A%20%22service1%22%2C%0A%20%20%22upstream\_id%22%3A%20%2210%22%0A%7D'%0A%60%60%60%0A%0A%0A%0A%0A%3Ecurl%20\-i%20\-X%20GET%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2F%22%20\-H%20%22Host%3A%20service1%22%20\-H%20%22ser\_key%3Aauth\-one%22%0A%0A%E8%BF%98%E6%9C%89%EF%BC%9Ajwt%20base\-auth%20%E4%B8%8D%E6%B5%8B%E4%BA%86%EF%BC%8C%E6%9C%89%E7%82%B9%E9%B8%A1%E8%82%8B%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%B5%AA%E8%B4%B9%E7%BD%91%E5%85%B3%E8%AE%A1%E7%AE%97%E6%97%B6%E9%97%B4%0A%0A%0A%0Adocker%E5%AE%89%E8%A3%85%0A\-\-\-%0A%0A%E5%B0%86%20Apache%20APISIX%20%E7%9A%84%20Docker%20%E9%95%9C%E5%83%8F%E4%B8%8B%E8%BD%BD%E5%88%B0%E6%9C%AC%E5%9C%B0%0A%3Egit%20clone%20https%3A%2F%2Fgithub.com%2Fapache%2Fapisix\-docker.git%0A%0A%E5%B0%86%E5%BD%93%E5%89%8D%E7%9A%84%E7%9B%AE%E5%BD%95%E5%88%87%E6%8D%A2%E5%88%B0%20apisix\-docker%2Fexample%20%E8%B7%AF%E5%BE%84%E4%B8%8B%0A%3Ecd%20apisix\-docker%2Fexample%0A%0A%E8%BF%90%E8%A1%8C%20docker\-compose%20%E5%91%BD%E4%BB%A4%EF%BC%8C%E5%AE%89%E8%A3%85%20Apache%20APISIX%0A%3Edocker\-compose%20\-p%20docker\-apisix%20up%20\-d%0A%0A%E8%BF%98%E6%98%AF%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%EF%BC%8Cdocker%20%E5%AE%89%E8%A3%85%E9%9D%A0%E8%B0%B1%EF%BC%8C%E5%88%86%E5%88%86%E9%92%9F%E6%90%9E%E5%AE%9A%E4%BA%86...%0A%0A%E7%9C%8B%E4%BA%86%E4%B8%8B%E5%AE%83%E7%9A%84%E7%BC%96%E6%8E%92%E6%96%87%E4%BB%B6%EF%BC%8C%E9%87%8C%E9%9D%A2%E5%8C%85%E5%90%AB%E7%9A%84%E4%B8%9C%E8%A5%BF%E8%BF%98%E6%8C%BA%E5%A4%9A%0A1.%20nginx\(%E4%B8%A4%E4%B8%AA\)%209081%209082%0A2.%20apisix%209080%209091%209443%209092%0A3.%20prometheus%209090%0A4.%20etcd%202379%0A5.%20grafana%203000%0A6.%20apisix\-dashboard%209000%0A%0A%E5%81%9C%E6%AD%A2%20%0A%3E%3Edocker\-compose%20\-p%20docker\-apisix%20stop%0A%0A%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%3Ecurl%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Fservices%2F%22%20\-H%20'X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1'%0A%0A%E7%9C%8B%E4%B8%80%E4%B8%8Bdashboard%0A%3Ehttp%3A%2F%2F8.142.177.235%3A9000%2F%0A%0A%E5%88%9D%E6%AD%A5%E6%B5%8B%E8%AF%95%0A\-\-\-\-\-%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAupstreams%0A%60%60%60%0Acurl%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Fupstreams%2F1%22%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%20\-X%20PUT%20\-d%20'%0A%7B%0A%20%20%22type%22%3A%20%22roundrobin%22%2C%0A%20%20%22nodes%22%3A%20%7B%0A%20%20%20%20%22httpbin.org%3A80%22%3A%201%0A%20%20%7D%0A%7D'%0A%60%60%60%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AArouter%2C%E5%B9%B6%E7%BB%91%E5%AE%9A%E5%88%B0%E4%B8%8A%E9%9D%A2%E7%9A%84upstreams%E4%B8%8A%0A%60%60%60%0Acurl%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fapisix%2Fadmin%2Froutes%2F1%22%20\-H%20%22X\-API\-KEY%3A%20edd1c9f034335f136f87ad84b625c8f1%22%20\-X%20PUT%20\-d%20'%0A%7B%0A%20%20%22uri%22%3A%20%22%2Fget%22%2C%0A%20%20%22host%22%3A%20%22httpbin.org%22%2C%0A%20%20%22upstream\_id%22%3A%20%221%22%0A%7D'%0A%60%60%60%0A%0A%E9%AA%8C%E8%AF%81%0A%3Ecurl%20\-i%20\-X%20GET%20%22http%3A%2F%2F127.0.0.1%3A9080%2Fget%3Ffoo1%3Dbar1%26foo2%3Dbar2%22%20\-H%20%22Host%3A%20httpbin.org%22%0A%0A%0A%E6%95%B0%E6%8D%AE%E6%B5%81%E8%BD%AC%0A\-\-\-\-%0Areq\-%3Eetcd\-%3Erouter\-%3Eplugin\-%3Efilter\-%3Efilter\-plugin\-%3Eupstream%0A\-%3Eplugin\-%3Ereal\-service%0A%0A%E6%A0%B8%E5%BF%83%EF%BC%9Arouter%20upstream%20plguins%0A%E6%88%91%E4%BB%AC%E7%9C%8B%E5%AE%9E%E9%99%85%E5%B0%B1%E6%98%AF%E9%80%9A%E8%BF%87plugins%E5%B0%86router%20upstream%E4%B8%B2%E8%81%94%E8%B5%B7%EF%BC%8C%E6%9C%80%E5%90%8E%E6%89%BE%E5%88%B0%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%0A%0Arouter%0A\-\-\-\-%0A%E5%8A%9F%E8%83%BD%EF%BC%9A%E9%80%9A%E8%BF%87%20req%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%89%BE%E5%88%B0%EF%BC%8Crouter%EF%BC%8C%E5%86%8D%20%E6%89%BE%E5%88%B0upstream%0A%0A%E5%85%88%E6%A0%B9%E6%8D%AE%E7%94%A8%E6%88%B7%E7%9A%84%E8%AF%B7%E6%B1%82%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%A6%82%EF%BC%9Aheader%20uri%20%EF%BC%8C%E5%8E%BBETCD%E9%87%8C%E5%8C%B9%E9%85%8D%EF%BC%8C%E6%89%BE%E5%88%B0%E4%B8%80%E4%B8%AArouter%EF%BC%8C%E7%84%B6%E5%90%8E%E6%A0%B9%E6%8D%AE%E8%AF%A5%E6%9D%A1router%E7%9A%84%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%89%A7%E8%A1%8C%E6%8F%92%E4%BB%B6%EF%BC%8C%E6%9C%80%E5%90%8E%E6%89%BE%E5%88%B0%E4%B8%80%E4%B8%AAupstream%0A%0Aplugin%0A%0A%7C%20key%20%7C%20desc%20%7Ctype%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20echo%20%7C%20%20%7C%20%20%7C%0A%7C%20grpc\-transcode%7C%20%20%7C%20%20%7C%0A%7C%20jwt%20%7C%20%20%7C%20%E9%AA%8C%E8%AF%81%20%7C%0A%7C%20key\-auth%20%7C%20%20%7C%E9%AA%8C%E8%AF%81%20%20%7C%0A%7C%20basic\-auth%20%7C%20%20%7C%20%E9%AA%8C%E8%AF%81%20%7C%0A%7C%20cros%20%7C%E8%B7%A8%E5%9F%9F%20%20%7C%20%20%7C%0A%7C%20ip\-restriction%20%7C%20%20%7C%E5%AE%89%E5%85%A8%20%20%7C%0A%7C%20ua\-restriction%20%7C%20%20%E8%BF%87%E6%BB%A4UA%E7%9A%84%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%BF%87%E6%BB%A4%E6%8E%89%E4%B8%80%E4%BA%9B%E7%88%AC%E8%99%AB%7C%E5%AE%89%E5%85%A8%20%20%7C%0A%7C%20referer\-restriction%20%7C%20%20%7C%20%E5%AE%89%E5%85%A8%20%7C%0A%7C%20limit\-count%20%7C%20%20%E6%A0%B9%E6%8D%AE%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E5%86%85%EF%BC%8C%E9%99%90%E5%88%B6%E8%AF%B7%E6%B1%82%E6%AC%A1%E6%95%B0%7C%E5%AE%89%E5%85%A8%20%20%7C%0A%7Climit\-req%7C%E6%A0%B9%E6%8D%AE%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E5%86%85%EF%BC%8C%E8%AF%B7%E6%B1%82%E9%A2%91%E7%8E%87%7C%0A%7Climit\-conn%7C%20%E6%A0%B9%E6%8D%AE%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E5%86%85%EF%BC%8C%E9%99%90%E5%88%B6%E8%BF%9E%E6%8E%A5%E6%95%B0\(%E5%B9%B6%E5%8F%91\)%7C%7C%0A%7Cip\-restriction%7CIP%20%E9%BB%91%2F%E7%99%BD%E5%90%8D%E5%8D%95%7C%0A%7Capi\-breaker%7C%E7%86%94%E6%96%AD%7C%0A%7Ctraffic\-split%7C%E5%88%86%E6%B5%81%7C%E6%B5%81%E6%8E%A7%7C%0A%7Crequest\-id%7C%7C%7C%0A%7Cprometheus%7C%E5%BC%80%E6%94%BEHTTP%E6%8E%A5%E5%8F%A3%E7%AD%89%E5%BE%85%E5%AF%B9%E6%96%B9%E6%9D%A5%E6%8A%93metrics%7C%E7%9B%91%E6%8E%A7%E7%BB%9F%E8%AE%A1%7C%0A%7Czipkin%7C%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%EF%BC%8C%E5%BE%97%E9%85%8D%E5%90%88request\-id%7C%E7%9B%91%E6%8E%A7%E7%BB%9F%E8%AE%A1%7C%0A%7Clog\-rotate%7C%E5%AF%B9%E6%9C%AC%E5%9C%B0%E7%A3%81%E7%9B%98%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E5%8A%A8%E6%80%81%E5%88%87%E5%89%B2%7C%E6%97%A5%E5%BF%97%7C%0A%7Ctcp\-logger%7C%E5%B0%86%E6%97%A5%E5%BF%97%E4%B8%BB%E5%8A%A8%E6%8E%A8%E9%80%813%E6%96%B9%7C%E6%97%A5%E5%BF%97%7C%0A%7Ctraffic\-split%7C%E5%88%86%E6%B5%81%7C%E6%B5%81%E6%8E%A7%7C%0A%0A%0A%E5%85%B3%E4%BA%8E%E5%AE%89%E5%85%A8%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%B0%86%E5%87%A0%E4%B8%AA%E5%90%88%E8%B5%B7%E6%9D%A5%E4%BD%BF%E7%94%A8%EF%BC%9A%0A%E5%8D%95%E4%B8%AA%E8%8A%82%E7%82%B9%EF%BC%8C%E5%B9%B6%E5%8F%91%E4%B8%8D%E8%83%BD%E8%B6%8510%E4%B8%AA%EF%BC%8C%E6%AF%8F%E7%A7%92%E6%9C%80%E5%A4%9A60%E6%AC%A1%E8%AF%B7%E6%B1%82%EF%BC%8C10%E7%A7%92%E6%9C%80%E5%A4%9A500%E4%B8%AA%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%B8%94UA%E6%AD%A3%E5%B8%B8%EF%BC%8C%E4%B8%8D%E5%9C%A8IP%E9%BB%91%E5%90%8D%E5%8D%95%E9%87%8C%EF%BC%8C%E4%B8%94key\-auth%E6%AD%A3%E5%B8%B8%0A%0A%E5%9F%BA%E6%9C%AC%E4%B8%8A%E8%83%BD%E8%BF%87%E6%BB%A4%E6%8E%89%E6%89%80%E6%9C%89%E5%88%9D%E7%BA%A7%E7%9A%84%E9%BB%91%E5%AE%A2%E7%9A%84DDOS%E6%94%BB%E5%87%BB%E4%BA%86%EF%BC%8C%E5%8F%A6%E5%A4%96%E5%B0%B1%E6%98%AF%E7%B2%92%E5%BA%A6%E8%83%BD%E5%B0%8F%E5%88%B0UID%20%E5%B0%B1%E5%AE%8C%E7%BE%8E%E4%BA%86%EF%BC%88%E4%B8%8D%E8%BF%87%E7%BD%91%E5%85%B3%E5%B1%82%E4%B9%9F%E4%B8%8D%E5%BA%94%E8%AF%A5%E5%85%B3%E6%B3%A8UID%EF%BC%89%0A%0A%0A%E9%AA%8C%E8%AF%81%E6%8F%92%E4%BB%B6%EF%BC%9A%20%20%0A%E7%95%A5%E6%9C%89%E7%82%B9%E7%89%B9%E6%AE%8A%EF%BC%8C%E4%BB%96%E5%BE%97%E7%BB%91%E5%AE%9A%E4%B8%80%E4%B8%AAconsumer%EF%BC%8C%E5%B9%B6%E4%B8%8D%E6%98%AF%E5%BA%94%E7%94%A8%E5%B1%82%E7%90%86%E8%A7%A3%E7%9A%84%EF%BC%8CJWT%E8%BD%AC%E5%87%BAUID%EF%BC%8C%E6%89%80%E4%BB%A5%EF%BC%8C%E5%AE%83%E7%9A%84%E4%BD%9C%E7%94%A8%E6%9B%B4%E5%83%8F%E6%98%AF%EF%BC%9A%E4%B8%8D%E5%85%81%E8%AE%B8%E6%89%80%E6%9C%89%E8%AF%B7%E6%B1%82%E9%83%BD%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE%E7%BD%91%E5%85%B3%EF%BC%8C%E7%AE%97%E6%98%AF%E4%B8%AA%E5%88%9D%E7%BA%A7%E9%98%B2%E5%BE%A1%E5%90%A7%E3%80%82%E7%BB%BC%E5%90%88%E7%9C%8B%E4%B8%8B%E6%9D%A5%EF%BC%9Akey\-auth%E6%9B%B4%E9%80%82%E5%90%88%E4%BA%9B%0A%0A%E6%8F%92%E4%BB%B6%20%0A%0A%E2%80%9Cremote\_addr%E2%80%9D%0A%E2%80%9Cserver\_addr%E2%80%9D%0A%E2%80%9Chttp\_x\_real\_ip%E2%80%9D%0A%E2%80%9Chttp\_x\_forwarded\_for%E2%80%9D%0A%0A%E5%AF%B9%E4%BA%8EIP%E5%81%9A%E5%AE%89%E5%85%A8%E6%9C%89%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%9A%0A%E4%BB%96%E6%98%AF%E6%8C%82%E5%9C%A8ROUTER%E4%B8%8A%E7%9A%84%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%90%8E%E9%9D%A2%E6%8C%82%E4%BA%864%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%8C%E7%AD%89%E4%BA%8E%E8%BF%99%E4%B8%AA%E9%99%90%E6%B5%81%E6%98%AF%E7%9B%B4%E6%8E%A5%E9%92%88%E5%AF%B94%E5%8F%B0%E6%9C%BA%E5%99%A8%E7%9A%84%E4%BA%86%E3%80%82%E3%80%82%E3%80%82%0A%0Alimit\-req%0A%E9%99%90%E5%88%B6%E8%AF%B7%E6%B1%82%E9%80%9F%E7%8E%87%0A1%E7%A7%92%E5%86%85%E5%8F%AA%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE%E4%B8%80%E6%AC%A1%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A4%A7%E4%BA%8E1%20%E5%B0%8F%E4%BA%8E3%E4%BC%9A%E8%A2%AB%E5%BB%B6%E8%BF%9F%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A4%A7%E4%BA%8E3%E7%9B%B4%E6%8E%A5reject%0A%E5%90%8C%E6%A0%B7%E4%B9%9F%E6%98%AF%E5%9F%BA%E4%BA%8EIP%EF%BC%8C%E8%B7%9F%E4%B8%8A%E9%9D%A2%E6%9C%89%E7%9B%B8%E5%90%8C%E9%97%AE%E9%A2%98%0A%0Aapi\-breaker%EF%BC%9A%E9%85%8D%E5%90%88check\-health%EF%BC%8C%E9%94%99%E8%AF%AF%E6%AC%A1%E6%95%B0%EF%BC%8C%E7%86%94%E6%96%AD%E8%8B%A5%E5%B9%B2%E6%97%B6%E9%97%B4%EF%BC%8C%E6%9C%80%E5%90%8E%E8%BE%BE%E5%88%B0%20%E9%98%80%E5%80%BC%E7%9B%B4%E6%8E%A5%E6%96%AD%E4%BA%86%EF%BC%8C%E4%BD%86%E6%98%AF%E4%B8%8D%E7%A1%AE%E5%AE%9Acheck\-health%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C%E6%98%AF%E9%92%88%E5%AF%B9%E4%B8%80%E7%BB%84%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%89%80%E6%9C%89%E6%9C%BA%E5%99%A8%EF%BC%8C%E8%BF%98%E6%98%AF%E6%9F%90%E4%B8%AA%E6%9C%BA%E5%99%A8%E3%80%82%0A%0Atraffic\-split%EF%BC%9A%E5%8F%AF%E4%BB%A5%E5%81%9A%E7%81%B0%E5%BA%A6%E5%8F%91%E5%B8%83%EF%BC%8C%E4%BD%86%E8%BF%98%E6%B2%A1%E7%A0%94%E7%A9%B6%E6%98%8E%E7%99%BD%0A%0A%0Aupstream%0A\-\-\-\-\-\-\-\-%0A%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AE\(ip%3Aport\)%E9%9B%86%E5%90%88%E8%A1%A8%EF%BC%8C%E6%88%96%E8%80%85%E7%90%86%E8%A7%A3%E4%B8%BA%EF%BC%9A%E4%B8%80%E7%BB%84%E6%9C%8D%E5%8A%A1%E7%9A%84%E9%9B%86%E7%BE%A4%0A%0A%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E4%B9%9F%E6%98%AF%E5%9C%A8%E8%BF%99%E9%87%8C%E5%81%9A%E7%9A%84%EF%BC%8C%E7%9B%AE%E5%89%8D%E6%94%AF%E6%8C%81%E5%A6%82%E4%B8%8B%E5%87%A0%E7%A7%8D%0A%7C%20key%20%7C%20dsec%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7Croundrobin%20%20%7C%E8%BD%AE%E8%AF%A2%EF%BC%8C%E5%B8%A6%E6%9D%83%E9%87%8D%20%20%7C%0A%7C%20chash%20%7C%E4%B8%80%E8%87%B4%E6%80%A7%E5%93%88%E5%B8%8C%20%20%7C%0A%7C%20ewma%20%7C%E9%80%89%E6%8B%A9%E5%BB%B6%E8%BF%9F%E6%9C%80%E5%B0%8F%E7%9A%84%E8%8A%82%E7%82%B9%20%20%7C%0A%7C%20least\_conn%20%7C%E9%80%89%E6%8B%A9%20\(active\_conn%20%2B%201\)%20%2F%20weight%20%E6%9C%80%E5%B0%8F%E7%9A%84%E8%8A%82%E7%82%B9%20%20%7C%0A%7C%20balancer%7C%20%E7%94%A8%E6%88%B7%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%20%7C%0A%0A%3E%E6%84%9F%E8%A7%89%E4%B9%9F%E5%A4%9F%E7%94%A8%0A%0A%0A1.%20%E8%BD%AE%E8%AF%A2%EF%BC%8C%E5%B8%A6%E6%9D%83%E9%87%8D%0A2.%20%E4%B8%80%E8%87%B4%E6%80%A7%E5%93%88%E5%B8%8C%0A%0A%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%0AEureka%20%2F%20Consul%20%0A%0Adiscovery\_type%0A%0A%0Ahealth\-check%0A%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5%0A%E6%9C%89%E7%82%B9%E9%B8%A1%E8%82%8B%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%90%AF%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%EF%BC%8C%E8%BF%99%E5%8A%9F%E8%83%BD%E6%B2%A1%E5%95%A5%E7%94%A8%E3%80%82%E3%80%82%E3%80%82%0A%E5%8F%A6%E5%A4%96%EF%BC%8C%E5%8F%AA%E6%9C%89%E6%9C%89%E8%AF%B7%E6%B1%82%E5%90%8Ehealth\-check%E6%89%8D%E4%BC%9A%E8%A7%A6%E5%8F%91%0A%0Aretries%0Ascheme%EF%BC%9Ahttp%20https%20grpc%0Aecho%0A%0A%0A%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E4%B8%8E%E6%B2%BB%E7%90%86%0A\-\-\-\-%0AEureka%20consul%20dns%0Adns%3A%E5%BF%BD%E7%95%A5%EF%BC%8C%E8%B4%9F%E8%BD%BD%E7%AD%96%E7%95%A5%E5%8D%95%E4%B8%80%EF%BC%8C%E9%83%A8%E7%BD%B2%E6%88%90%E6%9C%AC%E9%AB%98%0AEureka%3A%E5%BF%BD%E7%95%A5%EF%BC%8CJAVA%E7%9A%84%E4%B8%9C%E8%A5%BF%0Aconsul%3A%E4%B8%BB%E8%A6%81%E5%88%86%E6%9E%90%E8%BF%99%E4%B8%AA%0A%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-\-\-%0A%E6%84%9F%E8%A7%89%E7%BD%91%E5%85%B3%E8%BF%98%E6%98%AF%E6%8C%BA%E6%9C%89%E7%94%A8%E7%9A%84%EF%BC%8C%E9%97%B4%E6%8E%A5%E5%8F%8D%E6%98%A0%E4%BB%A5%E5%89%8D%E5%AF%B9%E7%BD%91%E5%85%B3%E8%AE%A4%E8%AF%86%E4%B8%8D%E8%B6%B3%EF%BC%8C%E4%B9%8B%E5%89%8D%E5%BE%88%E5%A4%9A%E5%9C%A8%E5%BA%94%E7%94%A8%E5%B1%82%E5%81%9A%E7%9A%84%E4%BA%8B%E6%83%85%E5%AE%8C%E5%85%A8%E9%83%BD%E5%8F%AF%E4%BB%A5%E6%8A%BD%E7%A6%BB%E5%87%BA%E6%9D%A5%E6%94%BE%E5%9C%A8%E7%BD%91%E5%85%B3%E5%B1%82%E7%BB%9F%E4%B8%80%E5%A4%84%E7%90%86%E3%80%82%0A%0A%E5%A6%82%E6%9E%9C%E5%AF%B9apisix%E5%81%9A%E4%B8%AA%E7%AE%80%E7%9F%AD%E5%AE%9A%E4%B9%89%EF%BC%9A%20%E5%B0%B1%E6%98%AF%E4%B8%AA%20nginx%20%E5%8D%87%E7%BA%A7%E7%89%88.%0A%0A%0Aetcd%20%E5%BC%80%E5%90%AFgrpc%20getway%20proxy%0A\-\-\-\-%0A%3E.%2Fetcd%20grpc\-proxy%20start%20\-\-endpoints%3Dhttp%3A%2F%2F127.0.0.1%3A2378%20\-\-listen\-addr%3D127.0.0.1%3A2379%0A%0A%0A%0A
