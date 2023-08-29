# grpc

## 概要

Remote Procedure Protocol:远程调用协议，A机器上的代码，通过函数调用B机器上的函数。应用于分布式。

Google Remote Procedure Protocol ：谷歌出口的一个RPC框架

> 有趣的是这个东西居然有很多语言版本，它不是一个单独的软件，可以把包嵌入到项目中使用，这个跟thfit差别好大。

支持如下语言：

C C\+\+ dart go java oc py ruby php node...

## 特点

1. 分布式
2. 跨语言
3. 统一的API定义文档

## 使用场景

1. 可以把复杂的大项拆分成若干个小的模块，再放到不同的服务器分布计算。
2. 项目的语言可以不局限一个，多语言 包容性 开发
3. 有一个统一的API接口文档，所有项目、语言统一维护

> 小项目，小公司没必要上这个东西，复杂度有点高。

## 安装

> 就以GO语言~为测试标准吧

先下2个包：

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

> protoc\-gen\-go 这东西我安protobuf的时候，已经安过了，跳过
> 
> 
> protoc\-gen\-go\-grpc 这个包好像不用安装，可能是因为安装过上面的包了吧

两个包都是辅助protoc脚本，生成特定语言的pb文件

这里注意：因为ETCD与高版本GRPC冲突，所以，只能降级protobuf grpc

protoc 3.5

protoc\-gen\-go@v1.2.0

## 使用

先看下目录结构：

```
├── make.sh
├── makepbservice.php   自动生成class-func的工具脚本
├── pb                 最终编译好，可用的GO文件
│   ├── common.pb.go    
│   └── demo.pb.go      
├── pbservice           自动生成func就是在这个文件夹子下
│   └── demo.proto.go   
└── proto
    ├── common.proto
    └── demo.proto     这个就是核心描述文件 
```

demo.proto,如下：

```
syntax = "proto3";

import "proto/common.proto";

package pb;
option go_package ="./;pb";

service First {
  rpc SayHello (RequestRegPlayer) returns (ResponseReg) {}
  rpc SayHi (RequestRegPlayer) returns (ResponseReg) {}
}

message RequestRegPlayer{
    int32               add_time    = 1;
    repeated    Player              player_list = 3;
    Player              player_info = 4;
    map<fixed64,Channel> channel_map = 5;
}

message Player{
    uint64       id             = 1;
    bytes       role_name       = 2;
    string      nickname        = 3;
    fixed32     status          = 4;
    float       score           = 5;
    PhoneType   phone_type      = 6;
    bool        sex            = 7;
    sint32      level           = 8;
}

message Channel{
    sint64 id = 1;
    string name = 2;
}

enum PhoneType {
    //allow_alias = true;
    mobile = 0;
    home = 1;
    word = 2;
}

message ResponseReg{
    bool    rs  = 1;
}

message MyHeader {
    Common common = 1;
}
```

这里定义了一个服务叫：First

## 生成PB文件

网上的方法：

> protoc \-I helloworld/ helloworld/helloworld.proto \-\-go\_out=plugins=grpc:helloworld

高版本protoc 方法

> protoc \-\-go\-grpc\_out=./pb ./proto/demo.proto

旧版本protoc 方法\(因为我的ETCD只能用grpc旧版，protoc也得跟着降版本\)

> protoc \-\-go\_out=plugins=grpc:. ./proto/demo.proto

## 代码阶段

server 和 client

详细的代码就不列了，自己百度吧

其中核心：监听一个TCP的IP:PORT，把这个监听传给上面生成好的PB即可。

## 原理

内部使用的是：http2.0，也就是说，虽然我开的是TCP协议监听，但整个交互是在TCP层面上又以HTTP传输。

HTTP2.0优缺点请跳转~

## grpc\-http2\-协议

既然是HTTP协议，那大体上相同：

HTTP头\+HTTP体

HTTP头=path\(/包名.服务名/方法名\)\+content\-type\(application/grpc\)

HTTP体=grpc包头\+业务数据

grpc包头=type类型位\+压缩标识位\+数据长度

数据读出来，根据方法名再找对应的proto类，写到类中，传给监听回调函数即完成

不太复杂感觉

## 实际应用

demo的方式，有些问题需要思考：

1. 每编译还得手动创建一个结构体，再挂上相关方法，有点麻烦
2. 如果有透传参数咋？
3. 如果有些公共处事件理咋办？如：日志、metrics等
4. 假设第1条生效，那能不能把用户的handle加进去呢？
5. 假设第4条成立，那么统一返回的值，是不是还得再封装一下？

解决办法：

1. 写个脚本，去分析.proto文件，帮忙生成相关类名\+函数名
2. 使用metadata透传值，毕竟是HTTP2.0，肯定有header
3. 第1条里的，加入公共代码
4. \(1\) 可以自己写，但牵涉到动态路由，有点麻烦呢。
    \(2\) c/s端增加拦截器
5. \(1\) 可以自己写
    \(2\) c/s端增加拦截器

## 拦截器

1. UnaryInterceptor 一元拦截器
2. StreamInterceptor 流拦截器
3. clientInterceptor 客户端

## rpc 对比 restful

1. rpc 性能要略好一些，毕竟是二进制传输，但依然还是HTTP协议。
2. rpc 可能更倾向于内部使用，分布式配合计算
3. restful 更适合一些对外的接口。
4. restful更简单，但凡有个浏览器都能访问。
5. rpc 倾向C/S结构，restful货币B/S结构

个人还是倾向restful，因为简单好用，调试容易~

## 总结

高级应用就得考虑各种各样的问题，而一旦考虑的多，就得聚合很多的代码进去，如：log metrics 链路追踪 错误报警 动态路由 统一回调结构体等等，又把项目复杂化了。原本如使用restful就挺简单，现在要加protobuf grpc库\(可能版本有兼容问题\)，再得加上编译过程\(学习protoœ\)，还得分配好目录结构，最后再加上这一堆代码...天呐~

