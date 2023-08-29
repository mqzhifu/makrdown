# NGINX-新

## nginx能做什么

核心功能：

|功能                  |例子                                          |
|----------------------|----------------------------------------------|
|实现了 HTTP/HTTPS 协议|可以做文件服务器                              |
|实现了 WEBSOCKET 协议 |浏览器也可以使用长连接                        |
|兼容OS                |win mac linux                                 |
|代理/反向代理         |给后端做代理，可以加一层日志、权限控制、头信息|
|负载均衡              |给后端一组服务器做负载均衡，并自带一些算法    |
|fastCgi               |PHP NODEJS                                    |

其它功能：

|功能          |例子                                          |
|--------------|----------------------------------------------|
|POP3/IMAP/SMTP|邮件处理                                      |
|MEMCACHE      |缓存处理                                      |
|LUA           |写些脚本，动态挂载到NGINX，控制NGINX的一些处理|
|RTMP          |视频流                                        |

小结：它的核心功能其实就是处理HTTP请求。不用你再去写代码了。并且它的性能高、稳定强、可扩展。

## nginx的核心优势

核心优点：

1. 性能高（处理发并发高）
2. 兼容主流平台
3. 满足主流功能的需求
4. 高度灵活，可模块化、可使用 LUA 嵌入开发、可用配置文件驱动

为什么性能高？

1. 小巧，并没有像 APACHE 融入太多功能。

> 原码文件一共就6MB左右\(未压缩的大小\)

1. 网络 IO 使用的是：EPOOL 模型

> epoll为什么快，参考另外一篇文章

## nginx 模块

从 src 目录分析：

|文件夹名|描述                                      |
|--------|------------------------------------------|
|core    |核心模块，nginx 大部分的操作都在这里      |
|event   |对 socketFD 的事件驱动处理                |
|http    |对 http/https 协议的处事                  |
|mail    |对邮件的处理                              |
|misc    |未知                                      |
|os      |网络、socketFD、文件IO（各种OS的兼容处理）|
|stream  |各种数据流的处理，upstream                |

core

|模块                                    |描述        |
|----------------------------------------|------------|
|array list string buffer tree hash queue|基础数据结构|
|crc crypt md5 sha1                      |加密算法    |
|cycle                                   |生存周期    |
|inet                                    |底层网络相关|
|log                                     |日志        |
|slab palloc                             |内存处理    |
|proxy                                   |代理        |
|regex                                   |正则        |
|config\_file                            |配置文件解析|

email

|模块  |描述|
|------|----|
|pop3  |    |
|realip|    |
|smtp  |    |
|auth  |    |

http

|模块                  |描述|
|----------------------|----|
|access                |    |
|addition\_filter      |    |
|auth\_basic           |    |
|auth\_request         |    |
|autoindex /index      |    |
|chatset\_filter       |    |
|fastcgi /scgi         |    |
|flv/ mp4              |    |
|geo/ geoip            |    |
|grpc                  |    |
|gunzip/gzip           |    |
|header\_filter        |    |
|image\_filter         |    |
|limit\_conn limit\_req|    |
|log                   |    |
|memcache              |    |
|mirror                |    |
|proxy                 |    |
|random\_index         |    |
|realip                |    |
|referer               |    |
|rewrite               |    |
|static                |    |
|stub\_status          |    |
|ssl                   |    |
|try\_files            |    |
|upstreamuserid\_filter|    |
|uwsgi                 |    |
|http2                 |    |

ngx\_core\_module:

|模块  |描述                                              |
|------|--------------------------------------------------|
|epoll |linux事件驱动库                                   |
|kqueue|unix linux事件驱动库                              |
|select|轮询 socketFD 操作库                              |
|core  |对 socketFD 的基础操作，如：accept connect openssl|

rds\-json\-nginx

lua\-nginx

## 进程组

master\+worker\+cache\(loader\+manager\)

master：主要管理 worker。如果某 wokrder 进程挂了，它会重新拉起

worker：处理下下游戏的网络请求。数量与CPU核数对应

> master也会创建 socker 监听端口，但最终还是会把处理权交给workder.

cache\-loader:上下游请求的缓存、常用文件的缓存

cache\-manager:主要是清理一些过期的缓存文件

## NGINX启动过程

这里主要是分析的 main 函数

1. ngx\_get\_options，主要用于解析命令行中的参数，

> 如：nginx \-s stop|start|restart

1. ngx\_time\_init，初始化并更新时间，

> 如： 全局变量 ngx\_cached\_time

1. ngx\_getpid，获取当前进程的pid。

> 用于发送重启，关闭等信号命令。

1. ngx\_log\_init，初始化日志，并得到日志的文件句柄ngx\_log\_file.fd
2. init\_cycle：Nginx的全局变量。

> 在内存池上创建一个默认大小1024的全局变量。

1. ngx\_save\_argv，保存Nginx命令行中的参数和变量,放到全局变量 ngx\_argv
2. ngx\_process\_options，将 ngx\_get\_options 中获得这些参数取值赋值到ngx\_cycle中。

> prefix, conf\_prefix, conf\_file, conf\_param等字段。

1. ngx\_os\_init：初始化系统相关变量，

> 如内存页面大小ngx\_pagesize,ngx\_cacheline\_size,最大连接数ngx\_max\_sockets等

1. ngx\_crc32\_table\_init，初始化一致性hash表，主要作用是加快查询
2. ngx\_add\_inherited\_sockets：主要是继承了socket的套接字。主要作用是热启动的时候需要平滑过渡
3. ngx\_preinit\_modules，主要是前置的初始化模块，对模块进行编号处理
4. ngx\_init\_cycle方法，完成全局变量cycle的初始化
5. ngx\_signal\_process，如果有信号，则进入ngx\_signal\_process方法。

> 例如./nginx \-s stop,则处理Nginx的停止信号

1. ngx\_get\_conf，得到核心模块 ngx\_core\_conf\_t 的配置文件指针
2. ngx\_create\_pidfile，创建pid文件。

> 例如：/usr/local/nginx\-1.4.7/nginx.pid

1. ngx\_master\_process\_cycle，这函数里面开始真正创建多个Nginx的子进程。这个方法包括子进程创建、事件监听、各种模块运行等都会包含进去

这里看，其主要的就是：

1. 创建全局变量
2. 解析/处理配置文件
3. 信号的注册与监听
4. 模块的初始化与创建
5. worker 进程组的创建、网络的监听

分析：ngx\_master\_process\_cycle

1. ngx\_master\_process\_cycle：
    1. 主进程进行信号的监听和处理
    2. 创建子进程
    3. 开启死循环模式
        ......
2. ngx\_start\_worker\_process
    1. 创建 N个wokrder进程（N=配置文件中的数量）
    2. 拿到新进程的 PID
    3. 将 PID 返回给 master ，用于信号处理
3. ngx\_spawn\_process:就是 fork个进程，拿个PID 做个简单的容错处理....
4. ngx\_worker\_process\_cycle：
    1. ngx\_workder\_process\_init
    2. ngx\_process\_events\_and\_timer
    3. 监听信号
    4. epool回调处理
    5. 开启死循环模式
5. ngx\_workder\_process\_init:
    1. 环境变量、全局变量
    2. 设备进程相关信息
6. ngx\_process\_events\_and\_timer：把自己注册到 epool 中去

## 配置文件

```
根配置 {
    ......
    http配置 {
        upstream配置 {
            ......
        }
        ......
        server配置块{
            ...... 
            location 配置块{
                ......
            }
        }
    }
}
```

优先集/包含关系：

> 根配置 \-\> http 配置 \-\> server配置块/upstream配置 \-\> location 配置块

根配置：这里主要是控制 nginx 相关，像： 主日志、epool、worker进程数等。基本上日常不用动，如果要动也是对高并发的优化处理。

http 配置：这里就对 HTTP 协议的整体配置，如：日志、最大连接数、数据压缩、限流等。跟上面差不多，很少改。

server 配置块：每一个域名就有一个 server 块。这里面就是具体的请求处理的配置块了。如：对域名、HTTS、公共页面配置\(404/403/503\)等。

location 配置块：这里就是具体到 URI/URL 、代理、跳转、URL重写、添加 http 头/响应头、FAST\-CGI等。

server 和 location 是日常程序员/运维 使用场景最多的。另外，server 跟 location 的一些配置是均可复用的，只是作用域不太一样。

## 跨域

add\_header Access\-Control\-Allow\-Origin \*;

add\_header Access\-Control\-Allow\-Headers X\-Requested\-With;

add\_header Access\-Control\-Allow\-Methods GET,POST,OPTIONS;

## 代理

只需要使用一个指令：proxy\_pass，即可。

跳转到3方网站：

```
location /baidu {
    proxy_pass http://www.baidu.com;
}
```

跳转到自己域内的一台服务器:

