


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
