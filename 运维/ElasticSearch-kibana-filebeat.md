# ElasticSearch/kibana/filebeat

# ElasticSearch

官网：https://www.elastic.co/cn/downloads/elasticsearch

添加一个用户，ES不允许ROOT启动

```
groupadd esgroup;
useradd esroot -g esgroup -p esroot
```

下载包，解压~设置权限

> chown \-R esroot:esgroup /soft/elasticsearch
> 
> 
> su \- esroot
> 
> 
> chown \-R esroot:esgroup es
> 
> 
> chmod 777 /data/logs/es

修改个数值，不然无法启动

> max virtual memory areas vm.max\_map\_count \[65530\] is too low, increase to at least \[262144\]

vim /etc/sysctl.conf

vm.max\_map\_count=262144

sudo sysctl \-p

编辑一下配置文件

> vim /soft/elasticsearch/config/elasticsearch.yml

```
cluster.name: my-es-cluster
node.name: node-76
path.data: /data/es
path.logs: /data/logs/es
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-76"]

```

修改个数值，不然无法启动

> vim jvm.options

\-Xmx512m

\-Xmx512m

启动

> su \- esroot
> 
> 
> /soft/elasticsearch/bin/elasticsearch \-d

```
future versions of Elasticsearch will require Java 11; your Java version from [/soft/jdk/jre] does not meet this requirement
```

注：ES里自带有JDK11，所以如果机器没安装JDK，就别装了，费力不讨好。

看下端口是否启动成功

> lsof \-i:9200

restful api 访问

> http://59.110.167.206:9200/

查看集群简单健康状态

> http://8.142.177.235:9200/\_cluster/health?pretty

查看集群状态

> http://8.142.177.235:9200/\_cluster/state

查看集群配置

> http://8.142.177.235:9200/\_cluster/settings

关闭ES

之前还有个HTTP\-API的方式，7.0以后给干掉了，目前没有太好的办法 ，只能KILL

> curl \-XPOST 'http://8.142.177.235:9200/\_shutdown

## ES基础学习

索引\(index\)：间接理解为，DBMS中的一个数据库，可分片、分副本

索引类型\(index\_type\)：间接理解为，DBMS中的，一个表，每个表有不同的字段值

文档\(doc\)：间接理解为，DBMS中的一行记录

映射\(mapping\)：索引和索引类型

> 7.0\-8.0官方已经宣布干掉index\_type，可以用使用：\_doc，做为index\_type name。结果就是：没有索引类型这个，索引下直接挂字段修饰符

> http://ip:port/索引/类型/文档ID 废弃 v:7.0\-
> 
> 
> http://ip:port/索引/\_doc/文档ID 使用 v7.0\+

document:

1. \_index：文档所在的索引名
2. \_type：文档所在的类型名
3. \_id：文档唯一 id
4. \_score：相关性算分
5. \_uid：组合 id，有\_type和\_id组成（从6.x开始\_type不再起作用，同\_id一样）
6. \_source：文档的原始 json 数据，可以从这里获取每个字段的内容
7. \_all：整合所有字段内容到该字段，默认下禁用

> document其实是2个部分，1：元数据，2：用户的真实数据

一个索引的定义主要包含2个KEY：setting mappings

1. setting:修改分片和副本数的
2. mappings:修改字段和类型的，是对一个索引的具体修饰，像是MYSQL的表，不同的是它更灵活。它支持动态创建，换个角度理解：是对doc的约束。

> http://59.110.167.206:9200/索引名/\_setting
> 
> 
> http://59.110.167.206:9200/my\_first\_index/\_mappings

查找索引下的数据

> curl \-XPOST 'http://59.110.167.206:9200/filebeat\-7.13.1\-2021.06.08\-000001/\_search?pretty' \-d '{"query": { "match\_all": {} },"from": 10,"size": 10}' \-H "Content\-Type: application/json"|grep messag

删除一个索引

> curl \-XDELETE 'http://59.110.167.206:9200/filebeat\-7.13.1\-2021.06.08\-000001'

查看索引列表

> http://59.110.167.206:9200/\_cat/indices?format=json&index=\[索引名称，可使用通配符\]

查看索引mapping

> http://59.110.167.206:9200/my\_first\_index/\_mapping?pretty

创建一个索引

```
curl -XPUT http://59.110.167.206:9200/my_first_index -H 'Content-Type: application/json' -d '
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas":0
  }
}'
```

> 创建索引的同时也可以一并创建索引类型

索引成功后，就是索引类型了，其实就是mapping，往一个索引里添加/注册mapping结构体

索引类型

1. type :String Numerical Date bool Array Object等
2. index:true|false ，是否被索引收录进去，就是：是否支持搜索
3. store:用户数据默认存于\_source中，也可以单独存储
4. analyzer

type补充

1. String 又包含：text keyword
2. byte，short，integer，long
3. float, half\_float, scaled\_float，double
4. integer\_range， long\_range， float\_range，double\_range，date\_range
5. Binary Geo\-point、Geo\-shape、IP、Join 等