```
location /user {
    proxy_pass http://192.168.1.1:8080/user;
}
```

跳转到某一个服务器集合:backend，并做负载均衡

```
location /user {
    proxy_pass backent;
}

upstream backend {
    server 192.168.1.1;
    server 192.168.1.2;
    server 192.168.1.3;
    ......
}
```

其它的参数：

\#上游请求推出错误时 或 超时

proxy\_next\_upstream error timeout http\_503 http\_504 http\_502;

\#上游 建立连接 超时时间

proxy\_connect\_timeout 500s;

\#上游 读取超时 时间

proxy\_read\_timeout 500s;

\#上游 发送超时 时间

proxy\_send\_timeout 500s;

\#请求上游戏时 在HTTP 头部加上 host 信息

proxy\_set\_header Host $http\_host;

\#请求上游戏时 在HTTP 头部加上 IP 信息

proxy\_set\_header X\-Real\-IP $remote\_addr;

\#请求上游戏时 在HTTP 头部加上 客户端的信息 信息

proxy\_set\_header X\-Forwarded\-For $remote\_addr;

nginx的代理确实简单，就一个 proxy\_pass 。剩下的管理员可任意配置使用。其实，代理的核心原理也不难，就是把客户端的请求信息保存，然后再发送一个完全一样的请求，发送给后端服务器。

## upstream

容器，定义一组 server

例如：定义一组叫 backend 的 server

```
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;
    server backup1.example.com:8080 backup;
}
```

## 负载均衡

简单说，它就依赖一个指令：upstream，在这个基础之上，再配置点信息就实现了，依然是很简单。

1. 轮询
    每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
    upstream backserver {
    server 192.168.0.14;
    server 192.168.0.15;
    }
2. 指定权重
    用于后端服务器性能不均的情况
    upstream backserver {
    server 192.168.0.14 weight=10;
    server 192.168.0.15 weight=10;
    }
3. IP绑定\-hash
    每个访客固定访问一个后端服务器，可以解决session的问题
4. fair
    按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    upstream backserver {
    server server1;
    server server2;
    fair;
    }
5. url\_hash
    按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    upstream backserver {
    server squid1:3128;
    server squid2:3128;
    hash $request\_uri;
    hash\_method crc32;
    }

## 一次HTTP请求，主要的11个阶段

|步骤                             |NGINX的模块                                |
|---------------------------------|-------------------------------------------|
|NGX\_HTTP\_POST\_READ\_PHASE = 0 |realIp                                     |
|NGX\_HTTP\_SERVER\_REWRITE\_PHASE|rewrite                                    |
|NGX\_HTTP\_FIND\_CONFIG\_PHASE   |                                           |
|NGX\_HTTP\_REWRITE\_PHASE        |rewrite                                    |
|NGX\_HTTP\_POST\_REWRITE\_PHASE  |                                           |
|NGX\_HTTP\_PREACCESS\_PHASE      |limit\_req \-\> limit\_conn                |
|NGX\_HTTP\_ACCESS\_PHASE         |                                           |
|NGX\_HTTP\_POST\_ACCESS\_PHASE   |access auth\_basic auth\_request           |
|NGX\_HTTP\_PRECONTENT\_PHASE     |try\_file mirrors                          |
|NGX\_HTTP\_CONTENT\_PHASE        |concat random\_index index autoindex static|
|NGX\_HTTP\_LOG\_PHASE            |access log                                 |

> 结构体：ngx\_http\_phases

具体分析11步骤：

1. NGX\_HTTP\_POST\_READ\_PHASE：
    拿取HTTP头信息，如：IP PORT 代理IP ，并保存到变量中，如：remote\_addr

> 这里还有个 real\_ip 模块，可选项

1. NGX\_HTTP\_SERVER\_REWRITE\_PHASE：
    寻找 server 块中的 rewrite
    1. 404 403 502 error\_page
    2. return
2. NGX\_HTTP\_FIND\_CONFIG\_PHASE
    寻找 location，nginx 会把每个 server 中的 每个 location 正则提取到一个二叉树中做索引，然后请求的URI在树中做搜索
3. NGX\_HTTP\_REWRITE\_PHASE
    寻找 location 块中的 rewrite。这里可能会重复执行，因为rewrite 跳转一次，再进来可能继续rewrite ....
4. NGX\_HTTP\_POST\_REWRITE\_PHASE
    rewrite后，为防止死循环
5. NGX\_HTTP\_PREACCESS\_PHASE
    认证预处理，如：limit\_conn  limit\_req
6. NGX\_HTTP\_ACCESS\_PHASE
    权限认证：
    1. IP限制，如： deny 192.168.1.1; allow 192.168.1.0/24;
    2. auth\_basic ：用户认证，浏览器自带的用户认证机制。
    3. auth\_request ：权限认证，当有一个请求进来，NGINX 并不直接请求后端的服务器，而是先发一个 auth 请求，后端如果返回200，则才会正常代理该请求到后端，否则直接拒绝
7. NGX\_HTTP\_POST\_ACCESS\_PHASE
    权限认证后置处理
8. NGX\_HTTP\_PRECONTENT\_PHASE
    就一个 try\_file 模块
    1. 可以给 location 找不到文件的时候，做个默认文件\(重定向\)
    2. 可以增加目录，用于寻找URI对应的静态文件
9. NGX\_HTTP\_CONTENT\_PHASE
    响应内容
10. NGX\_HTTP\_LOG\_PHASE
    日志

## 总结

nginx 或 apache ，做为 webservice 类型的开源软件，确实是方便。通过一个配置文件，用于驱动使用 webservice。程序员不需要再做额外的开发，就能使用 http 相关的服务。另外，nginx 还保证了稳定、性能、可扩展等等。真的是省事省人省力。

不过，过度使用开源免费的软件，也会拉低程序员的基础水平。对底层的理解不够深刻、培训机构的填鸭式教育陪着了一堆混子等等吧。但咋说呢，应该顺应时代发展。向前发展，有些底层的原理也不一定完全能理解。站在巨人的肩膀上！

