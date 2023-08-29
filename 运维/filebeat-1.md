# filebeat

## 安装

还是elastic 出品，直接下载解压

配置示例文件：filebeat.reference.yml（包含所有未过时的配置项）

正常加载的配置文件：filebeat.yml

详情配置说明请跳转

## 启动

> filebeat \-e \-c filebeat.yml
> 
> 
> ./filebeat \-e \-c filebeat.yml \-d "publish"

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

这东西会造成 自动创建索引的时候，共计5000来个mapping，关关关...

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

%0A%E5%AE%89%E8%A3%85%0A\-\-\-\-\-\-%0A%E8%BF%98%E6%98%AFelastic%20%E5%87%BA%E5%93%81%EF%BC%8C%E7%9B%B4%E6%8E%A5%E4%B8%8B%E8%BD%BD%E8%A7%A3%E5%8E%8B%0A%0A%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B%E6%96%87%E4%BB%B6%EF%BC%9Afilebeat.reference.yml%EF%BC%88%E5%8C%85%E5%90%AB%E6%89%80%E6%9C%89%E6%9C%AA%E8%BF%87%E6%97%B6%E7%9A%84%E9%85%8D%E7%BD%AE%E9%A1%B9%EF%BC%89%20%20%0A%E6%AD%A3%E5%B8%B8%E5%8A%A0%E8%BD%BD%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%9Afilebeat.yml%0A%E8%AF%A6%E6%83%85%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E%E8%AF%B7%E8%B7%B3%E8%BD%AC%0A%0A%0A%E5%90%AF%E5%8A%A8%20%20%0A\-\-\-\-\-%0A%3Efilebeat%20\-e%20\-c%20filebeat.yml%20%20%0A%3E.%2Ffilebeat%20%20\-e%20\-c%20filebeat.yml%20\-d%20%22publish%22%0A%0Afilebeat%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E7%9A%84%E7%B4%A2%E5%BC%95%E5%8F%AF%E8%83%BD%E6%98%AF%E5%8F%AA%E8%AF%BB%E6%9D%83%E9%99%90%EF%BC%8C%E6%94%B9%E4%B8%80%E4%B8%8B%EF%BC%9A%0A%60%60%60%0A%0Acurl%20\-XPUT%20\-H%20'Content\-Type%3A%20application%2Fjson'%20http%3A%2F%2F59.110.167.206%3A9200%2F\_settings%20\-d%20'%0A%7B%0A%20%20%20%20%22index%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22blocks%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22read\_only\_allow\_delete%22%3A%20%22false%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D'%0A%0Acurl%20\-XPUT%20\-H%20'Content\-Type%3A%20application%2Fjson'%20http%3A%2F%2F59.110.167.206%3A9200%2Ffilebeat\-7.13.1\-2021.06.08\-000001%2F\_settings%20\-d%20'%0A%7B%0A%20%20%20%20%22index%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22blocks%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22read\_only\_allow\_delete%22%3A%20%22false%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D'%0A%60%60%60%0A%0A%0A%0A%0A%0A%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E5%B7%B2%E5%BC%80%E5%90%AF%E7%9A%84modules%3A%0A%0A%3E.%2Ffilebeat%20modules%20list%0A%0A%E8%BF%99%E4%B8%9C%E8%A5%BF%E4%BC%9A%E9%80%A0%E6%88%90%20%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%85%B1%E8%AE%A15000%E6%9D%A5%E4%B8%AAmapping%EF%BC%8C%E5%85%B3%E5%85%B3%E5%85%B3...%0A%0A%60%60%60%0A.%2Ffilebeat%20modules%20disable%20aws%0A%0A%60%60%60%0A%0A%0A%E5%8E%9F%E7%90%86%0A\-\-\-\-\-\-%0AFilebeat%E6%B6%89%E5%8F%8A%E4%B8%A4%E4%B8%AA%E7%BB%84%E4%BB%B6%EF%BC%9A%E6%9F%A5%E6%89%BE%E5%99%A8prospector%EF%BC%88%E6%8E%A2%E6%B5%8B%E8%80%85%EF%BC%89%E5%92%8C%E9%87%87%E9%9B%86%E5%99%A8harvester%EF%BC%88%E6%94%B6%E5%89%B2%E8%80%85%EF%BC%89%EF%BC%8C%E6%9D%A5%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6\(tail%20file\)%E5%B9%B6%E5%B0%86%E4%BA%8B%E4%BB%B6%E6%95%B0%E6%8D%AE%E5%8F%91%E9%80%81%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9A%84%E8%BE%93%E5%87%BA%E3%80%82%0A%0Aharvester%EF%BC%88%E6%94%B6%E5%89%B2%E8%80%85%EF%BC%89%0A\-\-\-\-\-%0A%E8%AF%BB%E5%8F%96%E5%8D%95%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84%E5%86%85%E5%AE%B9%EF%BC%88%E9%80%90%E8%A1%8C%EF%BC%89%EF%BC%8C%E5%B9%B6%E5%B0%86%E5%86%85%E5%AE%B9%E5%8F%91%E9%80%81%E5%88%B0the%20output%EF%BC%8C%E6%AF%8F%E4%B8%AA%E6%96%87%E4%BB%B6%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AAharvester%2C%20harvester%E8%B4%9F%E8%B4%A3%E6%89%93%E5%BC%80%E5%92%8C%E5%85%B3%E9%97%AD%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BF%99%E6%84%8F%E5%91%B3%E7%9D%80%E5%9C%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6%E4%BF%9D%E6%8C%81%E6%89%93%E5%BC%80%E7%8A%B6%E6%80%81%E3%80%82%0A%0A%0Aprospector%EF%BC%88%E6%8E%A2%E6%B5%8B%E5%99%A8%EF%BC%89%0A\-\-\-\-\-%0A%E7%AE%A1%E7%90%86%20harvester%20%E5%B9%B6%E6%89%BE%E5%88%B0%E6%89%80%E6%9C%89%E8%A6%81%E8%AF%BB%E5%8F%96%E7%9A%84%E6%96%87%E4%BB%B6%E6%9D%A5%E6%BA%90%0A%0A%E6%AF%8F%E8%AE%BE%E7%BD%AE%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97%E7%9B%91%E5%90%AC%E8%B7%AF%E5%BE%84%20%EF%BC%8C%E5%8D%B3%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%20%20prospector