> 这支持的类型还真挺多...

创建一个索引类型,给my\_first\_index添加一个\<产品\>\<表\>

7.0以前是：http://59.110.167.206:9200/my\_first\_index/\_mapping/product

7.0以后是：http://59.110.167.206:9200/my\_first\_index/\_doc/product

```
curl -XPUT "http://59.110.167.206:9200/my_first_index/_doc/product" -H 'Content-Type: application/json' -d '
{
  "properties": {
    "title": {
      "type": "text",
      "index":true
    },
    "subtitle": {
      "type": "text"
    },
    "price": {
      "type": "float"
    },
    "isonline": {
      "type": "boolean"
    },
    "addtime": {
      "type": "date",
      "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
    },
    "stock": {
      "type": "integer"
    }
  }
}'
```

创建一个文档

```
curl -XPUT "http://59.110.167.206:9200/my_first_index/_doc/1" -H 'Content-Type: application/json' -d '
{
    "title": "shouji",
    "subtitle": "zhinengshouji",
    "price": 1200.02,
    "isonline": true,
    "addtime": "2021-06-01 12:00:00",
    "stock": 10
}'
```

## kibana

下载包，解压

同样，官方不推荐直接用ROOT（但也可以用），添加用户组,设置权限

> groupadd kibanagroup;
> 
> 
> useradd kibana \-g kibanagroup \-p kibana
> 
> 
> chown \-R kibana:kibanagroup /soft/kibana

修改个配置参数

> vim /soft/kibana/config/node.options
> 
> 
> \-\-max\-old\-space\-size=256

配置文件

> vim /soft/kibana/config/kibana.yml

```
server.port: 5601  
server.host: "0.0.0.0"  
elasticsearch.hosts: ["http://localhost:9200"] 
server.maxPayloadBytes: 10485760 #这里加了一个0，放大10倍
#logging.dest: logging.dest: /data/logs/kibana/start.log
```

启动

> bin/kibana \-\-allow\-root

看看端口号

> losf \-i:5601

看下api

> http://192.168.1.31:5601/status

这里注意下，kibana启动后会在ES里自动创建了3个索引：

1. .kibana\-event\-log\-7.10.0\-000001
2. .kibana\_task\_manager\_1
3. .kibana\_1

## filebeat

还是elastic 出品，golang编写，直接下载解压

配置示例文件：filebeat.reference.yml（包含所有未过时的配置项）

正常加载的配置文件：filebeat.yml

详情配置说明请跳转

查看下刚刚修改的配置文件语法：

> ./filebeat test config

## 启动

> filebeat \-e \-c filebeat.yml
> 
> 
> ./filebeat \-e \-c filebeat.yml \-d "\*"

filebeat自动创建的索引可能是只读权限，改一下：

```

curl -XPUT -H 'Content-Type: application/json' http://59.110.167.206:9200/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'

curl -XPUT -H 'Content-Type: application/json' http://59.110.167.206:9200/filebeat-7.13.1-2021.06.08-000001/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'
```

查看下已开启的modules:

> ./filebeat modules list

这东西会造成 自动创建索引的时候，共计5000来个mapping...

```
./filebeat modules disable aws

```

## 原理

Filebeat涉及两个组件：查找器prospector（探测者）和采集器harvester（收割者），来读取文件\(tail file\)并将事件数据发送到指定的输出。

## harvester（收割者）

读取单个文件的内容（逐行），并将内容发送到the output，每个文件启动一个harvester, harvester负责打开和关闭文件，这意味着在运行时文件描述符保持打开状态。

## prospector（探测器）

管理 harvester 并找到所有要读取的文件来源

每设置一个日志监听路径 ，即会有一个 prospector

启动成功后，用kibina

> kibana\-\>index management\-\>ck\-\*

查看filebeat在es中新建的相关索引文件，如果为空的话，注意：

1. input 的pash 是否正确
2. 目录下的日志文件是否存在且有内容

日志查看，需要得创建一个 搜索器

kibana\-\>index patterns\-\>

> ck\-local\-\*

error : Payload Too Large

> 修改 kibana.yml 中的server.maxPayloadBytes

