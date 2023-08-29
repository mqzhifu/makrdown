# http

## 概要

Hyper Text Transfer Protocol：超文本传输协议

> 只是一个协议

## 一次HTTP操作

先看流程图吧：

1. 用户在本机开启浏览器应用程序，然后输入域名
2. 浏览器会先请求DSN服务器，拿到IP

> 浏览器内存DNS缓存,/etc/host配置,主机dns缓存

1. 浏览器用IP\+PORT，与服务器端建立TCP连接，双方正常建立连接

> 本地的路由器\-\>到公网\-\>到对方的路由器
> 
> 
> 3次握手

1. 浏览器根据用户输入的URI，请求S端，获取HTML代码
2. 浏览器解析HTML代码
    1. HTML里还依赖一些资源，二次请求继续获取资源
    2. 将JS代码将给V8引擎执行
    3. 将最终的HTML套上CSS样式，送给沉浸器
3. 浏览器把最终渲染好的结果，显示在浏览器窗口内
4. 浏览器断开TCP连接

分析

1. 依然是建立TCP上
2. 一次请求正常过后，就会断开连接
3. 浏览器帮忙做了DNS解析 建立长连接 JS代码执行 图像渲染

## 协议详情

协议的格式大体分成两个部分：

1. 请求
2. 响应

> 基本上格式差不多，只是具体的KV值不太一样而以

## 协议详情\-请求

请求内容的格式分为四个部分：

1. 请求行\\r\\n
2. 头域\\r\\n
3. 空行
4. 消息体（可选）

> 其核心就是：header\+body

#### 请求行

> GET /index.php?a=1 HTTP/1.1
> 
> 
> 方法\[空格\]请求URI\[空格\]版本号\\r\\n

方法类型:

1. GET
2. POST
3. HEAD
4. PUT
5. DELETE
6. OPTIONS
7. TRACE
8. CONNECT

> 其实就：GET POST用的最多，然后head跨域会用一些，其它的基本很少很少使用
> 
> 
> 像：delete/put 直接p被：get/post 的URI中包含关键字给代替掉了，如：http://www.baidu.com/user/delete/id/1

#### 请求uri:Uniform Resource Identifier

用来标识你想请求服务端的资源

#### 版本号：

1. 1.0/1.1
2. 2.0

> 用来告知影响方：请求方当前的HTTP协议版本号，具体2.0如何处理后面再讲

#### 请求头

先看看一个demo:

```
Host:www.baidu.com \r\n
User-Agent:Mozilla/5.0 (Windows; U; Windows NT 6.1; zh-CN; rv:1.9.2.13) Gecko/20101203 Firefox/3.6.13
Cache-Controlmax-age=0          使用何种缓存机制
Data:Mon,30 Nov 2009 13:15:39发送请求的时间
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8   浏览器可接受的MIME类型；
Accept-Charset:GB2312,utf-8;    浏览器可接受的字符集
Accept-Encoding：gzip,deflate  浏览器能够进行解码的数据编码方式
Accept-Language：zh-cn,zh  浏览器所希望的语言种类
Authorization：授权信息
Connection：keep-alive 表示是否需要持久连接
Keep-Alive:115
Content-Length：表示请求消息正文的长度；
Cookie：这是最重要的请求头信息之一；
```

就是个KV值......主要就是告诉接收端：请求方的一些配置信息而以..... ，接收方，接到这些配置信息，根据请求方要求自行来定制化处理

> 从这里也能看出来，它是比较人性化，好扩展，但另外一个问题：如果直接对标TCP，如果它会将传输都带这么一堆的KV字符串，性能肯定不高

格式要求：首字母大写，单词间用中划线分隔

实际上这里面还有很多key，用处不一，只列些实际经常用的，另外：http协议它比较自由，只用一个空格分隔，至于你的KV也不一定全是写死，你也可以自定义一些，如：

```
X-Token:abcdabcdabcdabcdabcdabcdabcdabcdabcdabcd
X-Source:baidu
X-Access:IMtest
```

> 它的官方建议是：自定义的HEADE\-key，最好以大写X开头

几个常用的头key做个讲解吧：

|key             |desc                                |purpose                                                                                  |
|----------------|------------------------------------|-----------------------------------------------------------------------------------------|
|Host            |标识请求的域名                      |后端可根据域名做代理与分发                                                               |
|User\-Agent     |标识请求方的来源信息                |如：OS版本 设备类型 浏览器版本 等，这个经常用，尤其写爬虫的时候必须得用。反爬虫也会用到。|
|Accept          |告知接收方，请求方可以接收的数据类型|                                                                                         |
|Accept\-Charset |可接收\(优先\)的字符集字符集        |如：gbk gb2312 utf8，有时候乱码就是由此字段引起                                          |
|Accept\-Encoding|传输内容的编码格式                  |gzip 字符流，可用于压缩传输内容，加快传输性能                                            |
|Cache\-Control  |请求缓存                            |像js css这些纯静态文件，基本不会改变，也没必要每次请求都向S端拿                          |
|Connection      |是否短时间内开启长连接              |可避免每次连接重复创建TCP消耗性能                                                        |
|Cookie          |客户端存储的数据                    |HTTP是短连接，不存数据，没有状态，利用cookie就有状态数据了                               |
|Upgrade         |将连接升级成websocket               |长连接                                                                                   |
|Authorization   |当前请求方的用户名密码              |有些网站不需要自行实现登陆，仅通过浏览器做些简单的验证                                   |
|Referer         |上一个页面的来源                    |可做些反爬虫验证、页面跳转                                                               |
|Via             |代理                                |nginx代理再发往后端，后端想知道nginx的一些信息                                           |
|Range           |请求一部分                          |大文件，分多次请求下载                                                                   |

从这些头中分析，HTTP可做的事情还是很多的：

1. 性能提升：缓存、部分请求、ws长连接、短时间内长连接
2. 记录状态：cookie

正常编程就是基于这个协议里面有什么，才能做什么

#### 消息体

> abc=22&abcd=33 ，没什么可说的....

依然是个kv值形态

## 协议详情\-响应

1. HTTP状态行\\r\\n
2. 响应头\\r\\n
3. 3空行\\r\\n
4. 4可选的消息体

> 跟请求的格式一样一样的......

#### 响应\-行

> HTTP/1.1 200 OK
> 
> 
> 版本号\[空格\]状态码\[空格\]原因说明\\r\\n

HTTP/1.0 每次请求都需要建立新的TCP连接，连接不能复用。HTTP/1.1 新的请求可以在上次请求建立的TCP连接之上发送，连接可以复用。优点是减少重复进行TCP三次握手的开销，提高效率。

注意：在同一个TCP连接中，新的请求需要等上次请求收到响应后，才能发送。

#### 常用响应状态码：

|code|en desc              |cn desc                                               |purpose                                                                                       |
|----|---------------------|------------------------------------------------------|----------------------------------------------------------------------------------------------|
|200 |OK                   |客户端请求成功                                        |程序员最喜欢的状态码                                                                          |
|301 |Moved Permanently    |永久移动                                              |爬虫的时候遇到过，还有就是：某些网站可能变了域名也可以使用                                    |
|302 |Found                |临时移动                                              |同上，不过如果双方的类库支持自动跳转到新页面还好，如果不支持就很烦                            |
|304 |Not Modified         |未修改                                                |请求方可根据此值使用本地缓存文件，节约网络性能开销                                            |
|400 |Bad Request          |客户端请求有语法错误                                  |C端有公共库，基本上用不到，但如果自己实现HTTP协议会遇到类似问题                               |
|401 |Unauthorized         |请求未经授权                                          |与Authenticate头配合使用，实现简单登陆验证                                                    |
|403 |Forbidden            |服务器收到请求，但是拒绝提供服务                      |apache遇到的最多，大我是请求的资源没有权限                                                    |
|404 |Not Found            |请求资源不存在                                        |eg：输入了错误的URL                                                                           |
|500 |Internal Server Error|服务器发生不可预期的错误                              |像PHP JAVA 脚本执行出现错误，或者运行时判断参数不正确等等                                     |
|502 |Bad Gateway          |网关错                                                |大型公司所有的应用程序前面是要有网关，但中小公司基本上没遇到过，倒是前些年淘宝双11的时候遇到过|
|503 |Server Unavailable   |服务器当前不能处理客户端的请求，一段时间后可能恢复正常|未遇到过                                                                                      |

简单分析下：

|code |desc                  |                                                                     |
|-----|----------------------|---------------------------------------------------------------------|
|1开头|是C与S连接时的一些状态|因为开源的类库双方都有，且稳定，大多都记不住此状态值了......         |
|2开头|基本上都是成功        |忽略，大家都喜欢此状态                                               |
|3开头|请求的资源被移动/改变 |很少遇到，即使有，C端 类库也会自动跳转                               |
|4开头|大多是请求方的错误    |比较少遇到，因为C端的类库都比较稳定，真遇到了，后端就可以直接喷前端了|
|5开头|大多是服务端错        |请求方遇到类似的，直接开喷后端程序员吧                               |

#### 响应\-头

```
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html
Server: Apache/2.2.17 (Win32) PHP/5.2.9-2
Last-Modified: Sat, 22 Jan 2011 03:29:17 GMT
Content-Range: bytes 0-5
```

> 跟请求格式基本一样，就是KV

看一下常用的：

|key                           |desc                |purpose                                        |
|------------------------------|--------------------|-----------------------------------------------|
|Content\-Encoding             |传输的格式          |gzip 压缩                                      |
|Transfer\-Encoding            |报文格式\-分块编码  |算是对ws协议的优化吧                           |
|Last\-Modified                |该资源最后修改时间  |配合请求端来使用缓存机制，节约网络性能         |
|Connection                    |长连接              |请求方如果使用Upgrade做ws长连接                |
|Content\-Range                |分片传输时          |请求方想分段下载大文件时使用                   |
|Content\-Type                 |当前返回的内容类型  |当前S端返回的具体的内容，如：html json image js|
|Content\-Length               |当前返回的内容总长度|只是方便接收端处理，很少用                     |
|Location                      |重新获取资源位置    |如果有301 302 这种，可以用此参数跳转，很少用   |
|Access\-Control\-Allow\-Origin|允许跨域访问的域名  |跨域                                           |

常用 content\-type：

