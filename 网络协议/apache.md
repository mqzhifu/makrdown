# apache

解释

APACHE，以MODULE方式运行加载。

启动\-》等待请求\-》接收请求\-》分发进程/线程\-》请求PHP\-》接收返回数据\-》返回给C端\-》继续等待请求

启动：读取配置文件，初始化环境，加载各种module，进入等待请求阶段。

缓存

mod\_expires

\<IfModule expires\_module\>

ExpiresActive On

\#访问之后的一个月不再更新

ExpiresDefault "access plus 1 month"

\#访问之后的4周不再更新

\#ExpiresDefault "access plus 4 weeks"

\#访问之后的30天不再更新

\#ExpiresDefault "access plus 30 days"

\</IfModule\>

mod\_headers

\<IfModule headers\_module\>

# htm,html,txt类的文件缓存一个小时

\<filesmatch ".\(html|htm|txt\)$"\>

header set cache\-control "max\-age=3600"

\</filesmatch\>

# css, js, swf类的文件缓存一个星期

\<filesmatch ".\(css|js|swf\)$"\>

header set cache\-control "max\-age=604800"

\</filesmatch\>

\</IfModule\>

HTTP头协议中几个参数:Expires、Cache\-Control、Last\-Modified、ETag

Expires：过期时间

Cache\-Control：失效规则，no\-cache, must\-revalidate, max\-age=0

Last\-Modified：初次请求，S端会给出Last\-Modified的值，二次请求，会附带这个值，S端接收到这个参数，判断当前URL对应的文件是否发生了修改，如果有，更新文件，更新缓存内容~返回200，如果没有，返回304

ETag：类似TOKEN，增加安全

mod\_cache

mod\_disk\_cache ：文件缓存

mod\_mem\_cache：内存缓存

压缩

开启mod\_deflate.so mod\_headers.so

一个参数：Content\-encoding:gzip

KeepAlive

保持连接，当在短时间内有100个请求的时候，只建立1\-2个连接，注意是一定时间内保持连接，而非长连接

但是，开启KeepAlive，意味着短时间内需要维拉的连接增多，内存增加~而关闭了，需要处理的连接变多，关闭/启动 连接，增加了CPU的消耗

KeepAlive On

KeepAliveTimeout=10

MaxKeepAliveRequests

一次连接可以进行的HTTP请求的最大请求次数

timeout

这次请求和下次请求，间隔的时间，不要误以为是连接超时~时间

expires\_module

告诉浏览器缓存静态文件\(js css 等\)的时间

MPM（Multi\-Processing Module，多进程处理模块）：prefork, worker和event

编译的时候，可以加参数选择，\-\-with\-mpm=prefork|worker|event。event需要OS支持EPOLL~apachectl \-V，可以查看

prefork

多进程\+select，启动之后会预先启动一些进程，无须考虑线程安全的问题。

具体解释：参数配置与调优

\<IfModule mpm\_prefork\_module\>

StartServers 5 //启动后，派生多少个子进程

MinSpareServers 5 //最小空闲进程数

MaxSpareServers 10 //最大空闲进程数

MaxRequestWorkers 250 //最大可接收连接数

MaxConnectionsPerChild 0 //单个进程最大可提供多少个连接数

\</IfModule\>

启动时建立StartServers个子进程，然后按每秒创建指数级个进程直到达到MinSpareServers个进程（最多增到每秒32个）。MinSpareServers，设置空闲子进程的最大数量，默认为10。如果当前有超过MaxSpareServers数量的空闲子进程，那么父进程将杀死多余的子进程。此参数不要设的太大。如果你将该指令的值设置为比MinSpareServers小，Apache将会自动将其修成"MinSpareServers\+1"。

MaxRequestPerChild：指定每个进程处理了多少个请求后就自我毁灭。此参数好像没用了.....

MaxRequestWorkers ：最大接收连接数，超过将排队

优点是成熟稳定，缺点是比较消耗内存，而且并发支持受限于进程数量，对高并发支持稍差。

worker模式

多进程\+多线程\+SELECT。预先派生出若干进程（少量），每个进程再派生出若干线程，同时包括一个监听线程。

具体解释：参数配置与调优

\<IfModule mpm\_worker\_module\>

StartServers 3 //启动后，派生多少个子进程

MinSpareThreads 75 //最小空闲线程数

MaxSpareThreads 250 //最大空闲线程数

ThreadsPerChild 25 //每个进程包含多少个线程

MaxRequestWorkers 400 //最大可接收连接数

MaxConnectionsPerChild 0 //每个线程最大连接数

\</IfModule\>

大部分参数跟上面差不多

优点是，线程可共享进程的内存空间，占用内存少，并发性比较高

缺点是必须考虑线程安全。如果使用了keep\-alive方式，一个线程可能会被一直保持一个连接，但中间没有请求，直到超时。如果有多个线程被这样占据，在高并发场景下同样会出现无线程可用的情形。

event模式

多进程\+多线程\+epoll。是在2.4版本中才稳定发布的模式，它在worker的基础上，解决了keep\-alive连接不能释放的问题。event MPM中，会有一个专门的线程来管理这些keep\-alive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放。这样增强了高并发场景下的请求处理能力。epoll模式

具体解释：参数配置与调优

\<IfModule mpm\_worker\_module\>

StartServers 3 //启动后，派生多少个子进程

MinSpareThreads 75 //最小空闲线程数

MaxSpareThreads 250 //最大空闲线程数

ThreadsPerChild 25 //每个进程包含多少个线程

MaxRequestWorkers 400 //最大可接收连接数

MaxConnectionsPerChild 0 //每个线程最大连接数

\</IfModule\>

大部分参数跟上面差不多

虚拟目录访问控制

\<Directory /\>

\#指定根目录"/"启用Indexes、FollowSymLinks两种特性。

Options Indexes FollowSymLinks

AllowOverride all

Order allow,deny

Allow from all

\</Directory\>

AllowOverride：.htaccess

Options

All：表示除MultiViews之外的所有特性。这也是Options指令的默认设置。

None

FollowSymLinks：服务器允许在此目录中使用符号连接。如果该配置选项位于\<Location\>配置段中，将会被忽略。