rpc 只是个协议，GRPC thrif 才是真正实现了该协议的软件。但单看GRPC使用的HTTP2也没啥特别高深的东西，或者说没有实质性的创新与冲破。所以，没必要神话RPC，只是一种工具，具体还是得看使用场景。

## php、 go、protobuf

主要通信的就这3个东西，PHP \-\> protobuf \-\> GO

go安装比较简单，麻烦的是，安装 grpc 这个库，国内把google墙了，就得通过github 中转

先安装protobuf

wget protobuf\-all\-3.9.0.tar

tar \-zxvf

./autogen.sh

./configure \-\-prefix=/soft/protobuf

make

cp protoc /usr/bin/

protoc \-\-version

PHP扩展

wget https://pecl.php.net/get/grpc\-1.22.0.tgz

phpize

wget https://pecl.php.net/get/protobuf\-3.9.0.tgz

phpize

修改php.ini

extension=grpc.so

extension=protobuf.so

php \-m

这个扩展，对 GCC 有版本要求 。4.5 以上，所以还得再升级下GCC

wget http://gcc.skazkaforyou.com/releases/gcc\-4.8.2/gcc\-4.8.2.tar.gz

gcc \-v, g\+\+ \-v

这个是官方的grpc项目

git clone \-b $\(curl \-L https://grpc.io/release\) https://github.com/grpc/grpc

git clone https://github.com/grpc/grpc.git

//以上两个都行

//获取该项目的依赖的其它的包

git pull \-\-recurse\-submodules && git submodule update \-\-init \-\-recursive

//编译安装

make

make install

bin 下面的可执行文件，PHP在protoc.exe 文件的时候，需要个工具

cd examples/php

//安装 composer

curl \-sS https://getcomposer.org/installer | php

//用composer 安装 protobuf grpc ，工具类

php composer.phar install

//安装过程中，可能会报错，改下源

composer config repo.packagist composer https://packagist.phpcomposer.com

安装完后，会多一个vendor，查看下 protobuf grpc 文件都在不在

===========================================================

https://blog.csdn.net/java060515/article/details/84940938

wget https://dl.google.com/go/go1.12.7.linux\-amd64.tar.gz

解压

mv /soft/go

export GOROOT=/soft/go

export PATH="_P__A__T__H_:GOPATH/bin"

export GOWORK=/data0/www/gowork

export GOPATH=/data0/www/gowork //go get 目录

go get \-u github.com/golang/protobuf/proto

go get \-a github.com/golang/protobuf/protoc\-gen\-go

//很大概率会被墙，于是换套路

go install google.golang.org/grpc

src下新建 google.golang.org

cd google.golang.org

wget https://github.com/grpc/grpc\-go/archive/master.tar.gz

tar \-zxvf

mv grpc\-go\-master/ grpc

go install google.golang.org/grpc

依然会报错，大致是缺少依赖包，接着继续 安装

mkdir \-p src/golang.com/x

\#\!/bin/bash

MODULES="crypto net oauth2 sys text tools"

for module in _M__O__D__U__L__E__S__d__o__w__g__e__t__h__t__t__p__s_://_g__i__t__h__u__b_._c__o__m_/_g__o__l__a__n__g_/{module}/archive/master.tar.gz \-O _G__O__P__A__T__H_/_s__r__c_/_g__o__l__a__n__g_._o__r__g_/_x_/{module}.tar.gz

cd ${GOPATH}/src/golang.org/x && tar zxvf ${module}.tar.gz && mv ${module}\-master/ ${module}

done

依然报错，继续安装

wget https://github.com/google/go\-genproto/archive/master.tar.gz \-O ${GOPATH}/src/google.golang.org/genproto.tar.gz

cd ${GOPATH}/src/google.golang.org && tar zxvf genproto.tar.gz && mv go\-genproto\-master genproto

安装完成~NND

%E6%A6%82%E8%A6%81%0A\-\-\-\-\-\-%0ARemote%20Procedure%20Protocol%3A%E8%BF%9C%E7%A8%8B%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE%EF%BC%8CA%E6%9C%BA%E5%99%A8%E4%B8%8A%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E9%80%9A%E8%BF%87%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8B%E6%9C%BA%E5%99%A8%E4%B8%8A%E7%9A%84%E5%87%BD%E6%95%B0%E3%80%82%E5%BA%94%E7%94%A8%E4%BA%8E%E5%88%86%E5%B8%83%E5%BC%8F%E3%80%82%0AGoogle%20Remote%20Procedure%20Protocol%20%EF%BC%9A%E8%B0%B7%E6%AD%8C%E5%87%BA%E5%8F%A3%E7%9A%84%E4%B8%80%E4%B8%AARPC%E6%A1%86%E6%9E%B6%0A%0A%3E%E6%9C%89%E8%B6%A3%E7%9A%84%E6%98%AF%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%E5%B1%85%E7%84%B6%E6%9C%89%E5%BE%88%E5%A4%9A%E8%AF%AD%E8%A8%80%E7%89%88%E6%9C%AC%EF%BC%8C%E5%AE%83%E4%B8%8D%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8D%95%E7%8B%AC%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%8A%8A%E5%8C%85%E5%B5%8C%E5%85%A5%E5%88%B0%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8%EF%BC%8C%E8%BF%99%E4%B8%AA%E8%B7%9Fthfit%E5%B7%AE%E5%88%AB%E5%A5%BD%E5%A4%A7%E3%80%82%0A%0A%E6%94%AF%E6%8C%81%E5%A6%82%E4%B8%8B%E8%AF%AD%E8%A8%80%EF%BC%9A%0AC%20C%2B%2B%20dart%20go%20java%20oc%20py%20ruby%20php%20node...%0A%0A%E7%89%B9%E7%82%B9%0A\-\-\-\-%0A1.%20%E5%88%86%E5%B8%83%E5%BC%8F%0A2.%20%E8%B7%A8%E8%AF%AD%E8%A8%80%0A3.%20%E7%BB%9F%E4%B8%80%E7%9A%84API%E5%AE%9A%E4%B9%89%E6%96%87%E6%A1%A3%0A%0A%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%0A\-\-\-\-%0A1.%20%E5%8F%AF%E4%BB%A5%E6%8A%8A%E5%A4%8D%E6%9D%82%E7%9A%84%E5%A4%A7%E9%A1%B9%E6%8B%86%E5%88%86%E6%88%90%E8%8B%A5%E5%B9%B2%E4%B8%AA%E5%B0%8F%E7%9A%84%E6%A8%A1%E5%9D%97%EF%BC%8C%E5%86%8D%E6%94%BE%E5%88%B0%E4%B8%8D%E5%90%8C%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%88%86%E5%B8%83%E8%AE%A1%E7%AE%97%E3%80%82%0A2.%20%E9%A1%B9%E7%9B%AE%E7%9A%84%E8%AF%AD%E8%A8%80%E5%8F%AF%E4%BB%A5%E4%B8%8D%E5%B1%80%E9%99%90%E4%B8%80%E4%B8%AA%EF%BC%8C%E5%A4%9A%E8%AF%AD%E8%A8%80%20%E5%8C%85%E5%AE%B9%E6%80%A7%20%E5%BC%80%E5%8F%91%0A3.%20%E6%9C%89%E4%B8%80%E4%B8%AA%E7%BB%9F%E4%B8%80%E7%9A%84API%E6%8E%A5%E5%8F%A3%E6%96%87%E6%A1%A3%EF%BC%8C%E6%89%80%E6%9C%89%E9%A1%B9%E7%9B%AE%E3%80%81%E8%AF%AD%E8%A8%80%E7%BB%9F%E4%B8%80%E7%BB%B4%E6%8A%A4%0A%0A%3E%E5%B0%8F%E9%A1%B9%E7%9B%AE%EF%BC%8C%E5%B0%8F%E5%85%AC%E5%8F%B8%E6%B2%A1%E5%BF%85%E8%A6%81%E4%B8%8A%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%A4%8D%E6%9D%82%E5%BA%A6%E6%9C%89%E7%82%B9%E9%AB%98%E3%80%82%0A%0A%0A%E5%AE%89%E8%A3%85%0A\-\-\-\-\-%0A%3E%E5%B0%B1%E4%BB%A5GO%E8%AF%AD%E8%A8%80~%E4%B8%BA%E6%B5%8B%E8%AF%95%E6%A0%87%E5%87%86%E5%90%A7%0A%0A%E5%85%88%E4%B8%8B2%E4%B8%AA%E5%8C%85%EF%BC%9A%0A%60%60%60%0A%24%20go%20install%20google.golang.org%2Fprotobuf%2Fcmd%2Fprotoc\-gen\-go%40v1.26%0A%24%20go%20install%20google.golang.org%2Fgrpc%2Fcmd%2Fprotoc\-gen\-go\-grpc%40v1.1%0A%60%60%60%0A%0A%3Eprotoc\-gen\-go%20%E8%BF%99%E4%B8%9C%E8%A5%BF%E6%88%91%E5%AE%89protobuf%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%B7%B2%E7%BB%8F%E5%AE%89%E8%BF%87%E4%BA%86%EF%BC%8C%E8%B7%B3%E8%BF%87%0A%3Eprotoc\-gen\-go\-grpc%20%E8%BF%99%E4%B8%AA%E5%8C%85%E5%A5%BD%E5%83%8F%E4%B8%8D%E7%94%A8%E5%AE%89%E8%A3%85%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%98%AF%E5%9B%A0%E4%B8%BA%E5%AE%89%E8%A3%85%E8%BF%87%E4%B8%8A%E9%9D%A2%E7%9A%84%E5%8C%85%E4%BA%86%E5%90%A7%0A%0A%E4%B8%A4%E4%B8%AA%E5%8C%85%E9%83%BD%E6%98%AF%E8%BE%85%E5%8A%A9protoc%E8%84%9A%E6%9C%AC%EF%BC%8C%E7%94%9F%E6%88%90%E7%89%B9%E5%AE%9A%E8%AF%AD%E8%A8%80%E7%9A%84pb%E6%96%87%E4%BB%B6%20%0A%0A%0A%E8%BF%99%E9%87%8C%E6%B3%A8%E6%84%8F%EF%BC%9A%E5%9B%A0%E4%B8%BAETCD%E4%B8%8E%E9%AB%98%E7%89%88%E6%9C%ACGRPC%E5%86%B2%E7%AA%81%EF%BC%8C%E6%89%80%E4%BB%A5%EF%BC%8C%E5%8F%AA%E8%83%BD%E9%99%8D%E7%BA%A7protobuf%20grpc%20%0Aprotoc%203.5%0Aprotoc\-gen\-go%40v1.2.0%0A%0A%E4%BD%BF%E7%94%A8%0A\-\-\-\-%0A%0A%E5%85%88%E7%9C%8B%E4%B8%8B%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%EF%BC%9A%0A%60%60%60%0A%E2%94%9C%E2%94%80%E2%94%80%20make.sh%0A%E2%94%9C%E2%94%80%E2%94%80%20makepbservice.php%20%20%20%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90class\-func%E7%9A%84%E5%B7%A5%E5%85%B7%E8%84%9A%E6%9C%AC%0A%E2%94%9C%E2%94%80%E2%94%80%20pb%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%E6%9C%80%E7%BB%88%E7%BC%96%E8%AF%91%E5%A5%BD%EF%BC%8C%E5%8F%AF%E7%94%A8%E7%9A%84GO%E6%96%87%E4%BB%B6%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20common.pb.go%20%20%20%20%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20demo.pb.go%20%20%20%20%20%20%0A%E2%94%9C%E2%94%80%E2%94%80%20pbservice%20%20%20%20%20%20%20%20%20%20%20%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90func%E5%B0%B1%E6%98%AF%E5%9C%A8%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%AD%90%E4%B8%8B%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20demo.proto.go%20%20%20%0A%E2%94%94%E2%94%80%E2%94%80%20proto%0A%20%20%20%20%E2%94%9C%E2%94%80%E2%94%80%20common.proto%0A%20%20%20%20%E2%94%94%E2%94%80%E2%94%80%20demo.proto%20%20%20%20%20%E8%BF%99%E4%B8%AA%E5%B0%B1%E6%98%AF%E6%A0%B8%E5%BF%83%E6%8F%8F%E8%BF%B0%E6%96%87%E4%BB%B6%20%0A%60%60%60%0Ademo.proto%2C%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%60%60%60%0Asyntax%20%3D%20%22proto3%22%3B%0A%0Aimport%20%22proto%2Fcommon.proto%22%3B%0A%0Apackage%20pb%3B%0Aoption%20go\_package%20%3D%22.%2F%3Bpb%22%3B%0A%0Aservice%20First%20%7B%0A%20%20rpc%20SayHello%20\(RequestRegPlayer\)%20returns%20\(ResponseReg\)%20%7B%7D%0A%20%20rpc%20SayHi%20\(RequestRegPlayer\)%20returns%20\(ResponseReg\)%20%7B%7D%0A%7D%0A%0A%0A%0Amessage%20RequestRegPlayer%7B%0A%20%20%20%20int32%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20add\_time%20%20%20%20%3D%201%3B%0A%20%20%20%20repeated%20%20%20%20Player%20%20%20%20%20%20%20%20%20%20%20%20%20%20player\_list%20%3D%203%3B%0A%20%20%20%20Player%20%20%20%20%20%20%20%20%20%20%20%20%20%20player\_info%20%3D%204%3B%0A%20%20%20%20map%3Cfixed64%2CChannel%3E%20channel\_map%20%3D%205%3B%0A%7D%0A%0A%0Amessage%20Player%7B%0A%20%20%20%20uint64%20%20%20%20%20%20%20id%20%20%20%20%20%20%20%20%20%20%20%20%20%3D%201%3B%0A%20%20%20%20bytes%20%20%20%20%20%20%20role\_name%20%20%20%20%20%20%20%3D%202%3B%0A%20%20%20%20string%20%20%20%20%20%20nickname%20%20%20%20%20%20%20%20%3D%203%3B%0A%20%20%20%20fixed32%20%20%20%20%20status%20%20%20%20%20%20%20%20%20%20%3D%204%3B%0A%20%20%20%20float%20%20%20%20%20%20%20score%20%20%20%20%20%20%20%20%20%20%20%3D%205%3B%0A%20%20%20%20PhoneType%20%20%20phone\_type%20%20%20%20%20%20%3D%206%3B%0A%20%20%20%20bool%20%20%20%20%20%20%20%20sex%20%20%20%20%20%20%20%20%20%20%20%20%3D%207%3B%0A%20%20%20%20sint32%20%20%20%20%20%20level%20%20%20%20%20%20%20%20%20%20%20%3D%208%3B%0A%7D%0A%0Amessage%20Channel%7B%0A%20%20%20%20sint64%20id%20%3D%201%3B%0A%20%20%20%20string%20name%20%3D%202%3B%0A%7D%0A%0Aenum%20PhoneType%20%7B%0A%20%20%20%20%2F%2Fallow\_alias%20%3D%20true%3B%0A%20%20%20%20mobile%20%3D%200%3B%0A%20%20%20%20home%20%3D%201%3B%0A%20%20%20%20word%20%3D%202%3B%0A%7D%0A%0Amessage%20ResponseReg%7B%0A%20%20%20%20bool%20%20%20%20rs%20%20%3D%201%3B%0A%7D%0A%0Amessage%20MyHeader%20%7B%0A%20%20%20%20Common%20common%20%3D%201%3B%0A%7D%0A%60%60%60%0A%0A%E8%BF%99%E9%87%8C%E5%AE%9A%E4%B9%89%E4%BA%86%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%8F%AB%EF%BC%9AFirst%0A%0A%E7%94%9F%E6%88%90PB%E6%96%87%E4%BB%B6%0A\-\-\-\-%0A%0A%E7%BD%91%E4%B8%8A%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%9A%0A%3Eprotoc%20\-I%20helloworld%2F%20helloworld%2Fhelloworld.proto%20\-\-go\_out%3Dplugins%3Dgrpc%3Ahelloworld%0A%0A%E9%AB%98%E7%89%88%E6%9C%ACprotoc%20%E6%96%B9%E6%B3%95%0A%3Eprotoc%20%20\-\-go\-grpc\_out%3D.%2Fpb%20%20.%2Fproto%2Fdemo.proto%0A%0A%E6%97%A7%E7%89%88%E6%9C%ACprotoc%20%E6%96%B9%E6%B3%95\(%E5%9B%A0%E4%B8%BA%E6%88%91%E7%9A%84ETCD%E5%8F%AA%E8%83%BD%E7%94%A8grpc%E6%97%A7%E7%89%88%EF%BC%8Cprotoc%E4%B9%9F%E5%BE%97%E8%B7%9F%E7%9D%80%E9%99%8D%E7%89%88%E6%9C%AC\)%0A%3Eprotoc%20\-\-go\_out%3Dplugins%3Dgrpc%3A.%20.%2Fproto%2Fdemo.proto%0A%0A%0A%E4%BB%A3%E7%A0%81%E9%98%B6%E6%AE%B5%0A\-\-\-\-\-%0Aserver%20%E5%92%8C%20client%0A%0A%E8%AF%A6%E7%BB%86%E7%9A%84%E4%BB%A3%E7%A0%81%E5%B0%B1%E4%B8%8D%E5%88%97%E4%BA%86%EF%BC%8C%E8%87%AA%E5%B7%B1%E7%99%BE%E5%BA%A6%E5%90%A7%0A%0A%E5%85%B6%E4%B8%AD%E6%A0%B8%E5%BF%83%EF%BC%9A%E7%9B%91%E5%90%AC%E4%B8%80%E4%B8%AATCP%E7%9A%84IP%3APORT%EF%BC%8C%E6%8A%8A%E8%BF%99%E4%B8%AA%E7%9B%91%E5%90%AC%E4%BC%A0%E7%BB%99%E4%B8%8A%E9%9D%A2%E7%94%9F%E6%88%90%E5%A5%BD%E7%9A%84PB%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%E5%8E%9F%E7%90%86%0A\-\-\-\-%0A%E5%86%85%E9%83%A8%E4%BD%BF%E7%94%A8%E7%9A%84%E6%98%AF%EF%BC%9Ahttp2.0%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%EF%BC%8C%E8%99%BD%E7%84%B6%E6%88%91%E5%BC%80%E7%9A%84%E6%98%AFTCP%E5%8D%8F%E8%AE%AE%E7%9B%91%E5%90%AC%EF%BC%8C%E4%BD%86%E6%95%B4%E4%B8%AA%E4%BA%A4%E4%BA%92%E6%98%AF%E5%9C%A8TCP%E5%B1%82%E9%9D%A2%E4%B8%8A%E5%8F%88%E4%BB%A5HTTP%E4%BC%A0%E8%BE%93%E3%80%82%0A%0AHTTP2.0%E4%BC%98%E7%BC%BA%E7%82%B9%E8%AF%B7%E8%B7%B3%E8%BD%AC~%0A%0Agrpc\-http2\-%E5%8D%8F%E8%AE%AE%0A\-\-\-\-\-%0A%E6%97%A2%E7%84%B6%E6%98%AFHTTP%E5%8D%8F%E8%AE%AE%EF%BC%8C%E9%82%A3%E5%A4%A7%E4%BD%93%E4%B8%8A%E7%9B%B8%E5%90%8C%EF%BC%9A%0A%0AHTTP%E5%A4%B4%2BHTTP%E4%BD%93%0AHTTP%E5%A4%B4%3Dpath\(%2F%E5%8C%85%E5%90%8D.%E6%9C%8D%E5%8A%A1%E5%90%8D%2F%E6%96%B9%E6%B3%95%E5%90%8D\)%2Bcontent\-type\(application%2Fgrpc\)%0AHTTP%E4%BD%93%3Dgrpc%E5%8C%85%E5%A4%B4%2B%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%0Agrpc%E5%8C%85%E5%A4%B4%3Dtype%E7%B1%BB%E5%9E%8B%E4%BD%8D%2B%E5%8E%8B%E7%BC%A9%E6%A0%87%E8%AF%86%E4%BD%8D%2B%E6%95%B0%E6%8D%AE%E9%95%BF%E5%BA%A6%0A%0A%E6%95%B0%E6%8D%AE%E8%AF%BB%E5%87%BA%E6%9D%A5%EF%BC%8C%E6%A0%B9%E6%8D%AE%E6%96%B9%E6%B3%95%E5%90%8D%E5%86%8D%E6%89%BE%E5%AF%B9%E5%BA%94%E7%9A%84proto%E7%B1%BB%EF%BC%8C%E5%86%99%E5%88%B0%E7%B1%BB%E4%B8%AD%EF%BC%8C%E4%BC%A0%E7%BB%99%E7%9B%91%E5%90%AC%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%E5%8D%B3%E5%AE%8C%E6%88%90%0A%E4%B8%8D%E5%A4%AA%E5%A4%8D%E6%9D%82%E6%84%9F%E8%A7%89%0A%0A%E5%AE%9E%E9%99%85%E5%BA%94%E7%94%A8%0A\-\-\-\-\-\-\-%0Ademo%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C%E6%9C%89%E4%BA%9B%E9%97%AE%E9%A2%98%E9%9C%80%E8%A6%81%E6%80%9D%E8%80%83%EF%BC%9A%0A%0A1.%20%E6%AF%8F%E7%BC%96%E8%AF%91%E8%BF%98%E5%BE%97%E6%89%8B%E5%8A%A8%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E5%86%8D%E6%8C%82%E4%B8%8A%E7%9B%B8%E5%85%B3%E6%96%B9%E6%B3%95%EF%BC%8C%E6%9C%89%E7%82%B9%E9%BA%BB%E7%83%A6%0A2.%20%E5%A6%82%E6%9E%9C%E6%9C%89%E9%80%8F%E4%BC%A0%E5%8F%82%E6%95%B0%E5%92%8B%EF%BC%9F%0A3.%20%E5%A6%82%E6%9E%9C%E6%9C%89%E4%BA%9B%E5%85%AC%E5%85%B1%E5%A4%84%E4%BA%8B%E4%BB%B6%E7%90%86%E5%92%8B%E5%8A%9E%EF%BC%9F%E5%A6%82%EF%BC%9A%E6%97%A5%E5%BF%97%E3%80%81metrics%E7%AD%89%0A4.%20%E5%81%87%E8%AE%BE%E7%AC%AC1%E6%9D%A1%E7%94%9F%E6%95%88%EF%BC%8C%E9%82%A3%E8%83%BD%E4%B8%8D%E8%83%BD%E6%8A%8A%E7%94%A8%E6%88%B7%E7%9A%84handle%E5%8A%A0%E8%BF%9B%E5%8E%BB%E5%91%A2%EF%BC%9F%0A5.%20%E5%81%87%E8%AE%BE%E7%AC%AC4%E6%9D%A1%E6%88%90%E7%AB%8B%EF%BC%8C%E9%82%A3%E4%B9%88%E7%BB%9F%E4%B8%80%E8%BF%94%E5%9B%9E%E7%9A%84%E5%80%BC%EF%BC%8C%E6%98%AF%E4%B8%8D%E6%98%AF%E8%BF%98%E5%BE%97%E5%86%8D%E5%B0%81%E8%A3%85%E4%B8%80%E4%B8%8B%EF%BC%9F%0A%0A%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95%EF%BC%9A%0A%0A1.%20%E5%86%99%E4%B8%AA%E8%84%9A%E6%9C%AC%EF%BC%8C%E5%8E%BB%E5%88%86%E6%9E%90.proto%E6%96%87%E4%BB%B6%EF%BC%8C%E5%B8%AE%E5%BF%99%E7%94%9F%E6%88%90%E7%9B%B8%E5%85%B3%E7%B1%BB%E5%90%8D%2B%E5%87%BD%E6%95%B0%E5%90%8D%0A2.%20%E4%BD%BF%E7%94%A8metadata%E9%80%8F%E4%BC%A0%E5%80%BC%EF%BC%8C%E6%AF%95%E7%AB%9F%E6%98%AFHTTP2.0%EF%BC%8C%E8%82%AF%E5%AE%9A%E6%9C%89header%0A3.%20%E7%AC%AC1%E6%9D%A1%E9%87%8C%E7%9A%84%EF%BC%8C%E5%8A%A0%E5%85%A5%E5%85%AC%E5%85%B1%E4%BB%A3%E7%A0%81%0A4.%20%20\(1\)%20%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%B7%B1%E5%86%99%EF%BC%8C%E4%BD%86%E7%89%B5%E6%B6%89%E5%88%B0%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%EF%BC%8C%E6%9C%89%E7%82%B9%E9%BA%BB%E7%83%A6%E5%91%A2%E3%80%82%0A%20%20%20%20\(2\)%20c%2Fs%E7%AB%AF%E5%A2%9E%E5%8A%A0%E6%8B%A6%E6%88%AA%E5%99%A8%0A5.%20%20\(1\)%20%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%B7%B1%E5%86%99%20%0A%20%20%20%20\(2\)%20c%2Fs%E7%AB%AF%E5%A2%9E%E5%8A%A0%E6%8B%A6%E6%88%AA%E5%99%A8%0A%0A%E6%8B%A6%E6%88%AA%E5%99%A8%0A\-\-\-%0A1.%20UnaryInterceptor%20%20%E4%B8%80%E5%85%83%E6%8B%A6%E6%88%AA%E5%99%A8%0A2.%20StreamInterceptor%20%E6%B5%81%E6%8B%A6%E6%88%AA%E5%99%A8%0A3.%20clientInterceptor%20%E5%AE%A2%E6%88%B7%E7%AB%AF%0A%0A%0Arpc%20%E5%AF%B9%E6%AF%94%20restful%0A\-\-\-\-\-%0A1.%20rpc%20%E6%80%A7%E8%83%BD%E8%A6%81%E7%95%A5%E5%A5%BD%E4%B8%80%E4%BA%9B%EF%BC%8C%E6%AF%95%E7%AB%9F%E6%98%AF%E4%BA%8C%E8%BF%9B%E5%88%B6%E4%BC%A0%E8%BE%93%EF%BC%8C%E4%BD%86%E4%BE%9D%E7%84%B6%E8%BF%98%E6%98%AFHTTP%E5%8D%8F%E8%AE%AE%E3%80%82%0A2.%20rpc%20%E5%8F%AF%E8%83%BD%E6%9B%B4%E5%80%BE%E5%90%91%E4%BA%8E%E5%86%85%E9%83%A8%E4%BD%BF%E7%94%A8%EF%BC%8C%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E5%90%88%E8%AE%A1%E7%AE%97%0A3.%20restful%20%E6%9B%B4%E9%80%82%E5%90%88%E4%B8%80%E4%BA%9B%E5%AF%B9%E5%A4%96%E7%9A%84%E6%8E%A5%E5%8F%A3%E3%80%82%0A4.%20restful%E6%9B%B4%E7%AE%80%E5%8D%95%EF%BC%8C%E4%BD%86%E5%87%A1%E6%9C%89%E4%B8%AA%E6%B5%8F%E8%A7%88%E5%99%A8%E9%83%BD%E8%83%BD%E8%AE%BF%E9%97%AE%E3%80%82%0A5.%20rpc%20%E5%80%BE%E5%90%91C%2FS%E7%BB%93%E6%9E%84%EF%BC%8Crestful%E8%B4%A7%E5%B8%81B%2FS%E7%BB%93%E6%9E%84%0A%0A%E4%B8%AA%E4%BA%BA%E8%BF%98%E6%98%AF%E5%80%BE%E5%90%91restful%EF%BC%8C%E5%9B%A0%E4%B8%BA%E7%AE%80%E5%8D%95%E5%A5%BD%E7%94%A8%EF%BC%8C%E8%B0%83%E8%AF%95%E5%AE%B9%E6%98%93~%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-\-%0A%E9%AB%98%E7%BA%A7%E5%BA%94%E7%94%A8%E5%B0%B1%E5%BE%97%E8%80%83%E8%99%91%E5%90%84%E7%A7%8D%E5%90%84%E6%A0%B7%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%8C%E8%80%8C%E4%B8%80%E6%97%A6%E8%80%83%E8%99%91%E7%9A%84%E5%A4%9A%EF%BC%8C%E5%B0%B1%E5%BE%97%E8%81%9A%E5%90%88%E5%BE%88%E5%A4%9A%E7%9A%84%E4%BB%A3%E7%A0%81%E8%BF%9B%E5%8E%BB%EF%BC%8C%E5%A6%82%EF%BC%9Alog%20metrics%20%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%20%E9%94%99%E8%AF%AF%E6%8A%A5%E8%AD%A6%20%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%20%E7%BB%9F%E4%B8%80%E5%9B%9E%E8%B0%83%E7%BB%93%E6%9E%84%E4%BD%93%E7%AD%89%E7%AD%89%EF%BC%8C%E5%8F%88%E6%8A%8A%E9%A1%B9%E7%9B%AE%E5%A4%8D%E6%9D%82%E5%8C%96%E4%BA%86%E3%80%82%E5%8E%9F%E6%9C%AC%E5%A6%82%E4%BD%BF%E7%94%A8restful%E5%B0%B1%E6%8C%BA%E7%AE%80%E5%8D%95%EF%BC%8C%E7%8E%B0%E5%9C%A8%E8%A6%81%E5%8A%A0protobuf%20grpc%E5%BA%93\(%E5%8F%AF%E8%83%BD%E7%89%88%E6%9C%AC%E6%9C%89%E5%85%BC%E5%AE%B9%E9%97%AE%E9%A2%98\)%EF%BC%8C%E5%86%8D%E5%BE%97%E5%8A%A0%E4%B8%8A%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B\(%E5%AD%A6%E4%B9%A0proto%C5%93\)%EF%BC%8C%E8%BF%98%E5%BE%97%E5%88%86%E9%85%8D%E5%A5%BD%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%EF%BC%8C%E6%9C%80%E5%90%8E%E5%86%8D%E5%8A%A0%E4%B8%8A%E8%BF%99%E4%B8%80%E5%A0%86%E4%BB%A3%E7%A0%81...%E5%A4%A9%E5%91%90~%0A%0A%0Arpc%20%E5%8F%AA%E6%98%AF%E4%B8%AA%E5%8D%8F%E8%AE%AE%EF%BC%8CGRPC%20thrif%20%E6%89%8D%E6%98%AF%E7%9C%9F%E6%AD%A3%E5%AE%9E%E7%8E%B0%E4%BA%86%E8%AF%A5%E5%8D%8F%E8%AE%AE%E7%9A%84%E8%BD%AF%E4%BB%B6%E3%80%82%E4%BD%86%E5%8D%95%E7%9C%8BGRPC%E4%BD%BF%E7%94%A8%E7%9A%84HTTP2%E4%B9%9F%E6%B2%A1%E5%95%A5%E7%89%B9%E5%88%AB%E9%AB%98%E6%B7%B1%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E6%88%96%E8%80%85%E8%AF%B4%E6%B2%A1%E6%9C%89%E5%AE%9E%E8%B4%A8%E6%80%A7%E7%9A%84%E5%88%9B%E6%96%B0%E4%B8%8E%E5%86%B2%E7%A0%B4%E3%80%82%E6%89%80%E4%BB%A5%EF%BC%8C%E6%B2%A1%E5%BF%85%E8%A6%81%E7%A5%9E%E8%AF%9DRPC%EF%BC%8C%E5%8F%AA%E6%98%AF%E4%B8%80%E7%A7%8D%E5%B7%A5%E5%85%B7%EF%BC%8C%E5%85%B7%E4%BD%93%E8%BF%98%E6%98%AF%E5%BE%97%E7%9C%8B%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E3%80%82%0A%0Aphp%E3%80%81%20go%E3%80%81protobuf%0A\-\-\-\-\-\-%0A%E4%B8%BB%E8%A6%81%E9%80%9A%E4%BF%A1%E7%9A%84%E5%B0%B1%E8%BF%993%E4%B8%AA%E4%B8%9C%E8%A5%BF%EF%BC%8CPHP%20\-%3E%20protobuf%20\-%3E%20GO%0A%0Ago%E5%AE%89%E8%A3%85%E6%AF%94%E8%BE%83%E7%AE%80%E5%8D%95%EF%BC%8C%E9%BA%BB%E7%83%A6%E7%9A%84%E6%98%AF%EF%BC%8C%E5%AE%89%E8%A3%85%20grpc%20%E8%BF%99%E4%B8%AA%E5%BA%93%EF%BC%8C%E5%9B%BD%E5%86%85%E6%8A%8Agoogle%E5%A2%99%E4%BA%86%EF%BC%8C%E5%B0%B1%E5%BE%97%E9%80%9A%E8%BF%87github%20%E4%B8%AD%E8%BD%AC%0A%0A%E5%85%88%E5%AE%89%E8%A3%85protobuf%0Awget%20protobuf\-all\-3.9.0.tar%0Atar%20\-zxvf%20%0A.%2Fautogen.sh%0A.%2Fconfigure%20\-\-prefix%3D%2Fsoft%2Fprotobuf%0Amake%0Acp%20protoc%20%2Fusr%2Fbin%2F%0Aprotoc%20\-\-version%0A%0APHP%E6%89%A9%E5%B1%95%0Awget%20https%3A%2F%2Fpecl.php.net%2Fget%2Fgrpc\-1.22.0.tgz%0Aphpize%0A%0Awget%20https%3A%2F%2Fpecl.php.net%2Fget%2Fprotobuf\-3.9.0.tgz%0Aphpize%0A%0A%E4%BF%AE%E6%94%B9php.ini%20%0Aextension%3Dgrpc.so%0Aextension%3Dprotobuf.so%0A%0Aphp%20\-m%20%20%0A%0A%E8%BF%99%E4%B8%AA%E6%89%A9%E5%B1%95%EF%BC%8C%E5%AF%B9%20GCC%20%E6%9C%89%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82%20%E3%80%824.5%20%E4%BB%A5%E4%B8%8A%EF%BC%8C%E6%89%80%E4%BB%A5%E8%BF%98%E5%BE%97%E5%86%8D%E5%8D%87%E7%BA%A7%E4%B8%8BGCC%0Awget%20http%3A%2F%2Fgcc.skazkaforyou.com%2Freleases%2Fgcc\-4.8.2%2Fgcc\-4.8.2.tar.gz%0Agcc%20\-v%2C%20g%2B%2B%20\-v%0A%0A%0A%E8%BF%99%E4%B8%AA%E6%98%AF%E5%AE%98%E6%96%B9%E7%9A%84grpc%E9%A1%B9%E7%9B%AE%0Agit%20clone%20\-b%20%24\(curl%20\-L%20https%3A%2F%2Fgrpc.io%2Frelease\)%20https%3A%2F%2Fgithub.com%2Fgrpc%2Fgrpc%0Agit%20clone%20https%3A%2F%2Fgithub.com%2Fgrpc%2Fgrpc.git%0A%2F%2F%E4%BB%A5%E4%B8%8A%E4%B8%A4%E4%B8%AA%E9%83%BD%E8%A1%8C%0A%2F%2F%E8%8E%B7%E5%8F%96%E8%AF%A5%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BE%9D%E8%B5%96%E7%9A%84%E5%85%B6%E5%AE%83%E7%9A%84%E5%8C%85%0Agit%20pull%20\-\-recurse\-submodules%20%26%26%20git%20submodule%20update%20\-\-init%20\-\-recursive%0A%2F%2F%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%0Amake%0Amake%20install%0Abin%20%E4%B8%8B%E9%9D%A2%E7%9A%84%E5%8F%AF%E6%89%A7%E8%A1%8C%E6%96%87%E4%BB%B6%EF%BC%8CPHP%E5%9C%A8protoc.exe%20%E6%96%87%E4%BB%B6%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E9%9C%80%E8%A6%81%E4%B8%AA%E5%B7%A5%E5%85%B7%0A%0A%0Acd%20examples%2Fphp%0A%2F%2F%E5%AE%89%E8%A3%85%20composer%0Acurl%20\-sS%20https%3A%2F%2Fgetcomposer.org%2Finstaller%20%7C%20php%0A%2F%2F%E7%94%A8composer%20%E5%AE%89%E8%A3%85%20protobuf%20grpc%20%EF%BC%8C%E5%B7%A5%E5%85%B7%E7%B1%BB%0Aphp%20composer.phar%20install%0A%2F%2F%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B%E4%B8%AD%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%BC%9A%E6%8A%A5%E9%94%99%EF%BC%8C%E6%94%B9%E4%B8%8B%E6%BA%90%0Acomposer%20config%20repo.packagist%20composer%20https%3A%2F%2Fpackagist.phpcomposer.com%0A%0A%E5%AE%89%E8%A3%85%E5%AE%8C%E5%90%8E%EF%BC%8C%E4%BC%9A%E5%A4%9A%E4%B8%80%E4%B8%AAvendor%EF%BC%8C%E6%9F%A5%E7%9C%8B%E4%B8%8B%20protobuf%20grpc%20%E6%96%87%E4%BB%B6%E9%83%BD%E5%9C%A8%E4%B8%8D%E5%9C%A8%0A%0A%0A%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%0Ahttps%3A%2F%2Fblog.csdn.net%2Fjava060515%2Farticle%2Fdetails%2F84940938%0A%0A%0A%0Awget%20https%3A%2F%2Fdl.google.com%2Fgo%2Fgo1.12.7.linux\-amd64.tar.gz%0A%E8%A7%A3%E5%8E%8B%0Amv%20%2Fsoft%2Fgo%0Aexport%20GOROOT%3D%2Fsoft%2Fgo%0Aexport%20PATH%3D%22%24PATH%3A%24GOPATH%2Fbin%22%0Aexport%20GOWORK%3D%2Fdata0%2Fwww%2Fgowork%0Aexport%20GOPATH%3D%2Fdata0%2Fwww%2Fgowork%20%20%2F%2Fgo%20get%20%E7%9B%AE%E5%BD%95%20%0A%0A%0Ago%20get%20\-u%20github.com%2Fgolang%2Fprotobuf%2Fproto%0Ago%20get%20\-a%20github.com%2Fgolang%2Fprotobuf%2Fprotoc\-gen\-go%0A%0A%2F%2F%E5%BE%88%E5%A4%A7%E6%A6%82%E7%8E%87%E4%BC%9A%E8%A2%AB%E5%A2%99%EF%BC%8C%E4%BA%8E%E6%98%AF%E6%8D%A2%E5%A5%97%E8%B7%AF%0Ago%20install%20google.golang.org%2Fgrpc%0A%0Asrc%E4%B8%8B%E6%96%B0%E5%BB%BA%20google.golang.org%0Acd%20google.golang.org%0Awget%20https%3A%2F%2Fgithub.com%2Fgrpc%2Fgrpc\-go%2Farchive%2Fmaster.tar.gz%0Atar%20\-zxvf%0Amv%20grpc\-go\-master%2F%20grpc%0Ago%20install%20google.golang.org%2Fgrpc%0A%0A%0A%E4%BE%9D%E7%84%B6%E4%BC%9A%E6%8A%A5%E9%94%99%EF%BC%8C%E5%A4%A7%E8%87%B4%E6%98%AF%E7%BC%BA%E5%B0%91%E4%BE%9D%E8%B5%96%E5%8C%85%EF%BC%8C%E6%8E%A5%E7%9D%80%E7%BB%A7%E7%BB%AD%20%E5%AE%89%E8%A3%85%0A%0Amkdir%20\-p%20src%2Fgolang.com%2Fx%0A%0A%23\!%2Fbin%2Fbash%0AMODULES%3D%22crypto%20net%20oauth2%20sys%20text%20tools%22%0Afor%20module%20in%20%24%7BMODULES%7D%0Ado%0A%20%20%20%20wget%20https%3A%2F%2Fgithub.com%2Fgolang%2F%24%7Bmodule%7D%2Farchive%2Fmaster.tar.gz%20\-O%20%24%7BGOPATH%7D%2Fsrc%2Fgolang.org%2Fx%2F%24%7Bmodule%7D.tar.gz%0A%20%20%20%20cd%20%24%7BGOPATH%7D%2Fsrc%2Fgolang.org%2Fx%20%26%26%20tar%20zxvf%20%24%7Bmodule%7D.tar.gz%20%26%26%20mv%20%24%7Bmodule%7D\-master%2F%20%24%7Bmodule%7D%0Adone%0A%0A%E4%BE%9D%E7%84%B6%E6%8A%A5%E9%94%99%EF%BC%8C%E7%BB%A7%E7%BB%AD%E5%AE%89%E8%A3%85%0Awget%20https%3A%2F%2Fgithub.com%2Fgoogle%2Fgo\-genproto%2Farchive%2Fmaster.tar.gz%20\-O%20%24%7BGOPATH%7D%2Fsrc%2Fgoogle.golang.org%2Fgenproto.tar.gz%0Acd%20%24%7BGOPATH%7D%2Fsrc%2Fgoogle.golang.org%20%26%26%20tar%20zxvf%20genproto.tar.gz%20%26%26%20mv%20go\-genproto\-master%20genproto%0A%0A%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90~NND%0A
