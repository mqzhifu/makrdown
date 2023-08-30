# sapi

Server Application Programming Interface

概念：PHP以模块方式注入，如APACHE，有两次开始/结束，1、是APACHE启动初始化模块的时候，连带初始化PHP，当APACHE结束后，也连带执行PHP结束函数。2、APACHE启动后，当有一个请求过来，这是一次请求，一次请求被PHP处理结束后，这是一次请求结束。

请求前/启动

模块初始化阶段（MINIT），只执行一次

关闭模块 MSHUTDOWN

一次用户请求到来

请求开始 RINIT

请求结束 RSHUTDOWN

MINIT：APACHE在第一启动后，会再关闭模块，用于检查模块的配置文件，这个阶段模块可以自定义一些宏变量，同时会常驻内存

初始化若干全局变量、初始化若干常量、始化Zend引擎和核心组件、解析php.ini、全局操作函数的初始化、初始化静态构建的模块和共享模块\(MINIT\)、禁用函数和类、ACTIVATION、激活SAPI、环境初始化、模块请求初始化

PHP\_MINIT\_FUNCTION 初始化module时运行

PHP\_MSHUTDOWN\_FUNCTION 当module被卸载时运行

PHP\_RINIT\_FUNCTION 当一个REQUEST请求初始化时运行

PHP\_RSHUTDOWN\_FUNCTION 当一个REQUEST请求结束时运行

PHP\_MINFO\_FUNCTION 这个是设置phpinfo中这个模块的信息

PHP\_GINIT\_FUNCTION 初始化全局变量时

PHP\_GSHUTDOWN\_FUNCTION 释放全局变量时