Indexes：如果是目录，则显示下面所有文件列表

MultiViews

SymLinksIfOwnerMatch

ExecCGI

Includes

IncludesNOEXEC

ORDER与Allow DENY 配合使用

Allow ：允许访问的IP

DENY ：禁止访问的IP

REWRITE

. 匹配任何單字符

\[chars\] 匹配字符串:chars

\[^chars\] 不匹配字符串:chars

text1|text2 可選擇的字符串:text1或text2

? 匹配0到1個字符

- 匹配0到多個字符

- 匹配1到多個字符
    ^ 字符串開始標誌
    $ 字符串結束標誌
    \\n 轉義符標誌

反向引用 $N 用於 RewriteRule 中匹配的變量調用\(0 \<= N \<= 9\)

反向引用 %N 用於 RewriteCond 中最後一個匹配的變量調用\(1 \<= N \<= 9\)

RewriteCond適用的標誌符：

nocase|NC' \(no case\)忽略大小

ornext|OR' \(or next condition\)邏輯或，可以同時匹配多個RewriteCond條件

RewriteRule適用的標誌符：

redirect|R \[=code\] \(force redirect\)強迫重寫為基於http開頭的外部轉向\(注意URL的變化\) 如：\[R=301,L\]

forbidden|F' \(force URL to be forbidden\)重寫為禁止訪問

proxy|P' \(force proxy\)重寫為通過代理訪問的http路徑

last|L' \(last rule\)最後的重寫規則標誌，如果匹配，不再執行以後的規則

next|N' \(next round\)循環同一個規則，直到不能滿足匹配

chain|C' \(chained with next rule\)如果匹配該規則，則繼續下面的有Chain標誌的規則。

type|T=MIME\-type' \(force MIME type\)指定MIME類型

nosubreq|NS' \(used only if no internal sub\-request\)如果是內部子請求則跳過

nocase|NC' \(no case\)忽略大小

qsappend|QSA' \(query string append\)附加查詢字符串

noescape|NE' \(no URI escaping of output\)禁止URL中的字符自動轉義成%\[0\-9\]\+的形式。

passthrough|PT' \(pass through to next handler\)將重寫結果運用於mod\_alias

skip|S=num' \(skip next rule\(s\)\)跳過下面幾個規則

env|E=VAR:VAL' \(set environment variable\)添加環境變量

example：

RewriteEngine on

RewriteCond %{HTTP\_USER\_AGENT} ^MSIE \[NC,OR\]

RewriteCond %{HTTP\_USER\_AGENT} ^Opera \[NC\]

RewriteRule ^.\* \- \[F,L\] 這裡」\-」表示沒有替換，瀏覽器為IE和Opera的訪客將被禁止訪問。

example：

RewriteEngine On

RewriteBase /test

RewriteCond %{REQUEST\_FILENAME}.php \-f

RewriteRule \(\[^/\]\+\)$ /test/1.php \#for example: /test/admin =\> /test/admin.php RewriteRule \(\[^/\]\+\)\\.html /test/$1.php \[L\]

\#for example: /test/admin.html =\> /test/admin.php

參考資料：Apache Module mod\_rewrite