|key                                 |desc                                                                     |purpose                                             ||
|------------------------------------|-------------------------------------------------------------------------|----------------------------------------------------||
|text/html                           |HTML格式                                                                 |最常用的，也就是给浏览器解析/执行/渲染              ||
|text/xml                            |XML格式                                                                  |主要是一些配置文件使用，相比html更严谨，但也更复杂些||
|image/gif                           |gif图片格式                                                              |                                                    ||
|image/jpeg                          |jpg图片格式                                                              |                                                    ||
|image/png                           |png图片格式                                                              |                                                    ||
|application/javascript              |js代码                                                                   |                                                    ||
|application/json                    |JSON数据格式，这个最常用，前端从后端拿源数据就是这个                     |                                                    ||
|application/pdf                     |不太常用                                                                 |                                                    ||
|application/msword                  |不太常用                                                                 |                                                    ||
|application/octet\-stream           |二进制流数据（如常见的文件下载）                                         |                                                    ||
|application/x\-www\-form\-urlencoded|form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）|                                                    ||
|multipart/form\-data                |表单中进行文件上传时，就需要使用该格式                                   |                                                    ||

> 咋形容呢，其实content\-type只是一个标识，或者说有很大一部分是给浏览器使用的，比如：解析PDF EXCEL MP4 MP3 格式，浏览器可以直接使用\(浏览器插件\)，而我们日常使用最多的是json，像text/html这种 WEBSERVCIE就帮你搞好了。

总之：从类型中也能看出，浏览器能处理的数据类型还是挺多的，再配合是自带的插件，像：播放视频、音频，显示WORD EXCEL文档等等，那浏览器火爆也是必然的啊

#### 响应\-体

这个就是不同的content\-type 给出不同的内容了，也可以自行输出内容体，无所谓了，没啥重点可讲。

## http版本对比

1. http1.0
2. http1.1
3. http2.0
4. Chunked transfer\-coding

## http 1.0

这个对比的最基础版，不讲解了

## http 1.1

新的功能点：

1. head of line blocking 链接复用
2. 更多的缓存控制
3. 响应错误码增加
4. 传输性能优化

链接复用:1.0时，每个请求即一个新的TCP链接，优化成：只需要一次TCP链接，重复利用该链接，继续发请求.

具体到详细的协议中：connection: keep\-alive | connection: close

虽然可以一次发送N个请求，基本上不阻塞，但是响应方，得按请求方的请求时间，依次响应，严格看，还是会有阻塞问题。但请求方可以并行的发请求。

缓存控制好像也没啥讲的，就是比之前的缓存控制多一些头参数

```
If-Modefied-Since
If-Unmodified-Since
If-Match
If-None-Match
ET-tag
```

header中增加了host字段

性能优化：加入了状态码100，可以先试探S端是否正常，如果正常，再发BODY，勉强算是优化点传输性能吧，感觉挺鸡肋。还有一个，块传输：将一个大的body切成若干小块\(Chunked transfer\-coding\)，分批次传输，这个可以用到断点续传上。

总结：基本上就是一个链接复用算是常用些，其它的都挺鸡肋。

## http 2.0

1. 网络传输性能优化
2. 多路复用
3. 请求优先集
4. header 压缩
5. server push

1 2 3 条总结下来的核心是：二进制分帧

二进制分帧：在TCP 和 HTTP 层，中间加一层：二进制分帧

该层：请求方任何发，响应方任意响应，该层会把 两边的数据分成二进制的帧，做控制，再返回给两端。

多路复用：http 1.1 加入了连接复用，但是必须得按照请求顺序来响应，可能出现阻塞。如果在这中间再加一些控制代码，实现真的：单链接、无阻塞、多请求。

server push:S端在接收到C端一个主请求后，会自动把后续的资源请求的东西，也一并推送给C端，这样C端不用再单独发请求，直接从缓存中拿就行了。

缺点：单连接会因TCP线头阻塞（head\-of\-line blocking）的特性而传输速度受限。加上存在可能丢包的情况，其负面影响已超过压缩头部和优先级控制带来的好处。

总结：就一个多路复用确实能解决之前的阻塞问题~其它还是鸡肋。

## 常用实体

|显示结果|描述    |名称          |编号|
|--------|--------|--------------|----|
|\<      |小于号  |\<            |\<  |
|\>      |大于号  |\>            |\>  |
|&       |和号    |&             |&   |
|"       |引号    |"             |"   |
|'       |撇号    |' \(IE不支持\)|'   |
|©       |版权    |©             |©   |
|®      |注册商标|®            |®  |
|™      |商标    |™            |™  |
|¥       |元      |¥             |¥   |
|¥       |空格    |              |    |