ElasticSearch%20%0A%3D%3D%3D%3D%3D%0A%E5%AE%98%E7%BD%91%EF%BC%9Ahttps%3A%2F%2Fwww.elastic.co%2Fcn%2Fdownloads%2Felasticsearch%0A%0A%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E7%94%A8%E6%88%B7%EF%BC%8CES%E4%B8%8D%E5%85%81%E8%AE%B8ROOT%E5%90%AF%E5%8A%A8%0A%60%60%60%0Agroupadd%20esgroup%3B%0Auseradd%20esroot%20\-g%20esgroup%20\-p%20esroot%0A%60%60%60%0A%E4%B8%8B%E8%BD%BD%E5%8C%85%EF%BC%8C%E8%A7%A3%E5%8E%8B~%E8%AE%BE%E7%BD%AE%E6%9D%83%E9%99%90%0A%3Echown%20\-R%20esroot%3Aesgroup%20%2Fsoft%2Felasticsearch%20%20%0A%3Esu%20\-%20esroot%0A%3Echown%20\-R%20esroot%3Aesgroup%20es%20%20%0A%3Echmod%20777%20%2Fdata%2Flogs%2Fes%0A%0A%E4%BF%AE%E6%94%B9%E4%B8%AA%E6%95%B0%E5%80%BC%EF%BC%8C%E4%B8%8D%E7%84%B6%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8%0A%3Emax%20virtual%20memory%20areas%20vm.max\_map\_count%20%5B65530%5D%20is%20too%20low%2C%20increase%20to%20at%20least%20%5B262144%5D%0A%20%0Avim%20%2Fetc%2Fsysctl.conf%20%20%0Avm.max\_map\_count%3D262144%20%20%0Asudo%20sysctl%20\-p%20%20%0A%0A%E7%BC%96%E8%BE%91%E4%B8%80%E4%B8%8B%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%3Evim%20%2Fsoft%2Felasticsearch%2Fconfig%2Felasticsearch.yml%0A%60%60%60%0Acluster.name%3A%20my\-es\-cluster%0Anode.name%3A%20node\-76%0Apath.data%3A%20%2Fdata%2Fes%0Apath.logs%3A%20%2Fdata%2Flogs%2Fes%0Anetwork.host%3A%200.0.0.0%0Ahttp.port%3A%209200%0Acluster.initial\_master\_nodes%3A%20%5B%22node\-76%22%5D%0A%0A%60%60%60%0A%E4%BF%AE%E6%94%B9%E4%B8%AA%E6%95%B0%E5%80%BC%EF%BC%8C%E4%B8%8D%E7%84%B6%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8%0A%3Evim%20jvm.options%0A%0A\-Xmx512m%0A\-Xmx512m%0A%0A%E5%90%AF%E5%8A%A8%0A%3Esu%20%20\-%20esroot%20%20%0A%3E%2Fsoft%2Felasticsearch%2Fbin%2Felasticsearch%20\-d%20%20%0A%0A%60%60%60%0Afuture%20versions%20of%20Elasticsearch%20will%20require%20Java%2011%3B%20your%20Java%20version%20from%20%5B%2Fsoft%2Fjdk%2Fjre%5D%20does%20not%20meet%20this%20requirement%0A%60%60%60%0A%E6%B3%A8%EF%BC%9AES%E9%87%8C%E8%87%AA%E5%B8%A6%E6%9C%89JDK11%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A6%82%E6%9E%9C%E6%9C%BA%E5%99%A8%E6%B2%A1%E5%AE%89%E8%A3%85JDK%EF%BC%8C%E5%B0%B1%E5%88%AB%E8%A3%85%E4%BA%86%EF%BC%8C%E8%B4%B9%E5%8A%9B%E4%B8%8D%E8%AE%A8%E5%A5%BD%E3%80%82%0A%0A%E7%9C%8B%E4%B8%8B%E7%AB%AF%E5%8F%A3%E6%98%AF%E5%90%A6%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F%0A%3Elsof%20\-i%3A9200%0A%0Arestful%20api%20%E8%AE%BF%E9%97%AE%0A%3Ehttp%3A%2F%2F59.110.167.206%3A9200%2F%20%0A%0A%E6%9F%A5%E7%9C%8B%E9%9B%86%E7%BE%A4%E7%AE%80%E5%8D%95%E5%81%A5%E5%BA%B7%E7%8A%B6%E6%80%81%0A%3Ehttp%3A%2F%2F8.142.177.235%3A9200%2F\_cluster%2Fhealth%3Fpretty%0A%0A%E6%9F%A5%E7%9C%8B%E9%9B%86%E7%BE%A4%E7%8A%B6%E6%80%81%0A%3Ehttp%3A%2F%2F8.142.177.235%3A9200%2F\_cluster%2Fstate%0A%0A%E6%9F%A5%E7%9C%8B%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE%0A%3Ehttp%3A%2F%2F8.142.177.235%3A9200%2F\_cluster%2Fsettings%0A%0A%E5%85%B3%E9%97%ADES%0A%E4%B9%8B%E5%89%8D%E8%BF%98%E6%9C%89%E4%B8%AAHTTP\-API%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C7.0%E4%BB%A5%E5%90%8E%E7%BB%99%E5%B9%B2%E6%8E%89%E4%BA%86%EF%BC%8C%E7%9B%AE%E5%89%8D%E6%B2%A1%E6%9C%89%E5%A4%AA%E5%A5%BD%E7%9A%84%E5%8A%9E%E6%B3%95%20%EF%BC%8C%E5%8F%AA%E8%83%BDKILL%0A%3Ecurl%20\-XPOST%20'http%3A%2F%2F8.142.177.235%3A9200%2F\_shutdown%0A%0AES%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%0A\-\-\-\-\-%0A%0A%E7%B4%A2%E5%BC%95\(index\)%EF%BC%9A%E9%97%B4%E6%8E%A5%E7%90%86%E8%A7%A3%E4%B8%BA%EF%BC%8CDBMS%E4%B8%AD%E7%9A%84%E4%B8%80%E4%B8%AA%E6%95%B0%E6%8D%AE%E5%BA%93%EF%BC%8C%E5%8F%AF%E5%88%86%E7%89%87%E3%80%81%E5%88%86%E5%89%AF%E6%9C%AC%0A%0A%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B\(index\_type\)%EF%BC%9A%E9%97%B4%E6%8E%A5%E7%90%86%E8%A7%A3%E4%B8%BA%EF%BC%8CDBMS%E4%B8%AD%E7%9A%84%EF%BC%8C%E4%B8%80%E4%B8%AA%E8%A1%A8%EF%BC%8C%E6%AF%8F%E4%B8%AA%E8%A1%A8%E6%9C%89%E4%B8%8D%E5%90%8C%E7%9A%84%E5%AD%97%E6%AE%B5%E5%80%BC%0A%0A%E6%96%87%E6%A1%A3\(doc\)%EF%BC%9A%E9%97%B4%E6%8E%A5%E7%90%86%E8%A7%A3%E4%B8%BA%EF%BC%8CDBMS%E4%B8%AD%E7%9A%84%E4%B8%80%E8%A1%8C%E8%AE%B0%E5%BD%95%0A%0A%E6%98%A0%E5%B0%84\(mapping\)%EF%BC%9A%E7%B4%A2%E5%BC%95%E5%92%8C%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%0A%0A%0A%3E7.0\-8.0%E5%AE%98%E6%96%B9%E5%B7%B2%E7%BB%8F%E5%AE%A3%E5%B8%83%E5%B9%B2%E6%8E%89index\_type%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%94%A8%E4%BD%BF%E7%94%A8%EF%BC%9A\_doc%EF%BC%8C%E5%81%9A%E4%B8%BAindex\_type%20name%E3%80%82%E7%BB%93%E6%9E%9C%E5%B0%B1%E6%98%AF%EF%BC%9A%E6%B2%A1%E6%9C%89%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%E8%BF%99%E4%B8%AA%EF%BC%8C%E7%B4%A2%E5%BC%95%E4%B8%8B%E7%9B%B4%E6%8E%A5%E6%8C%82%E5%AD%97%E6%AE%B5%E4%BF%AE%E9%A5%B0%E7%AC%A6%0A%0A%3Ehttp%3A%2F%2Fip%3Aport%2F%E7%B4%A2%E5%BC%95%2F%E7%B1%BB%E5%9E%8B%2F%E6%96%87%E6%A1%A3ID%20%20%20%20%E5%BA%9F%E5%BC%83%20v%3A7.0\-%20%20%0A%3Ehttp%3A%2F%2Fip%3Aport%2F%E7%B4%A2%E5%BC%95%2F\_doc%2F%E6%96%87%E6%A1%A3ID%20%20%20%20%E4%BD%BF%E7%94%A8%20v7.0%2B%20%20%0A%0Adocument%3A%0A1.%20\_index%EF%BC%9A%E6%96%87%E6%A1%A3%E6%89%80%E5%9C%A8%E7%9A%84%E7%B4%A2%E5%BC%95%E5%90%8D%0A2.%20\_type%EF%BC%9A%E6%96%87%E6%A1%A3%E6%89%80%E5%9C%A8%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%90%8D%0A2.%20\_id%EF%BC%9A%E6%96%87%E6%A1%A3%E5%94%AF%E4%B8%80%20id%0A2.%20\_score%EF%BC%9A%E7%9B%B8%E5%85%B3%E6%80%A7%E7%AE%97%E5%88%86%0A2.%20\_uid%EF%BC%9A%E7%BB%84%E5%90%88%20id%EF%BC%8C%E6%9C%89\_type%E5%92%8C\_id%E7%BB%84%E6%88%90%EF%BC%88%E4%BB%8E6.x%E5%BC%80%E5%A7%8B\_type%E4%B8%8D%E5%86%8D%E8%B5%B7%E4%BD%9C%E7%94%A8%EF%BC%8C%E5%90%8C\_id%E4%B8%80%E6%A0%B7%EF%BC%89%0A2.%20\_source%EF%BC%9A%E6%96%87%E6%A1%A3%E7%9A%84%E5%8E%9F%E5%A7%8B%20json%20%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%BB%8E%E8%BF%99%E9%87%8C%E8%8E%B7%E5%8F%96%E6%AF%8F%E4%B8%AA%E5%AD%97%E6%AE%B5%E7%9A%84%E5%86%85%E5%AE%B9%0A2.%20\_all%EF%BC%9A%E6%95%B4%E5%90%88%E6%89%80%E6%9C%89%E5%AD%97%E6%AE%B5%E5%86%85%E5%AE%B9%E5%88%B0%E8%AF%A5%E5%AD%97%E6%AE%B5%EF%BC%8C%E9%BB%98%E8%AE%A4%E4%B8%8B%E7%A6%81%E7%94%A8%0A%0A%3Edocument%E5%85%B6%E5%AE%9E%E6%98%AF2%E4%B8%AA%E9%83%A8%E5%88%86%EF%BC%8C1%EF%BC%9A%E5%85%83%E6%95%B0%E6%8D%AE%EF%BC%8C2%EF%BC%9A%E7%94%A8%E6%88%B7%E7%9A%84%E7%9C%9F%E5%AE%9E%E6%95%B0%E6%8D%AE%0A%0A%0A%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%E7%9A%84%E5%AE%9A%E4%B9%89%E4%B8%BB%E8%A6%81%E5%8C%85%E5%90%AB2%E4%B8%AAKEY%EF%BC%9Asetting%20mappings%0A1.%20setting%3A%E4%BF%AE%E6%94%B9%E5%88%86%E7%89%87%E5%92%8C%E5%89%AF%E6%9C%AC%E6%95%B0%E7%9A%84%0A2.%20mappings%3A%E4%BF%AE%E6%94%B9%E5%AD%97%E6%AE%B5%E5%92%8C%E7%B1%BB%E5%9E%8B%E7%9A%84%EF%BC%8C%E6%98%AF%E5%AF%B9%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%E7%9A%84%E5%85%B7%E4%BD%93%E4%BF%AE%E9%A5%B0%EF%BC%8C%E5%83%8F%E6%98%AFMYSQL%E7%9A%84%E8%A1%A8%EF%BC%8C%E4%B8%8D%E5%90%8C%E7%9A%84%E6%98%AF%E5%AE%83%E6%9B%B4%E7%81%B5%E6%B4%BB%E3%80%82%E5%AE%83%E6%94%AF%E6%8C%81%E5%8A%A8%E6%80%81%E5%88%9B%E5%BB%BA%EF%BC%8C%E6%8D%A2%E4%B8%AA%E8%A7%92%E5%BA%A6%E7%90%86%E8%A7%A3%EF%BC%9A%E6%98%AF%E5%AF%B9doc%E7%9A%84%E7%BA%A6%E6%9D%9F%E3%80%82%0A%0A%3Ehttp%3A%2F%2F59.110.167.206%3A9200%2F%E7%B4%A2%E5%BC%95%E5%90%8D%2F\_setting%20%20%0A%3Ehttp%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_mappings%0A%0A%0A%0A%E6%9F%A5%E6%89%BE%E7%B4%A2%E5%BC%95%E4%B8%8B%E7%9A%84%E6%95%B0%E6%8D%AE%0A%3Ecurl%20\-XPOST%20'http%3A%2F%2F59.110.167.206%3A9200%2Ffilebeat\-7.13.1\-2021.06.08\-000001%2F\_search%3Fpretty'%20\-d%20'%7B%22query%22%3A%20%7B%20%22match\_all%22%3A%20%7B%7D%20%7D%2C%22from%22%3A%2010%2C%22size%22%3A%2010%7D'%20\-H%20%22Content\-Type%3A%20application%2Fjson%22%7Cgrep%20messag%0A%0A%E5%88%A0%E9%99%A4%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%0A%3Ecurl%20\-XDELETE%20'http%3A%2F%2F59.110.167.206%3A9200%2Ffilebeat\-7.13.1\-2021.06.08\-000001'%0A%0A%E6%9F%A5%E7%9C%8B%E7%B4%A2%E5%BC%95%E5%88%97%E8%A1%A8%0A%3Ehttp%3A%2F%2F59.110.167.206%3A9200%2F\_cat%2Findices%3Fformat%3Djson%26index%3D%5B%E7%B4%A2%E5%BC%95%E5%90%8D%E7%A7%B0%EF%BC%8C%E5%8F%AF%E4%BD%BF%E7%94%A8%E9%80%9A%E9%85%8D%E7%AC%A6%5D%0A%0A%E6%9F%A5%E7%9C%8B%E7%B4%A2%E5%BC%95mapping%0A%3Ehttp%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_mapping%3Fpretty%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%0A%60%60%60%0Acurl%20\-XPUT%20http%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%20\-H%20'Content\-Type%3A%20application%2Fjson'%20\-d%20'%0A%7B%0A%20%20%22settings%22%3A%20%7B%0A%20%20%20%20%22number\_of\_shards%22%3A%201%2C%0A%20%20%20%20%22number\_of\_replicas%22%3A0%0A%20%20%7D%0A%7D'%0A%60%60%60%0A%0A%3E%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E7%9A%84%E5%90%8C%E6%97%B6%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%B8%80%E5%B9%B6%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%0A%0A%E7%B4%A2%E5%BC%95%E6%88%90%E5%8A%9F%E5%90%8E%EF%BC%8C%E5%B0%B1%E6%98%AF%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%E4%BA%86%EF%BC%8C%E5%85%B6%E5%AE%9E%E5%B0%B1%E6%98%AFmapping%EF%BC%8C%E5%BE%80%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%E9%87%8C%E6%B7%BB%E5%8A%A0%2F%E6%B3%A8%E5%86%8Cmapping%E7%BB%93%E6%9E%84%E4%BD%93%0A%0A%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%0A1.%20type%20%3AString%20Numerical%20Date%20bool%20Array%20Object%E7%AD%89%0A2.%20index%3Atrue%7Cfalse%20%EF%BC%8C%E6%98%AF%E5%90%A6%E8%A2%AB%E7%B4%A2%E5%BC%95%E6%94%B6%E5%BD%95%E8%BF%9B%E5%8E%BB%EF%BC%8C%E5%B0%B1%E6%98%AF%EF%BC%9A%E6%98%AF%E5%90%A6%E6%94%AF%E6%8C%81%E6%90%9C%E7%B4%A2%0A3.%20store%3A%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E9%BB%98%E8%AE%A4%E5%AD%98%E4%BA%8E\_source%E4%B8%AD%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E5%8D%95%E7%8B%AC%E5%AD%98%E5%82%A8%0A4.%20analyzer%0A%0A%0Atype%E8%A1%A5%E5%85%85%0A1.%20String%20%E5%8F%88%E5%8C%85%E5%90%AB%EF%BC%9Atext%20keyword%0A2.%20byte%EF%BC%8Cshort%EF%BC%8Cinteger%EF%BC%8Clong%0A3.%20float%2C%20half\_float%2C%20scaled\_float%EF%BC%8Cdouble%0A4.%20integer\_range%EF%BC%8C%20long\_range%EF%BC%8C%20float\_range%EF%BC%8Cdouble\_range%EF%BC%8Cdate\_range%0A5.%20Binary%20Geo\-point%E3%80%81Geo\-shape%E3%80%81IP%E3%80%81Join%20%E7%AD%89%0A%0A%3E%E8%BF%99%E6%94%AF%E6%8C%81%E7%9A%84%E7%B1%BB%E5%9E%8B%E8%BF%98%E7%9C%9F%E6%8C%BA%E5%A4%9A...%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%E7%B1%BB%E5%9E%8B%2C%E7%BB%99my\_first\_index%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%3C%E4%BA%A7%E5%93%81%3E%3C%E8%A1%A8%3E%0A%0A7.0%E4%BB%A5%E5%89%8D%E6%98%AF%EF%BC%9Ahttp%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_mapping%2Fproduct%20%20%0A7.0%E4%BB%A5%E5%90%8E%E6%98%AF%EF%BC%9Ahttp%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_doc%2Fproduct%0A%60%60%60%0Acurl%20\-XPUT%20%22http%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_doc%2Fproduct%22%20\-H%20'Content\-Type%3A%20application%2Fjson'%20\-d%20'%0A%7B%0A%20%20%22properties%22%3A%20%7B%0A%20%20%20%20%22title%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22text%22%2C%0A%20%20%20%20%20%20%22index%22%3Atrue%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22subtitle%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22text%22%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22price%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22float%22%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22isonline%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22boolean%22%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22addtime%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22date%22%2C%0A%20%20%20%20%20%20%22format%22%3A%22yyyy\-MM\-dd%20HH%3Amm%3Ass%7C%7Cyyyy\-MM\-dd%22%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22stock%22%3A%20%7B%0A%20%20%20%20%20%20%22type%22%3A%20%22integer%22%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D'%0A%60%60%60%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%96%87%E6%A1%A3%0A%60%60%60%0Acurl%20\-XPUT%20%22http%3A%2F%2F59.110.167.206%3A9200%2Fmy\_first\_index%2F\_doc%2F1%22%20\-H%20'Content\-Type%3A%20application%2Fjson'%20\-d%20'%0A%7B%0A%20%20%20%20%22title%22%3A%20%22shouji%22%2C%0A%20%20%20%20%22subtitle%22%3A%20%22zhinengshouji%22%2C%0A%20%20%20%20%22price%22%3A%201200.02%2C%0A%20%20%20%20%22isonline%22%3A%20true%2C%0A%20%20%20%20%22addtime%22%3A%20%222021\-06\-01%2012%3A00%3A00%22%2C%0A%20%20%20%20%22stock%22%3A%2010%0A%7D'%0A%60%60%60%0A%0A%0A%0Akibana%0A\-\-\-\-\-\-\-%0A%E4%B8%8B%E8%BD%BD%E5%8C%85%EF%BC%8C%E8%A7%A3%E5%8E%8B%0A%E5%90%8C%E6%A0%B7%EF%BC%8C%E5%AE%98%E6%96%B9%E4%B8%8D%E6%8E%A8%E8%8D%90%E7%9B%B4%E6%8E%A5%E7%94%A8ROOT%EF%BC%88%E4%BD%86%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%94%A8%EF%BC%89%EF%BC%8C%E6%B7%BB%E5%8A%A0%E7%94%A8%E6%88%B7%E7%BB%84%2C%E8%AE%BE%E7%BD%AE%E6%9D%83%E9%99%90%0A%3Egroupadd%20kibanagroup%3B%20%20%20%20%0A%3Euseradd%20kibana%20\-g%20kibanagroup%20\-p%20kibana%0A%3Echown%20\-R%20kibana%3Akibanagroup%20%2Fsoft%2Fkibana%20%0A%0A%E4%BF%AE%E6%94%B9%E4%B8%AA%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%0A%3Evim%20%2Fsoft%2Fkibana%2Fconfig%2Fnode.options%20%20%0A\-\-max\-old\-space\-size%3D256%20%20%0A%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%3Evim%20%2Fsoft%2Fkibana%2Fconfig%2Fkibana.yml%20%20%0A%60%60%60%0Aserver.port%3A%205601%20%20%0Aserver.host%3A%20%220.0.0.0%22%20%20%0Aelasticsearch.hosts%3A%20%5B%22http%3A%2F%2Flocalhost%3A9200%22%5D%20%0Aserver.maxPayloadBytes%3A%2010485760%20%23%E8%BF%99%E9%87%8C%E5%8A%A0%E4%BA%86%E4%B8%80%E4%B8%AA0%EF%BC%8C%E6%94%BE%E5%A4%A710%E5%80%8D%0A%23logging.dest%3A%20logging.dest%3A%20%2Fdata%2Flogs%2Fkibana%2Fstart.log%0A%60%60%60%0A%0A%E5%90%AF%E5%8A%A8%0A%3Ebin%2Fkibana%20\-\-allow\-root%20%20%0A%0A%E7%9C%8B%E7%9C%8B%E7%AB%AF%E5%8F%A3%E5%8F%B7%0A%3Elosf%20\-i%3A5601%0A%0A%E7%9C%8B%E4%B8%8Bapi%0A%3Ehttp%3A%2F%2F192.168.1.31%3A5601%2Fstatus%0A%0A%0A%E8%BF%99%E9%87%8C%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%8Ckibana%E5%90%AF%E5%8A%A8%E5%90%8E%E4%BC%9A%E5%9C%A8ES%E9%87%8C%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E4%BA%863%E4%B8%AA%E7%B4%A2%E5%BC%95%EF%BC%9A%0A1.%20.kibana\-event\-log\-7.10.0\-000001%0A2.%20.kibana\_task\_manager\_1%0A3.%20.kibana\_1%0A%0A%0A%0Afilebeat%0A\-\-\-\-\-\-%0A%E8%BF%98%E6%98%AFelastic%20%E5%87%BA%E5%93%81%EF%BC%8Cgolang%E7%BC%96%E5%86%99%EF%BC%8C%E7%9B%B4%E6%8E%A5%E4%B8%8B%E8%BD%BD%E8%A7%A3%E5%8E%8B%0A%0A%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B%E6%96%87%E4%BB%B6%EF%BC%9Afilebeat.reference.yml%EF%BC%88%E5%8C%85%E5%90%AB%E6%89%80%E6%9C%89%E6%9C%AA%E8%BF%87%E6%97%B6%E7%9A%84%E9%85%8D%E7%BD%AE%E9%A1%B9%EF%BC%89%20%20%0A%E6%AD%A3%E5%B8%B8%E5%8A%A0%E8%BD%BD%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%9Afilebeat.yml%0A%E8%AF%A6%E6%83%85%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E%E8%AF%B7%E8%B7%B3%E8%BD%AC%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E5%88%9A%E5%88%9A%E4%BF%AE%E6%94%B9%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AF%AD%E6%B3%95%EF%BC%9A%0A%3E.%2Ffilebeat%20test%20config%0A%0A%E5%90%AF%E5%8A%A8%20%20%0A\-\-\-\-\-%0A%3Efilebeat%20\-e%20\-c%20filebeat.yml%20%20%0A%3E.%2Ffilebeat%20%20\-e%20\-c%20filebeat.yml%20\-d%20%22\*%22%0A%0Afilebeat%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E7%9A%84%E7%B4%A2%E5%BC%95%E5%8F%AF%E8%83%BD%E6%98%AF%E5%8F%AA%E8%AF%BB%E6%9D%83%E9%99%90%EF%BC%8C%E6%94%B9%E4%B8%80%E4%B8%8B%EF%BC%9A%0A%60%60%60%0A%0Acurl%20\-XPUT%20\-H%20'Content\-Type%3A%20application%2Fjson'%20http%3A%2F%2F59.110.167.206%3A9200%2F\_settings%20\-d%20'%0A%7B%0A%20%20%20%20%22index%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22blocks%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22read\_only\_allow\_delete%22%3A%20%22false%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D'%0A%0Acurl%20\-XPUT%20\-H%20'Content\-Type%3A%20application%2Fjson'%20http%3A%2F%2F59.110.167.206%3A9200%2Ffilebeat\-7.13.1\-2021.06.08\-000001%2F\_settings%20\-d%20'%0A%7B%0A%20%20%20%20%22index%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22blocks%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22read\_only\_allow\_delete%22%3A%20%22false%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D'%0A%60%60%60%0A%0A%0A%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E5%B7%B2%E5%BC%80%E5%90%AF%E7%9A%84modules%3A%0A%0A%3E.%2Ffilebeat%20modules%20list%0A%0A%E8%BF%99%E4%B8%9C%E8%A5%BF%E4%BC%9A%E9%80%A0%E6%88%90%20%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%85%B1%E8%AE%A15000%E6%9D%A5%E4%B8%AAmapping...%0A%0A%60%60%60%0A.%2Ffilebeat%20modules%20disable%20aws%0A%0A%60%60%60%0A%0A%0A%E5%8E%9F%E7%90%86%0A\-\-\-\-\-\-%0AFilebeat%E6%B6%89%E5%8F%8A%E4%B8%A4%E4%B8%AA%E7%BB%84%E4%BB%B6%EF%BC%9A%E6%9F%A5%E6%89%BE%E5%99%A8prospector%EF%BC%88%E6%8E%A2%E6%B5%8B%E8%80%85%EF%BC%89%E5%92%8C%E9%87%87%E9%9B%86%E5%99%A8harvester%EF%BC%88%E6%94%B6%E5%89%B2%E8%80%85%EF%BC%89%EF%BC%8C%E6%9D%A5%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6\(tail%20file\)%E5%B9%B6%E5%B0%86%E4%BA%8B%E4%BB%B6%E6%95%B0%E6%8D%AE%E5%8F%91%E9%80%81%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9A%84%E8%BE%93%E5%87%BA%E3%80%82%0A%0Aharvester%EF%BC%88%E6%94%B6%E5%89%B2%E8%80%85%EF%BC%89%0A\-\-\-\-\-%0A%E8%AF%BB%E5%8F%96%E5%8D%95%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84%E5%86%85%E5%AE%B9%EF%BC%88%E9%80%90%E8%A1%8C%EF%BC%89%EF%BC%8C%E5%B9%B6%E5%B0%86%E5%86%85%E5%AE%B9%E5%8F%91%E9%80%81%E5%88%B0the%20output%EF%BC%8C%E6%AF%8F%E4%B8%AA%E6%96%87%E4%BB%B6%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AAharvester%2C%20harvester%E8%B4%9F%E8%B4%A3%E6%89%93%E5%BC%80%E5%92%8C%E5%85%B3%E9%97%AD%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BF%99%E6%84%8F%E5%91%B3%E7%9D%80%E5%9C%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6%E4%BF%9D%E6%8C%81%E6%89%93%E5%BC%80%E7%8A%B6%E6%80%81%E3%80%82%0A%0A%0Aprospector%EF%BC%88%E6%8E%A2%E6%B5%8B%E5%99%A8%EF%BC%89%0A\-\-\-\-\-%0A%E7%AE%A1%E7%90%86%20harvester%20%E5%B9%B6%E6%89%BE%E5%88%B0%E6%89%80%E6%9C%89%E8%A6%81%E8%AF%BB%E5%8F%96%E7%9A%84%E6%96%87%E4%BB%B6%E6%9D%A5%E6%BA%90%0A%0A%E6%AF%8F%E8%AE%BE%E7%BD%AE%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97%E7%9B%91%E5%90%AC%E8%B7%AF%E5%BE%84%20%EF%BC%8C%E5%8D%B3%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%20%20prospector%0A%0A%0A%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F%E5%90%8E%EF%BC%8C%E7%94%A8kibina%0A%3Ekibana\-%3Eindex%20management\-%3Eck\-\*%0A%0A%E6%9F%A5%E7%9C%8Bfilebeat%E5%9C%A8es%E4%B8%AD%E6%96%B0%E5%BB%BA%E7%9A%84%E7%9B%B8%E5%85%B3%E7%B4%A2%E5%BC%95%E6%96%87%E4%BB%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%BA%E7%A9%BA%E7%9A%84%E8%AF%9D%EF%BC%8C%E6%B3%A8%E6%84%8F%EF%BC%9A%0A1.%20input%20%E7%9A%84pash%20%E6%98%AF%E5%90%A6%E6%AD%A3%E7%A1%AE%0A2.%20%E7%9B%AE%E5%BD%95%E4%B8%8B%E7%9A%84%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E6%98%AF%E5%90%A6%E5%AD%98%E5%9C%A8%E4%B8%94%E6%9C%89%E5%86%85%E5%AE%B9%0A%0A%E6%97%A5%E5%BF%97%E6%9F%A5%E7%9C%8B%EF%BC%8C%E9%9C%80%E8%A6%81%E5%BE%97%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20%E6%90%9C%E7%B4%A2%E5%99%A8%0Akibana\-%3Eindex%20patterns\-%3E%0A%0A%3Eck\-local\-\*%0A%0Aerror%20%3A%20Payload%20Too%20Large%0A%3E%E4%BF%AE%E6%94%B9%20kibana.yml%20%E4%B8%AD%E7%9A%84server.maxPayloadBytes