%E8%A7%A3%E9%87%8A%0A%0AAPACHE%EF%BC%8C%E4%BB%A5MODULE%E6%96%B9%E5%BC%8F%E8%BF%90%E8%A1%8C%E5%8A%A0%E8%BD%BD%E3%80%82%0A%0A%E5%90%AF%E5%8A%A8\-%E3%80%8B%E7%AD%89%E5%BE%85%E8%AF%B7%E6%B1%82\-%E3%80%8B%E6%8E%A5%E6%94%B6%E8%AF%B7%E6%B1%82\-%E3%80%8B%E5%88%86%E5%8F%91%E8%BF%9B%E7%A8%8B%2F%E7%BA%BF%E7%A8%8B\-%E3%80%8B%E8%AF%B7%E6%B1%82PHP\-%E3%80%8B%E6%8E%A5%E6%94%B6%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE\-%E3%80%8B%E8%BF%94%E5%9B%9E%E7%BB%99C%E7%AB%AF\-%E3%80%8B%E7%BB%A7%E7%BB%AD%E7%AD%89%E5%BE%85%E8%AF%B7%E6%B1%82%0A%0A%E5%90%AF%E5%8A%A8%EF%BC%9A%E8%AF%BB%E5%8F%96%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E5%88%9D%E5%A7%8B%E5%8C%96%E7%8E%AF%E5%A2%83%EF%BC%8C%E5%8A%A0%E8%BD%BD%E5%90%84%E7%A7%8Dmodule%EF%BC%8C%E8%BF%9B%E5%85%A5%E7%AD%89%E5%BE%85%E8%AF%B7%E6%B1%82%E9%98%B6%E6%AE%B5%E3%80%82%0A%0A%0A%E7%BC%93%E5%AD%98%0A%0Amod\_expires%0A%3CIfModule%20expires\_module%3E%20%20%0AExpiresActive%20On%20%20%0A%23%E8%AE%BF%E9%97%AE%E4%B9%8B%E5%90%8E%E7%9A%84%E4%B8%80%E4%B8%AA%E6%9C%88%E4%B8%8D%E5%86%8D%E6%9B%B4%E6%96%B0%20%20%0AExpiresDefault%20%22access%20plus%201%20month%22%20%20%20%0A%23%E8%AE%BF%E9%97%AE%E4%B9%8B%E5%90%8E%E7%9A%844%E5%91%A8%E4%B8%8D%E5%86%8D%E6%9B%B4%E6%96%B0%20%20%0A%23ExpiresDefault%20%22access%20plus%204%20weeks%22%20%20%0A%23%E8%AE%BF%E9%97%AE%E4%B9%8B%E5%90%8E%E7%9A%8430%E5%A4%A9%E4%B8%8D%E5%86%8D%E6%9B%B4%E6%96%B0%20%20%0A%23ExpiresDefault%20%22access%20plus%2030%20days%22%20%20%0A%3C%2FIfModule%3E%20%0A%0A%0Amod\_headers%0A%0A%3CIfModule%20headers\_module%3E%20%20%20%20%0A%23%20htm%2Chtml%2Ctxt%E7%B1%BB%E7%9A%84%E6%96%87%E4%BB%B6%E7%BC%93%E5%AD%98%E4%B8%80%E4%B8%AA%E5%B0%8F%E6%97%B6%20%20%20%20%0A%3Cfilesmatch%20%22%5C.\(html%7Chtm%7Ctxt\)%24%22%3E%20%20%20%20%0Aheader%20set%20cache\-control%20%22max\-age%3D3600%22%20%20%20%20%0A%3C%2Ffilesmatch%3E%20%20%20%20%0A%20%20%20%20%0A%23%20css%2C%20js%2C%20swf%E7%B1%BB%E7%9A%84%E6%96%87%E4%BB%B6%E7%BC%93%E5%AD%98%E4%B8%80%E4%B8%AA%E6%98%9F%E6%9C%9F%20%20%20%20%0A%3Cfilesmatch%20%22%5C.\(css%7Cjs%7Cswf\)%24%22%3E%20%20%20%20%0Aheader%20set%20cache\-control%20%22max\-age%3D604800%22%20%20%20%20%0A%3C%2Ffilesmatch%3E%20%20%0A%3C%2FIfModule%3E%20%0A%0A%0AHTTP%E5%A4%B4%E5%8D%8F%E8%AE%AE%E4%B8%AD%E5%87%A0%E4%B8%AA%E5%8F%82%E6%95%B0%3AExpires%E3%80%81Cache\-Control%E3%80%81Last\-Modified%E3%80%81ETag%0AExpires%EF%BC%9A%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4%0ACache\-Control%EF%BC%9A%E5%A4%B1%E6%95%88%E8%A7%84%E5%88%99%EF%BC%8Cno\-cache%2C%20must\-revalidate%2C%20max\-age%3D0%0ALast\-Modified%EF%BC%9A%E5%88%9D%E6%AC%A1%E8%AF%B7%E6%B1%82%EF%BC%8CS%E7%AB%AF%E4%BC%9A%E7%BB%99%E5%87%BALast\-Modified%E7%9A%84%E5%80%BC%EF%BC%8C%E4%BA%8C%E6%AC%A1%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%BC%9A%E9%99%84%E5%B8%A6%E8%BF%99%E4%B8%AA%E5%80%BC%EF%BC%8CS%E7%AB%AF%E6%8E%A5%E6%94%B6%E5%88%B0%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%EF%BC%8C%E5%88%A4%E6%96%AD%E5%BD%93%E5%89%8DURL%E5%AF%B9%E5%BA%94%E7%9A%84%E6%96%87%E4%BB%B6%E6%98%AF%E5%90%A6%E5%8F%91%E7%94%9F%E4%BA%86%E4%BF%AE%E6%94%B9%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89%EF%BC%8C%E6%9B%B4%E6%96%B0%E6%96%87%E4%BB%B6%EF%BC%8C%E6%9B%B4%E6%96%B0%E7%BC%93%E5%AD%98%E5%86%85%E5%AE%B9~%E8%BF%94%E5%9B%9E200%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%EF%BC%8C%E8%BF%94%E5%9B%9E304%0AETag%EF%BC%9A%E7%B1%BB%E4%BC%BCTOKEN%EF%BC%8C%E5%A2%9E%E5%8A%A0%E5%AE%89%E5%85%A8%0A%0A%0Amod\_cache%0Amod\_disk\_cache%20%EF%BC%9A%E6%96%87%E4%BB%B6%E7%BC%93%E5%AD%98%0Amod\_mem\_cache%EF%BC%9A%E5%86%85%E5%AD%98%E7%BC%93%E5%AD%98%0A%0A%E5%8E%8B%E7%BC%A9%0A%0A%E5%BC%80%E5%90%AFmod\_deflate.so%20%09mod\_headers.so%0A%E4%B8%80%E4%B8%AA%E5%8F%82%E6%95%B0%EF%BC%9AContent\-encoding%3Agzip%0A%0A%0AKeepAlive%0A%E4%BF%9D%E6%8C%81%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%BD%93%E5%9C%A8%E7%9F%AD%E6%97%B6%E9%97%B4%E5%86%85%E6%9C%89100%E4%B8%AA%E8%AF%B7%E6%B1%82%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%8F%AA%E5%BB%BA%E7%AB%8B1\-2%E4%B8%AA%E8%BF%9E%E6%8E%A5%EF%BC%8C%E6%B3%A8%E6%84%8F%E6%98%AF%E4%B8%80%E5%AE%9A%E6%97%B6%E9%97%B4%E5%86%85%E4%BF%9D%E6%8C%81%E8%BF%9E%E6%8E%A5%EF%BC%8C%E8%80%8C%E9%9D%9E%E9%95%BF%E8%BF%9E%E6%8E%A5%0A%0A%0A%E4%BD%86%E6%98%AF%EF%BC%8C%E5%BC%80%E5%90%AFKeepAlive%EF%BC%8C%E6%84%8F%E5%91%B3%E7%9D%80%E7%9F%AD%E6%97%B6%E9%97%B4%E5%86%85%E9%9C%80%E8%A6%81%E7%BB%B4%E6%8B%89%E7%9A%84%E8%BF%9E%E6%8E%A5%E5%A2%9E%E5%A4%9A%EF%BC%8C%E5%86%85%E5%AD%98%E5%A2%9E%E5%8A%A0~%E8%80%8C%E5%85%B3%E9%97%AD%E4%BA%86%EF%BC%8C%E9%9C%80%E8%A6%81%E5%A4%84%E7%90%86%E7%9A%84%E8%BF%9E%E6%8E%A5%E5%8F%98%E5%A4%9A%EF%BC%8C%E5%85%B3%E9%97%AD%2F%E5%90%AF%E5%8A%A8%20%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%A2%9E%E5%8A%A0%E4%BA%86CPU%E7%9A%84%E6%B6%88%E8%80%97%0A%0AKeepAlive%20On%0AKeepAliveTimeout%3D10%0AMaxKeepAliveRequests%0A%E4%B8%80%E6%AC%A1%E8%BF%9E%E6%8E%A5%E5%8F%AF%E4%BB%A5%E8%BF%9B%E8%A1%8C%E7%9A%84HTTP%E8%AF%B7%E6%B1%82%E7%9A%84%E6%9C%80%E5%A4%A7%E8%AF%B7%E6%B1%82%E6%AC%A1%E6%95%B0%0A%0Atimeout%20%0A%0A%E8%BF%99%E6%AC%A1%E8%AF%B7%E6%B1%82%E5%92%8C%E4%B8%8B%E6%AC%A1%E8%AF%B7%E6%B1%82%EF%BC%8C%E9%97%B4%E9%9A%94%E7%9A%84%E6%97%B6%E9%97%B4%EF%BC%8C%E4%B8%8D%E8%A6%81%E8%AF%AF%E4%BB%A5%E4%B8%BA%E6%98%AF%E8%BF%9E%E6%8E%A5%E8%B6%85%E6%97%B6~%E6%97%B6%E9%97%B4%0A%0Aexpires\_module%0A%E5%91%8A%E8%AF%89%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6\(js%20css%20%E7%AD%89\)%E7%9A%84%E6%97%B6%E9%97%B4%0A%0A%0A%0AMPM%EF%BC%88Multi\-Processing%20Module%EF%BC%8C%E5%A4%9A%E8%BF%9B%E7%A8%8B%E5%A4%84%E7%90%86%E6%A8%A1%E5%9D%97%EF%BC%89%EF%BC%9Aprefork%2C%20worker%E5%92%8Cevent%0A%0A%E7%BC%96%E8%AF%91%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%8A%A0%E5%8F%82%E6%95%B0%E9%80%89%E6%8B%A9%EF%BC%8C\-\-with\-mpm%3Dprefork%7Cworker%7Cevent%E3%80%82event%E9%9C%80%E8%A6%81OS%E6%94%AF%E6%8C%81EPOLL~apachectl%20\-V%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%9F%A5%E7%9C%8B%0A%0A%0Aprefork%0A%0A%E5%A4%9A%E8%BF%9B%E7%A8%8B%2Bselect%EF%BC%8C%E5%90%AF%E5%8A%A8%E4%B9%8B%E5%90%8E%E4%BC%9A%E9%A2%84%E5%85%88%E5%90%AF%E5%8A%A8%E4%B8%80%E4%BA%9B%E8%BF%9B%E7%A8%8B%EF%BC%8C%E6%97%A0%E9%A1%BB%E8%80%83%E8%99%91%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E9%97%AE%E9%A2%98%E3%80%82%0A%0A%E5%85%B7%E4%BD%93%E8%A7%A3%E9%87%8A%EF%BC%9A%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE%E4%B8%8E%E8%B0%83%E4%BC%98%0A%0A%3CIfModule%20mpm\_prefork\_module%3E%0AStartServers%205%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E5%90%AF%E5%8A%A8%E5%90%8E%EF%BC%8C%E6%B4%BE%E7%94%9F%E5%A4%9A%E5%B0%91%E4%B8%AA%E5%AD%90%E8%BF%9B%E7%A8%8B%0AMinSpareServers%205%20%20%20%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%B0%8F%E7%A9%BA%E9%97%B2%E8%BF%9B%E7%A8%8B%E6%95%B0%0AMaxSpareServers%2010%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E7%A9%BA%E9%97%B2%E8%BF%9B%E7%A8%8B%E6%95%B0%0AMaxRequestWorkers%20250%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E5%8F%AF%E6%8E%A5%E6%94%B6%E8%BF%9E%E6%8E%A5%E6%95%B0%0AMaxConnectionsPerChild%200%20%20%2F%2F%E5%8D%95%E4%B8%AA%E8%BF%9B%E7%A8%8B%E6%9C%80%E5%A4%A7%E5%8F%AF%E6%8F%90%E4%BE%9B%E5%A4%9A%E5%B0%91%E4%B8%AA%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%3C%2FIfModule%3E%0A%0A%20%E5%90%AF%E5%8A%A8%E6%97%B6%E5%BB%BA%E7%AB%8BStartServers%E4%B8%AA%E5%AD%90%E8%BF%9B%E7%A8%8B%EF%BC%8C%E7%84%B6%E5%90%8E%E6%8C%89%E6%AF%8F%E7%A7%92%E5%88%9B%E5%BB%BA%E6%8C%87%E6%95%B0%E7%BA%A7%E4%B8%AA%E8%BF%9B%E7%A8%8B%E7%9B%B4%E5%88%B0%E8%BE%BE%E5%88%B0MinSpareServers%E4%B8%AA%E8%BF%9B%E7%A8%8B%EF%BC%88%E6%9C%80%E5%A4%9A%E5%A2%9E%E5%88%B0%E6%AF%8F%E7%A7%9232%E4%B8%AA%EF%BC%89%E3%80%82MinSpareServers%EF%BC%8C%E8%AE%BE%E7%BD%AE%E7%A9%BA%E9%97%B2%E5%AD%90%E8%BF%9B%E7%A8%8B%E7%9A%84%E6%9C%80%E5%A4%A7%E6%95%B0%E9%87%8F%EF%BC%8C%E9%BB%98%E8%AE%A4%E4%B8%BA10%E3%80%82%E5%A6%82%E6%9E%9C%E5%BD%93%E5%89%8D%E6%9C%89%E8%B6%85%E8%BF%87MaxSpareServers%E6%95%B0%E9%87%8F%E7%9A%84%E7%A9%BA%E9%97%B2%E5%AD%90%E8%BF%9B%E7%A8%8B%EF%BC%8C%E9%82%A3%E4%B9%88%E7%88%B6%E8%BF%9B%E7%A8%8B%E5%B0%86%E6%9D%80%E6%AD%BB%E5%A4%9A%E4%BD%99%E7%9A%84%E5%AD%90%E8%BF%9B%E7%A8%8B%E3%80%82%E6%AD%A4%E5%8F%82%E6%95%B0%E4%B8%8D%E8%A6%81%E8%AE%BE%E7%9A%84%E5%A4%AA%E5%A4%A7%E3%80%82%E5%A6%82%E6%9E%9C%E4%BD%A0%E5%B0%86%E8%AF%A5%E6%8C%87%E4%BB%A4%E7%9A%84%E5%80%BC%E8%AE%BE%E7%BD%AE%E4%B8%BA%E6%AF%94MinSpareServers%E5%B0%8F%EF%BC%8CApache%E5%B0%86%E4%BC%9A%E8%87%AA%E5%8A%A8%E5%B0%86%E5%85%B6%E4%BF%AE%E6%88%90%22MinSpareServers%2B1%22%E3%80%82%0A%0AMaxRequestPerChild%EF%BC%9A%E6%8C%87%E5%AE%9A%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%A4%84%E7%90%86%E4%BA%86%E5%A4%9A%E5%B0%91%E4%B8%AA%E8%AF%B7%E6%B1%82%E5%90%8E%E5%B0%B1%E8%87%AA%E6%88%91%E6%AF%81%E7%81%AD%E3%80%82%E6%AD%A4%E5%8F%82%E6%95%B0%E5%A5%BD%E5%83%8F%E6%B2%A1%E7%94%A8%E4%BA%86.....%0AMaxRequestWorkers%20%EF%BC%9A%E6%9C%80%E5%A4%A7%E6%8E%A5%E6%94%B6%E8%BF%9E%E6%8E%A5%E6%95%B0%EF%BC%8C%E8%B6%85%E8%BF%87%E5%B0%86%E6%8E%92%E9%98%9F%0A%0A%E4%BC%98%E7%82%B9%E6%98%AF%E6%88%90%E7%86%9F%E7%A8%B3%E5%AE%9A%EF%BC%8C%E7%BC%BA%E7%82%B9%E6%98%AF%E6%AF%94%E8%BE%83%E6%B6%88%E8%80%97%E5%86%85%E5%AD%98%EF%BC%8C%E8%80%8C%E4%B8%94%E5%B9%B6%E5%8F%91%E6%94%AF%E6%8C%81%E5%8F%97%E9%99%90%E4%BA%8E%E8%BF%9B%E7%A8%8B%E6%95%B0%E9%87%8F%EF%BC%8C%E5%AF%B9%E9%AB%98%E5%B9%B6%E5%8F%91%E6%94%AF%E6%8C%81%E7%A8%8D%E5%B7%AE%E3%80%82%0A%0A%0Aworker%E6%A8%A1%E5%BC%8F%0A%0A%E5%A4%9A%E8%BF%9B%E7%A8%8B%2B%E5%A4%9A%E7%BA%BF%E7%A8%8B%2BSELECT%E3%80%82%E9%A2%84%E5%85%88%E6%B4%BE%E7%94%9F%E5%87%BA%E8%8B%A5%E5%B9%B2%E8%BF%9B%E7%A8%8B%EF%BC%88%E5%B0%91%E9%87%8F%EF%BC%89%EF%BC%8C%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%86%8D%E6%B4%BE%E7%94%9F%E5%87%BA%E8%8B%A5%E5%B9%B2%E7%BA%BF%E7%A8%8B%EF%BC%8C%E5%90%8C%E6%97%B6%E5%8C%85%E6%8B%AC%E4%B8%80%E4%B8%AA%E7%9B%91%E5%90%AC%E7%BA%BF%E7%A8%8B%E3%80%82%0A%0A%E5%85%B7%E4%BD%93%E8%A7%A3%E9%87%8A%EF%BC%9A%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE%E4%B8%8E%E8%B0%83%E4%BC%98%0A%0A%3CIfModule%20mpm\_worker\_module%3E%0A%20%20%20%20StartServers%20%20%20%20%20%20%20%20%20%20%20%20%203%20%20%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E5%90%AF%E5%8A%A8%E5%90%8E%EF%BC%8C%E6%B4%BE%E7%94%9F%E5%A4%9A%E5%B0%91%E4%B8%AA%E5%AD%90%E8%BF%9B%E7%A8%8B%0A%20%20%20%20MinSpareThreads%20%20%20%20%20%20%20%20%2075%20%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%B0%8F%E7%A9%BA%E9%97%B2%E7%BA%BF%E7%A8%8B%E6%95%B0%0A%20%20%20%20MaxSpareThreads%20%20%20%20%20%20%20%20250%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E7%A9%BA%E9%97%B2%E7%BA%BF%E7%A8%8B%E6%95%B0%0A%20%20%20%20ThreadsPerChild%20%20%20%20%20%20%20%20%2025%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%8C%85%E5%90%AB%E5%A4%9A%E5%B0%91%E4%B8%AA%E7%BA%BF%E7%A8%8B%0A%20%20%20%20MaxRequestWorkers%20%20%20%20%20%20400%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E5%8F%AF%E6%8E%A5%E6%94%B6%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%20%20%20%20MaxConnectionsPerChild%20%20%200%20%20%20%20%20%20%2F%2F%E6%AF%8F%E4%B8%AA%E7%BA%BF%E7%A8%8B%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%3C%2FIfModule%3E%0A%0A%0A%E5%A4%A7%E9%83%A8%E5%88%86%E5%8F%82%E6%95%B0%E8%B7%9F%E4%B8%8A%E9%9D%A2%E5%B7%AE%E4%B8%8D%E5%A4%9A%0A%0A%0A%E4%BC%98%E7%82%B9%E6%98%AF%EF%BC%8C%E7%BA%BF%E7%A8%8B%E5%8F%AF%E5%85%B1%E4%BA%AB%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%86%85%E5%AD%98%E7%A9%BA%E9%97%B4%EF%BC%8C%E5%8D%A0%E7%94%A8%E5%86%85%E5%AD%98%E5%B0%91%EF%BC%8C%E5%B9%B6%E5%8F%91%E6%80%A7%E6%AF%94%E8%BE%83%E9%AB%98%0A%0A%E7%BC%BA%E7%82%B9%E6%98%AF%E5%BF%85%E9%A1%BB%E8%80%83%E8%99%91%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E3%80%82%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8%E4%BA%86keep\-alive%E6%96%B9%E5%BC%8F%EF%BC%8C%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%8F%AF%E8%83%BD%E4%BC%9A%E8%A2%AB%E4%B8%80%E7%9B%B4%E4%BF%9D%E6%8C%81%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%EF%BC%8C%E4%BD%86%E4%B8%AD%E9%97%B4%E6%B2%A1%E6%9C%89%E8%AF%B7%E6%B1%82%EF%BC%8C%E7%9B%B4%E5%88%B0%E8%B6%85%E6%97%B6%E3%80%82%E5%A6%82%E6%9E%9C%E6%9C%89%E5%A4%9A%E4%B8%AA%E7%BA%BF%E7%A8%8B%E8%A2%AB%E8%BF%99%E6%A0%B7%E5%8D%A0%E6%8D%AE%EF%BC%8C%E5%9C%A8%E9%AB%98%E5%B9%B6%E5%8F%91%E5%9C%BA%E6%99%AF%E4%B8%8B%E5%90%8C%E6%A0%B7%E4%BC%9A%E5%87%BA%E7%8E%B0%E6%97%A0%E7%BA%BF%E7%A8%8B%E5%8F%AF%E7%94%A8%E7%9A%84%E6%83%85%E5%BD%A2%E3%80%82%0A%0A%0A%0Aevent%E6%A8%A1%E5%BC%8F%0A%0A%E5%A4%9A%E8%BF%9B%E7%A8%8B%2B%E5%A4%9A%E7%BA%BF%E7%A8%8B%2Bepoll%E3%80%82%E6%98%AF%E5%9C%A82.4%E7%89%88%E6%9C%AC%E4%B8%AD%E6%89%8D%E7%A8%B3%E5%AE%9A%E5%8F%91%E5%B8%83%E7%9A%84%E6%A8%A1%E5%BC%8F%EF%BC%8C%E5%AE%83%E5%9C%A8worker%E7%9A%84%E5%9F%BA%E7%A1%80%E4%B8%8A%EF%BC%8C%E8%A7%A3%E5%86%B3%E4%BA%86keep\-alive%E8%BF%9E%E6%8E%A5%E4%B8%8D%E8%83%BD%E9%87%8A%E6%94%BE%E7%9A%84%E9%97%AE%E9%A2%98%E3%80%82event%20MPM%E4%B8%AD%EF%BC%8C%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%E4%B8%93%E9%97%A8%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%9D%A5%E7%AE%A1%E7%90%86%E8%BF%99%E4%BA%9Bkeep\-alive%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%BA%BF%E7%A8%8B%EF%BC%8C%E5%BD%93%E6%9C%89%E7%9C%9F%E5%AE%9E%E8%AF%B7%E6%B1%82%E8%BF%87%E6%9D%A5%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%B0%86%E8%AF%B7%E6%B1%82%E4%BC%A0%E9%80%92%E7%BB%99%E6%9C%8D%E5%8A%A1%E7%BA%BF%E7%A8%8B%EF%BC%8C%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%AF%95%E5%90%8E%EF%BC%8C%E5%8F%88%E5%85%81%E8%AE%B8%E5%AE%83%E9%87%8A%E6%94%BE%E3%80%82%E8%BF%99%E6%A0%B7%E5%A2%9E%E5%BC%BA%E4%BA%86%E9%AB%98%E5%B9%B6%E5%8F%91%E5%9C%BA%E6%99%AF%E4%B8%8B%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E8%83%BD%E5%8A%9B%E3%80%82epoll%E6%A8%A1%E5%BC%8F%0A%0A%0A%E5%85%B7%E4%BD%93%E8%A7%A3%E9%87%8A%EF%BC%9A%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE%E4%B8%8E%E8%B0%83%E4%BC%98%0A%0A%3CIfModule%20mpm\_worker\_module%3E%0A%20%20%20%20StartServers%20%20%20%20%20%20%20%20%20%20%20%20%203%20%20%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E5%90%AF%E5%8A%A8%E5%90%8E%EF%BC%8C%E6%B4%BE%E7%94%9F%E5%A4%9A%E5%B0%91%E4%B8%AA%E5%AD%90%E8%BF%9B%E7%A8%8B%0A%20%20%20%20MinSpareThreads%20%20%20%20%20%20%20%20%2075%20%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%B0%8F%E7%A9%BA%E9%97%B2%E7%BA%BF%E7%A8%8B%E6%95%B0%0A%20%20%20%20MaxSpareThreads%20%20%20%20%20%20%20%20250%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E7%A9%BA%E9%97%B2%E7%BA%BF%E7%A8%8B%E6%95%B0%0A%20%20%20%20ThreadsPerChild%20%20%20%20%20%20%20%20%2025%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%8C%85%E5%90%AB%E5%A4%9A%E5%B0%91%E4%B8%AA%E7%BA%BF%E7%A8%8B%0A%20%20%20%20MaxRequestWorkers%20%20%20%20%20%20400%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E5%8F%AF%E6%8E%A5%E6%94%B6%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%20%20%20%20MaxConnectionsPerChild%20%20%200%20%20%20%20%20%20%2F%2F%E6%AF%8F%E4%B8%AA%E7%BA%BF%E7%A8%8B%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%3C%2FIfModule%3E%0A%0A%0A%E5%A4%A7%E9%83%A8%E5%88%86%E5%8F%82%E6%95%B0%E8%B7%9F%E4%B8%8A%E9%9D%A2%E5%B7%AE%E4%B8%8D%E5%A4%9A%0A%0A%0A%0A%0A%0A%0A%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%95%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6%0A%3CDirectory%20%2F%3E%0A%20%20%20%20%20%23%E6%8C%87%E5%AE%9A%E6%A0%B9%E7%9B%AE%E5%BD%95%22%2F%22%E5%90%AF%E7%94%A8Indexes%E3%80%81FollowSymLinks%E4%B8%A4%E7%A7%8D%E7%89%B9%E6%80%A7%E3%80%82%0A%20%20%20%20Options%20Indexes%20FollowSymLinks%0A%20%20%20%20AllowOverride%20all%0A%20%20%20%20Order%20allow%2Cdeny%0A%20%20%20%20Allow%20from%20all%0A%3C%2FDirectory%3E%0A%0A%0AAllowOverride%EF%BC%9A.htaccess%0AOptions%20%0AAll%EF%BC%9A%E8%A1%A8%E7%A4%BA%E9%99%A4MultiViews%E4%B9%8B%E5%A4%96%E7%9A%84%E6%89%80%E6%9C%89%E7%89%B9%E6%80%A7%E3%80%82%E8%BF%99%E4%B9%9F%E6%98%AFOptions%E6%8C%87%E4%BB%A4%E7%9A%84%E9%BB%98%E8%AE%A4%E8%AE%BE%E7%BD%AE%E3%80%82%0ANone%0AFollowSymLinks%EF%BC%9A%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%85%81%E8%AE%B8%E5%9C%A8%E6%AD%A4%E7%9B%AE%E5%BD%95%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%AC%A6%E5%8F%B7%E8%BF%9E%E6%8E%A5%E3%80%82%E5%A6%82%E6%9E%9C%E8%AF%A5%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9%E4%BD%8D%E4%BA%8E%3CLocation%3E%E9%85%8D%E7%BD%AE%E6%AE%B5%E4%B8%AD%EF%BC%8C%E5%B0%86%E4%BC%9A%E8%A2%AB%E5%BF%BD%E7%95%A5%E3%80%82%0AIndexes%EF%BC%9A%E5%A6%82%E6%9E%9C%E6%98%AF%E7%9B%AE%E5%BD%95%EF%BC%8C%E5%88%99%E6%98%BE%E7%A4%BA%E4%B8%8B%E9%9D%A2%E6%89%80%E6%9C%89%E6%96%87%E4%BB%B6%E5%88%97%E8%A1%A8%0AMultiViews%0ASymLinksIfOwnerMatch%0AExecCGI%0AIncludes%0AIncludesNOEXEC%0A%0A%0AORDER%E4%B8%8EAllow%20%20DENY%20%E9%85%8D%E5%90%88%E4%BD%BF%E7%94%A8%0AAllow%20%20%EF%BC%9A%E5%85%81%E8%AE%B8%E8%AE%BF%E9%97%AE%E7%9A%84IP%0ADENY%20%EF%BC%9A%E7%A6%81%E6%AD%A2%E8%AE%BF%E9%97%AE%E7%9A%84IP%0A%0A%0AREWRITE%0A%0A%0A.%20%E5%8C%B9%E9%85%8D%E4%BB%BB%E4%BD%95%E5%96%AE%E5%AD%97%E7%AC%A6%0A%5Bchars%5D%20%E5%8C%B9%E9%85%8D%E5%AD%97%E7%AC%A6%E4%B8%B2%3Achars%0A%5B%5Echars%5D%20%E4%B8%8D%E5%8C%B9%E9%85%8D%E5%AD%97%E7%AC%A6%E4%B8%B2%3Achars%0Atext1%7Ctext2%20%E5%8F%AF%E9%81%B8%E6%93%87%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%3Atext1%E6%88%96text2%0A%3F%20%E5%8C%B9%E9%85%8D0%E5%88%B01%E5%80%8B%E5%AD%97%E7%AC%A6%0A\*%20%E5%8C%B9%E9%85%8D0%E5%88%B0%E5%A4%9A%E5%80%8B%E5%AD%97%E7%AC%A6%0A%2B%20%E5%8C%B9%E9%85%8D1%E5%88%B0%E5%A4%9A%E5%80%8B%E5%AD%97%E7%AC%A6%0A%5E%20%E5%AD%97%E7%AC%A6%E4%B8%B2%E9%96%8B%E5%A7%8B%E6%A8%99%E8%AA%8C%0A%24%20%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B5%90%E6%9D%9F%E6%A8%99%E8%AA%8C%0A%5Cn%20%E8%BD%89%E7%BE%A9%E7%AC%A6%E6%A8%99%E8%AA%8C%0A%0A%E5%8F%8D%E5%90%91%E5%BC%95%E7%94%A8%20%24N%20%E7%94%A8%E6%96%BC%20RewriteRule%20%E4%B8%AD%E5%8C%B9%E9%85%8D%E7%9A%84%E8%AE%8A%E9%87%8F%E8%AA%BF%E7%94%A8\(0%20%3C%3D%20N%20%3C%3D%209\)%0A%E5%8F%8D%E5%90%91%E5%BC%95%E7%94%A8%20%25N%20%E7%94%A8%E6%96%BC%20RewriteCond%20%E4%B8%AD%E6%9C%80%E5%BE%8C%E4%B8%80%E5%80%8B%E5%8C%B9%E9%85%8D%E7%9A%84%E8%AE%8A%E9%87%8F%E8%AA%BF%E7%94%A8\(1%20%3C%3D%20N%20%3C%3D%209\)%0A%0ARewriteCond%E9%81%A9%E7%94%A8%E7%9A%84%E6%A8%99%E8%AA%8C%E7%AC%A6%EF%BC%9A%0Anocase%7CNC'%20\(no%20case\)%E5%BF%BD%E7%95%A5%E5%A4%A7%E5%B0%8F%0Aornext%7COR'%20\(or%20next%20condition\)%E9%82%8F%E8%BC%AF%E6%88%96%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%90%8C%E6%99%82%E5%8C%B9%E9%85%8D%E5%A4%9A%E5%80%8BRewriteCond%E6%A2%9D%E4%BB%B6%0A%0ARewriteRule%E9%81%A9%E7%94%A8%E7%9A%84%E6%A8%99%E8%AA%8C%E7%AC%A6%EF%BC%9A%0Aredirect%7CR%20%5B%3Dcode%5D%20\(force%20redirect\)%E5%BC%B7%E8%BF%AB%E9%87%8D%E5%AF%AB%E7%82%BA%E5%9F%BA%E6%96%BChttp%E9%96%8B%E9%A0%AD%E7%9A%84%E5%A4%96%E9%83%A8%E8%BD%89%E5%90%91\(%E6%B3%A8%E6%84%8FURL%E7%9A%84%E8%AE%8A%E5%8C%96\)%20%E5%A6%82%EF%BC%9A%5BR%3D301%2CL%5D%0Aforbidden%7CF'%20\(force%20URL%20to%20be%20forbidden\)%E9%87%8D%E5%AF%AB%E7%82%BA%E7%A6%81%E6%AD%A2%E8%A8%AA%E5%95%8F%0Aproxy%7CP'%20\(force%20proxy\)%E9%87%8D%E5%AF%AB%E7%82%BA%E9%80%9A%E9%81%8E%E4%BB%A3%E7%90%86%E8%A8%AA%E5%95%8F%E7%9A%84http%E8%B7%AF%E5%BE%91%0Alast%7CL'%20\(last%20rule\)%E6%9C%80%E5%BE%8C%E7%9A%84%E9%87%8D%E5%AF%AB%E8%A6%8F%E5%89%87%E6%A8%99%E8%AA%8C%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%8C%B9%E9%85%8D%EF%BC%8C%E4%B8%8D%E5%86%8D%E5%9F%B7%E8%A1%8C%E4%BB%A5%E5%BE%8C%E7%9A%84%E8%A6%8F%E5%89%87%0Anext%7CN'%20\(next%20round\)%E5%BE%AA%E7%92%B0%E5%90%8C%E4%B8%80%E5%80%8B%E8%A6%8F%E5%89%87%EF%BC%8C%E7%9B%B4%E5%88%B0%E4%B8%8D%E8%83%BD%E6%BB%BF%E8%B6%B3%E5%8C%B9%E9%85%8D%0Achain%7CC'%20\(chained%20with%20next%20rule\)%E5%A6%82%E6%9E%9C%E5%8C%B9%E9%85%8D%E8%A9%B2%E8%A6%8F%E5%89%87%EF%BC%8C%E5%89%87%E7%B9%BC%E7%BA%8C%E4%B8%8B%E9%9D%A2%E7%9A%84%E6%9C%89Chain%E6%A8%99%E8%AA%8C%E7%9A%84%E8%A6%8F%E5%89%87%E3%80%82%0Atype%7CT%3DMIME\-type'%20\(force%20MIME%20type\)%E6%8C%87%E5%AE%9AMIME%E9%A1%9E%E5%9E%8B%0Anosubreq%7CNS'%20\(used%20only%20if%20no%20internal%20sub\-request\)%E5%A6%82%E6%9E%9C%E6%98%AF%E5%85%A7%E9%83%A8%E5%AD%90%E8%AB%8B%E6%B1%82%E5%89%87%E8%B7%B3%E9%81%8E%0Anocase%7CNC'%20\(no%20case\)%E5%BF%BD%E7%95%A5%E5%A4%A7%E5%B0%8F%0Aqsappend%7CQSA'%20\(query%20string%20append\)%E9%99%84%E5%8A%A0%E6%9F%A5%E8%A9%A2%E5%AD%97%E7%AC%A6%E4%B8%B2%0Anoescape%7CNE'%20\(no%20URI%20escaping%20of%20output\)%E7%A6%81%E6%AD%A2URL%E4%B8%AD%E7%9A%84%E5%AD%97%E7%AC%A6%E8%87%AA%E5%8B%95%E8%BD%89%E7%BE%A9%E6%88%90%25%5B0\-9%5D%2B%E7%9A%84%E5%BD%A2%E5%BC%8F%E3%80%82%0Apassthrough%7CPT'%20\(pass%20through%20to%20next%20handler\)%E5%B0%87%E9%87%8D%E5%AF%AB%E7%B5%90%E6%9E%9C%E9%81%8B%E7%94%A8%E6%96%BCmod\_alias%0Askip%7CS%3Dnum'%20\(skip%20next%20rule\(s\)\)%E8%B7%B3%E9%81%8E%E4%B8%8B%E9%9D%A2%E5%B9%BE%E5%80%8B%E8%A6%8F%E5%89%87%0Aenv%7CE%3DVAR%3AVAL'%20\(set%20environment%20variable\)%E6%B7%BB%E5%8A%A0%E7%92%B0%E5%A2%83%E8%AE%8A%E9%87%8F%0A%0Aexample%EF%BC%9A%0ARewriteEngine%20on%0ARewriteCond%20%25%7BHTTP\_USER\_AGENT%7D%20%5EMSIE%20%5BNC%2COR%5D%0ARewriteCond%20%25%7BHTTP\_USER\_AGENT%7D%20%5EOpera%20%5BNC%5D%0ARewriteRule%20%5E.\*%20\-%20%5BF%2CL%5D%20%E9%80%99%E8%A3%A1%E3%80%8D\-%E3%80%8D%E8%A1%A8%E7%A4%BA%E6%B2%92%E6%9C%89%E6%9B%BF%E6%8F%9B%EF%BC%8C%E7%80%8F%E8%A6%BD%E5%99%A8%E7%82%BAIE%E5%92%8COpera%E7%9A%84%E8%A8%AA%E5%AE%A2%E5%B0%87%E8%A2%AB%E7%A6%81%E6%AD%A2%E8%A8%AA%E5%95%8F%E3%80%82%0A%0Aexample%EF%BC%9A%0ARewriteEngine%20On%0ARewriteBase%20%2Ftest%0ARewriteCond%20%25%7BREQUEST\_FILENAME%7D.php%20\-f%0ARewriteRule%20\(%5B%5E%2F%5D%2B\)%24%20%2Ftest%2F%241.php%0A%23for%20example%3A%20%2Ftest%2Fadmin%20%3D%3E%20%2Ftest%2Fadmin.php%0ARewriteRule%20\(%5B%5E%2F%5D%2B\)%5C.html%24%20%2Ftest%2F%241.php%20%5BL%5D%0A%23for%20example%3A%20%2Ftest%2Fadmin.html%20%3D%3E%20%2Ftest%2Fadmin.php%0A%E5%8F%83%E8%80%83%E8%B3%87%E6%96%99%EF%BC%9AApache%20Module%20mod\_rewrite
