# etcd

https://github.com/etcd\-io/etcd/releases

解压后，两个主要的文件：

1、etcd

2、etcdctl

先启动守护进程,看下效果

> ./etcd

配置文件

> mkdir config
> 
> 
> cd config
> 
> 
> touch et\_m1.yml et\_m2.yml et\_m3.yml

数据目录

> mkdir /soft/etcd/data/
> 
> 
> cd /soft/etcd/data/
> 
> 
> mkdir et\_m1 et\_m2 et\_m3
> 
> 
> chmod 777 /soft/etcd/data
> 
> 
> chmod \-R 700 /soft/etcd/data/\*

> vim m1.yml

```

name: "et_m1"
data-dir: "/soft/etcd/data/et_m1"
log-level: "debug"

listen-peer-urls: 'http://172.26.150.7:2380'
listen-client-urls: 'http://0.0.0.0:2379'
initial-advertise-peer-urls: 'http://172.26.150.7:2380'
advertise-client-urls: 'http://0.0.0.0:2379'

initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster'

initial-cluster: 'et_m1=http://172.26.150.7:2380,et_m2=http://172.26.150.7:2382,et_m3=http://172.26.150.7:2384'

enable-v2: true #如果用V2版本，使用HTTP的方式，得打开，不然404

```

> 如果间单机，此配置必须得有：listen\-client\-urls: 'http://0.0.0.0:2379'

2379：CLIENT、HTTP API 端口

2380：各集群节点通信端口

启动

./etcd \-\-config\-file=/soft/etcd/config/cfg.conf

重启启动

> cp /soft/etcd/etcd /usr/local/bin
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m1.yml
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m2.yml
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m3.yml

http v2 测试，添加一个数据

> curl http://39.106.65.76:2379/v2/keys/message \-X PUT \-d value="Hello world"

http v2 测试，获取一个数据

> curl http://39.106.65.76:2379/v2/keys/message

etcdctl ，指令行的日常操作，跟HTTP API 一样

> ./etcdctl put test 1111
> 
> 
> ./etcdctl get test

在使用 etcdctl ，有个V2 V3切换

> export ETCDCTL\_API=2
> 
> 
> export ETCDCTL\_API=3

新的版本ETCD均是V3\-API，使用了GRPC，具说性能提升1/3，最可恨的是V3不兼容V2，不能共存，V3：是放弃了HTTP，全用GRPC，如果非要使用HTTP，可以再搞个grpc getway ，用HTTP\+JSON发送过来，中转下

## etcd 添加服务

touch /usr/lib/systemd/system/etcd.service

vim /usr/lib/systemd/system/etcd.service