%E6%A6%82%E8%A6%81%0A\-\-\-%0AHyper%20Text%20Transfer%20Protocol%EF%BC%9A%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE%0A%3E%E5%8F%AA%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8D%8F%E8%AE%AE%0A%0A%E4%B8%80%E6%AC%A1HTTP%E6%93%8D%E4%BD%9C%0A\-\-\-\-%0A%E5%85%88%E7%9C%8B%E6%B5%81%E7%A8%8B%E5%9B%BE%E5%90%A7%EF%BC%9A%0A%0A%0A%0A%0A1.%20%E7%94%A8%E6%88%B7%E5%9C%A8%E6%9C%AC%E6%9C%BA%E5%BC%80%E5%90%AF%E6%B5%8F%E8%A7%88%E5%99%A8%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%EF%BC%8C%E7%84%B6%E5%90%8E%E8%BE%93%E5%85%A5%E5%9F%9F%E5%90%8D%0A2.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%BC%9A%E5%85%88%E8%AF%B7%E6%B1%82DSN%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E6%8B%BF%E5%88%B0IP%0A%3E%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E5%AD%98DNS%E7%BC%93%E5%AD%98%2C%2Fetc%2Fhost%E9%85%8D%E7%BD%AE%2C%E4%B8%BB%E6%9C%BAdns%E7%BC%93%E5%AD%98%0A3.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E7%94%A8IP%2BPORT%EF%BC%8C%E4%B8%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E5%BB%BA%E7%AB%8BTCP%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%8F%8C%E6%96%B9%E6%AD%A3%E5%B8%B8%E5%BB%BA%E7%AB%8B%E8%BF%9E%E6%8E%A5%0A%3E%E6%9C%AC%E5%9C%B0%E7%9A%84%E8%B7%AF%E7%94%B1%E5%99%A8\-%3E%E5%88%B0%E5%85%AC%E7%BD%91\-%3E%E5%88%B0%E5%AF%B9%E6%96%B9%E7%9A%84%E8%B7%AF%E7%94%B1%E5%99%A8%0A%3E3%E6%AC%A1%E6%8F%A1%E6%89%8B%0A3.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E6%A0%B9%E6%8D%AE%E7%94%A8%E6%88%B7%E8%BE%93%E5%85%A5%E7%9A%84URI%EF%BC%8C%E8%AF%B7%E6%B1%82S%E7%AB%AF%EF%BC%8C%E8%8E%B7%E5%8F%96HTML%E4%BB%A3%E7%A0%81%0A4.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E8%A7%A3%E6%9E%90HTML%E4%BB%A3%E7%A0%81%0A%20%20%20%201.%20HTML%E9%87%8C%E8%BF%98%E4%BE%9D%E8%B5%96%E4%B8%80%E4%BA%9B%E8%B5%84%E6%BA%90%EF%BC%8C%E4%BA%8C%E6%AC%A1%E8%AF%B7%E6%B1%82%E7%BB%A7%E7%BB%AD%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90%0A%20%20%20%202.%20%E5%B0%86JS%E4%BB%A3%E7%A0%81%E5%B0%86%E7%BB%99V8%E5%BC%95%E6%93%8E%E6%89%A7%E8%A1%8C%0A%20%20%20%203.%20%E5%B0%86%E6%9C%80%E7%BB%88%E7%9A%84HTML%E5%A5%97%E4%B8%8ACSS%E6%A0%B7%E5%BC%8F%EF%BC%8C%E9%80%81%E7%BB%99%E6%B2%89%E6%B5%B8%E5%99%A8%0A5.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8A%8A%E6%9C%80%E7%BB%88%E6%B8%B2%E6%9F%93%E5%A5%BD%E7%9A%84%E7%BB%93%E6%9E%9C%EF%BC%8C%E6%98%BE%E7%A4%BA%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E7%AA%97%E5%8F%A3%E5%86%85%0A6.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E6%96%AD%E5%BC%80TCP%E8%BF%9E%E6%8E%A5%0A%0A%E5%88%86%E6%9E%90%0A1.%20%E4%BE%9D%E7%84%B6%E6%98%AF%E5%BB%BA%E7%AB%8BTCP%E4%B8%8A%0A2.%20%E4%B8%80%E6%AC%A1%E8%AF%B7%E6%B1%82%E6%AD%A3%E5%B8%B8%E8%BF%87%E5%90%8E%EF%BC%8C%E5%B0%B1%E4%BC%9A%E6%96%AD%E5%BC%80%E8%BF%9E%E6%8E%A5%0A3.%20%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B8%AE%E5%BF%99%E5%81%9A%E4%BA%86DNS%E8%A7%A3%E6%9E%90%20%E5%BB%BA%E7%AB%8B%E9%95%BF%E8%BF%9E%E6%8E%A5%20JS%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%20%E5%9B%BE%E5%83%8F%E6%B8%B2%E6%9F%93%0A%0A%E5%8D%8F%E8%AE%AE%E8%AF%A6%E6%83%85%0A\-\-\-\-%0A%E5%8D%8F%E8%AE%AE%E7%9A%84%E6%A0%BC%E5%BC%8F%E5%A4%A7%E4%BD%93%E5%88%86%E6%88%90%E4%B8%A4%E4%B8%AA%E9%83%A8%E5%88%86%EF%BC%9A%0A1.%20%E8%AF%B7%E6%B1%82%0A2.%20%E5%93%8D%E5%BA%94%0A%3E%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%A0%BC%E5%BC%8F%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E5%8F%AA%E6%98%AF%E5%85%B7%E4%BD%93%E7%9A%84KV%E5%80%BC%E4%B8%8D%E5%A4%AA%E4%B8%80%E6%A0%B7%E8%80%8C%E4%BB%A5%0A%0A%E5%8D%8F%E8%AE%AE%E8%AF%A6%E6%83%85\-%E8%AF%B7%E6%B1%82%0A\-\-\-%0A%0A%E8%AF%B7%E6%B1%82%E5%86%85%E5%AE%B9%E7%9A%84%E6%A0%BC%E5%BC%8F%E5%88%86%E4%B8%BA%E5%9B%9B%E4%B8%AA%E9%83%A8%E5%88%86%EF%BC%9A%0A1.%20%E8%AF%B7%E6%B1%82%E8%A1%8C%5Cr%5Cn%0A2.%20%E5%A4%B4%E5%9F%9F%5Cr%5Cn%0A3.%20%E7%A9%BA%E8%A1%8C%0A4.%20%E6%B6%88%E6%81%AF%E4%BD%93%EF%BC%88%E5%8F%AF%E9%80%89%EF%BC%89%0A%0A%3E%E5%85%B6%E6%A0%B8%E5%BF%83%E5%B0%B1%E6%98%AF%EF%BC%9Aheader%2Bbody%0A%0A%23%23%23%23%20%E8%AF%B7%E6%B1%82%E8%A1%8C%0A%3EGET%20%2Findex.php%3Fa%3D1%20HTTP%2F1.1%0A%3E%E6%96%B9%E6%B3%95%5B%E7%A9%BA%E6%A0%BC%5D%E8%AF%B7%E6%B1%82URI%5B%E7%A9%BA%E6%A0%BC%5D%E7%89%88%E6%9C%AC%E5%8F%B7%5Cr%5Cn%0A%0A%E6%96%B9%E6%B3%95%E7%B1%BB%E5%9E%8B%3A%0A1.%20GET%20%0A2.%20POST%20%0A3.%20HEAD%20%0A3.%20PUT%20%0A4.%20DELETE%0A4.%20OPTIONS%0A5.%20TRACE%0A5.%20CONNECT%0A%0A%3E%E5%85%B6%E5%AE%9E%E5%B0%B1%EF%BC%9AGET%20POST%E7%94%A8%E7%9A%84%E6%9C%80%E5%A4%9A%EF%BC%8C%E7%84%B6%E5%90%8Ehead%E8%B7%A8%E5%9F%9F%E4%BC%9A%E7%94%A8%E4%B8%80%E4%BA%9B%EF%BC%8C%E5%85%B6%E5%AE%83%E7%9A%84%E5%9F%BA%E6%9C%AC%E5%BE%88%E5%B0%91%E5%BE%88%E5%B0%91%E4%BD%BF%E7%94%A8%0A%3E%E5%83%8F%EF%BC%9Adelete%2Fput%20%E7%9B%B4%E6%8E%A5p%E8%A2%AB%EF%BC%9Aget%2Fpost%20%E7%9A%84URI%E4%B8%AD%E5%8C%85%E5%90%AB%E5%85%B3%E9%94%AE%E5%AD%97%E7%BB%99%E4%BB%A3%E6%9B%BF%E6%8E%89%E4%BA%86%EF%BC%8C%E5%A6%82%EF%BC%9Ahttp%3A%2F%2Fwww.baidu.com%2Fuser%2Fdelete%2Fid%2F1%0A%0A%0A%23%23%23%23%20%E8%AF%B7%E6%B1%82uri%3AUniform%20Resource%20Identifier%20%0A%E7%94%A8%E6%9D%A5%E6%A0%87%E8%AF%86%E4%BD%A0%E6%83%B3%E8%AF%B7%E6%B1%82%E6%9C%8D%E5%8A%A1%E7%AB%AF%E7%9A%84%E8%B5%84%E6%BA%90%0A%0A%23%23%23%23%20%E7%89%88%E6%9C%AC%E5%8F%B7%EF%BC%9A%0A1.%201.0%2F1.1%0A2.%202.0%0A%3E%E7%94%A8%E6%9D%A5%E5%91%8A%E7%9F%A5%E5%BD%B1%E5%93%8D%E6%96%B9%EF%BC%9A%E8%AF%B7%E6%B1%82%E6%96%B9%E5%BD%93%E5%89%8D%E7%9A%84HTTP%E5%8D%8F%E8%AE%AE%E7%89%88%E6%9C%AC%E5%8F%B7%EF%BC%8C%E5%85%B7%E4%BD%932.0%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E5%90%8E%E9%9D%A2%E5%86%8D%E8%AE%B2%0A%0A%0A%23%23%23%23%20%E8%AF%B7%E6%B1%82%E5%A4%B4%0A%E5%85%88%E7%9C%8B%E7%9C%8B%E4%B8%80%E4%B8%AAdemo%3A%0A%60%60%60%0AHost%3Awww.baidu.com%20%5Cr%5Cn%0AUser\-Agent%3AMozilla%2F5.0%20\(Windows%3B%20U%3B%20Windows%20NT%206.1%3B%20zh\-CN%3B%20rv%3A1.9.2.13\)%20Gecko%2F20101203%20Firefox%2F3.6.13%0ACache\-Controlmax\-age%3D0%20%20%20%20%20%20%20%20%20%20%E4%BD%BF%E7%94%A8%E4%BD%95%E7%A7%8D%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%0AData%3AMon%2C30%20Nov%202009%2013%3A15%3A39%E5%8F%91%E9%80%81%E8%AF%B7%E6%B1%82%E7%9A%84%E6%97%B6%E9%97%B4%0AAccept%EF%BC%9Atext%2Fhtml%2Capplication%2Fxhtml%2Bxml%2Capplication%2Fxml%3Bq%3D0.9%2C\*%2F\*%3Bq%3D0.8%20%20%20%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E6%8E%A5%E5%8F%97%E7%9A%84MIME%E7%B1%BB%E5%9E%8B%EF%BC%9B%0AAccept\-Charset%3AGB2312%2Cutf\-8%3B%20%20%20%20%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E6%8E%A5%E5%8F%97%E7%9A%84%E5%AD%97%E7%AC%A6%E9%9B%86%0AAccept\-Encoding%EF%BC%9Agzip%2Cdeflate%20%20%E6%B5%8F%E8%A7%88%E5%99%A8%E8%83%BD%E5%A4%9F%E8%BF%9B%E8%A1%8C%E8%A7%A3%E7%A0%81%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BC%96%E7%A0%81%E6%96%B9%E5%BC%8F%0AAccept\-Language%EF%BC%9Azh\-cn%2Czh%20%20%E6%B5%8F%E8%A7%88%E5%99%A8%E6%89%80%E5%B8%8C%E6%9C%9B%E7%9A%84%E8%AF%AD%E8%A8%80%E7%A7%8D%E7%B1%BB%0AAuthorization%EF%BC%9A%E6%8E%88%E6%9D%83%E4%BF%A1%E6%81%AF%0AConnection%EF%BC%9Akeep\-alive%20%E8%A1%A8%E7%A4%BA%E6%98%AF%E5%90%A6%E9%9C%80%E8%A6%81%E6%8C%81%E4%B9%85%E8%BF%9E%E6%8E%A5%0AKeep\-Alive%3A115%0AContent\-Length%EF%BC%9A%E8%A1%A8%E7%A4%BA%E8%AF%B7%E6%B1%82%E6%B6%88%E6%81%AF%E6%AD%A3%E6%96%87%E7%9A%84%E9%95%BF%E5%BA%A6%EF%BC%9B%0ACookie%EF%BC%9A%E8%BF%99%E6%98%AF%E6%9C%80%E9%87%8D%E8%A6%81%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%B4%E4%BF%A1%E6%81%AF%E4%B9%8B%E4%B8%80%EF%BC%9B%0A%60%60%60%0A%0A%E5%B0%B1%E6%98%AF%E4%B8%AAKV%E5%80%BC......%E4%B8%BB%E8%A6%81%E5%B0%B1%E6%98%AF%E5%91%8A%E8%AF%89%E6%8E%A5%E6%94%B6%E7%AB%AF%EF%BC%9A%E8%AF%B7%E6%B1%82%E6%96%B9%E7%9A%84%E4%B8%80%E4%BA%9B%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%E8%80%8C%E4%BB%A5.....%20%EF%BC%8C%E6%8E%A5%E6%94%B6%E6%96%B9%EF%BC%8C%E6%8E%A5%E5%88%B0%E8%BF%99%E4%BA%9B%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%A0%B9%E6%8D%AE%E8%AF%B7%E6%B1%82%E6%96%B9%E8%A6%81%E6%B1%82%E8%87%AA%E8%A1%8C%E6%9D%A5%E5%AE%9A%E5%88%B6%E5%8C%96%E5%A4%84%E7%90%86%0A%3E%E4%BB%8E%E8%BF%99%E9%87%8C%E4%B9%9F%E8%83%BD%E7%9C%8B%E5%87%BA%E6%9D%A5%EF%BC%8C%E5%AE%83%E6%98%AF%E6%AF%94%E8%BE%83%E4%BA%BA%E6%80%A7%E5%8C%96%EF%BC%8C%E5%A5%BD%E6%89%A9%E5%B1%95%EF%BC%8C%E4%BD%86%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%9A%E5%A6%82%E6%9E%9C%E7%9B%B4%E6%8E%A5%E5%AF%B9%E6%A0%87TCP%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%AE%83%E4%BC%9A%E5%B0%86%E4%BC%A0%E8%BE%93%E9%83%BD%E5%B8%A6%E8%BF%99%E4%B9%88%E4%B8%80%E5%A0%86%E7%9A%84KV%E5%AD%97%E7%AC%A6%E4%B8%B2%EF%BC%8C%E6%80%A7%E8%83%BD%E8%82%AF%E5%AE%9A%E4%B8%8D%E9%AB%98%0A%0A%E6%A0%BC%E5%BC%8F%E8%A6%81%E6%B1%82%EF%BC%9A%E9%A6%96%E5%AD%97%E6%AF%8D%E5%A4%A7%E5%86%99%EF%BC%8C%E5%8D%95%E8%AF%8D%E9%97%B4%E7%94%A8%E4%B8%AD%E5%88%92%E7%BA%BF%E5%88%86%E9%9A%94%0A%0A%E5%AE%9E%E9%99%85%E4%B8%8A%E8%BF%99%E9%87%8C%E9%9D%A2%E8%BF%98%E6%9C%89%E5%BE%88%E5%A4%9Akey%EF%BC%8C%E7%94%A8%E5%A4%84%E4%B8%8D%E4%B8%80%EF%BC%8C%E5%8F%AA%E5%88%97%E4%BA%9B%E5%AE%9E%E9%99%85%E7%BB%8F%E5%B8%B8%E7%94%A8%E7%9A%84%EF%BC%8C%E5%8F%A6%E5%A4%96%EF%BC%9Ahttp%E5%8D%8F%E8%AE%AE%E5%AE%83%E6%AF%94%E8%BE%83%E8%87%AA%E7%94%B1%EF%BC%8C%E5%8F%AA%E7%94%A8%E4%B8%80%E4%B8%AA%E7%A9%BA%E6%A0%BC%E5%88%86%E9%9A%94%EF%BC%8C%E8%87%B3%E4%BA%8E%E4%BD%A0%E7%9A%84KV%E4%B9%9F%E4%B8%8D%E4%B8%80%E5%AE%9A%E5%85%A8%E6%98%AF%E5%86%99%E6%AD%BB%EF%BC%8C%E4%BD%A0%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%80%E4%BA%9B%EF%BC%8C%E5%A6%82%EF%BC%9A%0A%60%60%60%0AX\-Token%3Aabcdabcdabcdabcdabcdabcdabcdabcdabcdabcd%0AX\-Source%3Abaidu%0AX\-Access%3AIMtest%0A%60%60%60%0A%3E%E5%AE%83%E7%9A%84%E5%AE%98%E6%96%B9%E5%BB%BA%E8%AE%AE%E6%98%AF%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84HEADE\-key%EF%BC%8C%E6%9C%80%E5%A5%BD%E4%BB%A5%E5%A4%A7%E5%86%99X%E5%BC%80%E5%A4%B4%0A%0A%E5%87%A0%E4%B8%AA%E5%B8%B8%E7%94%A8%E7%9A%84%E5%A4%B4key%E5%81%9A%E4%B8%AA%E8%AE%B2%E8%A7%A3%E5%90%A7%EF%BC%9A%0A%0A%7C%20key%20%7C%20desc%20%7Cpurpose%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C\-\-\-%20%7C%0A%7C%20Host%20%7C%20%20%E6%A0%87%E8%AF%86%E8%AF%B7%E6%B1%82%E7%9A%84%E5%9F%9F%E5%90%8D%20%7C%E5%90%8E%E7%AB%AF%E5%8F%AF%E6%A0%B9%E6%8D%AE%E5%9F%9F%E5%90%8D%E5%81%9A%E4%BB%A3%E7%90%86%E4%B8%8E%E5%88%86%E5%8F%91%7C%0A%7C%20User\-Agent%20%7C%20%E6%A0%87%E8%AF%86%E8%AF%B7%E6%B1%82%E6%96%B9%E7%9A%84%E6%9D%A5%E6%BA%90%E4%BF%A1%E6%81%AF%20%7C%E5%A6%82%EF%BC%9AOS%E7%89%88%E6%9C%AC%20%E8%AE%BE%E5%A4%87%E7%B1%BB%E5%9E%8B%20%E6%B5%8F%E8%A7%88%E5%99%A8%E7%89%88%E6%9C%AC%20%E7%AD%89%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%BB%8F%E5%B8%B8%E7%94%A8%EF%BC%8C%E5%B0%A4%E5%85%B6%E5%86%99%E7%88%AC%E8%99%AB%E7%9A%84%E6%97%B6%E5%80%99%E5%BF%85%E9%A1%BB%E5%BE%97%E7%94%A8%E3%80%82%E5%8F%8D%E7%88%AC%E8%99%AB%E4%B9%9F%E4%BC%9A%E7%94%A8%E5%88%B0%E3%80%82%7C%0A%7C%20Accept%20%7C%E5%91%8A%E7%9F%A5%E6%8E%A5%E6%94%B6%E6%96%B9%EF%BC%8C%E8%AF%B7%E6%B1%82%E6%96%B9%E5%8F%AF%E4%BB%A5%E6%8E%A5%E6%94%B6%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%20%20%7C%7C%0A%7C%20Accept\-Charset%20%7C%E5%8F%AF%E6%8E%A5%E6%94%B6\(%E4%BC%98%E5%85%88\)%E7%9A%84%E5%AD%97%E7%AC%A6%E9%9B%86%E5%AD%97%E7%AC%A6%E9%9B%86%20%20%7C%E5%A6%82%EF%BC%9Agbk%20gb2312%20utf8%EF%BC%8C%E6%9C%89%E6%97%B6%E5%80%99%E4%B9%B1%E7%A0%81%E5%B0%B1%E6%98%AF%E7%94%B1%E6%AD%A4%E5%AD%97%E6%AE%B5%E5%BC%95%E8%B5%B7%7C%0A%7CAccept\-Encoding%20%7C%E4%BC%A0%E8%BE%93%E5%86%85%E5%AE%B9%E7%9A%84%E7%BC%96%E7%A0%81%E6%A0%BC%E5%BC%8F%7Cgzip%20%E5%AD%97%E7%AC%A6%E6%B5%81%EF%BC%8C%E5%8F%AF%E7%94%A8%E4%BA%8E%E5%8E%8B%E7%BC%A9%E4%BC%A0%E8%BE%93%E5%86%85%E5%AE%B9%EF%BC%8C%E5%8A%A0%E5%BF%AB%E4%BC%A0%E8%BE%93%E6%80%A7%E8%83%BD%7C%0A%7CCache\-Control%7C%E8%AF%B7%E6%B1%82%E7%BC%93%E5%AD%98%7C%E5%83%8Fjs%20css%E8%BF%99%E4%BA%9B%E7%BA%AF%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8D%E4%BC%9A%E6%94%B9%E5%8F%98%EF%BC%8C%E4%B9%9F%E6%B2%A1%E5%BF%85%E8%A6%81%E6%AF%8F%E6%AC%A1%E8%AF%B7%E6%B1%82%E9%83%BD%E5%90%91S%E7%AB%AF%E6%8B%BF%7C%0A%7C%20Connection%20%7C%20%E6%98%AF%E5%90%A6%E7%9F%AD%E6%97%B6%E9%97%B4%E5%86%85%E5%BC%80%E5%90%AF%E9%95%BF%E8%BF%9E%E6%8E%A5%7C%E5%8F%AF%E9%81%BF%E5%85%8D%E6%AF%8F%E6%AC%A1%E8%BF%9E%E6%8E%A5%E9%87%8D%E5%A4%8D%E5%88%9B%E5%BB%BATCP%E6%B6%88%E8%80%97%E6%80%A7%E8%83%BD%7C%0A%7C%20Cookie%20%7C%20%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AD%98%E5%82%A8%E7%9A%84%E6%95%B0%E6%8D%AE%20%7CHTTP%E6%98%AF%E7%9F%AD%E8%BF%9E%E6%8E%A5%EF%BC%8C%E4%B8%8D%E5%AD%98%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%B2%A1%E6%9C%89%E7%8A%B6%E6%80%81%EF%BC%8C%E5%88%A9%E7%94%A8cookie%E5%B0%B1%E6%9C%89%E7%8A%B6%E6%80%81%E6%95%B0%E6%8D%AE%E4%BA%86%7C%0A%7C%20Upgrade%20%7C%20%E5%B0%86%E8%BF%9E%E6%8E%A5%E5%8D%87%E7%BA%A7%E6%88%90websocket%20%7C%E9%95%BF%E8%BF%9E%E6%8E%A5%7C%0A%7C%20Authorization%20%7C%E5%BD%93%E5%89%8D%E8%AF%B7%E6%B1%82%E6%96%B9%E7%9A%84%E7%94%A8%E6%88%B7%E5%90%8D%E5%AF%86%E7%A0%81%20%20%7C%E6%9C%89%E4%BA%9B%E7%BD%91%E7%AB%99%E4%B8%8D%E9%9C%80%E8%A6%81%E8%87%AA%E8%A1%8C%E5%AE%9E%E7%8E%B0%E7%99%BB%E9%99%86%EF%BC%8C%E4%BB%85%E9%80%9A%E8%BF%87%E6%B5%8F%E8%A7%88%E5%99%A8%E5%81%9A%E4%BA%9B%E7%AE%80%E5%8D%95%E7%9A%84%E9%AA%8C%E8%AF%81%7C%0A%7C%20Referer%20%7C%20%E4%B8%8A%E4%B8%80%E4%B8%AA%E9%A1%B5%E9%9D%A2%E7%9A%84%E6%9D%A5%E6%BA%90%20%7C%E5%8F%AF%E5%81%9A%E4%BA%9B%E5%8F%8D%E7%88%AC%E8%99%AB%E9%AA%8C%E8%AF%81%E3%80%81%E9%A1%B5%E9%9D%A2%E8%B7%B3%E8%BD%AC%7C%0A%7C%20Via%20%7C%20%E4%BB%A3%E7%90%86%20%7Cnginx%E4%BB%A3%E7%90%86%E5%86%8D%E5%8F%91%E5%BE%80%E5%90%8E%E7%AB%AF%EF%BC%8C%E5%90%8E%E7%AB%AF%E6%83%B3%E7%9F%A5%E9%81%93nginx%E7%9A%84%E4%B8%80%E4%BA%9B%E4%BF%A1%E6%81%AF%7C%0A%7C%20Range%20%7C%20%E8%AF%B7%E6%B1%82%E4%B8%80%E9%83%A8%E5%88%86%20%20%7C%E5%A4%A7%E6%96%87%E4%BB%B6%EF%BC%8C%E5%88%86%E5%A4%9A%E6%AC%A1%E8%AF%B7%E6%B1%82%E4%B8%8B%E8%BD%BD%7C%0A%0A%E4%BB%8E%E8%BF%99%E4%BA%9B%E5%A4%B4%E4%B8%AD%E5%88%86%E6%9E%90%EF%BC%8CHTTP%E5%8F%AF%E5%81%9A%E7%9A%84%E4%BA%8B%E6%83%85%E8%BF%98%E6%98%AF%E5%BE%88%E5%A4%9A%E7%9A%84%EF%BC%9A%0A1.%20%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%EF%BC%9A%E7%BC%93%E5%AD%98%E3%80%81%E9%83%A8%E5%88%86%E8%AF%B7%E6%B1%82%E3%80%81ws%E9%95%BF%E8%BF%9E%E6%8E%A5%E3%80%81%E7%9F%AD%E6%97%B6%E9%97%B4%E5%86%85%E9%95%BF%E8%BF%9E%E6%8E%A5%0A2.%20%E8%AE%B0%E5%BD%95%E7%8A%B6%E6%80%81%EF%BC%9Acookie%0A%0A%E6%AD%A3%E5%B8%B8%E7%BC%96%E7%A8%8B%E5%B0%B1%E6%98%AF%E5%9F%BA%E4%BA%8E%E8%BF%99%E4%B8%AA%E5%8D%8F%E8%AE%AE%E9%87%8C%E9%9D%A2%E6%9C%89%E4%BB%80%E4%B9%88%EF%BC%8C%E6%89%8D%E8%83%BD%E5%81%9A%E4%BB%80%E4%B9%88%0A%0A%23%23%23%23%20%E6%B6%88%E6%81%AF%E4%BD%93%0A%3Eabc%3D22%26abcd%3D33%20%EF%BC%8C%E6%B2%A1%E4%BB%80%E4%B9%88%E5%8F%AF%E8%AF%B4%E7%9A%84....%0A%0A%E4%BE%9D%E7%84%B6%E6%98%AF%E4%B8%AAkv%E5%80%BC%E5%BD%A2%E6%80%81%0A%0A%0A%E5%8D%8F%E8%AE%AE%E8%AF%A6%E6%83%85\-%E5%93%8D%E5%BA%94%0A\-\-\-\-%0A%0A1.%20HTTP%E7%8A%B6%E6%80%81%E8%A1%8C%5Cr%5Cn%0A2.%20%E5%93%8D%E5%BA%94%E5%A4%B4%5Cr%5Cn%0A3.%203%E7%A9%BA%E8%A1%8C%5Cr%5Cn%0A4.%204%E5%8F%AF%E9%80%89%E7%9A%84%E6%B6%88%E6%81%AF%E4%BD%93%0A%20%0A%3E%E8%B7%9F%E8%AF%B7%E6%B1%82%E7%9A%84%E6%A0%BC%E5%BC%8F%E4%B8%80%E6%A0%B7%E4%B8%80%E6%A0%B7%E7%9A%84......%20%0A%20%0A%23%23%23%23%20%E5%93%8D%E5%BA%94\-%E8%A1%8C%0A%0A%3EHTTP%2F1.1%20200%20OK%0A%3E%E7%89%88%E6%9C%AC%E5%8F%B7%5B%E7%A9%BA%E6%A0%BC%5D%E7%8A%B6%E6%80%81%E7%A0%81%5B%E7%A9%BA%E6%A0%BC%5D%E5%8E%9F%E5%9B%A0%E8%AF%B4%E6%98%8E%5Cr%5Cn%20%0A%20%0AHTTP%2F1.0%20%E6%AF%8F%E6%AC%A1%E8%AF%B7%E6%B1%82%E9%83%BD%E9%9C%80%E8%A6%81%E5%BB%BA%E7%AB%8B%E6%96%B0%E7%9A%84TCP%E8%BF%9E%E6%8E%A5%EF%BC%8C%E8%BF%9E%E6%8E%A5%E4%B8%8D%E8%83%BD%E5%A4%8D%E7%94%A8%E3%80%82HTTP%2F1.1%20%E6%96%B0%E7%9A%84%E8%AF%B7%E6%B1%82%E5%8F%AF%E4%BB%A5%E5%9C%A8%E4%B8%8A%E6%AC%A1%E8%AF%B7%E6%B1%82%E5%BB%BA%E7%AB%8B%E7%9A%84TCP%E8%BF%9E%E6%8E%A5%E4%B9%8B%E4%B8%8A%E5%8F%91%E9%80%81%EF%BC%8C%E8%BF%9E%E6%8E%A5%E5%8F%AF%E4%BB%A5%E5%A4%8D%E7%94%A8%E3%80%82%E4%BC%98%E7%82%B9%E6%98%AF%E5%87%8F%E5%B0%91%E9%87%8D%E5%A4%8D%E8%BF%9B%E8%A1%8CTCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B%E7%9A%84%E5%BC%80%E9%94%80%EF%BC%8C%E6%8F%90%E9%AB%98%E6%95%88%E7%8E%87%E3%80%82%0A%E6%B3%A8%E6%84%8F%EF%BC%9A%E5%9C%A8%E5%90%8C%E4%B8%80%E4%B8%AATCP%E8%BF%9E%E6%8E%A5%E4%B8%AD%EF%BC%8C%E6%96%B0%E7%9A%84%E8%AF%B7%E6%B1%82%E9%9C%80%E8%A6%81%E7%AD%89%E4%B8%8A%E6%AC%A1%E8%AF%B7%E6%B1%82%E6%94%B6%E5%88%B0%E5%93%8D%E5%BA%94%E5%90%8E%EF%BC%8C%E6%89%8D%E8%83%BD%E5%8F%91%E9%80%81%E3%80%82%20%0A%20%0A%20%0A%23%23%23%23%20%E5%B8%B8%E7%94%A8%E5%93%8D%E5%BA%94%E7%8A%B6%E6%80%81%E7%A0%81%EF%BC%9A%0A%0A%7C%20code%20%7Cen%20desc%20%20%7Ccn%20desc%20%20%7C%20purpose%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20200%20%7C%20OK%20%7C%20%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%B7%E6%B1%82%E6%88%90%E5%8A%9F%20%7C%20%E7%A8%8B%E5%BA%8F%E5%91%98%E6%9C%80%E5%96%9C%E6%AC%A2%E7%9A%84%E7%8A%B6%E6%80%81%E7%A0%81%20%7C%0A%7C%20301%20%7C%20Moved%20Permanently%20%7C%20%E6%B0%B8%E4%B9%85%E7%A7%BB%E5%8A%A8%20%7C%20%E7%88%AC%E8%99%AB%E7%9A%84%E6%97%B6%E5%80%99%E9%81%87%E5%88%B0%E8%BF%87%EF%BC%8C%E8%BF%98%E6%9C%89%E5%B0%B1%E6%98%AF%EF%BC%9A%E6%9F%90%E4%BA%9B%E7%BD%91%E7%AB%99%E5%8F%AF%E8%83%BD%E5%8F%98%E4%BA%86%E5%9F%9F%E5%90%8D%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%20%7C%0A%7C%20302%20%7C%20Found%20%7C%20%E4%B8%B4%E6%97%B6%E7%A7%BB%E5%8A%A8%20%7C%20%20%E5%90%8C%E4%B8%8A%EF%BC%8C%E4%B8%8D%E8%BF%87%E5%A6%82%E6%9E%9C%E5%8F%8C%E6%96%B9%E7%9A%84%E7%B1%BB%E5%BA%93%E6%94%AF%E6%8C%81%E8%87%AA%E5%8A%A8%E8%B7%B3%E8%BD%AC%E5%88%B0%E6%96%B0%E9%A1%B5%E9%9D%A2%E8%BF%98%E5%A5%BD%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%94%AF%E6%8C%81%E5%B0%B1%E5%BE%88%E7%83%A6%20%7C%0A%7C%20304%20%7C%20Not%20Modified%20%7C%20%E6%9C%AA%E4%BF%AE%E6%94%B9%20%7C%E8%AF%B7%E6%B1%82%E6%96%B9%E5%8F%AF%E6%A0%B9%E6%8D%AE%E6%AD%A4%E5%80%BC%E4%BD%BF%E7%94%A8%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E6%96%87%E4%BB%B6%EF%BC%8C%E8%8A%82%E7%BA%A6%E7%BD%91%E7%BB%9C%E6%80%A7%E8%83%BD%E5%BC%80%E9%94%80%20%7C%0A%7C%20400%20%7C%20Bad%20Request%20%7C%20%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%B7%E6%B1%82%E6%9C%89%E8%AF%AD%E6%B3%95%E9%94%99%E8%AF%AF%20%7CC%E7%AB%AF%E6%9C%89%E5%85%AC%E5%85%B1%E5%BA%93%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E7%94%A8%E4%B8%8D%E5%88%B0%EF%BC%8C%E4%BD%86%E5%A6%82%E6%9E%9C%E8%87%AA%E5%B7%B1%E5%AE%9E%E7%8E%B0HTTP%E5%8D%8F%E8%AE%AE%E4%BC%9A%E9%81%87%E5%88%B0%E7%B1%BB%E4%BC%BC%E9%97%AE%E9%A2%98%20%20%7C%0A%7C%20401%20%7C%20Unauthorized%20%7C%20%E8%AF%B7%E6%B1%82%E6%9C%AA%E7%BB%8F%E6%8E%88%E6%9D%83%7C%E4%B8%8EAuthenticate%E5%A4%B4%E9%85%8D%E5%90%88%E4%BD%BF%E7%94%A8%EF%BC%8C%E5%AE%9E%E7%8E%B0%E7%AE%80%E5%8D%95%E7%99%BB%E9%99%86%E9%AA%8C%E8%AF%81%20%20%7C%20%20%0A%7C%20403%20%7C%20Forbidden%20%20%7C%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%94%B6%E5%88%B0%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%BD%86%E6%98%AF%E6%8B%92%E7%BB%9D%E6%8F%90%E4%BE%9B%E6%9C%8D%E5%8A%A1%20%7C%20apache%E9%81%87%E5%88%B0%E7%9A%84%E6%9C%80%E5%A4%9A%EF%BC%8C%E5%A4%A7%E6%88%91%E6%98%AF%E8%AF%B7%E6%B1%82%E7%9A%84%E8%B5%84%E6%BA%90%E6%B2%A1%E6%9C%89%E6%9D%83%E9%99%90%20%7C%0A%7C%20404%20%7C%20Not%20Found%20%7C%20%E8%AF%B7%E6%B1%82%E8%B5%84%E6%BA%90%E4%B8%8D%E5%AD%98%E5%9C%A8%20%7C%20eg%EF%BC%9A%E8%BE%93%E5%85%A5%E4%BA%86%E9%94%99%E8%AF%AF%E7%9A%84URL%20%7C%0A%7C%20500%20%7C%20Internal%20Server%20Error%20%20%7C%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%91%E7%94%9F%E4%B8%8D%E5%8F%AF%E9%A2%84%E6%9C%9F%E7%9A%84%E9%94%99%E8%AF%AF%20%7C%20%E5%83%8FPHP%20JAVA%20%E8%84%9A%E6%9C%AC%E6%89%A7%E8%A1%8C%E5%87%BA%E7%8E%B0%E9%94%99%E8%AF%AF%EF%BC%8C%E6%88%96%E8%80%85%E8%BF%90%E8%A1%8C%E6%97%B6%E5%88%A4%E6%96%AD%E5%8F%82%E6%95%B0%E4%B8%8D%E6%AD%A3%E7%A1%AE%E7%AD%89%E7%AD%89%20%7C%0A%7C%20502%20%7C%20Bad%20Gateway%20%7C%20%E7%BD%91%E5%85%B3%E9%94%99%7C%E5%A4%A7%E5%9E%8B%E5%85%AC%E5%8F%B8%E6%89%80%E6%9C%89%E7%9A%84%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%89%8D%E9%9D%A2%E6%98%AF%E8%A6%81%E6%9C%89%E7%BD%91%E5%85%B3%EF%BC%8C%E4%BD%86%E4%B8%AD%E5%B0%8F%E5%85%AC%E5%8F%B8%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%B2%A1%E9%81%87%E5%88%B0%E8%BF%87%EF%BC%8C%E5%80%92%E6%98%AF%E5%89%8D%E4%BA%9B%E5%B9%B4%E6%B7%98%E5%AE%9D%E5%8F%8C11%E7%9A%84%E6%97%B6%E5%80%99%E9%81%87%E5%88%B0%E8%BF%87%20%20%7C%0A%7C%20503%20%7C%20Server%20Unavailable%20%7C%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%BD%93%E5%89%8D%E4%B8%8D%E8%83%BD%E5%A4%84%E7%90%86%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%B8%80%E6%AE%B5%E6%97%B6%E9%97%B4%E5%90%8E%E5%8F%AF%E8%83%BD%E6%81%A2%E5%A4%8D%E6%AD%A3%E5%B8%B8%20%7C%20%E6%9C%AA%E9%81%87%E5%88%B0%E8%BF%87%20%7C%0A%0A%E7%AE%80%E5%8D%95%E5%88%86%E6%9E%90%E4%B8%8B%EF%BC%9A%0A%0A%7C%20code%20%7C%20desc%20%7C%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%201%E5%BC%80%E5%A4%B4%20%7C%20%E6%98%AFC%E4%B8%8ES%E8%BF%9E%E6%8E%A5%E6%97%B6%E7%9A%84%E4%B8%80%E4%BA%9B%E7%8A%B6%E6%80%81%20%7C%20%E5%9B%A0%E4%B8%BA%E5%BC%80%E6%BA%90%E7%9A%84%E7%B1%BB%E5%BA%93%E5%8F%8C%E6%96%B9%E9%83%BD%E6%9C%89%EF%BC%8C%E4%B8%94%E7%A8%B3%E5%AE%9A%EF%BC%8C%E5%A4%A7%E5%A4%9A%E9%83%BD%E8%AE%B0%E4%B8%8D%E4%BD%8F%E6%AD%A4%E7%8A%B6%E6%80%81%E5%80%BC%E4%BA%86......%20%7C%0A%7C%202%E5%BC%80%E5%A4%B4%20%7C%20%E5%9F%BA%E6%9C%AC%E4%B8%8A%E9%83%BD%E6%98%AF%E6%88%90%E5%8A%9F%20%7C%20%E5%BF%BD%E7%95%A5%EF%BC%8C%E5%A4%A7%E5%AE%B6%E9%83%BD%E5%96%9C%E6%AC%A2%E6%AD%A4%E7%8A%B6%E6%80%81%20%7C%0A%7C%203%E5%BC%80%E5%A4%B4%20%7C%20%E8%AF%B7%E6%B1%82%E7%9A%84%E8%B5%84%E6%BA%90%E8%A2%AB%E7%A7%BB%E5%8A%A8%2F%E6%94%B9%E5%8F%98%20%7C%20%E5%BE%88%E5%B0%91%E9%81%87%E5%88%B0%EF%BC%8C%E5%8D%B3%E4%BD%BF%E6%9C%89%EF%BC%8CC%E7%AB%AF%20%E7%B1%BB%E5%BA%93%E4%B9%9F%E4%BC%9A%E8%87%AA%E5%8A%A8%E8%B7%B3%E8%BD%AC%20%7C%0A%7C%204%E5%BC%80%E5%A4%B4%20%7C%20%E5%A4%A7%E5%A4%9A%E6%98%AF%E8%AF%B7%E6%B1%82%E6%96%B9%E7%9A%84%E9%94%99%E8%AF%AF%20%20%7C%20%E6%AF%94%E8%BE%83%E5%B0%91%E9%81%87%E5%88%B0%EF%BC%8C%E5%9B%A0%E4%B8%BAC%E7%AB%AF%E7%9A%84%E7%B1%BB%E5%BA%93%E9%83%BD%E6%AF%94%E8%BE%83%E7%A8%B3%E5%AE%9A%EF%BC%8C%E7%9C%9F%E9%81%87%E5%88%B0%E4%BA%86%EF%BC%8C%E5%90%8E%E7%AB%AF%E5%B0%B1%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E5%96%B7%E5%89%8D%E7%AB%AF%E4%BA%86%20%7C%0A%7C%205%E5%BC%80%E5%A4%B4%20%7C%20%E5%A4%A7%E5%A4%9A%E6%98%AF%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%94%99%20%7C%20%E8%AF%B7%E6%B1%82%E6%96%B9%E9%81%87%E5%88%B0%E7%B1%BB%E4%BC%BC%E7%9A%84%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%BC%80%E5%96%B7%E5%90%8E%E7%AB%AF%E7%A8%8B%E5%BA%8F%E5%91%98%E5%90%A7%20%7C%0A%0A%20%0A%23%23%23%23%20%E5%93%8D%E5%BA%94\-%E5%A4%B4%0A%20%0A%60%60%60%0AKeep\-Alive%3A%20timeout%3D5%2C%20max%3D100%0AConnection%3A%20Keep\-Alive%0AContent\-Type%3A%20text%2Fhtml%0AServer%3A%20Apache%2F2.2.17%20\(Win32\)%20PHP%2F5.2.9\-2%0ALast\-Modified%3A%20Sat%2C%2022%20Jan%202011%2003%3A29%3A17%20GMT%0AContent\-Range%3A%20bytes%200\-5%0A%60%60%60%0A%3E%E8%B7%9F%E8%AF%B7%E6%B1%82%E6%A0%BC%E5%BC%8F%E5%9F%BA%E6%9C%AC%E4%B8%80%E6%A0%B7%EF%BC%8C%E5%B0%B1%E6%98%AFKV%0A%0A%E7%9C%8B%E4%B8%80%E4%B8%8B%E5%B8%B8%E7%94%A8%E7%9A%84%EF%BC%9A%0A%0A%7Ckey%20%20%7C%20desc%20%7C%20purpose%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20Content\-Encoding%20%7C%20%E4%BC%A0%E8%BE%93%E7%9A%84%E6%A0%BC%E5%BC%8F%20%7Cgzip%20%E5%8E%8B%E7%BC%A9%20%20%7C%0A%7C%20Transfer\-Encoding%20%7C%E6%8A%A5%E6%96%87%E6%A0%BC%E5%BC%8F\-%E5%88%86%E5%9D%97%E7%BC%96%E7%A0%81%20%20%7C%20%E7%AE%97%E6%98%AF%E5%AF%B9ws%E5%8D%8F%E8%AE%AE%E7%9A%84%E4%BC%98%E5%8C%96%E5%90%A7%20%7C%0A%7C%20Last\-Modified%20%7C%20%E8%AF%A5%E8%B5%84%E6%BA%90%E6%9C%80%E5%90%8E%E4%BF%AE%E6%94%B9%E6%97%B6%E9%97%B4%20%7C%20%E9%85%8D%E5%90%88%E8%AF%B7%E6%B1%82%E7%AB%AF%E6%9D%A5%E4%BD%BF%E7%94%A8%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%EF%BC%8C%E8%8A%82%E7%BA%A6%E7%BD%91%E7%BB%9C%E6%80%A7%E8%83%BD%20%7C%0A%7C%20Connection%7C%20%E9%95%BF%E8%BF%9E%E6%8E%A5%20%7C%20%E8%AF%B7%E6%B1%82%E6%96%B9%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8Upgrade%E5%81%9Aws%E9%95%BF%E8%BF%9E%E6%8E%A5%20%7C%0A%7C%20Content\-Range%20%7C%20%E5%88%86%E7%89%87%E4%BC%A0%E8%BE%93%E6%97%B6%20%7C%20%E8%AF%B7%E6%B1%82%E6%96%B9%E6%83%B3%E5%88%86%E6%AE%B5%E4%B8%8B%E8%BD%BD%E5%A4%A7%E6%96%87%E4%BB%B6%E6%97%B6%E4%BD%BF%E7%94%A8%20%20%7C%0A%7C%20Content\-Type%20%7C%20%E5%BD%93%E5%89%8D%E8%BF%94%E5%9B%9E%E7%9A%84%E5%86%85%E5%AE%B9%E7%B1%BB%E5%9E%8B%20%7C%20%E5%BD%93%E5%89%8DS%E7%AB%AF%E8%BF%94%E5%9B%9E%E7%9A%84%E5%85%B7%E4%BD%93%E7%9A%84%E5%86%85%E5%AE%B9%EF%BC%8C%E5%A6%82%EF%BC%9Ahtml%20json%20image%20js%20%7C%0A%7C%20Content\-Length%20%7C%E5%BD%93%E5%89%8D%E8%BF%94%E5%9B%9E%E7%9A%84%E5%86%85%E5%AE%B9%E6%80%BB%E9%95%BF%E5%BA%A6%20%20%7C%20%E5%8F%AA%E6%98%AF%E6%96%B9%E4%BE%BF%E6%8E%A5%E6%94%B6%E7%AB%AF%E5%A4%84%E7%90%86%EF%BC%8C%E5%BE%88%E5%B0%91%E7%94%A8%20%7C%0A%7C%20Location%20%7C%20%E9%87%8D%E6%96%B0%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90%E4%BD%8D%E7%BD%AE%20%20%7C%20%E5%A6%82%E6%9E%9C%E6%9C%89301%20302%20%E8%BF%99%E7%A7%8D%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%94%A8%E6%AD%A4%E5%8F%82%E6%95%B0%E8%B7%B3%E8%BD%AC%EF%BC%8C%E5%BE%88%E5%B0%91%E7%94%A8%20%20%7C%0A%7C%20Access\-Control\-Allow\-Origin%20%20%7C%20%E5%85%81%E8%AE%B8%E8%B7%A8%E5%9F%9F%E8%AE%BF%E9%97%AE%E7%9A%84%E5%9F%9F%E5%90%8D%20%7C%E8%B7%A8%E5%9F%9F%20%20%7C%0A%0A%0A%E5%B8%B8%E7%94%A8%20content\-type%EF%BC%9A%0A%7C%20key%20%7C%20desc%20%7Cpurpose%20%20%7C%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20text%2Fhtml%20%7C%20HTML%E6%A0%BC%E5%BC%8F%20%7C%20%E6%9C%80%E5%B8%B8%E7%94%A8%E7%9A%84%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E7%BB%99%E6%B5%8F%E8%A7%88%E5%99%A8%E8%A7%A3%E6%9E%90%2F%E6%89%A7%E8%A1%8C%2F%E6%B8%B2%E6%9F%93%20%7C%20%20%7C%0A%7C%20text%2Fxml%20%7C%20XML%E6%A0%BC%E5%BC%8F%20%20%7C%20%E4%B8%BB%E8%A6%81%E6%98%AF%E4%B8%80%E4%BA%9B%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BD%BF%E7%94%A8%EF%BC%8C%E7%9B%B8%E6%AF%94html%E6%9B%B4%E4%B8%A5%E8%B0%A8%EF%BC%8C%E4%BD%86%E4%B9%9F%E6%9B%B4%E5%A4%8D%E6%9D%82%E4%BA%9B%20%7C%20%20%7C%0A%7C%20image%2Fgif%20%7C%20gif%E5%9B%BE%E7%89%87%E6%A0%BC%E5%BC%8F%20%20%7C%20%20%7C%20%20%7C%0A%7C%20image%2Fjpeg%20%7C%20jpg%E5%9B%BE%E7%89%87%E6%A0%BC%E5%BC%8F%20%20%7C%20%20%7C%20%20%7C%0A%7C%20image%2Fpng%20%7C%20png%E5%9B%BE%E7%89%87%E6%A0%BC%E5%BC%8F%20%7C%20%20%7C%20%20%7C%0A%7C%20application%2Fjavascript%20%7C%20js%E4%BB%A3%E7%A0%81%20%7C%20%20%7C%20%20%7C%0A%7C%20application%2Fjson%20%7C%20JSON%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%9C%80%E5%B8%B8%E7%94%A8%EF%BC%8C%E5%89%8D%E7%AB%AF%E4%BB%8E%E5%90%8E%E7%AB%AF%E6%8B%BF%E6%BA%90%E6%95%B0%E6%8D%AE%E5%B0%B1%E6%98%AF%E8%BF%99%E4%B8%AA%20%7C%20%20%7C%20%20%7C%0A%7C%20application%2Fpdf%20%7C%20%E4%B8%8D%E5%A4%AA%E5%B8%B8%E7%94%A8%7C%20%7C%20%7C%0A%7C%20application%2Fmsword%20%7C%20%20%E4%B8%8D%E5%A4%AA%E5%B8%B8%E7%94%A8%7C%20%7C%20%7C%0A%7C%20application%2Foctet\-stream%20%7C%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%B5%81%E6%95%B0%E6%8D%AE%EF%BC%88%E5%A6%82%E5%B8%B8%E8%A7%81%E7%9A%84%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD%EF%BC%89%20%7C%20%7C%20%7C%0A%7C%20application%2Fx\-www\-form\-urlencoded%7C%20%20form%E8%A1%A8%E5%8D%95%E6%95%B0%E6%8D%AE%E8%A2%AB%E7%BC%96%E7%A0%81%E4%B8%BAkey%2Fvalue%E6%A0%BC%E5%BC%8F%E5%8F%91%E9%80%81%E5%88%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88%E8%A1%A8%E5%8D%95%E9%BB%98%E8%AE%A4%E7%9A%84%E6%8F%90%E4%BA%A4%E6%95%B0%E6%8D%AE%E7%9A%84%E6%A0%BC%E5%BC%8F%EF%BC%89%7C%20%7C%20%7C%0A%7C%20multipart%2Fform\-data%7C%E8%A1%A8%E5%8D%95%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%97%B6%EF%BC%8C%E5%B0%B1%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8%E8%AF%A5%E6%A0%BC%E5%BC%8F%20%7C%20%7C%20%7C%0A%0A%0A%3E%E5%92%8B%E5%BD%A2%E5%AE%B9%E5%91%A2%EF%BC%8C%E5%85%B6%E5%AE%9Econtent\-type%E5%8F%AA%E6%98%AF%E4%B8%80%E4%B8%AA%E6%A0%87%E8%AF%86%EF%BC%8C%E6%88%96%E8%80%85%E8%AF%B4%E6%9C%89%E5%BE%88%E5%A4%A7%E4%B8%80%E9%83%A8%E5%88%86%E6%98%AF%E7%BB%99%E6%B5%8F%E8%A7%88%E5%99%A8%E4%BD%BF%E7%94%A8%E7%9A%84%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9A%E8%A7%A3%E6%9E%90PDF%20EXCEL%20MP4%20MP3%20%E6%A0%BC%E5%BC%8F%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8\(%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8F%92%E4%BB%B6\)%EF%BC%8C%E8%80%8C%E6%88%91%E4%BB%AC%E6%97%A5%E5%B8%B8%E4%BD%BF%E7%94%A8%E6%9C%80%E5%A4%9A%E7%9A%84%E6%98%AFjson%EF%BC%8C%E5%83%8Ftext%2Fhtml%E8%BF%99%E7%A7%8D%20WEBSERVCIE%E5%B0%B1%E5%B8%AE%E4%BD%A0%E6%90%9E%E5%A5%BD%E4%BA%86%E3%80%82%0A%0A%E6%80%BB%E4%B9%8B%EF%BC%9A%E4%BB%8E%E7%B1%BB%E5%9E%8B%E4%B8%AD%E4%B9%9F%E8%83%BD%E7%9C%8B%E5%87%BA%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E8%83%BD%E5%A4%84%E7%90%86%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E8%BF%98%E6%98%AF%E6%8C%BA%E5%A4%9A%E7%9A%84%EF%BC%8C%E5%86%8D%E9%85%8D%E5%90%88%E6%98%AF%E8%87%AA%E5%B8%A6%E7%9A%84%E6%8F%92%E4%BB%B6%EF%BC%8C%E5%83%8F%EF%BC%9A%E6%92%AD%E6%94%BE%E8%A7%86%E9%A2%91%E3%80%81%E9%9F%B3%E9%A2%91%EF%BC%8C%E6%98%BE%E7%A4%BAWORD%20EXCEL%E6%96%87%E6%A1%A3%E7%AD%89%E7%AD%89%EF%BC%8C%E9%82%A3%E6%B5%8F%E8%A7%88%E5%99%A8%E7%81%AB%E7%88%86%E4%B9%9F%E6%98%AF%E5%BF%85%E7%84%B6%E7%9A%84%E5%95%8A%0A%0A%23%23%23%23%20%E5%93%8D%E5%BA%94\-%E4%BD%93%0A%E8%BF%99%E4%B8%AA%E5%B0%B1%E6%98%AF%E4%B8%8D%E5%90%8C%E7%9A%84content\-type%20%E7%BB%99%E5%87%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E5%86%85%E5%AE%B9%E4%BA%86%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%87%AA%E8%A1%8C%E8%BE%93%E5%87%BA%E5%86%85%E5%AE%B9%E4%BD%93%EF%BC%8C%E6%97%A0%E6%89%80%E8%B0%93%E4%BA%86%EF%BC%8C%E6%B2%A1%E5%95%A5%E9%87%8D%E7%82%B9%E5%8F%AF%E8%AE%B2%E3%80%82%0A%0Ahttp%E7%89%88%E6%9C%AC%E5%AF%B9%E6%AF%94%0A\-\-\-\-\-\-\-%0A1.%20http1.0%0A2.%20http1.1%0A3.%20http2.0%0A4.%20Chunked%20transfer\-coding%0A%0Ahttp%201.0%0A\-\-\-\-\-%0A%E8%BF%99%E4%B8%AA%E5%AF%B9%E6%AF%94%E7%9A%84%E6%9C%80%E5%9F%BA%E7%A1%80%E7%89%88%EF%BC%8C%E4%B8%8D%E8%AE%B2%E8%A7%A3%E4%BA%86%0A%0Ahttp%201.1%0A\-\-\-\-%0A%E6%96%B0%E7%9A%84%E5%8A%9F%E8%83%BD%E7%82%B9%EF%BC%9A%20%0A1.%20head%20of%20line%20blocking%20%E9%93%BE%E6%8E%A5%E5%A4%8D%E7%94%A8%0A2.%20%E6%9B%B4%E5%A4%9A%E7%9A%84%E7%BC%93%E5%AD%98%E6%8E%A7%E5%88%B6%0A3.%20%E5%93%8D%E5%BA%94%E9%94%99%E8%AF%AF%E7%A0%81%E5%A2%9E%E5%8A%A0%0A4.%20%E4%BC%A0%E8%BE%93%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%0A%0A%E9%93%BE%E6%8E%A5%E5%A4%8D%E7%94%A8%3A1.0%E6%97%B6%EF%BC%8C%E6%AF%8F%E4%B8%AA%E8%AF%B7%E6%B1%82%E5%8D%B3%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84TCP%E9%93%BE%E6%8E%A5%EF%BC%8C%E4%BC%98%E5%8C%96%E6%88%90%EF%BC%9A%E5%8F%AA%E9%9C%80%E8%A6%81%E4%B8%80%E6%AC%A1TCP%E9%93%BE%E6%8E%A5%EF%BC%8C%E9%87%8D%E5%A4%8D%E5%88%A9%E7%94%A8%E8%AF%A5%E9%93%BE%E6%8E%A5%EF%BC%8C%E7%BB%A7%E7%BB%AD%E5%8F%91%E8%AF%B7%E6%B1%82.%0A%E5%85%B7%E4%BD%93%E5%88%B0%E8%AF%A6%E7%BB%86%E7%9A%84%E5%8D%8F%E8%AE%AE%E4%B8%AD%EF%BC%9Aconnection%3A%20keep\-alive%20%7C%20connection%3A%20close%0A%E8%99%BD%E7%84%B6%E5%8F%AF%E4%BB%A5%E4%B8%80%E6%AC%A1%E5%8F%91%E9%80%81N%E4%B8%AA%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E4%B8%8D%E9%98%BB%E5%A1%9E%EF%BC%8C%E4%BD%86%E6%98%AF%E5%93%8D%E5%BA%94%E6%96%B9%EF%BC%8C%E5%BE%97%E6%8C%89%E8%AF%B7%E6%B1%82%E6%96%B9%E7%9A%84%E8%AF%B7%E6%B1%82%E6%97%B6%E9%97%B4%EF%BC%8C%E4%BE%9D%E6%AC%A1%E5%93%8D%E5%BA%94%EF%BC%8C%E4%B8%A5%E6%A0%BC%E7%9C%8B%EF%BC%8C%E8%BF%98%E6%98%AF%E4%BC%9A%E6%9C%89%E9%98%BB%E5%A1%9E%E9%97%AE%E9%A2%98%E3%80%82%E4%BD%86%E8%AF%B7%E6%B1%82%E6%96%B9%E5%8F%AF%E4%BB%A5%E5%B9%B6%E8%A1%8C%E7%9A%84%E5%8F%91%E8%AF%B7%E6%B1%82%E3%80%82%0A%0A%E7%BC%93%E5%AD%98%E6%8E%A7%E5%88%B6%E5%A5%BD%E5%83%8F%E4%B9%9F%E6%B2%A1%E5%95%A5%E8%AE%B2%E7%9A%84%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%AF%94%E4%B9%8B%E5%89%8D%E7%9A%84%E7%BC%93%E5%AD%98%E6%8E%A7%E5%88%B6%E5%A4%9A%E4%B8%80%E4%BA%9B%E5%A4%B4%E5%8F%82%E6%95%B0%0A%60%60%60%0AIf\-Modefied\-Since%0AIf\-Unmodified\-Since%0AIf\-Match%0AIf\-None\-Match%0AET\-tag%0A%60%60%60%0A%0Aheader%E4%B8%AD%E5%A2%9E%E5%8A%A0%E4%BA%86host%E5%AD%97%E6%AE%B5%0A%0A%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%EF%BC%9A%E5%8A%A0%E5%85%A5%E4%BA%86%E7%8A%B6%E6%80%81%E7%A0%81100%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%85%88%E8%AF%95%E6%8E%A2S%E7%AB%AF%E6%98%AF%E5%90%A6%E6%AD%A3%E5%B8%B8%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%AD%A3%E5%B8%B8%EF%BC%8C%E5%86%8D%E5%8F%91BODY%EF%BC%8C%E5%8B%89%E5%BC%BA%E7%AE%97%E6%98%AF%E4%BC%98%E5%8C%96%E7%82%B9%E4%BC%A0%E8%BE%93%E6%80%A7%E8%83%BD%E5%90%A7%EF%BC%8C%E6%84%9F%E8%A7%89%E6%8C%BA%E9%B8%A1%E8%82%8B%E3%80%82%E8%BF%98%E6%9C%89%E4%B8%80%E4%B8%AA%EF%BC%8C%E5%9D%97%E4%BC%A0%E8%BE%93%EF%BC%9A%E5%B0%86%E4%B8%80%E4%B8%AA%E5%A4%A7%E7%9A%84body%E5%88%87%E6%88%90%E8%8B%A5%E5%B9%B2%E5%B0%8F%E5%9D%97\(Chunked%20transfer\-coding\)%EF%BC%8C%E5%88%86%E6%89%B9%E6%AC%A1%E4%BC%A0%E8%BE%93%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%8F%AF%E4%BB%A5%E7%94%A8%E5%88%B0%E6%96%AD%E7%82%B9%E7%BB%AD%E4%BC%A0%E4%B8%8A%E3%80%82%0A%0A%E6%80%BB%E7%BB%93%EF%BC%9A%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E9%93%BE%E6%8E%A5%E5%A4%8D%E7%94%A8%E7%AE%97%E6%98%AF%E5%B8%B8%E7%94%A8%E4%BA%9B%EF%BC%8C%E5%85%B6%E5%AE%83%E7%9A%84%E9%83%BD%E6%8C%BA%E9%B8%A1%E8%82%8B%E3%80%82%0A%0A%0Ahttp%202.0%0A\-\-\-\-%0A1.%20%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%0A2.%20%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%0A3.%20%E8%AF%B7%E6%B1%82%E4%BC%98%E5%85%88%E9%9B%86%0A2.%20header%20%E5%8E%8B%E7%BC%A9%0A4.%20server%20push%0A%0A%0A%0A1%202%203%20%E6%9D%A1%E6%80%BB%E7%BB%93%E4%B8%8B%E6%9D%A5%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AF%EF%BC%9A%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%88%86%E5%B8%A7%0A%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%88%86%E5%B8%A7%EF%BC%9A%E5%9C%A8TCP%20%E5%92%8C%20HTTP%20%E5%B1%82%EF%BC%8C%E4%B8%AD%E9%97%B4%E5%8A%A0%E4%B8%80%E5%B1%82%EF%BC%9A%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%88%86%E5%B8%A7%0A%E8%AF%A5%E5%B1%82%EF%BC%9A%E8%AF%B7%E6%B1%82%E6%96%B9%E4%BB%BB%E4%BD%95%E5%8F%91%EF%BC%8C%E5%93%8D%E5%BA%94%E6%96%B9%E4%BB%BB%E6%84%8F%E5%93%8D%E5%BA%94%EF%BC%8C%E8%AF%A5%E5%B1%82%E4%BC%9A%E6%8A%8A%20%E4%B8%A4%E8%BE%B9%E7%9A%84%E6%95%B0%E6%8D%AE%E5%88%86%E6%88%90%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%9A%84%E5%B8%A7%EF%BC%8C%E5%81%9A%E6%8E%A7%E5%88%B6%EF%BC%8C%E5%86%8D%E8%BF%94%E5%9B%9E%E7%BB%99%E4%B8%A4%E7%AB%AF%E3%80%82%0A%0A%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%EF%BC%9Ahttp%201.1%20%E5%8A%A0%E5%85%A5%E4%BA%86%E8%BF%9E%E6%8E%A5%E5%A4%8D%E7%94%A8%EF%BC%8C%E4%BD%86%E6%98%AF%E5%BF%85%E9%A1%BB%E5%BE%97%E6%8C%89%E7%85%A7%E8%AF%B7%E6%B1%82%E9%A1%BA%E5%BA%8F%E6%9D%A5%E5%93%8D%E5%BA%94%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E9%98%BB%E5%A1%9E%E3%80%82%E5%A6%82%E6%9E%9C%E5%9C%A8%E8%BF%99%E4%B8%AD%E9%97%B4%E5%86%8D%E5%8A%A0%E4%B8%80%E4%BA%9B%E6%8E%A7%E5%88%B6%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%AE%9E%E7%8E%B0%E7%9C%9F%E7%9A%84%EF%BC%9A%E5%8D%95%E9%93%BE%E6%8E%A5%E3%80%81%E6%97%A0%E9%98%BB%E5%A1%9E%E3%80%81%E5%A4%9A%E8%AF%B7%E6%B1%82%E3%80%82%0A%0Aserver%20push%3AS%E7%AB%AF%E5%9C%A8%E6%8E%A5%E6%94%B6%E5%88%B0C%E7%AB%AF%E4%B8%80%E4%B8%AA%E4%B8%BB%E8%AF%B7%E6%B1%82%E5%90%8E%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%8A%A8%E6%8A%8A%E5%90%8E%E7%BB%AD%E7%9A%84%E8%B5%84%E6%BA%90%E8%AF%B7%E6%B1%82%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E4%B9%9F%E4%B8%80%E5%B9%B6%E6%8E%A8%E9%80%81%E7%BB%99C%E7%AB%AF%EF%BC%8C%E8%BF%99%E6%A0%B7C%E7%AB%AF%E4%B8%8D%E7%94%A8%E5%86%8D%E5%8D%95%E7%8B%AC%E5%8F%91%E8%AF%B7%E6%B1%82%EF%BC%8C%E7%9B%B4%E6%8E%A5%E4%BB%8E%E7%BC%93%E5%AD%98%E4%B8%AD%E6%8B%BF%E5%B0%B1%E8%A1%8C%E4%BA%86%E3%80%82%0A%0A%0A%E7%BC%BA%E7%82%B9%EF%BC%9A%E5%8D%95%E8%BF%9E%E6%8E%A5%E4%BC%9A%E5%9B%A0TCP%E7%BA%BF%E5%A4%B4%E9%98%BB%E5%A1%9E%EF%BC%88head\-of\-line%20blocking%EF%BC%89%E7%9A%84%E7%89%B9%E6%80%A7%E8%80%8C%E4%BC%A0%E8%BE%93%E9%80%9F%E5%BA%A6%E5%8F%97%E9%99%90%E3%80%82%E5%8A%A0%E4%B8%8A%E5%AD%98%E5%9C%A8%E5%8F%AF%E8%83%BD%E4%B8%A2%E5%8C%85%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%85%B6%E8%B4%9F%E9%9D%A2%E5%BD%B1%E5%93%8D%E5%B7%B2%E8%B6%85%E8%BF%87%E5%8E%8B%E7%BC%A9%E5%A4%B4%E9%83%A8%E5%92%8C%E4%BC%98%E5%85%88%E7%BA%A7%E6%8E%A7%E5%88%B6%E5%B8%A6%E6%9D%A5%E7%9A%84%E5%A5%BD%E5%A4%84%E3%80%82%0A%0A%E6%80%BB%E7%BB%93%EF%BC%9A%E5%B0%B1%E4%B8%80%E4%B8%AA%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E7%A1%AE%E5%AE%9E%E8%83%BD%E8%A7%A3%E5%86%B3%E4%B9%8B%E5%89%8D%E7%9A%84%E9%98%BB%E5%A1%9E%E9%97%AE%E9%A2%98~%E5%85%B6%E5%AE%83%E8%BF%98%E6%98%AF%E9%B8%A1%E8%82%8B%E3%80%82%0A%0A%0A%E5%B8%B8%E7%94%A8%E5%AE%9E%E4%BD%93%0A\-\-\-%0A%7C%20%E6%98%BE%E7%A4%BA%E7%BB%93%E6%9E%9C%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%20%20%E5%90%8D%E7%A7%B0%7C%20%E7%BC%96%E5%8F%B7%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20%3C%20%7C%20%E5%B0%8F%E4%BA%8E%E5%8F%B7%20%7C%20%26lt%3B%20%7C%26%2360%3B%20%7C%0A%7C%20%3E%20%7C%20%E5%A4%A7%E4%BA%8E%E5%8F%B7%20%7C%20%26gt%3B%20%7C%20%26%2362%3B%20%7C%0A%7C%26%20%20%7C%20%E5%92%8C%E5%8F%B7%20%7C%20%26amp%3B%20%7C%26%2338%3B%20%20%7C%0A%7C%20%22%20%7C%20%E5%BC%95%E5%8F%B7%20%7C%20%20%26quot%3B%7C%20%20%26%2334%3B%7C%0A%7C%20'%20%7C%20%E6%92%87%E5%8F%B7%20%7C%20%26apos%3B%20\(IE%E4%B8%8D%E6%94%AF%E6%8C%81\)%20%7C%20%26%2339%3B%20%7C%0A%7C%20%C2%A9%20%7C%E7%89%88%E6%9D%83%20%20%7C%20%26copy%3B%20%7C%20%26%23169%3B%20%7C%0A%7C%20%C2%AE%20%7C%20%E6%B3%A8%E5%86%8C%E5%95%86%E6%A0%87%20%7C%20%26reg%3B%20%7C%20%26%23174%3B%20%7C%0A%7C%20%E2%84%A2%20%7C%20%E5%95%86%E6%A0%87%20%7C%20%26trade%3B%20%20%7C%20%26%238482%3B%20%7C%0A%7C%C2%A5%7C%E5%85%83%7C%26yen%3B%7C%26%23165%3B%7C%0A%7C%C2%A5%7C%E7%A9%BA%E6%A0%BC%7C%09%26nbsp%3B%7C%09%26%23160%3B%7C%0A%0A%0A
