# mac dir 说明

/Application

/Library/Application

/System/Applications

/System/Volumes/Data/Applications

/Users/wangdongyan/Applications

/Users/wangdongyan/Library/Application Support

/Volumes

## unix/linux 传统目录

/bin :日常操作系统的命令 ls cd rm

/sbin:日常管理系统的命令 fdisk ifconfig ping

/usr:3方类库

bin:后期安装的一些可执行脚本命令

sbin:后期安装的一些可执行脚本命令

local/bin:用户自己编译的软件

/etc:所有配置文件

/dev:硬件设备，如：硬盘驱动

/tmp:临时文件

/var:可能产生变化的东西，如：日志文件

## mac os

```
Applications 应用程序目录，默认所有的GUI应用程序都安装在这里；
Library 系统的数据文件、帮助文件、文档等等；
    Application
    Application Support:三方应用插件
    Caches
    Documentation
    Fonts
    
Network 网络节点存放目录，不过我的电脑里没这个目录呢
System 他只包含一个名为Library的目录，这个子目录中存放了系统的绝大部分组件，如各种framework，以及内核模块，字体文件等等。
    Library
        cache:好大....
        fonts:
        Sounds:
        desktop pic:系统自带的桌面图片
        openssl
    Applications:系统自带的软件，如：app store、books、Calculator、music、photo 等
    
Users 存放用户的个人资料和配置。每个用户有自己的单独目录。等同于linux下/home
    wangdongyan
        Applications
        Desktop
        Documents
        Downloads
        Library
            Application Support:三方应用插件
        Movies
        Music
        Pictures
        Public 你可以把需要与其它用户共享的文件放在这个目录中，默认状态下，这个目录可以被其它所有用户访问
        各种软件的配置文件、缓存文件
        
Volumes 文件系统挂载点存放目录。有时候安装一些非APP Store下载的包，首次会挂载到这里
cores 内核转储文件存放目录。当一个进程崩溃时，如果系统允许则会产生转储文件。
private 里面的子目录存放了/tmp, /var, /etc等链接目录的目标目录。

```

