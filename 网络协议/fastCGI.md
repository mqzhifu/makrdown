# fastCGI

CGI全称"通用网关接口"（Common Gateway Interface），用于HTTP服务器与其它机器上的程序服务通信交流的一种工具，CGI程序须运行在网络服务器上。

FastCGI是一个可伸缩地、高速地在HTTP服务器和动态脚本语言间通信的接口（FastCGI接口在Linux下是socket（可以是文件socket，也可以是ip socket）），主要优点是把动态语言和HTTP服务器分离开来。多数流行的HTTP服务器都支持FastCGI，包括Apache、Nginx和lightpd。

