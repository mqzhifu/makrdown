# go 项目容器编排

## 前置处理

服务器使用的是 ubutu，少许指令不同，不同OS换个指令即可

先查看下当前服务器是否已经安装过：MYSQL REDIS

> systemctl status mysql.service
> 
> 
> systemctl status redis.service

> service mysql stop
> 
> 
> service redis top

## 安装docker

lsb\_release \-a

> 我这里是 ubutu 20.04

注：我是全程开的ROOT，如果非ROOT 记得：sudo

如果OS里有旧的 DOCKER ，先检查一下，再删除了：

> apt\-get remove docker docker\-engine docker.io containerd runc

安装一些前置包：

> apt\-get install ca\-certificates curl gnupg lsb\-release

这里他会连带着安装一堆的依赖库

使用下面的 curl 导入源仓库的 GPG key：

> curl \-fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt\-key add \-

> apt\-get install software\-properties\-common

将 Docker APT 软件源添加到你的系统：

> add\-apt\-repository "deb \[arch=amd64\] https://download.docker.com/linux/ubuntu $\(lsb\_release \-cs\) stable"

> add\-apt\-repository
> 
> 
> "deb \[arch=amd64\] https://mirrors.aliyun.com/docker\-ce/linux/ubuntu
> 
> 
> $\(lsb\_release \-cs\)
> 
> 
> stable"

#### 安装 docker

> apt install docker\-ce docker\-ce\-cli containerd.io

查看 docker 状态

> systemctl status docker

## 安装MYSQL 5.7

下载镜像 ：

> docker pull mysql:5.7

mkdir /data/docker/mysql57/conf

mkdir /data/docker/mysql57/data

mkdir /data/docker/mysql57/log

首次启动：

docker run \-d \-p 3306:3306 \-\-name ckMysq57 

\-\-privileged=true 

\-v /data/docker/mysql57/conf:/etc/mysql/mysql.conf.d 

\-v /data/docker/mysql57/log:/var/log/ 

\-v /data/docker/mysql57/data:/var/lib/mysql 

\-e MYSQL\_USER="ckck" \-e MYSQL\_PASSWORD="123456" 

\-e MYSQL\_ROOT\_PASSWORD="123456" 

\-e \-\-character\-set\-server=utf8 

\-e \-\-collation\-server=utf8\_general\_ci 

mysql:5.7

如果启动失败：

> docker logs XXXX

## 安装 redis5

下载镜像：

> docker pull redis:5.0.3

创建挂载目录：

> mkdir \-p /data/docker/redis

创建配置文件:

> touch /data/docker/redis/redis.conf

```
loglevel notice
requirepass 123456
```

这里不用挂载了，直接启动：

> docker run \-d \-\-name ckRedis5 \-p 6379:6379
> 
> 
> redis:5.0.3 \-\-requirepass 111111

挂载式启动

> docker run \-d \-\-name ckRedis5 \-p 6379:6379
> 
> 
> \-v /data/docker/redis/redis.conf:/etc/redis/redis.conf
> 
> 
> redis:5.0.3

## 部署项目

最好先登陆，不然 docker build 会出错:

> docker login

```
#golang 环境
FROM golang:1.18-alpine AS builder
#FROM golang:1.18 AS build

#使用 alpine ，可以减少镜像大小。但是 alpine 默认的源在国内访问不了，需要修改为国内的源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

#安装编译需要的环境gcc等
RUN apk add build-base

#设置代码的工作目录，容器启动直接进入此目录
WORKDIR /app

#设置GOLANG的环境变量
ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct

#复制项目代码
COPY . .

#下载 goland 依赖包
#RUN go version;go env -w GO111MODULE=on;go env -w GOPROXY=https://goproxy.cn,direct;
RUN go mod tidy;

#编译项目代码
RUN go build -o ar120

#帮助文档
#RUN go install github.com/swaggo/swag/cmd/swag@v1.7.9;
#RUN $HOME/go/bin/swag init --parseDependency --parseInternal --parseDepth 3;
#RUN swag -v 

#开放的端口号，注：这里需要看一下项目中的配置文件，要保持一致
EXPOSE 3333 5555

CMD [ "./ar120","-e","5"]
```

开始创建镜像

> docker build \-t zgoframe:v2 .

执行镜像

> docker run \-d \-p 3333:3333 \-p 5555:5555 \-\-name myself2 zgoframe:v2

进到容器里看看

> docker exec \-it 4ea6d72da1d9 /bin/sh