%2FApplication%20%0A%2FLibrary%2FApplication%20%0A%2FSystem%2FApplications%20%0A%2FSystem%2FVolumes%2FData%2FApplications%0A%2FUsers%2Fwangdongyan%2FApplications%0A%2FUsers%2Fwangdongyan%2FLibrary%2FApplication%20Support%0A%2FVolumes%0A%0Acmd%20%2B%20shift%20%2B%20.%20%20%3A%E6%89%93%E5%BC%80%E9%9A%90%E8%97%8F%E6%96%87%E4%BB%B6%E5%A4%B9%0A%0Aunix%2Flinux%20%E4%BC%A0%E7%BB%9F%E7%9B%AE%E5%BD%95%0A\-\-\-\-\-\-\-\-\-\-\-\-%0A%2Fbin%20%3A%E6%97%A5%E5%B8%B8%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%91%BD%E4%BB%A4%20ls%20cd%20rm%20%20%20%0A%2Fsbin%3A%E6%97%A5%E5%B8%B8%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%91%BD%E4%BB%A4%20fdisk%20ifconfig%20ping%20%20%0A%2Fusr%3A3%E6%96%B9%E7%B1%BB%E5%BA%93%20%20%0A%20%20%20%20bin%3A%E5%90%8E%E6%9C%9F%E5%AE%89%E8%A3%85%E7%9A%84%E4%B8%80%E4%BA%9B%E5%8F%AF%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E5%91%BD%E4%BB%A4%0A%20%20%20%20sbin%3A%E5%90%8E%E6%9C%9F%E5%AE%89%E8%A3%85%E7%9A%84%E4%B8%80%E4%BA%9B%E5%8F%AF%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E5%91%BD%E4%BB%A4%0A%20%20%20%20local%2Fbin%3A%E7%94%A8%E6%88%B7%E8%87%AA%E5%B7%B1%E7%BC%96%E8%AF%91%E7%9A%84%E8%BD%AF%E4%BB%B6%0A%2Fetc%3A%E6%89%80%E6%9C%89%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%2Fdev%3A%E7%A1%AC%E4%BB%B6%E8%AE%BE%E5%A4%87%EF%BC%8C%E5%A6%82%EF%BC%9A%E7%A1%AC%E7%9B%98%E9%A9%B1%E5%8A%A8%0A%2Ftmp%3A%E4%B8%B4%E6%97%B6%E6%96%87%E4%BB%B6%0A%2Fvar%3A%E5%8F%AF%E8%83%BD%E4%BA%A7%E7%94%9F%E5%8F%98%E5%8C%96%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%A6%82%EF%BC%9A%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%0A%0Amac%20os%20%0A\-\-\-\-\-\-%0A%60%60%60%0AApplications%20%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E7%9B%AE%E5%BD%95%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%89%80%E6%9C%89%E7%9A%84GUI%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E9%83%BD%E5%AE%89%E8%A3%85%E5%9C%A8%E8%BF%99%E9%87%8C%EF%BC%9B%0ALibrary%20%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E3%80%81%E5%B8%AE%E5%8A%A9%E6%96%87%E4%BB%B6%E3%80%81%E6%96%87%E6%A1%A3%E7%AD%89%E7%AD%89%EF%BC%9B%0A%20%20%20%20Application%0A%20%20%20%20Application%20Support%3A%E4%B8%89%E6%96%B9%E5%BA%94%E7%94%A8%E6%8F%92%E4%BB%B6%0A%20%20%20%20Caches%0A%20%20%20%20Documentation%0A%20%20%20%20Fonts%0A%20%20%20%20%0ANetwork%20%E7%BD%91%E7%BB%9C%E8%8A%82%E7%82%B9%E5%AD%98%E6%94%BE%E7%9B%AE%E5%BD%95%EF%BC%8C%E4%B8%8D%E8%BF%87%E6%88%91%E7%9A%84%E7%94%B5%E8%84%91%E9%87%8C%E6%B2%A1%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E5%91%A2%0ASystem%20%E4%BB%96%E5%8F%AA%E5%8C%85%E5%90%AB%E4%B8%80%E4%B8%AA%E5%90%8D%E4%B8%BALibrary%E7%9A%84%E7%9B%AE%E5%BD%95%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%AD%90%E7%9B%AE%E5%BD%95%E4%B8%AD%E5%AD%98%E6%94%BE%E4%BA%86%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%BB%9D%E5%A4%A7%E9%83%A8%E5%88%86%E7%BB%84%E4%BB%B6%EF%BC%8C%E5%A6%82%E5%90%84%E7%A7%8Dframework%EF%BC%8C%E4%BB%A5%E5%8F%8A%E5%86%85%E6%A0%B8%E6%A8%A1%E5%9D%97%EF%BC%8C%E5%AD%97%E4%BD%93%E6%96%87%E4%BB%B6%E7%AD%89%E7%AD%89%E3%80%82%0A%20%20%20%20Library%0A%20%20%20%20%20%20%20%20cache%3A%E5%A5%BD%E5%A4%A7....%0A%20%20%20%20%20%20%20%20fonts%3A%0A%20%20%20%20%20%20%20%20Sounds%3A%0A%20%20%20%20%20%20%20%20desktop%20pic%3A%E7%B3%BB%E7%BB%9F%E8%87%AA%E5%B8%A6%E7%9A%84%E6%A1%8C%E9%9D%A2%E5%9B%BE%E7%89%87%0A%20%20%20%20%20%20%20%20openssl%0A%20%20%20%20Applications%3A%E7%B3%BB%E7%BB%9F%E8%87%AA%E5%B8%A6%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%A6%82%EF%BC%9Aapp%20store%E3%80%81books%E3%80%81Calculator%E3%80%81music%E3%80%81photo%20%E7%AD%89%0A%20%20%20%20%0AUsers%20%E5%AD%98%E6%94%BE%E7%94%A8%E6%88%B7%E7%9A%84%E4%B8%AA%E4%BA%BA%E8%B5%84%E6%96%99%E5%92%8C%E9%85%8D%E7%BD%AE%E3%80%82%E6%AF%8F%E4%B8%AA%E7%94%A8%E6%88%B7%E6%9C%89%E8%87%AA%E5%B7%B1%E7%9A%84%E5%8D%95%E7%8B%AC%E7%9B%AE%E5%BD%95%E3%80%82%E7%AD%89%E5%90%8C%E4%BA%8Elinux%E4%B8%8B%2Fhome%0A%20%20%20%20wangdongyan%0A%20%20%20%20%20%20%20%20Applications%0A%20%20%20%20%20%20%20%20Desktop%0A%20%20%20%20%20%20%20%20Documents%0A%20%20%20%20%20%20%20%20Downloads%0A%20%20%20%20%20%20%20%20Library%0A%20%20%20%20%20%20%20%20%20%20%20%20Application%20Support%3A%E4%B8%89%E6%96%B9%E5%BA%94%E7%94%A8%E6%8F%92%E4%BB%B6%0A%20%20%20%20%20%20%20%20Movies%0A%20%20%20%20%20%20%20%20Music%0A%20%20%20%20%20%20%20%20Pictures%0A%20%20%20%20%20%20%20%20Public%20%E4%BD%A0%E5%8F%AF%E4%BB%A5%E6%8A%8A%E9%9C%80%E8%A6%81%E4%B8%8E%E5%85%B6%E5%AE%83%E7%94%A8%E6%88%B7%E5%85%B1%E4%BA%AB%E7%9A%84%E6%96%87%E4%BB%B6%E6%94%BE%E5%9C%A8%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E4%B8%AD%EF%BC%8C%E9%BB%98%E8%AE%A4%E7%8A%B6%E6%80%81%E4%B8%8B%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E5%8F%AF%E4%BB%A5%E8%A2%AB%E5%85%B6%E5%AE%83%E6%89%80%E6%9C%89%E7%94%A8%E6%88%B7%E8%AE%BF%E9%97%AE%0A%20%20%20%20%20%20%20%20%E5%90%84%E7%A7%8D%E8%BD%AF%E4%BB%B6%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E3%80%81%E7%BC%93%E5%AD%98%E6%96%87%E4%BB%B6%0A%20%20%20%20%20%20%20%20%0AVolumes%20%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%8C%82%E8%BD%BD%E7%82%B9%E5%AD%98%E6%94%BE%E7%9B%AE%E5%BD%95%E3%80%82%E6%9C%89%E6%97%B6%E5%80%99%E5%AE%89%E8%A3%85%E4%B8%80%E4%BA%9B%E9%9D%9EAPP%20Store%E4%B8%8B%E8%BD%BD%E7%9A%84%E5%8C%85%EF%BC%8C%E9%A6%96%E6%AC%A1%E4%BC%9A%E6%8C%82%E8%BD%BD%E5%88%B0%E8%BF%99%E9%87%8C%0Acores%20%E5%86%85%E6%A0%B8%E8%BD%AC%E5%82%A8%E6%96%87%E4%BB%B6%E5%AD%98%E6%94%BE%E7%9B%AE%E5%BD%95%E3%80%82%E5%BD%93%E4%B8%80%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%B4%A9%E6%BA%83%E6%97%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E7%B3%BB%E7%BB%9F%E5%85%81%E8%AE%B8%E5%88%99%E4%BC%9A%E4%BA%A7%E7%94%9F%E8%BD%AC%E5%82%A8%E6%96%87%E4%BB%B6%E3%80%82%0Aprivate%20%E9%87%8C%E9%9D%A2%E7%9A%84%E5%AD%90%E7%9B%AE%E5%BD%95%E5%AD%98%E6%94%BE%E4%BA%86%2Ftmp%2C%20%2Fvar%2C%20%2Fetc%E7%AD%89%E9%93%BE%E6%8E%A5%E7%9B%AE%E5%BD%95%E7%9A%84%E7%9B%AE%E6%A0%87%E7%9B%AE%E5%BD%95%E3%80%82%0A%0A%60%60%60%0A%0A%0A%0A%0A
