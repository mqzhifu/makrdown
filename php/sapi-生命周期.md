
Server Application Programming Interface

PHP 是脚本语言，弱类型语言，更像是一个嵌入型的语言，至少早期是这样。
也因为小巧才被广泛使用

更多的时候，它需要配置 web-service 来使用，主要是做一些逻辑处理。

以 apache 为例，它是嵌入到 apache 里的。
以 nginx 为例，它相对更像是一门正常的语言，可以启动进程，使用 fastCgi 与其通信。

不管什么方式，一次请求至少有2个阶段，一次启动至少有2个阶段：

| 成员变量 | 描述 |
| :--- | :--- |
| PHP_MINIT | 模块初始化 |
| PHP_MSHUTDOWN | 模块结束 |
| PHP_RINIT | PHP 初始化 |
| PHP_RSHUTDOWN | PHP 结束 |
|  |  |
|  |  |

模块初始化阶段（MINIT），只执行一次

关闭模块 MSHUTDOWN

一次用户请求到来

请求开始 RINIT

请求结束 RSHUTDOWN

MINIT：APACHE在第一启动后，会再关闭模块，用于检查模块的配置文件，这个阶段模块可以自定义一些宏变量，同时会常驻内存

初始化若干全局变量、初始化若干常量、始化Zend引擎和核心组件、解析php.ini、全局操作函数的初始化、初始化静态构建的模块和共享模块\(MINIT\)、禁用函数和类、ACTIVATION、激活SAPI、环境初始化、模块请求初始化

PHP\_MINIT\_FUNCTION 初始化 module 时运行

PHP\_MSHUTDOWN\_FUNCTION 当module被卸载时运行

PHP\_RINIT\_FUNCTION 当一个REQUEST请求初始化时运行

PHP\_RSHUTDOWN\_FUNCTION 当一个REQUEST请求结束时运行

PHP\_MINFO\_FUNCTION 这个是设置phpinfo中这个模块的信息

PHP\_GINIT\_FUNCTION 初始化全局变量时

PHP\_GSHUTDOWN\_FUNCTION 释放全局变量时

# sapi

当PHP 被嵌入到APACHE中后，其中它就是一个完整的模块，像是一个图灵机：只有一个输入  和  输出。具体输入的参数 和 输出的参数规则，就是 sapi 

有了 sapi ，间接等于PHP更加通用化，可以适用很多场景，如：APACHE NGINX