%E5%89%8D%E7%BD%AE%E5%A4%84%E7%90%86%0A\-\-\-\-%0A%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%BF%E7%94%A8%E7%9A%84%E6%98%AF%20ubutu%EF%BC%8C%E5%B0%91%E8%AE%B8%E6%8C%87%E4%BB%A4%E4%B8%8D%E5%90%8C%EF%BC%8C%E4%B8%8D%E5%90%8COS%E6%8D%A2%E4%B8%AA%E6%8C%87%E4%BB%A4%E5%8D%B3%E5%8F%AF%0A%0A%E5%85%88%E6%9F%A5%E7%9C%8B%E4%B8%8B%E5%BD%93%E5%89%8D%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%98%AF%E5%90%A6%E5%B7%B2%E7%BB%8F%E5%AE%89%E8%A3%85%E8%BF%87%EF%BC%9AMYSQL%20REDIS%0A%0A%3Esystemctl%20status%20mysql.service%0A%3Esystemctl%20status%20redis.service%0A%0A%3Eservice%20mysql%20stop%0A%3Eservice%20redis%20top%0A%0A%0A%0A%0A%0A%E5%AE%89%E8%A3%85docker%0A\-\-\-\-%0A%20lsb\_release%20\-a%0A%20%3E%E6%88%91%E8%BF%99%E9%87%8C%E6%98%AF%20%20ubutu%2020.04%0A%0A%E6%B3%A8%EF%BC%9A%E6%88%91%E6%98%AF%E5%85%A8%E7%A8%8B%E5%BC%80%E7%9A%84ROOT%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%9D%9EROOT%20%E8%AE%B0%E5%BE%97%EF%BC%9Asudo%0A%0A%E5%A6%82%E6%9E%9COS%E9%87%8C%E6%9C%89%E6%97%A7%E7%9A%84%20DOCKER%20%EF%BC%8C%E5%85%88%E6%A3%80%E6%9F%A5%E4%B8%80%E4%B8%8B%EF%BC%8C%E5%86%8D%E5%88%A0%E9%99%A4%E4%BA%86%EF%BC%9A%0A%3Eapt\-get%20remove%20docker%20docker\-engine%20docker.io%20containerd%20runc%0A%0A%E5%AE%89%E8%A3%85%E4%B8%80%E4%BA%9B%E5%89%8D%E7%BD%AE%E5%8C%85%EF%BC%9A%0A%3Eapt\-get%20install%20ca\-certificates%20curl%20gnupg%20lsb\-release%0A%0A%E8%BF%99%E9%87%8C%E4%BB%96%E4%BC%9A%E8%BF%9E%E5%B8%A6%E7%9D%80%E5%AE%89%E8%A3%85%E4%B8%80%E5%A0%86%E7%9A%84%E4%BE%9D%E8%B5%96%E5%BA%93%0A%0A%E4%BD%BF%E7%94%A8%E4%B8%8B%E9%9D%A2%E7%9A%84%20curl%20%E5%AF%BC%E5%85%A5%E6%BA%90%E4%BB%93%E5%BA%93%E7%9A%84%20GPG%20key%EF%BC%9A%0A%3Ecurl%20\-fsSL%20https%3A%2F%2Fdownload.docker.com%2Flinux%2Fubuntu%2Fgpg%20%7C%20sudo%20apt\-key%20add%20\-%0A%0A%0A%3Eapt\-get%20install%20software\-properties\-common%0A%0A%E5%B0%86%20Docker%20APT%20%E8%BD%AF%E4%BB%B6%E6%BA%90%E6%B7%BB%E5%8A%A0%E5%88%B0%E4%BD%A0%E7%9A%84%E7%B3%BB%E7%BB%9F%EF%BC%9A%0A%3Eadd\-apt\-repository%20%22deb%20%5Barch%3Damd64%5D%20https%3A%2F%2Fdownload.docker.com%2Flinux%2Fubuntu%20%24\(lsb\_release%20\-cs\)%20stable%22%0A%0A%3Eadd\-apt\-repository%20%5C%0A%20%20%20%20%20%22deb%20%5Barch%3Damd64%5D%20https%3A%2F%2Fmirrors.aliyun.com%2Fdocker\-ce%2Flinux%2Fubuntu%20%5C%0A%20%20%20%20%20%24\(lsb\_release%20\-cs\)%20%5C%0A%20%20%20%20%20stable%22%0A%0A%23%23%23%23%20%E5%AE%89%E8%A3%85%20docker%0A%3Eapt%20install%20docker\-ce%20docker\-ce\-cli%20containerd.io%0A%0A%0A%E6%9F%A5%E7%9C%8B%20docker%20%E7%8A%B6%E6%80%81%0A%3Esystemctl%20status%20docker%0A%0A%0A%0A%0A%0A%E5%AE%89%E8%A3%85MYSQL%205.7%0A\-\-\-%0A%E4%B8%8B%E8%BD%BD%E9%95%9C%E5%83%8F%20%EF%BC%9A%0A%3Edocker%20pull%20mysql%3A5.7%0A%0Amkdir%20%2Fdata%2Fdocker%2Fmysql57%2Fconf%0Amkdir%20%2Fdata%2Fdocker%2Fmysql57%2Fdata%0Amkdir%20%2Fdata%2Fdocker%2Fmysql57%2Flog%0A%0A%E9%A6%96%E6%AC%A1%E5%90%AF%E5%8A%A8%EF%BC%9A%0A%0Adocker%20run%20\-d%20\-p%203306%3A3306%20\-\-name%20ckMysq57%20%20%20%5C%0A\-\-privileged%3Dtrue%20%20%5C%0A\-v%20%2Fdata%2Fdocker%2Fmysql57%2Fconf%3A%2Fetc%2Fmysql%2Fmysql.conf.d%20%5C%0A\-v%20%2Fdata%2Fdocker%2Fmysql57%2Flog%3A%2Fvar%2Flog%2F%20%5C%0A\-v%20%2Fdata%2Fdocker%2Fmysql57%2Fdata%3A%2Fvar%2Flib%2Fmysql%20%5C%0A\-e%20MYSQL\_USER%3D%22ckck%22%20\-e%20MYSQL\_PASSWORD%3D%22123456%22%20%5C%0A\-e%20MYSQL\_ROOT\_PASSWORD%3D%22123456%22%20%20%5C%0A\-e%20\-\-character\-set\-server%3Dutf8%20%5C%0A\-e%20\-\-collation\-server%3Dutf8\_general\_ci%20%5C%0Amysql%3A5.7%20%0A%0A%E5%A6%82%E6%9E%9C%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%EF%BC%9A%0A%3Edocker%20logs%20XXXX%0A%0A%0A%0A%E5%AE%89%E8%A3%85%20redis5%0A\-\-\-\-%0A%E4%B8%8B%E8%BD%BD%E9%95%9C%E5%83%8F%EF%BC%9A%0A%3Edocker%20pull%20redis%3A5.0.3%0A%0A%E5%88%9B%E5%BB%BA%E6%8C%82%E8%BD%BD%E7%9B%AE%E5%BD%95%EF%BC%9A%0A%3Emkdir%20\-p%20%2Fdata%2Fdocker%2Fredis%0A%0A%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%3A%0A%3Etouch%20%2Fdata%2Fdocker%2Fredis%2Fredis.conf%0A%60%60%60%0Aloglevel%20notice%0Arequirepass%20123456%0A%60%60%60%0A%0A%E8%BF%99%E9%87%8C%E4%B8%8D%E7%94%A8%E6%8C%82%E8%BD%BD%E4%BA%86%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%90%AF%E5%8A%A8%EF%BC%9A%0A%3Edocker%20run%20\-d%20\-\-name%20ckRedis5%20\-p%206379%3A6379%20%5C%0Aredis%3A5.0.3%20%20\-\-requirepass%20111111%0A%0A%E6%8C%82%E8%BD%BD%E5%BC%8F%E5%90%AF%E5%8A%A8%0A%3Edocker%20run%20\-d%20\-\-name%20ckRedis5%20\-p%206379%3A6379%20%20%5C%0A\-v%20%2Fdata%2Fdocker%2Fredis%2Fredis.conf%3A%2Fetc%2Fredis%2Fredis.conf%20%5C%0Aredis%3A5.0.3%20%0A%0A%0A%E9%83%A8%E7%BD%B2%E9%A1%B9%E7%9B%AE%0A\-\-\-\-%0A%E6%9C%80%E5%A5%BD%E5%85%88%E7%99%BB%E9%99%86%EF%BC%8C%E4%B8%8D%E7%84%B6%20docker%20build%20%E4%BC%9A%E5%87%BA%E9%94%99%3A%0A%3Edocker%20login%0A%0A%0A%60%60%60%0A%23golang%20%E7%8E%AF%E5%A2%83%0AFROM%20golang%3A1.18\-alpine%20AS%20builder%0A%23FROM%20golang%3A1.18%20AS%20build%0A%0A%23%E4%BD%BF%E7%94%A8%20alpine%20%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%87%8F%E5%B0%91%E9%95%9C%E5%83%8F%E5%A4%A7%E5%B0%8F%E3%80%82%E4%BD%86%E6%98%AF%20alpine%20%E9%BB%98%E8%AE%A4%E7%9A%84%E6%BA%90%E5%9C%A8%E5%9B%BD%E5%86%85%E8%AE%BF%E9%97%AE%E4%B8%8D%E4%BA%86%EF%BC%8C%E9%9C%80%E8%A6%81%E4%BF%AE%E6%94%B9%E4%B8%BA%E5%9B%BD%E5%86%85%E7%9A%84%E6%BA%90%0ARUN%20sed%20\-i%20's%2Fdl\-cdn.alpinelinux.org%2Fmirrors.aliyun.com%2Fg'%20%2Fetc%2Fapk%2Frepositories%0A%0A%23%E5%AE%89%E8%A3%85%E7%BC%96%E8%AF%91%E9%9C%80%E8%A6%81%E7%9A%84%E7%8E%AF%E5%A2%83gcc%E7%AD%89%0ARUN%20apk%20add%20build\-base%0A%0A%23%E8%AE%BE%E7%BD%AE%E4%BB%A3%E7%A0%81%E7%9A%84%E5%B7%A5%E4%BD%9C%E7%9B%AE%E5%BD%95%EF%BC%8C%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E7%9B%B4%E6%8E%A5%E8%BF%9B%E5%85%A5%E6%AD%A4%E7%9B%AE%E5%BD%95%0AWORKDIR%20%2Fapp%0A%0A%23%E8%AE%BE%E7%BD%AEGOLANG%E7%9A%84%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%0AENV%20GO111MODULE%3Don%20%5C%0A%20%20%20%20GOPROXY%3Dhttps%3A%2F%2Fgoproxy.cn%2Cdirect%0A%0A%23%E5%A4%8D%E5%88%B6%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81%0ACOPY%20.%20.%0A%0A%23%E4%B8%8B%E8%BD%BD%20goland%20%E4%BE%9D%E8%B5%96%E5%8C%85%0A%23RUN%20go%20version%3Bgo%20env%20\-w%20GO111MODULE%3Don%3Bgo%20env%20\-w%20GOPROXY%3Dhttps%3A%2F%2Fgoproxy.cn%2Cdirect%3B%0ARUN%20go%20mod%20tidy%3B%0A%0A%23%E7%BC%96%E8%AF%91%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81%0ARUN%20go%20build%20\-o%20ar120%0A%0A%23%E5%B8%AE%E5%8A%A9%E6%96%87%E6%A1%A3%0A%23RUN%20go%20install%20github.com%2Fswaggo%2Fswag%2Fcmd%2Fswag%40v1.7.9%3B%0A%23RUN%20%24HOME%2Fgo%2Fbin%2Fswag%20init%20\-\-parseDependency%20\-\-parseInternal%20\-\-parseDepth%203%3B%0A%23RUN%20swag%20\-v%20%0A%0A%23%E5%BC%80%E6%94%BE%E7%9A%84%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%8C%E6%B3%A8%EF%BC%9A%E8%BF%99%E9%87%8C%E9%9C%80%E8%A6%81%E7%9C%8B%E4%B8%80%E4%B8%8B%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E8%A6%81%E4%BF%9D%E6%8C%81%E4%B8%80%E8%87%B4%0AEXPOSE%203333%205555%0A%0ACMD%20%5B%20%22.%2Far120%22%2C%22\-e%22%2C%225%22%5D%0A%60%60%60%0A%0A%E5%BC%80%E5%A7%8B%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F%0A%3Edocker%20build%20\-t%20zgoframe%3Av2%20.%0A%0A%E6%89%A7%E8%A1%8C%E9%95%9C%E5%83%8F%0A%3Edocker%20run%20\-d%20\-p%203333%3A3333%20\-p%205555%3A5555%20\-\-name%20myself2%20zgoframe%3Av2%0A%0A%E8%BF%9B%E5%88%B0%E5%AE%B9%E5%99%A8%E9%87%8C%E7%9C%8B%E7%9C%8B%0A%3Edocker%20exec%20\-it%20%204ea6d72da1d9%20%2Fbin%2Fsh