nginx%E8%83%BD%E5%81%9A%E4%BB%80%E4%B9%88%0A\-\-\-\-\-%0A%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%EF%BC%9A%0A%0A%0A%7C%E5%8A%9F%E8%83%BD%20%20%7C%E4%BE%8B%E5%AD%90%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%20%E5%AE%9E%E7%8E%B0%E4%BA%86%20HTTP%2FHTTPS%20%E5%8D%8F%E8%AE%AE%20%7C%20%E5%8F%AF%E4%BB%A5%E5%81%9A%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8%20%7C%0A%7C%20%E5%AE%9E%E7%8E%B0%E4%BA%86%20WEBSOCKET%20%E5%8D%8F%E8%AE%AE%20%7C%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E9%95%BF%E8%BF%9E%E6%8E%A5%20%7C%0A%7C%E5%85%BC%E5%AE%B9OS%20%20%7Cwin%20mac%20linux%20%20%7C%0A%7C%20%E4%BB%A3%E7%90%86%2F%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%20%7C%20%E7%BB%99%E5%90%8E%E7%AB%AF%E5%81%9A%E4%BB%A3%E7%90%86%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%8A%A0%E4%B8%80%E5%B1%82%E6%97%A5%E5%BF%97%E3%80%81%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6%E3%80%81%E5%A4%B4%E4%BF%A1%E6%81%AF%20%7C%0A%7C%20%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%20%7C%20%E7%BB%99%E5%90%8E%E7%AB%AF%E4%B8%80%E7%BB%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%81%9A%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%EF%BC%8C%E5%B9%B6%E8%87%AA%E5%B8%A6%E4%B8%80%E4%BA%9B%E7%AE%97%E6%B3%95%20%7C%0A%7C%20fastCgi%20%7C%20PHP%20NODEJS%20%7C%0A%0A%E5%85%B6%E5%AE%83%E5%8A%9F%E8%83%BD%EF%BC%9A%0A%7C%E5%8A%9F%E8%83%BD%20%7C%20%E4%BE%8B%E5%AD%90%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%20POP3%2FIMAP%2FSMTP%20%7C%E9%82%AE%E4%BB%B6%E5%A4%84%E7%90%86%20%20%7C%0A%7C%20MEMCACHE%20%7C%20%E7%BC%93%E5%AD%98%E5%A4%84%E7%90%86%20%7C%0A%7C%20LUA%20%7C%E5%86%99%E4%BA%9B%E8%84%9A%E6%9C%AC%EF%BC%8C%E5%8A%A8%E6%80%81%E6%8C%82%E8%BD%BD%E5%88%B0NGINX%EF%BC%8C%E6%8E%A7%E5%88%B6NGINX%E7%9A%84%E4%B8%80%E4%BA%9B%E5%A4%84%E7%90%86%20%20%7C%0A%7C%20RTMP%20%20%7C%E8%A7%86%E9%A2%91%E6%B5%81%20%20%7C%0A%0A%0A%E5%B0%8F%E7%BB%93%EF%BC%9A%E5%AE%83%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%E5%85%B6%E5%AE%9E%E5%B0%B1%E6%98%AF%E5%A4%84%E7%90%86HTTP%E8%AF%B7%E6%B1%82%E3%80%82%E4%B8%8D%E7%94%A8%E4%BD%A0%E5%86%8D%E5%8E%BB%E5%86%99%E4%BB%A3%E7%A0%81%E4%BA%86%E3%80%82%E5%B9%B6%E4%B8%94%E5%AE%83%E7%9A%84%E6%80%A7%E8%83%BD%E9%AB%98%E3%80%81%E7%A8%B3%E5%AE%9A%E5%BC%BA%E3%80%81%E5%8F%AF%E6%89%A9%E5%B1%95%E3%80%82%0A%0Anginx%E7%9A%84%E6%A0%B8%E5%BF%83%E4%BC%98%E5%8A%BF%0A\-\-\-%0A%E6%A0%B8%E5%BF%83%E4%BC%98%E7%82%B9%EF%BC%9A%0A1.%20%E6%80%A7%E8%83%BD%E9%AB%98%EF%BC%88%E5%A4%84%E7%90%86%E5%8F%91%E5%B9%B6%E5%8F%91%E9%AB%98%EF%BC%89%0A2.%20%E5%85%BC%E5%AE%B9%E4%B8%BB%E6%B5%81%E5%B9%B3%E5%8F%B0%0A3.%20%E6%BB%A1%E8%B6%B3%E4%B8%BB%E6%B5%81%E5%8A%9F%E8%83%BD%E7%9A%84%E9%9C%80%E6%B1%82%0A5.%20%E9%AB%98%E5%BA%A6%E7%81%B5%E6%B4%BB%EF%BC%8C%E5%8F%AF%E6%A8%A1%E5%9D%97%E5%8C%96%E3%80%81%E5%8F%AF%E4%BD%BF%E7%94%A8%20LUA%20%E5%B5%8C%E5%85%A5%E5%BC%80%E5%8F%91%E3%80%81%E5%8F%AF%E7%94%A8%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E9%A9%B1%E5%8A%A8%0A%0A%E4%B8%BA%E4%BB%80%E4%B9%88%E6%80%A7%E8%83%BD%E9%AB%98%EF%BC%9F%0A1.%20%E5%B0%8F%E5%B7%A7%EF%BC%8C%E5%B9%B6%E6%B2%A1%E6%9C%89%E5%83%8F%20APACHE%20%E8%9E%8D%E5%85%A5%E5%A4%AA%E5%A4%9A%E5%8A%9F%E8%83%BD%E3%80%82%0A%3E%E5%8E%9F%E7%A0%81%E6%96%87%E4%BB%B6%E4%B8%80%E5%85%B1%E5%B0%B16MB%E5%B7%A6%E5%8F%B3\(%E6%9C%AA%E5%8E%8B%E7%BC%A9%E7%9A%84%E5%A4%A7%E5%B0%8F\)%0A2.%20%E7%BD%91%E7%BB%9C%20IO%20%E4%BD%BF%E7%94%A8%E7%9A%84%E6%98%AF%EF%BC%9AEPOOL%20%E6%A8%A1%E5%9E%8B%0A%3Eepoll%E4%B8%BA%E4%BB%80%E4%B9%88%E5%BF%AB%EF%BC%8C%E5%8F%82%E8%80%83%E5%8F%A6%E5%A4%96%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%0A%0Anginx%20%E6%A8%A1%E5%9D%97%0A\-\-\-\-\-\-%0A%E4%BB%8E%20src%20%E7%9B%AE%E5%BD%95%E5%88%86%E6%9E%90%EF%BC%9A%0A%0A%7C%20%E6%96%87%E4%BB%B6%E5%A4%B9%E5%90%8D%20%7C%E6%8F%8F%E8%BF%B0%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20core%20%7C%20%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%EF%BC%8Cnginx%20%E5%A4%A7%E9%83%A8%E5%88%86%E7%9A%84%E6%93%8D%E4%BD%9C%E9%83%BD%E5%9C%A8%E8%BF%99%E9%87%8C%20%20%7C%0A%7C%20event%20%20%7C%20%E5%AF%B9%20socketFD%20%E7%9A%84%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E5%A4%84%E7%90%86%20%20%20%7C%0A%7C%20http%20%20%7C%20%E5%AF%B9%20http%2Fhttps%20%E5%8D%8F%E8%AE%AE%E7%9A%84%E5%A4%84%E4%BA%8B%20%20%7C%0A%7C%20mail%20%20%7C%20%E5%AF%B9%E9%82%AE%E4%BB%B6%E7%9A%84%E5%A4%84%E7%90%86%20%20%7C%0A%7C%20misc%20%20%7C%20%E6%9C%AA%E7%9F%A5%20%20%7C%0A%7C%20os%20%7C%20%E7%BD%91%E7%BB%9C%E3%80%81socketFD%E3%80%81%E6%96%87%E4%BB%B6IO%EF%BC%88%E5%90%84%E7%A7%8DOS%E7%9A%84%E5%85%BC%E5%AE%B9%E5%A4%84%E7%90%86%EF%BC%89%20%7C%0A%7C%20stream%20%7C%E5%90%84%E7%A7%8D%E6%95%B0%E6%8D%AE%E6%B5%81%E7%9A%84%E5%A4%84%E7%90%86%EF%BC%8Cupstream%20%7C%0A%0A%0Acore%0A%0A%7C%E6%A8%A1%E5%9D%97%20%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20array%20list%20string%20buffer%20tree%20hash%20queue%20%7C%20%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%20%7C%0A%7C%20crc%20crypt%20md5%20sha1%20%20%7C%20%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%20%20%7C%0A%7C%20cycle%20%7C%20%E7%94%9F%E5%AD%98%E5%91%A8%E6%9C%9F%20%7C%0A%7C%20inet%20%7C%20%E5%BA%95%E5%B1%82%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3%20%7C%0A%7Clog%20%20%7C%20%E6%97%A5%E5%BF%97%20%7C%0A%7C%20slab%20palloc%20%7C%E5%86%85%E5%AD%98%E5%A4%84%E7%90%86%20%20%7C%0A%7Cproxy%20%20%7C%20%E4%BB%A3%E7%90%86%20%7C%0A%7C%20regex%20%20%20%20%7C%E6%AD%A3%E5%88%99%20%20%7C%0A%7C%20%20config\_file%20%7C%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90%20%20%7C%0A%0Aemail%0A%0A%7C%20%E6%A8%A1%E5%9D%97%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7Cpop3%20%20%7C%20%20%7C%0A%7C%20realip%20%7C%20%20%7C%0A%7C%20smtp%20%7C%20%20%7C%0A%7C%20auth%20%7C%20%20%7C%0A%0A%0A%0A%0A%0Ahttp%20%0A%7C%20%E6%A8%A1%E5%9D%97%20%7C%E6%8F%8F%E8%BF%B0%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7Caccess%20%20%20%7C%20%20%7C%0A%7C%20addition\_filter%20%7C%20%20%7C%0A%7C%20auth\_basic%20%20%7C%20%20%7C%0A%7C%20%20auth\_request%20%7C%20%20%7C%0A%7C%20autoindex%20%2Findex%20%20%7C%20%20%7C%0A%7C%20%20chatset\_filter%20%20%7C%20%20%7C%0A%7Cfastcgi%20%2Fscgi%20%20%7C%20%20%7C%0A%7C%20%20flv%2F%20mp4%20%7C%20%20%7C%0A%7C%20%20geo%2F%20geoip%20%7C%20%20%7C%0A%7C%20grpc%20%20%7C%20%20%7C%0A%7C%20%20gunzip%2Fgzip%20%20%7C%20%20%7C%0A%7Cheader\_filter%20%20%20%7C%20%20%7C%0A%7C%20image\_filter%20%7C%20%20%7C%0A%7C%20limit\_conn%20limit\_req%20%20%7C%20%20%7C%0A%7C%20log%20%20%7C%20%7C%0A%7Cmemcache%20%20%7C%20%7C%0A%7C%20mirror%20%7C%20%7C%0A%7Cproxy%20%20%7C%20%7C%0A%7Crandom\_index%20%7C%20%7C%0A%7Crealip%20%20%7C%20%7C%0A%7C%20referer%20%7C%20%7C%0A%7Crewrite%20%20%7C%20%7C%0A%7Cstatic%20%20%7C%20%7C%0A%7C%20stub\_status%20%20%7C%20%7C%0A%7Cssl%20%7C%20%7C%0A%7Ctry\_files%20%7C%20%7C%0A%7Cupstreamuserid\_filter%20%7C%20%7C%0A%7Cuwsgi%20%7C%20%7C%0A%7Chttp2%20%7C%20%7C%0A%0A%0Angx\_core\_module%3A%0A%0A%7C%20%E6%A8%A1%E5%9D%97%7C%20%E6%8F%8F%E8%BF%B0%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7Cepoll%20%20%7C%20linux%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E5%BA%93%20%7C%0A%7C%20kqueue%20%7C%20unix%20linux%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E5%BA%93%20%20%7C%0A%7C%20select%20%7C%20%E8%BD%AE%E8%AF%A2%20socketFD%20%E6%93%8D%E4%BD%9C%E5%BA%93%20%20%7C%0A%7C%20core%20%20%7C%20%E5%AF%B9%20socketFD%20%E7%9A%84%E5%9F%BA%E7%A1%80%E6%93%8D%E4%BD%9C%EF%BC%8C%E5%A6%82%EF%BC%9Aaccept%20connect%20openssl%20%20%7C%0A%0A%0Ards\-json\-nginx%0Alua\-nginx%0A%0A%0A%0A%0A%0A%0A%E8%BF%9B%E7%A8%8B%E7%BB%84%0A\-\-\-\-\-%0Amaster%2Bworker%2Bcache\(loader%2Bmanager\)%0A%0Amaster%EF%BC%9A%E4%B8%BB%E8%A6%81%E7%AE%A1%E7%90%86%20worker%E3%80%82%E5%A6%82%E6%9E%9C%E6%9F%90%20wokrder%20%E8%BF%9B%E7%A8%8B%E6%8C%82%E4%BA%86%EF%BC%8C%E5%AE%83%E4%BC%9A%E9%87%8D%E6%96%B0%E6%8B%89%E8%B5%B7%0Aworker%EF%BC%9A%E5%A4%84%E7%90%86%E4%B8%8B%E4%B8%8B%E6%B8%B8%E6%88%8F%E7%9A%84%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E3%80%82%E6%95%B0%E9%87%8F%E4%B8%8ECPU%E6%A0%B8%E6%95%B0%E5%AF%B9%E5%BA%94%0A%3Emaster%E4%B9%9F%E4%BC%9A%E5%88%9B%E5%BB%BA%20socker%20%E7%9B%91%E5%90%AC%E7%AB%AF%E5%8F%A3%EF%BC%8C%E4%BD%86%E6%9C%80%E7%BB%88%E8%BF%98%E6%98%AF%E4%BC%9A%E6%8A%8A%E5%A4%84%E7%90%86%E6%9D%83%E4%BA%A4%E7%BB%99workder.%0A%0Acache\-loader%3A%E4%B8%8A%E4%B8%8B%E6%B8%B8%E8%AF%B7%E6%B1%82%E7%9A%84%E7%BC%93%E5%AD%98%E3%80%81%E5%B8%B8%E7%94%A8%E6%96%87%E4%BB%B6%E7%9A%84%E7%BC%93%E5%AD%98%0Acache\-manager%3A%E4%B8%BB%E8%A6%81%E6%98%AF%E6%B8%85%E7%90%86%E4%B8%80%E4%BA%9B%E8%BF%87%E6%9C%9F%E7%9A%84%E7%BC%93%E5%AD%98%E6%96%87%E4%BB%B6%0A%0ANGINX%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%0A\-\-\-\-%0A%E8%BF%99%E9%87%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E5%88%86%E6%9E%90%E7%9A%84%20main%20%E5%87%BD%E6%95%B0%0A%0A1.%20ngx\_get\_options%EF%BC%8C%E4%B8%BB%E8%A6%81%E7%94%A8%E4%BA%8E%E8%A7%A3%E6%9E%90%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%AD%E7%9A%84%E5%8F%82%E6%95%B0%EF%BC%8C%0A%3E%20%E5%A6%82%EF%BC%9Anginx%20\-s%20stop%7Cstart%7Crestart%0A2.%20ngx\_time\_init%EF%BC%8C%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B9%B6%E6%9B%B4%E6%96%B0%E6%97%B6%E9%97%B4%EF%BC%8C%0A%3E%20%E5%A6%82%EF%BC%9A%20%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%20ngx\_cached\_time%0A3.%20ngx\_getpid%EF%BC%8C%E8%8E%B7%E5%8F%96%E5%BD%93%E5%89%8D%E8%BF%9B%E7%A8%8B%E7%9A%84pid%E3%80%82%0A%3E%E7%94%A8%E4%BA%8E%E5%8F%91%E9%80%81%E9%87%8D%E5%90%AF%EF%BC%8C%E5%85%B3%E9%97%AD%E7%AD%89%E4%BF%A1%E5%8F%B7%E5%91%BD%E4%BB%A4%E3%80%82%0A4.%20ngx\_log\_init%EF%BC%8C%E5%88%9D%E5%A7%8B%E5%8C%96%E6%97%A5%E5%BF%97%EF%BC%8C%E5%B9%B6%E5%BE%97%E5%88%B0%E6%97%A5%E5%BF%97%E7%9A%84%E6%96%87%E4%BB%B6%E5%8F%A5%E6%9F%84ngx\_log\_file.fd%0A5.%20init\_cycle%EF%BC%9ANginx%E7%9A%84%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E3%80%82%0A%3E%E5%9C%A8%E5%86%85%E5%AD%98%E6%B1%A0%E4%B8%8A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%BB%98%E8%AE%A4%E5%A4%A7%E5%B0%8F1024%E7%9A%84%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E3%80%82%0A6.%20ngx\_save\_argv%EF%BC%8C%E4%BF%9D%E5%AD%98Nginx%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%AD%E7%9A%84%E5%8F%82%E6%95%B0%E5%92%8C%E5%8F%98%E9%87%8F%2C%E6%94%BE%E5%88%B0%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%20ngx\_argv%0A7.%20ngx\_process\_options%EF%BC%8C%E5%B0%86%20ngx\_get\_options%20%E4%B8%AD%E8%8E%B7%E5%BE%97%E8%BF%99%E4%BA%9B%E5%8F%82%E6%95%B0%E5%8F%96%E5%80%BC%E8%B5%8B%E5%80%BC%E5%88%B0ngx\_cycle%E4%B8%AD%E3%80%82%0A%3Eprefix%2C%20conf\_prefix%2C%20conf\_file%2C%20conf\_param%E7%AD%89%E5%AD%97%E6%AE%B5%E3%80%82%0A8.%20ngx\_os\_init%EF%BC%9A%E5%88%9D%E5%A7%8B%E5%8C%96%E7%B3%BB%E7%BB%9F%E7%9B%B8%E5%85%B3%E5%8F%98%E9%87%8F%EF%BC%8C%0A%3E%E5%A6%82%E5%86%85%E5%AD%98%E9%A1%B5%E9%9D%A2%E5%A4%A7%E5%B0%8Fngx\_pagesize%2Cngx\_cacheline\_size%2C%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0ngx\_max\_sockets%E7%AD%89%0A9.%20ngx\_crc32\_table\_init%EF%BC%8C%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E8%87%B4%E6%80%A7hash%E8%A1%A8%EF%BC%8C%E4%B8%BB%E8%A6%81%E4%BD%9C%E7%94%A8%E6%98%AF%E5%8A%A0%E5%BF%AB%E6%9F%A5%E8%AF%A2%0A10.%20ngx\_add\_inherited\_sockets%EF%BC%9A%E4%B8%BB%E8%A6%81%E6%98%AF%E7%BB%A7%E6%89%BF%E4%BA%86socket%E7%9A%84%E5%A5%97%E6%8E%A5%E5%AD%97%E3%80%82%E4%B8%BB%E8%A6%81%E4%BD%9C%E7%94%A8%E6%98%AF%E7%83%AD%E5%90%AF%E5%8A%A8%E7%9A%84%E6%97%B6%E5%80%99%E9%9C%80%E8%A6%81%E5%B9%B3%E6%BB%91%E8%BF%87%E6%B8%A1%0A11.%20ngx\_preinit\_modules%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E5%89%8D%E7%BD%AE%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E6%A8%A1%E5%9D%97%EF%BC%8C%E5%AF%B9%E6%A8%A1%E5%9D%97%E8%BF%9B%E8%A1%8C%E7%BC%96%E5%8F%B7%E5%A4%84%E7%90%86%0A12.%20ngx\_init\_cycle%E6%96%B9%E6%B3%95%EF%BC%8C%E5%AE%8C%E6%88%90%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8Fcycle%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%0A13.%20ngx\_signal\_process%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89%E4%BF%A1%E5%8F%B7%EF%BC%8C%E5%88%99%E8%BF%9B%E5%85%A5ngx\_signal\_process%E6%96%B9%E6%B3%95%E3%80%82%0A%3E%E4%BE%8B%E5%A6%82.%2Fnginx%20\-s%20stop%2C%E5%88%99%E5%A4%84%E7%90%86Nginx%E7%9A%84%E5%81%9C%E6%AD%A2%E4%BF%A1%E5%8F%B7%0A14.%20ngx\_get\_conf%EF%BC%8C%E5%BE%97%E5%88%B0%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%20ngx\_core\_conf\_t%20%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%8C%87%E9%92%88%0A15.%20ngx\_create\_pidfile%EF%BC%8C%E5%88%9B%E5%BB%BApid%E6%96%87%E4%BB%B6%E3%80%82%0A%3E%E4%BE%8B%E5%A6%82%EF%BC%9A%2Fusr%2Flocal%2Fnginx\-1.4.7%2Fnginx.pid%0A16.%20ngx\_master\_process\_cycle%EF%BC%8C%E8%BF%99%E5%87%BD%E6%95%B0%E9%87%8C%E9%9D%A2%E5%BC%80%E5%A7%8B%E7%9C%9F%E6%AD%A3%E5%88%9B%E5%BB%BA%E5%A4%9A%E4%B8%AANginx%E7%9A%84%E5%AD%90%E8%BF%9B%E7%A8%8B%E3%80%82%E8%BF%99%E4%B8%AA%E6%96%B9%E6%B3%95%E5%8C%85%E6%8B%AC%E5%AD%90%E8%BF%9B%E7%A8%8B%E5%88%9B%E5%BB%BA%E3%80%81%E4%BA%8B%E4%BB%B6%E7%9B%91%E5%90%AC%E3%80%81%E5%90%84%E7%A7%8D%E6%A8%A1%E5%9D%97%E8%BF%90%E8%A1%8C%E7%AD%89%E9%83%BD%E4%BC%9A%E5%8C%85%E5%90%AB%E8%BF%9B%E5%8E%BB%0A%0A%E8%BF%99%E9%87%8C%E7%9C%8B%EF%BC%8C%E5%85%B6%E4%B8%BB%E8%A6%81%E7%9A%84%E5%B0%B1%E6%98%AF%EF%BC%9A%0A1.%20%E5%88%9B%E5%BB%BA%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%0A2.%20%E8%A7%A3%E6%9E%90%2F%E5%A4%84%E7%90%86%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A3.%20%E4%BF%A1%E5%8F%B7%E7%9A%84%E6%B3%A8%E5%86%8C%E4%B8%8E%E7%9B%91%E5%90%AC%0A4.%20%E6%A8%A1%E5%9D%97%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%8E%E5%88%9B%E5%BB%BA%0A5.%20worker%20%E8%BF%9B%E7%A8%8B%E7%BB%84%E7%9A%84%E5%88%9B%E5%BB%BA%E3%80%81%E7%BD%91%E7%BB%9C%E7%9A%84%E7%9B%91%E5%90%AC%0A%0A%E5%88%86%E6%9E%90%EF%BC%9Angx\_master\_process\_cycle%0A%0A%0A1.%20ngx\_master\_process\_cycle%EF%BC%9A%0A%20%20%20%201.%20%E4%B8%BB%E8%BF%9B%E7%A8%8B%E8%BF%9B%E8%A1%8C%E4%BF%A1%E5%8F%B7%E7%9A%84%E7%9B%91%E5%90%AC%E5%92%8C%E5%A4%84%E7%90%86%0A%20%20%20%202.%20%E5%88%9B%E5%BB%BA%E5%AD%90%E8%BF%9B%E7%A8%8B%0A%20%20%20%203.%20%E5%BC%80%E5%90%AF%E6%AD%BB%E5%BE%AA%E7%8E%AF%E6%A8%A1%E5%BC%8F%0A%20%20%20%20......%0A2.%20ngx\_start\_worker\_process%0A%20%20%20%201.%20%E5%88%9B%E5%BB%BA%20N%E4%B8%AAwokrder%E8%BF%9B%E7%A8%8B%EF%BC%88N%3D%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84%E6%95%B0%E9%87%8F%EF%BC%89%0A%20%20%20%202.%20%E6%8B%BF%E5%88%B0%E6%96%B0%E8%BF%9B%E7%A8%8B%E7%9A%84%20PID%0A%20%20%20%203.%20%E5%B0%86%20PID%20%E8%BF%94%E5%9B%9E%E7%BB%99%20master%20%EF%BC%8C%E7%94%A8%E4%BA%8E%E4%BF%A1%E5%8F%B7%E5%A4%84%E7%90%86%0A3.%20ngx\_spawn\_process%3A%E5%B0%B1%E6%98%AF%20fork%E4%B8%AA%E8%BF%9B%E7%A8%8B%EF%BC%8C%E6%8B%BF%E4%B8%AAPID%20%E5%81%9A%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%AE%B9%E9%94%99%E5%A4%84%E7%90%86....%0A4.%20ngx\_worker\_process\_cycle%EF%BC%9A%0A%20%20%20%201.%20%20ngx\_workder\_process\_init%0A%20%20%20%202.%20ngx\_process\_events\_and\_timer%0A%20%20%20%203.%20%E7%9B%91%E5%90%AC%E4%BF%A1%E5%8F%B7%0A%20%20%20%204.%20epool%E5%9B%9E%E8%B0%83%E5%A4%84%E7%90%86%0A%20%20%20%205.%20%E5%BC%80%E5%90%AF%E6%AD%BB%E5%BE%AA%E7%8E%AF%E6%A8%A1%E5%BC%8F%0A5.%20ngx\_workder\_process\_init%3A%0A%20%20%20%201.%20%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E3%80%81%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%0A%20%20%20%202.%20%E8%AE%BE%E5%A4%87%E8%BF%9B%E7%A8%8B%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF%0A6.%20ngx\_process\_events\_and\_timer%EF%BC%9A%E6%8A%8A%E8%87%AA%E5%B7%B1%E6%B3%A8%E5%86%8C%E5%88%B0%20epool%20%E4%B8%AD%E5%8E%BB%0A%0A%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A\-\-\-\-%0A%60%60%60%0A%E6%A0%B9%E9%85%8D%E7%BD%AE%20%7B%0A%20%20%20%20......%0A%20%20%20%20http%E9%85%8D%E7%BD%AE%20%7B%0A%20%20%20%20%20%20%20%20upstream%E9%85%8D%E7%BD%AE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20......%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20......%0A%20%20%20%20%20%20%20%20server%E9%85%8D%E7%BD%AE%E5%9D%97%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20......%20%0A%20%20%20%20%20%20%20%20%20%20%20%20location%20%E9%85%8D%E7%BD%AE%E5%9D%97%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20......%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%E4%BC%98%E5%85%88%E9%9B%86%2F%E5%8C%85%E5%90%AB%E5%85%B3%E7%B3%BB%EF%BC%9A%0A%3E%E6%A0%B9%E9%85%8D%E7%BD%AE%20\-%3E%20http%20%E9%85%8D%E7%BD%AE%20\-%3E%20server%E9%85%8D%E7%BD%AE%E5%9D%97%2Fupstream%E9%85%8D%E7%BD%AE%20%20\-%3E%20location%20%E9%85%8D%E7%BD%AE%E5%9D%97%0A%0A%E6%A0%B9%E9%85%8D%E7%BD%AE%EF%BC%9A%E8%BF%99%E9%87%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E6%8E%A7%E5%88%B6%20nginx%20%E7%9B%B8%E5%85%B3%EF%BC%8C%E5%83%8F%EF%BC%9A%20%E4%B8%BB%E6%97%A5%E5%BF%97%E3%80%81epool%E3%80%81worker%E8%BF%9B%E7%A8%8B%E6%95%B0%E7%AD%89%E3%80%82%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%97%A5%E5%B8%B8%E4%B8%8D%E7%94%A8%E5%8A%A8%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%A6%81%E5%8A%A8%E4%B9%9F%E6%98%AF%E5%AF%B9%E9%AB%98%E5%B9%B6%E5%8F%91%E7%9A%84%E4%BC%98%E5%8C%96%E5%A4%84%E7%90%86%E3%80%82%0A%0Ahttp%20%E9%85%8D%E7%BD%AE%EF%BC%9A%E8%BF%99%E9%87%8C%E5%B0%B1%E5%AF%B9%20HTTP%20%E5%8D%8F%E8%AE%AE%E7%9A%84%E6%95%B4%E4%BD%93%E9%85%8D%E7%BD%AE%EF%BC%8C%E5%A6%82%EF%BC%9A%E6%97%A5%E5%BF%97%E3%80%81%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0%E3%80%81%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9%E3%80%81%E9%99%90%E6%B5%81%E7%AD%89%E3%80%82%E8%B7%9F%E4%B8%8A%E9%9D%A2%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E5%BE%88%E5%B0%91%E6%94%B9%E3%80%82%0A%0A%0Aserver%20%E9%85%8D%E7%BD%AE%E5%9D%97%EF%BC%9A%E6%AF%8F%E4%B8%80%E4%B8%AA%E5%9F%9F%E5%90%8D%E5%B0%B1%E6%9C%89%E4%B8%80%E4%B8%AA%20server%20%E5%9D%97%E3%80%82%E8%BF%99%E9%87%8C%E9%9D%A2%E5%B0%B1%E6%98%AF%E5%85%B7%E4%BD%93%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E7%9A%84%E9%85%8D%E7%BD%AE%E5%9D%97%E4%BA%86%E3%80%82%E5%A6%82%EF%BC%9A%E5%AF%B9%E5%9F%9F%E5%90%8D%E3%80%81HTTS%E3%80%81%E5%85%AC%E5%85%B1%E9%A1%B5%E9%9D%A2%E9%85%8D%E7%BD%AE\(404%2F403%2F503\)%E7%AD%89%E3%80%82%0A%0Alocation%20%E9%85%8D%E7%BD%AE%E5%9D%97%EF%BC%9A%E8%BF%99%E9%87%8C%E5%B0%B1%E6%98%AF%E5%85%B7%E4%BD%93%E5%88%B0%20URI%2FURL%20%E3%80%81%E4%BB%A3%E7%90%86%E3%80%81%E8%B7%B3%E8%BD%AC%E3%80%81URL%E9%87%8D%E5%86%99%E3%80%81%E6%B7%BB%E5%8A%A0%20http%20%E5%A4%B4%2F%E5%93%8D%E5%BA%94%E5%A4%B4%E3%80%81FAST\-CGI%E7%AD%89%E3%80%82%0A%0Aserver%20%E5%92%8C%20location%20%E6%98%AF%E6%97%A5%E5%B8%B8%E7%A8%8B%E5%BA%8F%E5%91%98%2F%E8%BF%90%E7%BB%B4%20%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E6%9C%80%E5%A4%9A%E7%9A%84%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8Cserver%20%E8%B7%9F%20location%20%E7%9A%84%E4%B8%80%E4%BA%9B%E9%85%8D%E7%BD%AE%E6%98%AF%E5%9D%87%E5%8F%AF%E5%A4%8D%E7%94%A8%E7%9A%84%EF%BC%8C%E5%8F%AA%E6%98%AF%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8D%E5%A4%AA%E4%B8%80%E6%A0%B7%E3%80%82%0A%0A%0A%0A%0A%E8%B7%A8%E5%9F%9F%0A\-\-\-\-%0Aadd\_header%20Access\-Control\-Allow\-Origin%20\*%3B%0Aadd\_header%20Access\-Control\-Allow\-Headers%20X\-Requested\-With%3B%0Aadd\_header%20Access\-Control\-Allow\-Methods%20GET%2CPOST%2COPTIONS%3B%0A%0A%E4%BB%A3%E7%90%86%0A\-\-\-%0A%E5%8F%AA%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8%E4%B8%80%E4%B8%AA%E6%8C%87%E4%BB%A4%EF%BC%9Aproxy\_pass%EF%BC%8C%E5%8D%B3%E5%8F%AF%E3%80%82%0A%20%E8%B7%B3%E8%BD%AC%E5%88%B03%E6%96%B9%E7%BD%91%E7%AB%99%EF%BC%9A%0A%60%60%60%0Alocation%20%2Fbaidu%20%7B%0A%20%20%20%20proxy\_pass%20http%3A%2F%2Fwww.baidu.com%3B%0A%7D%0A%60%60%60%0A%E8%B7%B3%E8%BD%AC%E5%88%B0%E8%87%AA%E5%B7%B1%E5%9F%9F%E5%86%85%E7%9A%84%E4%B8%80%E5%8F%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%3A%0A%60%60%60%0Alocation%20%2Fuser%20%7B%0A%20%20%20%20proxy\_pass%20http%3A%2F%2F192.168.1.1%3A8080%2Fuser%3B%0A%7D%0A%60%60%60%0A%E8%B7%B3%E8%BD%AC%E5%88%B0%E6%9F%90%E4%B8%80%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%9B%86%E5%90%88%3Abackend%EF%BC%8C%E5%B9%B6%E5%81%9A%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%0A%60%60%60%0Alocation%20%2Fuser%20%7B%0A%20%20%20%20proxy\_pass%20backent%3B%0A%7D%0A%0Aupstream%20backend%20%7B%0A%20%20%20%20server%20192.168.1.1%3B%0A%20%20%20%20server%20192.168.1.2%3B%0A%20%20%20%20server%20192.168.1.3%3B%0A%20%20%20%20......%0A%7D%0A%60%60%60%0A%0A%E5%85%B6%E5%AE%83%E7%9A%84%E5%8F%82%E6%95%B0%EF%BC%9A%0A%0A%23%E4%B8%8A%E6%B8%B8%E8%AF%B7%E6%B1%82%E6%8E%A8%E5%87%BA%E9%94%99%E8%AF%AF%E6%97%B6%20%E6%88%96%20%E8%B6%85%E6%97%B6%0Aproxy\_next\_upstream%20error%20timeout%20http\_503%20http\_504%20http\_502%3B%20%20%0A%23%E4%B8%8A%E6%B8%B8%20%E5%BB%BA%E7%AB%8B%E8%BF%9E%E6%8E%A5%20%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%0Aproxy\_connect\_timeout%20500s%3B%0A%23%E4%B8%8A%E6%B8%B8%20%E8%AF%BB%E5%8F%96%E8%B6%85%E6%97%B6%20%E6%97%B6%E9%97%B4%0Aproxy\_read\_timeout%20500s%3B%0A%23%E4%B8%8A%E6%B8%B8%20%E5%8F%91%E9%80%81%E8%B6%85%E6%97%B6%20%E6%97%B6%E9%97%B4%0Aproxy\_send\_timeout%20500s%3B%0A%23%E8%AF%B7%E6%B1%82%E4%B8%8A%E6%B8%B8%E6%88%8F%E6%97%B6%20%E5%9C%A8HTTP%20%E5%A4%B4%E9%83%A8%E5%8A%A0%E4%B8%8A%20host%20%E4%BF%A1%E6%81%AF%0Aproxy\_set\_header%20Host%20%24http\_host%3B%0A%23%E8%AF%B7%E6%B1%82%E4%B8%8A%E6%B8%B8%E6%88%8F%E6%97%B6%20%E5%9C%A8HTTP%20%E5%A4%B4%E9%83%A8%E5%8A%A0%E4%B8%8A%20IP%20%E4%BF%A1%E6%81%AF%0Aproxy\_set\_header%20X\-Real\-IP%20%24remote\_addr%3B%0A%23%E8%AF%B7%E6%B1%82%E4%B8%8A%E6%B8%B8%E6%88%8F%E6%97%B6%20%E5%9C%A8HTTP%20%E5%A4%B4%E9%83%A8%E5%8A%A0%E4%B8%8A%20%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E4%BF%A1%E6%81%AF%20%E4%BF%A1%E6%81%AF%0Aproxy\_set\_header%20X\-Forwarded\-For%20%24remote\_addr%3B%0A%0Anginx%E7%9A%84%E4%BB%A3%E7%90%86%E7%A1%AE%E5%AE%9E%E7%AE%80%E5%8D%95%EF%BC%8C%E5%B0%B1%E4%B8%80%E4%B8%AA%20proxy\_pass%20%E3%80%82%E5%89%A9%E4%B8%8B%E7%9A%84%E7%AE%A1%E7%90%86%E5%91%98%E5%8F%AF%E4%BB%BB%E6%84%8F%E9%85%8D%E7%BD%AE%E4%BD%BF%E7%94%A8%E3%80%82%E5%85%B6%E5%AE%9E%EF%BC%8C%E4%BB%A3%E7%90%86%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86%E4%B9%9F%E4%B8%8D%E9%9A%BE%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%8A%8A%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E8%AF%B7%E6%B1%82%E4%BF%A1%E6%81%AF%E4%BF%9D%E5%AD%98%EF%BC%8C%E7%84%B6%E5%90%8E%E5%86%8D%E5%8F%91%E9%80%81%E4%B8%80%E4%B8%AA%E5%AE%8C%E5%85%A8%E4%B8%80%E6%A0%B7%E7%9A%84%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%8F%91%E9%80%81%E7%BB%99%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E3%80%82%0A%0A%0Aupstream%0A\-\-\-\-\-\-\-\-%0A%E5%AE%B9%E5%99%A8%EF%BC%8C%E5%AE%9A%E4%B9%89%E4%B8%80%E7%BB%84%20server%0A%E4%BE%8B%E5%A6%82%EF%BC%9A%E5%AE%9A%E4%B9%89%E4%B8%80%E7%BB%84%E5%8F%AB%20backend%20%E7%9A%84%20server%0A%60%60%60%0Aupstream%20backend%20%7B%0A%20%20%20%20server%20backend1.example.com%20weight%3D5%3B%0A%20%20%20%20server%20127.0.0.1%3A8080%20max\_fails%3D3%20fail\_timeout%3D30s%3B%0A%20%20%20%20server%20unix%3A%2Ftmp%2Fbackend3%3B%0A%20%20%20%20server%20backup1.example.com%3A8080%20backup%3B%0A%7D%0A%60%60%60%0A%0A%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%0A\-\-\-\-\-%0A%E7%AE%80%E5%8D%95%E8%AF%B4%EF%BC%8C%E5%AE%83%E5%B0%B1%E4%BE%9D%E8%B5%96%E4%B8%80%E4%B8%AA%E6%8C%87%E4%BB%A4%EF%BC%9Aupstream%EF%BC%8C%E5%9C%A8%E8%BF%99%E4%B8%AA%E5%9F%BA%E7%A1%80%E4%B9%8B%E4%B8%8A%EF%BC%8C%E5%86%8D%E9%85%8D%E7%BD%AE%E7%82%B9%E4%BF%A1%E6%81%AF%E5%B0%B1%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%8C%E4%BE%9D%E7%84%B6%E6%98%AF%E5%BE%88%E7%AE%80%E5%8D%95%E3%80%82%0A%0A1.%20%E8%BD%AE%E8%AF%A2%0A%E6%AF%8F%E4%B8%AA%E8%AF%B7%E6%B1%82%E6%8C%89%E6%97%B6%E9%97%B4%E9%A1%BA%E5%BA%8F%E9%80%90%E4%B8%80%E5%88%86%E9%85%8D%E5%88%B0%E4%B8%8D%E5%90%8C%E7%9A%84%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8down%E6%8E%89%EF%BC%8C%E8%83%BD%E8%87%AA%E5%8A%A8%E5%89%94%E9%99%A4%E3%80%82%0Aupstream%20backserver%20%7B%0A%20%20%20%20server%20192.168.0.14%3B%0A%20%20%20%20server%20192.168.0.15%3B%0A%7D%0A%0A2.%20%E6%8C%87%E5%AE%9A%E6%9D%83%E9%87%8D%0A%E7%94%A8%E4%BA%8E%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%80%A7%E8%83%BD%E4%B8%8D%E5%9D%87%E7%9A%84%E6%83%85%E5%86%B5%0Aupstream%20backserver%20%7B%0A%20%20%20%20server%20192.168.0.14%20weight%3D10%3B%0A%20%20%20%20server%20192.168.0.15%20weight%3D10%3B%0A%7D%0A3.%20IP%E7%BB%91%E5%AE%9A\-hash%0A%E6%AF%8F%E4%B8%AA%E8%AE%BF%E5%AE%A2%E5%9B%BA%E5%AE%9A%E8%AE%BF%E9%97%AE%E4%B8%80%E4%B8%AA%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%A7%A3%E5%86%B3session%E7%9A%84%E9%97%AE%E9%A2%98%0A4.%20fair%0A%E6%8C%89%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E5%93%8D%E5%BA%94%E6%97%B6%E9%97%B4%E6%9D%A5%E5%88%86%E9%85%8D%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%93%8D%E5%BA%94%E6%97%B6%E9%97%B4%E7%9F%AD%E7%9A%84%E4%BC%98%E5%85%88%E5%88%86%E9%85%8D%E3%80%82%0Aupstream%20backserver%20%7B%0A%20%20%20%20server%20server1%3B%0A%20%20%20%20server%20server2%3B%0A%20%20%20%20fair%3B%0A%7D%0A5.%20url\_hash%0A%E6%8C%89%E8%AE%BF%E9%97%AEurl%E7%9A%84hash%E7%BB%93%E6%9E%9C%E6%9D%A5%E5%88%86%E9%85%8D%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%BD%BF%E6%AF%8F%E4%B8%AAurl%E5%AE%9A%E5%90%91%E5%88%B0%E5%90%8C%E4%B8%80%E4%B8%AA%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%BA%E7%BC%93%E5%AD%98%E6%97%B6%E6%AF%94%E8%BE%83%E6%9C%89%E6%95%88%E3%80%82%0Aupstream%20backserver%20%7B%0A%20%20%20%20server%20squid1%3A3128%3B%0A%20%20%20%20server%20squid2%3A3128%3B%0A%20%20%20%20hash%20%24request\_uri%3B%0A%20%20%20%20hash\_method%20crc32%3B%0A%7D%0A%0A%0A%0A%0A%0A%0A%0A%E4%B8%80%E6%AC%A1HTTP%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%B8%BB%E8%A6%81%E7%9A%8411%E4%B8%AA%E9%98%B6%E6%AE%B5%0A\-\-\-\-\-%0A%7C%E6%AD%A5%E9%AA%A4%20%20%7CNGINX%E7%9A%84%E6%A8%A1%E5%9D%97%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7CNGX\_HTTP\_POST\_READ\_PHASE%C2%A0%3D%C2%A00%20%7C%20realIp%20%7C%0A%7C%20NGX\_HTTP\_SERVER\_REWRITE\_PHASE%20%7Crewrite%20%20%7C%0A%7CNGX\_HTTP\_FIND\_CONFIG\_PHASE%20%20%7C%20%20%7C%0A%7C%20NGX\_HTTP\_REWRITE\_PHASE%20%7C%20rewrite%20%7C%0A%7C%20NGX\_HTTP\_POST\_REWRITE\_PHASE%20%7C%20%20%7C%0A%7C%20NGX\_HTTP\_PREACCESS\_PHASE%20%7Climit\_req%20\-%3E%20%20limit\_conn%20%20%7C%0A%7CNGX\_HTTP\_ACCESS\_PHASE%20%20%7C%20%20%7C%0A%7C%20NGX\_HTTP\_POST\_ACCESS\_PHASE%20%7C%20access%20auth\_basic%20auth\_request%20%20%7C%0A%7C%20NGX\_HTTP\_PRECONTENT\_PHASE%20%7C%20try\_file%20mirrors%20%7C%0A%7C%20NGX\_HTTP\_CONTENT\_PHASE%20%7C%20concat%20random\_index%20%20index%20autoindex%20static%20%20%7C%0A%7C%20NGX\_HTTP\_LOG\_PHASE%20%7C%20access%20log%20%7C%0A%20%20%0A%3E%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%9Angx\_http\_phases%0A%0A%0A%E5%85%B7%E4%BD%93%E5%88%86%E6%9E%9011%E6%AD%A5%E9%AA%A4%EF%BC%9A%0A%0A1.%20NGX\_HTTP\_POST\_READ\_PHASE%EF%BC%9A%0A%E6%8B%BF%E5%8F%96HTTP%E5%A4%B4%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%A6%82%EF%BC%9AIP%20PORT%20%E4%BB%A3%E7%90%86IP%20%EF%BC%8C%E5%B9%B6%E4%BF%9D%E5%AD%98%E5%88%B0%E5%8F%98%E9%87%8F%E4%B8%AD%EF%BC%8C%E5%A6%82%EF%BC%9Aremote\_addr%C2%A0%20%0A%3E%E8%BF%99%E9%87%8C%E8%BF%98%E6%9C%89%E4%B8%AA%20real\_ip%20%E6%A8%A1%E5%9D%97%EF%BC%8C%E5%8F%AF%E9%80%89%E9%A1%B9%0A%0A2.%20NGX\_HTTP\_SERVER\_REWRITE\_PHASE%EF%BC%9A%0A%E5%AF%BB%E6%89%BE%20server%20%E5%9D%97%E4%B8%AD%E7%9A%84%20rewrite%0A%20%20%20%201.%20404%20403%20502%20error\_page%0A%20%20%20%202.%20return%20%0A3.%20NGX\_HTTP\_FIND\_CONFIG\_PHASE%0A%E5%AF%BB%E6%89%BE%20location%EF%BC%8Cnginx%20%E4%BC%9A%E6%8A%8A%E6%AF%8F%E4%B8%AA%20server%20%E4%B8%AD%E7%9A%84%20%E6%AF%8F%E4%B8%AA%20location%20%E6%AD%A3%E5%88%99%E6%8F%90%E5%8F%96%E5%88%B0%E4%B8%80%E4%B8%AA%E4%BA%8C%E5%8F%89%E6%A0%91%E4%B8%AD%E5%81%9A%E7%B4%A2%E5%BC%95%EF%BC%8C%E7%84%B6%E5%90%8E%E8%AF%B7%E6%B1%82%E7%9A%84URI%E5%9C%A8%E6%A0%91%E4%B8%AD%E5%81%9A%E6%90%9C%E7%B4%A2%0A%0A4.%20NGX\_HTTP\_REWRITE\_PHASE%0A%E5%AF%BB%E6%89%BE%20location%20%E5%9D%97%E4%B8%AD%E7%9A%84%20rewrite%E3%80%82%E8%BF%99%E9%87%8C%E5%8F%AF%E8%83%BD%E4%BC%9A%E9%87%8D%E5%A4%8D%E6%89%A7%E8%A1%8C%EF%BC%8C%E5%9B%A0%E4%B8%BArewrite%20%E8%B7%B3%E8%BD%AC%E4%B8%80%E6%AC%A1%EF%BC%8C%E5%86%8D%E8%BF%9B%E6%9D%A5%E5%8F%AF%E8%83%BD%E7%BB%A7%E7%BB%ADrewrite%20....%0A%0A5.%20NGX\_HTTP\_POST\_REWRITE\_PHASE%0Arewrite%E5%90%8E%EF%BC%8C%E4%B8%BA%E9%98%B2%E6%AD%A2%E6%AD%BB%E5%BE%AA%E7%8E%AF%0A%0A6.%20NGX\_HTTP\_PREACCESS\_PHASE%0A%E8%AE%A4%E8%AF%81%E9%A2%84%E5%A4%84%E7%90%86%EF%BC%8C%E5%A6%82%EF%BC%9Alimit\_conn%C2%A0%20%20limit\_req%C2%A0%0A%0A7.%20NGX\_HTTP\_ACCESS\_PHASE%0A%E6%9D%83%E9%99%90%E8%AE%A4%E8%AF%81%EF%BC%9A%0A%20%20%20%201.%20IP%E9%99%90%E5%88%B6%EF%BC%8C%E5%A6%82%EF%BC%9A%20deny%20192.168.1.1%3B%20allow%20192.168.1.0%2F24%3B%0A%20%20%20%202.%20auth\_basic%C2%A0%EF%BC%9A%E7%94%A8%E6%88%B7%E8%AE%A4%E8%AF%81%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E8%87%AA%E5%B8%A6%E7%9A%84%E7%94%A8%E6%88%B7%E8%AE%A4%E8%AF%81%E6%9C%BA%E5%88%B6%E3%80%82%0A%20%20%20%203.%20auth\_request%C2%A0%EF%BC%9A%E6%9D%83%E9%99%90%E8%AE%A4%E8%AF%81%EF%BC%8C%E5%BD%93%E6%9C%89%E4%B8%80%E4%B8%AA%E8%AF%B7%E6%B1%82%E8%BF%9B%E6%9D%A5%EF%BC%8CNGINX%20%E5%B9%B6%E4%B8%8D%E7%9B%B4%E6%8E%A5%E8%AF%B7%E6%B1%82%E5%90%8E%E7%AB%AF%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E8%80%8C%E6%98%AF%E5%85%88%E5%8F%91%E4%B8%80%E4%B8%AA%20auth%20%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%90%8E%E7%AB%AF%E5%A6%82%E6%9E%9C%E8%BF%94%E5%9B%9E200%EF%BC%8C%E5%88%99%E6%89%8D%E4%BC%9A%E6%AD%A3%E5%B8%B8%E4%BB%A3%E7%90%86%E8%AF%A5%E8%AF%B7%E6%B1%82%E5%88%B0%E5%90%8E%E7%AB%AF%EF%BC%8C%E5%90%A6%E5%88%99%E7%9B%B4%E6%8E%A5%E6%8B%92%E7%BB%9D%0A%20%20%20%20%0A%0A8.%20NGX\_HTTP\_POST\_ACCESS\_PHASE%0A%E6%9D%83%E9%99%90%E8%AE%A4%E8%AF%81%E5%90%8E%E7%BD%AE%E5%A4%84%E7%90%86%0A%0A9.%20NGX\_HTTP\_PRECONTENT\_PHASE%0A%E5%B0%B1%E4%B8%80%E4%B8%AA%20try\_file%20%E6%A8%A1%E5%9D%97%0A%20%20%20%201.%20%E5%8F%AF%E4%BB%A5%E7%BB%99%20location%20%E6%89%BE%E4%B8%8D%E5%88%B0%E6%96%87%E4%BB%B6%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%81%9A%E4%B8%AA%E9%BB%98%E8%AE%A4%E6%96%87%E4%BB%B6\(%E9%87%8D%E5%AE%9A%E5%90%91\)%0A%20%20%20%202.%20%E5%8F%AF%E4%BB%A5%E5%A2%9E%E5%8A%A0%E7%9B%AE%E5%BD%95%EF%BC%8C%E7%94%A8%E4%BA%8E%E5%AF%BB%E6%89%BEURI%E5%AF%B9%E5%BA%94%E7%9A%84%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%0A%20%0A10.%20NGX\_HTTP\_CONTENT\_PHASE%0A%E5%93%8D%E5%BA%94%E5%86%85%E5%AE%B9%0A%0A11.%20NGX\_HTTP\_LOG\_PHASE%0A%E6%97%A5%E5%BF%97%0A%0A%0A%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-\-%0Anginx%20%E6%88%96%20apache%20%EF%BC%8C%E5%81%9A%E4%B8%BA%20webservice%20%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%BC%80%E6%BA%90%E8%BD%AF%E4%BB%B6%EF%BC%8C%E7%A1%AE%E5%AE%9E%E6%98%AF%E6%96%B9%E4%BE%BF%E3%80%82%E9%80%9A%E8%BF%87%E4%B8%80%E4%B8%AA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E7%94%A8%E4%BA%8E%E9%A9%B1%E5%8A%A8%E4%BD%BF%E7%94%A8%20webservice%E3%80%82%E7%A8%8B%E5%BA%8F%E5%91%98%E4%B8%8D%E9%9C%80%E8%A6%81%E5%86%8D%E5%81%9A%E9%A2%9D%E5%A4%96%E7%9A%84%E5%BC%80%E5%8F%91%EF%BC%8C%E5%B0%B1%E8%83%BD%E4%BD%BF%E7%94%A8%20http%20%E7%9B%B8%E5%85%B3%E7%9A%84%E6%9C%8D%E5%8A%A1%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8Cnginx%20%E8%BF%98%E4%BF%9D%E8%AF%81%E4%BA%86%E7%A8%B3%E5%AE%9A%E3%80%81%E6%80%A7%E8%83%BD%E3%80%81%E5%8F%AF%E6%89%A9%E5%B1%95%E7%AD%89%E7%AD%89%E3%80%82%E7%9C%9F%E7%9A%84%E6%98%AF%E7%9C%81%E4%BA%8B%E7%9C%81%E4%BA%BA%E7%9C%81%E5%8A%9B%E3%80%82%0A%0A%E4%B8%8D%E8%BF%87%EF%BC%8C%E8%BF%87%E5%BA%A6%E4%BD%BF%E7%94%A8%E5%BC%80%E6%BA%90%E5%85%8D%E8%B4%B9%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%8C%E4%B9%9F%E4%BC%9A%E6%8B%89%E4%BD%8E%E7%A8%8B%E5%BA%8F%E5%91%98%E7%9A%84%E5%9F%BA%E7%A1%80%E6%B0%B4%E5%B9%B3%E3%80%82%E5%AF%B9%E5%BA%95%E5%B1%82%E7%9A%84%E7%90%86%E8%A7%A3%E4%B8%8D%E5%A4%9F%E6%B7%B1%E5%88%BB%E3%80%81%E5%9F%B9%E8%AE%AD%E6%9C%BA%E6%9E%84%E7%9A%84%E5%A1%AB%E9%B8%AD%E5%BC%8F%E6%95%99%E8%82%B2%E9%99%AA%E7%9D%80%E4%BA%86%E4%B8%80%E5%A0%86%E6%B7%B7%E5%AD%90%E7%AD%89%E7%AD%89%E5%90%A7%E3%80%82%E4%BD%86%E5%92%8B%E8%AF%B4%E5%91%A2%EF%BC%8C%E5%BA%94%E8%AF%A5%E9%A1%BA%E5%BA%94%E6%97%B6%E4%BB%A3%E5%8F%91%E5%B1%95%E3%80%82%E5%90%91%E5%89%8D%E5%8F%91%E5%B1%95%EF%BC%8C%E6%9C%89%E4%BA%9B%E5%BA%95%E5%B1%82%E7%9A%84%E5%8E%9F%E7%90%86%E4%B9%9F%E4%B8%8D%E4%B8%80%E5%AE%9A%E5%AE%8C%E5%85%A8%E8%83%BD%E7%90%86%E8%A7%A3%E3%80%82%E7%AB%99%E5%9C%A8%E5%B7%A8%E4%BA%BA%E7%9A%84%E8%82%A9%E8%86%80%E4%B8%8A%EF%BC%81%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A