```
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network-online.target local-fs.target remote-fs.target time-sync.target
Wants=network-online.target local-fs.target remote-fs.target time-sync.target

[Service]
User=root
Type=notify
#Environment=ETCD_DATA_DIR=/soft/etcd/data
#Environment=ETCD_NAME=%m

#WorkingDirectory=/soft/etcd/
#EnvironmentFile=-/soft/etcd/config/et_m1.yml
ExecStart=/soft/etcd/etcd --config-file=/soft/etcd/config/et_m1.yml

Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

systemctl enable etcd.service

Created symlink from /etc/systemd/system/multi\-user.target.wants/etcd.service to /usr/lib/systemd/system/etcd.service.

systemctl start etcd.service;

## 安全

查看下先

etcdctl user list

添加用户/设置密码

etcdctl user add root

开启验证功能

./etcdctl auth enable \-\-user='root'

角色与目录权限

这里先忽略掉吧

## 原理

```
Raft：etcd所采用的保证分布式系统强一致性的算法。
Node：一个Raft状态机实例。
Member： 一个etcd实例。它管理着一个Node，并且可以为客户端请求提供服务。
Cluster：由多个Member构成可以协同工作的etcd集群。
Peer：对同一个etcd集群中另外一个Member的称呼。
Client： 向etcd集群发送HTTP请求的客户端。
WAL：预写式日志，etcd用于持久化存储的日志格式。
snapshot：etcd防止WAL文件过多而设置的快照，存储etcd数据状态。
Proxy：etcd的一种模式，为etcd集群提供反向代理服务。
Leader：Raft算法中通过竞选而产生的处理所有数据提交的节点。
Follower：竞选失败的节点作为Raft中的从属节点，为算法提供强一致性保证。
Candidate：当Follower超过一定时间接收不到Leader的心跳时转变为Candidate开始竞选。
Term：某个节点成为Leader到下一次竞选时间，称为一个Term。
Index：数据项编号。Raft中通过Term和Index来定位数据。
```

%0Ahttps%3A%2F%2Fgithub.com%2Fetcd\-io%2Fetcd%2Freleases%0A%0A%E8%A7%A3%E5%8E%8B%E5%90%8E%EF%BC%8C%E4%B8%A4%E4%B8%AA%E4%B8%BB%E8%A6%81%E7%9A%84%E6%96%87%E4%BB%B6%EF%BC%9A%20%20%0A1%E3%80%81etcd%20%20%0A2%E3%80%81etcdctl%20%20%0A%0A%0A%E5%85%88%E5%90%AF%E5%8A%A8%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%2C%E7%9C%8B%E4%B8%8B%E6%95%88%E6%9E%9C%20%20%0A%3E.%2Fetcd%0A%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%20%20%0A%3Emkdir%20config%20%20%0A%3Ecd%20config%20%20%0A%3Etouch%20et\_m1.yml%20et\_m2.yml%20et\_m3.yml%20%20%0A%0A%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95%0A%3Emkdir%20%2Fsoft%2Fetcd%2Fdata%2F%20%20%0A%3Ecd%20%2Fsoft%2Fetcd%2Fdata%2F%20%20%0A%3Emkdir%20et\_m1%20et\_m2%20et\_m3%20%20%0A%3Echmod%20%20777%20%2Fsoft%2Fetcd%2Fdata%20%20%0A%3Echmod%20\-R%20700%20%2Fsoft%2Fetcd%2Fdata%2F\*%0A%0A%3Evim%20m1.yml%20%0A%0A%60%60%60%0A%0Aname%3A%20%22et\_m1%22%0Adata\-dir%3A%20%22%2Fsoft%2Fetcd%2Fdata%2Fet\_m1%22%0Alog\-level%3A%20%22debug%22%0A%0Alisten\-peer\-urls%3A%20'http%3A%2F%2F172.26.150.7%3A2380'%0Alisten\-client\-urls%3A%20'http%3A%2F%2F0.0.0.0%3A2379'%0Ainitial\-advertise\-peer\-urls%3A%20'http%3A%2F%2F172.26.150.7%3A2380'%0Aadvertise\-client\-urls%3A%20'http%3A%2F%2F0.0.0.0%3A2379'%0A%0A%0Ainitial\-cluster\-state%3A%20'new'%0Ainitial\-cluster\-token%3A%20'etcd\-cluster'%0A%0Ainitial\-cluster%3A%20'et\_m1%3Dhttp%3A%2F%2F172.26.150.7%3A2380%2Cet\_m2%3Dhttp%3A%2F%2F172.26.150.7%3A2382%2Cet\_m3%3Dhttp%3A%2F%2F172.26.150.7%3A2384'%0A%0Aenable\-v2%3A%20true%20%23%E5%A6%82%E6%9E%9C%E7%94%A8V2%E7%89%88%E6%9C%AC%EF%BC%8C%E4%BD%BF%E7%94%A8HTTP%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C%E5%BE%97%E6%89%93%E5%BC%80%EF%BC%8C%E4%B8%8D%E7%84%B6404%0A%0A%0A%60%60%60%0A%0A%3E%E5%A6%82%E6%9E%9C%E9%97%B4%E5%8D%95%E6%9C%BA%EF%BC%8C%E6%AD%A4%E9%85%8D%E7%BD%AE%E5%BF%85%E9%A1%BB%E5%BE%97%E6%9C%89%EF%BC%9Alisten\-client\-urls%3A%20'http%3A%2F%2F0.0.0.0%3A2379'%0A%0A2379%EF%BC%9ACLIENT%E3%80%81HTTP%20API%20%E7%AB%AF%E5%8F%A3%20%20%0A2380%EF%BC%9A%E5%90%84%E9%9B%86%E7%BE%A4%E8%8A%82%E7%82%B9%E9%80%9A%E4%BF%A1%E7%AB%AF%E5%8F%A3%0A%0A%E5%90%AF%E5%8A%A8%0A.%2Fetcd%20\-\-config\-file%3D%2Fsoft%2Fetcd%2Fconfig%2Fcfg.conf%0A%0A%E9%87%8D%E5%90%AF%E5%90%AF%E5%8A%A8%20%20%0A%3Ecp%20%2Fsoft%2Fetcd%2Fetcd%20%2Fusr%2Flocal%2Fbin%20%20%0A%3Eetcd%20\-\-config\-file%3D%2Fsoft%2Fetcd%2Fconfig%2Fet\_m1.yml%20%20%0A%3Eetcd%20\-\-config\-file%3D%2Fsoft%2Fetcd%2Fconfig%2Fet\_m2.yml%20%20%0A%3Eetcd%20\-\-config\-file%3D%2Fsoft%2Fetcd%2Fconfig%2Fet\_m3.yml%20%0A%0A%0Ahttp%20v2%20%E6%B5%8B%E8%AF%95%EF%BC%8C%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E6%95%B0%E6%8D%AE%20%20%20%20%0A%3Ecurl%20http%3A%2F%2F39.106.65.76%3A2379%2Fv2%2Fkeys%2Fmessage%20\-X%20PUT%20\-d%20value%3D%22Hello%20world%22%0A%0Ahttp%20v2%20%E6%B5%8B%E8%AF%95%EF%BC%8C%E8%8E%B7%E5%8F%96%E4%B8%80%E4%B8%AA%E6%95%B0%E6%8D%AE%20%20%0A%3Ecurl%20http%3A%2F%2F39.106.65.76%3A2379%2Fv2%2Fkeys%2Fmessage%0A%0A%0Aetcdctl%20%EF%BC%8C%E6%8C%87%E4%BB%A4%E8%A1%8C%E7%9A%84%E6%97%A5%E5%B8%B8%E6%93%8D%E4%BD%9C%EF%BC%8C%E8%B7%9FHTTP%20API%20%E4%B8%80%E6%A0%B7%20%20%0A%3E.%2Fetcdctl%20put%20test%201111%20%20%0A%3E.%2Fetcdctl%20get%20test%20%20%0A%0A%E5%9C%A8%E4%BD%BF%E7%94%A8%20etcdctl%20%EF%BC%8C%E6%9C%89%E4%B8%AAV2%20V3%E5%88%87%E6%8D%A2%0A%3Eexport%20ETCDCTL\_API%3D2%20%20%0A%3Eexport%20ETCDCTL\_API%3D3%0A%0A%0A%E6%96%B0%E7%9A%84%E7%89%88%E6%9C%ACETCD%E5%9D%87%E6%98%AFV3\-API%EF%BC%8C%E4%BD%BF%E7%94%A8%E4%BA%86GRPC%EF%BC%8C%E5%85%B7%E8%AF%B4%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%871%2F3%EF%BC%8C%E6%9C%80%E5%8F%AF%E6%81%A8%E7%9A%84%E6%98%AFV3%E4%B8%8D%E5%85%BC%E5%AE%B9V2%EF%BC%8C%E4%B8%8D%E8%83%BD%E5%85%B1%E5%AD%98%EF%BC%8CV3%EF%BC%9A%E6%98%AF%E6%94%BE%E5%BC%83%E4%BA%86HTTP%EF%BC%8C%E5%85%A8%E7%94%A8GRPC%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%9D%9E%E8%A6%81%E4%BD%BF%E7%94%A8HTTP%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%86%8D%E6%90%9E%E4%B8%AAgrpc%20getway%20%EF%BC%8C%E7%94%A8HTTP%2BJSON%E5%8F%91%E9%80%81%E8%BF%87%E6%9D%A5%EF%BC%8C%E4%B8%AD%E8%BD%AC%E4%B8%8B%0A%0A%0Aetcd%20%E6%B7%BB%E5%8A%A0%E6%9C%8D%E5%8A%A1%0A\-\-\-%0Atouch%20%2Fusr%2Flib%2Fsystemd%2Fsystem%2Fetcd.service%0Avim%20%2Fusr%2Flib%2Fsystemd%2Fsystem%2Fetcd.service%0A%60%60%60%0A%5BUnit%5D%0ADescription%3Detcd%20key\-value%20store%0ADocumentation%3Dhttps%3A%2F%2Fgithub.com%2Fetcd\-io%2Fetcd%0AAfter%3Dnetwork\-online.target%20local\-fs.target%20remote\-fs.target%20time\-sync.target%0AWants%3Dnetwork\-online.target%20local\-fs.target%20remote\-fs.target%20time\-sync.target%0A%0A%5BService%5D%0AUser%3Droot%0AType%3Dnotify%0A%23Environment%3DETCD\_DATA\_DIR%3D%2Fsoft%2Fetcd%2Fdata%0A%23Environment%3DETCD\_NAME%3D%25m%0A%0A%23WorkingDirectory%3D%2Fsoft%2Fetcd%2F%0A%23EnvironmentFile%3D\-%2Fsoft%2Fetcd%2Fconfig%2Fet\_m1.yml%0AExecStart%3D%2Fsoft%2Fetcd%2Fetcd%20\-\-config\-file%3D%2Fsoft%2Fetcd%2Fconfig%2Fet\_m1.yml%0A%0ARestart%3Dalways%0ARestartSec%3D10s%0ALimitNOFILE%3D40000%0A%0A%5BInstall%5D%0AWantedBy%3Dmulti\-user.target%0A%60%60%60%0Asystemctl%20enable%20etcd.service%0ACreated%20symlink%20from%20%2Fetc%2Fsystemd%2Fsystem%2Fmulti\-user.target.wants%2Fetcd.service%20to%20%2Fusr%2Flib%2Fsystemd%2Fsystem%2Fetcd.service.%0A%0Asystemctl%20start%20etcd.service%3B%0A%0A%0A%E5%AE%89%E5%85%A8%0A\-\-\-%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E5%85%88%0Aetcdctl%20user%20list%0A%E6%B7%BB%E5%8A%A0%E7%94%A8%E6%88%B7%2F%E8%AE%BE%E7%BD%AE%E5%AF%86%E7%A0%81%0Aetcdctl%20user%20add%20root%20%0A%0A%E5%BC%80%E5%90%AF%E9%AA%8C%E8%AF%81%E5%8A%9F%E8%83%BD%0A.%2Fetcdctl%20auth%20enable%20\-\-user%3D'root'%0A%0A%E8%A7%92%E8%89%B2%E4%B8%8E%E7%9B%AE%E5%BD%95%E6%9D%83%E9%99%90%0A%E8%BF%99%E9%87%8C%E5%85%88%E5%BF%BD%E7%95%A5%E6%8E%89%E5%90%A7%0A%0A%E5%8E%9F%E7%90%86%0A\-\-\-\-\-\-\-%0A%0A%0A%60%60%60%0ARaft%EF%BC%9Aetcd%E6%89%80%E9%87%87%E7%94%A8%E7%9A%84%E4%BF%9D%E8%AF%81%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E5%BC%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%9A%84%E7%AE%97%E6%B3%95%E3%80%82%0ANode%EF%BC%9A%E4%B8%80%E4%B8%AARaft%E7%8A%B6%E6%80%81%E6%9C%BA%E5%AE%9E%E4%BE%8B%E3%80%82%0AMember%EF%BC%9A%20%E4%B8%80%E4%B8%AAetcd%E5%AE%9E%E4%BE%8B%E3%80%82%E5%AE%83%E7%AE%A1%E7%90%86%E7%9D%80%E4%B8%80%E4%B8%AANode%EF%BC%8C%E5%B9%B6%E4%B8%94%E5%8F%AF%E4%BB%A5%E4%B8%BA%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%B7%E6%B1%82%E6%8F%90%E4%BE%9B%E6%9C%8D%E5%8A%A1%E3%80%82%0ACluster%EF%BC%9A%E7%94%B1%E5%A4%9A%E4%B8%AAMember%E6%9E%84%E6%88%90%E5%8F%AF%E4%BB%A5%E5%8D%8F%E5%90%8C%E5%B7%A5%E4%BD%9C%E7%9A%84etcd%E9%9B%86%E7%BE%A4%E3%80%82%0APeer%EF%BC%9A%E5%AF%B9%E5%90%8C%E4%B8%80%E4%B8%AAetcd%E9%9B%86%E7%BE%A4%E4%B8%AD%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AAMember%E7%9A%84%E7%A7%B0%E5%91%BC%E3%80%82%0AClient%EF%BC%9A%20%E5%90%91etcd%E9%9B%86%E7%BE%A4%E5%8F%91%E9%80%81HTTP%E8%AF%B7%E6%B1%82%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF%E3%80%82%0AWAL%EF%BC%9A%E9%A2%84%E5%86%99%E5%BC%8F%E6%97%A5%E5%BF%97%EF%BC%8Cetcd%E7%94%A8%E4%BA%8E%E6%8C%81%E4%B9%85%E5%8C%96%E5%AD%98%E5%82%A8%E7%9A%84%E6%97%A5%E5%BF%97%E6%A0%BC%E5%BC%8F%E3%80%82%0Asnapshot%EF%BC%9Aetcd%E9%98%B2%E6%AD%A2WAL%E6%96%87%E4%BB%B6%E8%BF%87%E5%A4%9A%E8%80%8C%E8%AE%BE%E7%BD%AE%E7%9A%84%E5%BF%AB%E7%85%A7%EF%BC%8C%E5%AD%98%E5%82%A8etcd%E6%95%B0%E6%8D%AE%E7%8A%B6%E6%80%81%E3%80%82%0AProxy%EF%BC%9Aetcd%E7%9A%84%E4%B8%80%E7%A7%8D%E6%A8%A1%E5%BC%8F%EF%BC%8C%E4%B8%BAetcd%E9%9B%86%E7%BE%A4%E6%8F%90%E4%BE%9B%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E3%80%82%0ALeader%EF%BC%9ARaft%E7%AE%97%E6%B3%95%E4%B8%AD%E9%80%9A%E8%BF%87%E7%AB%9E%E9%80%89%E8%80%8C%E4%BA%A7%E7%94%9F%E7%9A%84%E5%A4%84%E7%90%86%E6%89%80%E6%9C%89%E6%95%B0%E6%8D%AE%E6%8F%90%E4%BA%A4%E7%9A%84%E8%8A%82%E7%82%B9%E3%80%82%0AFollower%EF%BC%9A%E7%AB%9E%E9%80%89%E5%A4%B1%E8%B4%A5%E7%9A%84%E8%8A%82%E7%82%B9%E4%BD%9C%E4%B8%BARaft%E4%B8%AD%E7%9A%84%E4%BB%8E%E5%B1%9E%E8%8A%82%E7%82%B9%EF%BC%8C%E4%B8%BA%E7%AE%97%E6%B3%95%E6%8F%90%E4%BE%9B%E5%BC%BA%E4%B8%80%E8%87%B4%E6%80%A7%E4%BF%9D%E8%AF%81%E3%80%82%0ACandidate%EF%BC%9A%E5%BD%93Follower%E8%B6%85%E8%BF%87%E4%B8%80%E5%AE%9A%E6%97%B6%E9%97%B4%E6%8E%A5%E6%94%B6%E4%B8%8D%E5%88%B0Leader%E7%9A%84%E5%BF%83%E8%B7%B3%E6%97%B6%E8%BD%AC%E5%8F%98%E4%B8%BACandidate%E5%BC%80%E5%A7%8B%E7%AB%9E%E9%80%89%E3%80%82%0ATerm%EF%BC%9A%E6%9F%90%E4%B8%AA%E8%8A%82%E7%82%B9%E6%88%90%E4%B8%BALeader%E5%88%B0%E4%B8%8B%E4%B8%80%E6%AC%A1%E7%AB%9E%E9%80%89%E6%97%B6%E9%97%B4%EF%BC%8C%E7%A7%B0%E4%B8%BA%E4%B8%80%E4%B8%AATerm%E3%80%82%0AIndex%EF%BC%9A%E6%95%B0%E6%8D%AE%E9%A1%B9%E7%BC%96%E5%8F%B7%E3%80%82Raft%E4%B8%AD%E9%80%9A%E8%BF%87Term%E5%92%8CIndex%E6%9D%A5%E5%AE%9A%E4%BD%8D%E6%95%B0%E6%8D%AE%E3%80%82%0A%60%60%60%0A
