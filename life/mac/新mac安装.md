# 新mac安装

## U盘安装

1. 插上U盘
2. 打开mac自带disk工具
3. U盘格式化成：Mac os 扩展\(日志式\)
4. 打开官网地址

> https://support.apple.com/zh\-cn/HT201372

1. 下载os镜像
2. 制作U盘引导\+OS镜像

> sudo /Applications/Install\\ macOS\\ Catalina.app/Contents/Resources/createinstallmedia \-\-volume /Volumes/MyVolume

> sudo /Applications/Install\\ macOS\\ Ventura.app/Contents/Resources/createinstallmedia \-\-volume /Volumes/MyVolume

1. command \+ R ,进入恢复模式，上面菜单栏，打开：允许外部安装系统
2. options \+ R ,选择那个U盘
3. 格式化掉MAC 硬盘的所有内容，注：给硬盘起个新名字，不然他就自动带个：data

## OS初始硬盘容量

目前硬盘已使用容量：11.05 \+ 5.5

## OS基础设置

1. 系统设置 \-\> 显示器 排列 \-\> 打开显示器镜像
2. 系统设置\-\>时间与日期 \-\> 时间改成24小时，并显示日期
3. 设置 \-\> 键盘 \-\> 快捷键：\(修改快捷键\)
    1. caps键切换输入法
    2. 聚集 \-\> OP \+ SPACE （切换聚集）
    3. 输入法 \-\> cmd \+ space\(切换输入法\)
    4. 调度中心 \-\> ctrl \+ d（显示桌面）
    5. 服务 \-\> cmd \+ shift \+ f （快速打开finder）
4. 系统设置 \-\> 键盘 \-\> 输出法 \-\> 删除掉拼音
5. 系统设置 \-\> 软件更新 \-\> 关闭mac 自动更新
6. 系统设置 \-\> 安装性与隐私 \-\> 防火墙 \-\> 关闭
7. 系统设置 \-\> 节能 设置锁屏幕 时间
8. 点击状态 栏 上的电池LOGO 显示电池 百分比
9. 系统设置 \-\> 安全性与隐私 \-\> 通用 \-\> 选择“任何来源”

> 如果没有'任何来源选项'，执行：sudo spctl \-\-master\-disable

## 基础软件安装

1. app store
    1. thor
    2. the unarchier
    3. wechat
    4. lemon
    5. qq
    6. lark
    7. 万年厍
    8. foxit
    9. 参考另外一个[文�]()�
2. 本地文件安装
    1.搜狗五笔输入法
    > OS设置:键盘\-\>删了拼音输入法
    1. hyper switch
    2. 阿里云盘
    3. cDock
    4. foobar
    5. HandShaker
    6. IINA
    7. iTerm
    8. openmtp
    9. Paste
    10. SecureCRT
    11. google chrome
    12. element\-key
    13. 迅雷

## karainer\-element

更换的健位：

|原健|更换|
|----|----|
|fn  |cmd |
|cmd |fn  |

选择complex modifications

新建一个 add rule

选择 import

|原健  |更换         |
|------|-------------|
|return|open         |
|use f2|rename       |
|del   |move to trash|

## 设置finder

1. 显示 \-\>路径\(目录\)栏 状态栏
2. 自定义工具栏 加上：前进后退 路径显示 新建文件 删除
3. 偏好设置：ctrl \+ ,
    1. 配置一下左侧栏目显示
    2. 显示所有文件扩展名
    3. 搜索改成：只搜索当前文件夹子
    4. 在桌面上 显示硬盘
4. 将data film 目录拖到边栏中去

## 开启ROOT

1. 设置 用户 登陆选项\(左下角\) 网络账号服务器 加入 打开目录实用工具 上方菜单 编辑 启用root 设置密码
2. spotlight 直接输入：目录实用工具 更快

## 关闭csrutil（SIP）

1. 重启 ， command \+ r ,进入恢复模式
2. 上方菜单中 开启一个shell

> csrutil disable

1. 重启

## 创建能用文件目录

mount \-uw /

mkdir /data

## xcode & command tools

下载地址：

> https://developer.apple.com/download/all/?q=xcode

OS版本与xcode对应版本

> https://xcodereleases.com/

慢长的下载吧。。。。10G\+400M\+

解压xcode.xip ,插入到 application 中，双击，它还得安装一会

安装command tools

## brew

> /bin/zsh \-c "$\(curl \-fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh\)"

这个可以，但依赖GIT，建议先安装好xcode 和command tools

安装过程中，风扇好响 。。。且又是一排 红字，大概意思是：以ROOT运行brew非常危险（去你妹的，老子的电脑，我想怎么用ROOT就怎么用！）

```
brew update
brew install wget
brew install tree
brew install ffmpeg
```

## 设置cDock

安装 cDock

系统设置 \-\> 程序坞 \-》 关闭 显示最近打开的APP

## 翻墙软件

https://hkscloud.men/\#/login

安装 clashX

## alfred

[请点击]()

## github\-ssh

ssh\-keygen \-t rsa \-C "mqzhifu@sina.com"

却是github setting ssh 中添加一个公钥

## item2

具体就不写了，参照另外一个[文档]()

记得配置login.sh os\_startup.sh

## vim

[请点击]()

## 设置开机自启

thor

magent

Hyper switch

## 关闭spotlight索引

sudo launchctl unload \-w /System/Library/LaunchAgents/com.apple.Spotlight.plist

sudo launchctl load \-w /System/Library/LaunchAgents/com.apple.Spotlight.plist

sudo launchctl unload \-w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

sudo launchctl load \-w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

关闭自动索引

> sudo mdutil \-a \-i off

删除索引

> sudo mdutil \-a \-E

重新打开

> sudo mdutil \-a \-i on

重建索引

> sudo mdutil \-E

## 最后硬盘空间

折腾完剩下：422个G

chown \-R $\(whoami\) $\(brew \-\-prefix\)/\*

## 移动硬盘被坑

移动硬盘格式不同，WIN下是NTFS，MAC下是HFS\+EXFAT，可以读，但不能写，如果想要写ntfs需要额外最少花39员买个软件，且39元只能支持一台设备、读写速度慢、无法保证MAC更新版本继续使用。我TMD也是贱，非要自己尝试破解，结果1.8T的数据全丢，不少都是N年积累下来的禁播电影，还有海伟的高清美剧，操了！

先不考试丢的数据了，查查解决办法吧

具体方法：

1、MAC下磁盘工具\-\>抹除\-\>再分区成exFat

2、WIN下执行

> chkdsk F: /F

两端都试了下没问题，多了3个文件夹子

> .fseventsd
> 
> 
> .Spotlight\-V100
> 
> 
> FOUND.000

总结

之前是MAC下兼容WIN

现在改成WIN去兼容MAC

WIN下超简单，一条指令，MAC上就得花钱，而且exFat好像还是microsoft 发明的，另外，就算用付费软件，且最高VIP付费，MAC写NTFS的速度依然慢，且有丢数据风险 ，NND，SB的苹果。

虽然数据丢了，但丢就丢了吧，有些不怎么看，还能下的，再重新找找资源，下不了的就算了。当年手杀熊猫算是熟悉了WIN的基本结构，不停的重做系统搞坏硬盘，算是熟悉了WIN的硬盘操作，MAC下也得损失点，不然哪里来的熟悉呢~当是学费吧。

另外，WIN下麻烦点，每次都得执行一下指令才能使用，但是总比花钱要强，而且，现在大部分时间使用MAC办公，经常传东西也是MAC ，效果更好些。

U%E7%9B%98%E5%AE%89%E8%A3%85%0A\-\-\-\-\-%0A1.%20%E6%8F%92%E4%B8%8AU%E7%9B%98%0A2.%20%E6%89%93%E5%BC%80mac%E8%87%AA%E5%B8%A6disk%E5%B7%A5%E5%85%B7%0A3.%20U%E7%9B%98%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%88%90%EF%BC%9AMac%20os%20%E6%89%A9%E5%B1%95\(%E6%97%A5%E5%BF%97%E5%BC%8F\)%0A4.%20%E6%89%93%E5%BC%80%E5%AE%98%E7%BD%91%E5%9C%B0%E5%9D%80%0A%3Ehttps%3A%2F%2Fsupport.apple.com%2Fzh\-cn%2FHT201372%0A5.%20%E4%B8%8B%E8%BD%BDos%E9%95%9C%E5%83%8F%0A6.%20%E5%88%B6%E4%BD%9CU%E7%9B%98%E5%BC%95%E5%AF%BC%2BOS%E9%95%9C%E5%83%8F%0A%3Esudo%20%2FApplications%2FInstall%5C%20macOS%5C%20Catalina.app%2FContents%2FResources%2Fcreateinstallmedia%20\-\-volume%20%2FVolumes%2FMyVolume%0A%0A%3Esudo%20%2FApplications%2FInstall%5C%20macOS%5C%20Ventura.app%2FContents%2FResources%2Fcreateinstallmedia%20\-\-volume%20%2FVolumes%2FMyVolume%C2%A0%0A%0A%0A%0A7.%20command%20%2B%20R%20%2C%E8%BF%9B%E5%85%A5%E6%81%A2%E5%A4%8D%E6%A8%A1%E5%BC%8F%EF%BC%8C%E4%B8%8A%E9%9D%A2%E8%8F%9C%E5%8D%95%E6%A0%8F%EF%BC%8C%E6%89%93%E5%BC%80%EF%BC%9A%E5%85%81%E8%AE%B8%E5%A4%96%E9%83%A8%E5%AE%89%E8%A3%85%E7%B3%BB%E7%BB%9F%0A8.%20options%20%2B%20R%20%2C%E9%80%89%E6%8B%A9%E9%82%A3%E4%B8%AAU%E7%9B%98%0A9.%20%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%8E%89MAC%20%E7%A1%AC%E7%9B%98%E7%9A%84%E6%89%80%E6%9C%89%E5%86%85%E5%AE%B9%EF%BC%8C%E6%B3%A8%EF%BC%9A%E7%BB%99%E7%A1%AC%E7%9B%98%E8%B5%B7%E4%B8%AA%E6%96%B0%E5%90%8D%E5%AD%97%EF%BC%8C%E4%B8%8D%E7%84%B6%E4%BB%96%E5%B0%B1%E8%87%AA%E5%8A%A8%E5%B8%A6%E4%B8%AA%EF%BC%9Adata%0A%0A%0AOS%E5%88%9D%E5%A7%8B%E7%A1%AC%E7%9B%98%E5%AE%B9%E9%87%8F%0A\-\-\-\-\-%0A%E7%9B%AE%E5%89%8D%E7%A1%AC%E7%9B%98%E5%B7%B2%E4%BD%BF%E7%94%A8%E5%AE%B9%E9%87%8F%EF%BC%9A11.05%20%2B%205.5%20%20%0A%0AOS%E5%9F%BA%E7%A1%80%E8%AE%BE%E7%BD%AE%0A\-\-\-\-\-%0A1.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E6%98%BE%E7%A4%BA%E5%99%A8%20%E6%8E%92%E5%88%97%20\-%3E%20%E6%89%93%E5%BC%80%E6%98%BE%E7%A4%BA%E5%99%A8%E9%95%9C%E5%83%8F%0A2.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE\-%3E%E6%97%B6%E9%97%B4%E4%B8%8E%E6%97%A5%E6%9C%9F%20\-%3E%20%E6%97%B6%E9%97%B4%E6%94%B9%E6%88%9024%E5%B0%8F%E6%97%B6%EF%BC%8C%E5%B9%B6%E6%98%BE%E7%A4%BA%E6%97%A5%E6%9C%9F%0A3.%20%E8%AE%BE%E7%BD%AE%20\-%3E%20%E9%94%AE%E7%9B%98%20\-%3E%20%E5%BF%AB%E6%8D%B7%E9%94%AE%EF%BC%9A\(%E4%BF%AE%E6%94%B9%E5%BF%AB%E6%8D%B7%E9%94%AE\)%0A%20%20%20%201.%20caps%E9%94%AE%E5%88%87%E6%8D%A2%E8%BE%93%E5%85%A5%E6%B3%95%20%0A%20%20%20%202.%20%E8%81%9A%E9%9B%86%20%20\-%3E%20%20OP%20%2B%20SPACE%20%EF%BC%88%E5%88%87%E6%8D%A2%E8%81%9A%E9%9B%86%EF%BC%89%0A%20%20%20%203.%20%E8%BE%93%E5%85%A5%E6%B3%95%20\-%3E%20cmd%20%2B%20space\(%E5%88%87%E6%8D%A2%E8%BE%93%E5%85%A5%E6%B3%95\)%0A%20%20%20%204.%20%E8%B0%83%E5%BA%A6%E4%B8%AD%E5%BF%83%20\-%3E%20%20ctrl%20%2B%20d%EF%BC%88%E6%98%BE%E7%A4%BA%E6%A1%8C%E9%9D%A2%EF%BC%89%0A%20%20%20%205.%20%E6%9C%8D%E5%8A%A1%20%20%20\-%3E%20cmd%20%2B%20shift%20%2B%20f%20%EF%BC%88%E5%BF%AB%E9%80%9F%E6%89%93%E5%BC%80finder%EF%BC%89%0A3.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E9%94%AE%E7%9B%98%20\-%3E%20%E8%BE%93%E5%87%BA%E6%B3%95%20\-%3E%20%E5%88%A0%E9%99%A4%E6%8E%89%E6%8B%BC%E9%9F%B3%0A4.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E8%BD%AF%E4%BB%B6%E6%9B%B4%E6%96%B0%20\-%3E%20%E5%85%B3%E9%97%ADmac%20%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%0A5.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E5%AE%89%E8%A3%85%E6%80%A7%E4%B8%8E%E9%9A%90%E7%A7%81%20\-%3E%20%E9%98%B2%E7%81%AB%E5%A2%99%20\-%3E%20%E5%85%B3%E9%97%AD%0A6.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E8%8A%82%E8%83%BD%20%E8%AE%BE%E7%BD%AE%E9%94%81%E5%B1%8F%E5%B9%95%20%E6%97%B6%E9%97%B4%20%0A7.%20%E7%82%B9%E5%87%BB%E7%8A%B6%E6%80%81%20%E6%A0%8F%20%E4%B8%8A%E7%9A%84%E7%94%B5%E6%B1%A0LOGO%20%20%E6%98%BE%E7%A4%BA%E7%94%B5%E6%B1%A0%20%E7%99%BE%E5%88%86%E6%AF%94%0A6.%20%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E5%AE%89%E5%85%A8%E6%80%A7%E4%B8%8E%E9%9A%90%E7%A7%81%20\-%3E%20%E9%80%9A%E7%94%A8%20\-%3E%20%E9%80%89%E6%8B%A9%E2%80%9C%E4%BB%BB%E4%BD%95%E6%9D%A5%E6%BA%90%E2%80%9D%0A%3E%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89'%E4%BB%BB%E4%BD%95%E6%9D%A5%E6%BA%90%E9%80%89%E9%A1%B9'%EF%BC%8C%E6%89%A7%E8%A1%8C%EF%BC%9Asudo%20spctl%20\-\-master\-disable%0A%0A%E5%9F%BA%E7%A1%80%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%0A\-\-\-\-%0A1.%20app%20store%0A%20%20%20%201.%20thor%0A%20%20%20%202.%20the%20unarchier%0A%20%20%20%203.%20wechat%0A%20%20%20%204.%20lemon%0A%20%20%20%204.%20qq%0A%20%20%20%205.%20lark%0A%20%20%20%206.%20%E4%B8%87%E5%B9%B4%E5%8E%8D%0A%20%20%20%207.%20foxit%0A%20%20%20%208.%20%E5%8F%82%E8%80%83%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%5B%E6%96%87%E7%AB%A0%5D\(evernote%3A%2F%2F%2Fview%2F35868219%2Fs61%2F75094959\-dacc\-4511\-aecd\-1e2cd2a160df%2F75094959\-dacc\-4511\-aecd\-1e2cd2a160df%2F\)%0A2.%20%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%AE%89%E8%A3%85%0A%20%20%20%201.%E6%90%9C%E7%8B%97%E4%BA%94%E7%AC%94%E8%BE%93%E5%85%A5%E6%B3%95%0A%20%20%20%20%3E%20OS%E8%AE%BE%E7%BD%AE%3A%E9%94%AE%E7%9B%98\-%3E%E5%88%A0%E4%BA%86%E6%8B%BC%E9%9F%B3%E8%BE%93%E5%85%A5%E6%B3%95%0A%20%20%20%202.%20hyper%20switch%0A%20%20%20%203.%20%E9%98%BF%E9%87%8C%E4%BA%91%E7%9B%98%0A%20%20%20%204.%20cDock%0A%20%20%20%205.%20foobar%0A%20%20%20%206.%20HandShaker%0A%20%20%20%207.%20IINA%0A%20%20%20%208.%20iTerm%0A%20%20%20%209.%20openmtp%0A%20%20%20%2010.%20Paste%0A%20%20%20%2011.%20SecureCRT%0A%20%20%20%2012.%20google%20chrome%0A%20%20%20%2013.%20element\-key%0A%20%20%20%2014.%20%E8%BF%85%E9%9B%B7%0A%0Akarainer\-element%0A\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-%0A%0A%E6%9B%B4%E6%8D%A2%E7%9A%84%E5%81%A5%E4%BD%8D%EF%BC%9A%0A%7C%E5%8E%9F%E5%81%A5%7C%E6%9B%B4%E6%8D%A2%7C%0A%7C\-\-\-\-%7C\-\-\-\-\-%7C%0A%7Cfn%7Ccmd%7C%0A%7Ccmd%7Cfn%7C%0A%0A%E9%80%89%E6%8B%A9complex%20modifications%0A%E6%96%B0%E5%BB%BA%E4%B8%80%E4%B8%AA%20add%20rule%0A%E9%80%89%E6%8B%A9%20import%20%0A%7C%E5%8E%9F%E5%81%A5%7C%E6%9B%B4%E6%8D%A2%7C%0A%7C\-\-\-\-%7C\-\-\-\-\-%7C%0A%7Creturn%7Copen%7C%0A%7Cuse%20f2%7Crename%7C%0A%7Cdel%7Cmove%20to%20trash%7C%0A%0A%0A%E8%AE%BE%E7%BD%AEfinder%0A\-\-\-\-\-%0A1.%20%E6%98%BE%E7%A4%BA%20\-%3E%E8%B7%AF%E5%BE%84\(%E7%9B%AE%E5%BD%95\)%E6%A0%8F%20%E7%8A%B6%E6%80%81%E6%A0%8F%0A2.%20%E8%87%AA%E5%AE%9A%E4%B9%89%E5%B7%A5%E5%85%B7%E6%A0%8F%20%E5%8A%A0%E4%B8%8A%EF%BC%9A%E5%89%8D%E8%BF%9B%E5%90%8E%E9%80%80%20%E8%B7%AF%E5%BE%84%E6%98%BE%E7%A4%BA%20%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%20%E5%88%A0%E9%99%A4%0A3.%20%E5%81%8F%E5%A5%BD%E8%AE%BE%E7%BD%AE%EF%BC%9Actrl%20%2B%20%2C%20%20%20%0A%20%20%20%201.%20%E9%85%8D%E7%BD%AE%E4%B8%80%E4%B8%8B%E5%B7%A6%E4%BE%A7%E6%A0%8F%E7%9B%AE%E6%98%BE%E7%A4%BA%0A%20%20%20%202.%20%E6%98%BE%E7%A4%BA%E6%89%80%E6%9C%89%E6%96%87%E4%BB%B6%E6%89%A9%E5%B1%95%E5%90%8D%0A%20%20%20%203.%20%E6%90%9C%E7%B4%A2%E6%94%B9%E6%88%90%EF%BC%9A%E5%8F%AA%E6%90%9C%E7%B4%A2%E5%BD%93%E5%89%8D%E6%96%87%E4%BB%B6%E5%A4%B9%E5%AD%90%0A%20%20%20%204.%20%E5%9C%A8%E6%A1%8C%E9%9D%A2%E4%B8%8A%20%E6%98%BE%E7%A4%BA%E7%A1%AC%E7%9B%98%0A4.%20%E5%B0%86data%20film%20%E7%9B%AE%E5%BD%95%E6%8B%96%E5%88%B0%E8%BE%B9%E6%A0%8F%E4%B8%AD%E5%8E%BB%0A%0A%E5%BC%80%E5%90%AFROOT%0A\-\-\-\-%0A1.%20%E8%AE%BE%E7%BD%AE%20%E7%94%A8%E6%88%B7%20%E7%99%BB%E9%99%86%E9%80%89%E9%A1%B9\(%E5%B7%A6%E4%B8%8B%E8%A7%92\)%20%E7%BD%91%E7%BB%9C%E8%B4%A6%E5%8F%B7%E6%9C%8D%E5%8A%A1%E5%99%A8%20%E5%8A%A0%E5%85%A5%20%E6%89%93%E5%BC%80%E7%9B%AE%E5%BD%95%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7%20%E4%B8%8A%E6%96%B9%E8%8F%9C%E5%8D%95%20%E7%BC%96%E8%BE%91%20%E5%90%AF%E7%94%A8root%20%E8%AE%BE%E7%BD%AE%E5%AF%86%E7%A0%81%0A2.%20spotlight%20%E7%9B%B4%E6%8E%A5%E8%BE%93%E5%85%A5%EF%BC%9A%E7%9B%AE%E5%BD%95%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7%20%E6%9B%B4%E5%BF%AB%0A%0A%E5%85%B3%E9%97%ADcsrutil%EF%BC%88SIP%EF%BC%89%0A\-\-\-\-\-%0A1.%20%E9%87%8D%E5%90%AF%20%EF%BC%8C%20command%20%2B%20r%20%2C%E8%BF%9B%E5%85%A5%E6%81%A2%E5%A4%8D%E6%A8%A1%E5%BC%8F%0A2.%20%E4%B8%8A%E6%96%B9%E8%8F%9C%E5%8D%95%E4%B8%AD%20%E5%BC%80%E5%90%AF%E4%B8%80%E4%B8%AAshell%0A%3Ecsrutil%20disable%0A%0A3.%20%E9%87%8D%E5%90%AF%0A%0A%E5%88%9B%E5%BB%BA%E8%83%BD%E7%94%A8%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95%0A\-\-\-\-\-%0Amount%20\-uw%20%2F%0Amkdir%20%2Fdata%0A%0Axcode%20%26%20command%20tools%0A\-\-\-\-%0A%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80%EF%BC%9A%0A%3Ehttps%3A%2F%2Fdeveloper.apple.com%2Fdownload%2Fall%2F%3Fq%3Dxcode%0A%0AOS%E7%89%88%E6%9C%AC%E4%B8%8Excode%E5%AF%B9%E5%BA%94%E7%89%88%E6%9C%AC%0A%3Ehttps%3A%2F%2Fxcodereleases.com%2F%0A%0A%E6%85%A2%E9%95%BF%E7%9A%84%E4%B8%8B%E8%BD%BD%E5%90%A7%E3%80%82%E3%80%82%E3%80%82%E3%80%8210G%2B400M%2B%0A%0A%E8%A7%A3%E5%8E%8Bxcode.xip%20%2C%E6%8F%92%E5%85%A5%E5%88%B0%20application%20%E4%B8%AD%EF%BC%8C%E5%8F%8C%E5%87%BB%EF%BC%8C%E5%AE%83%E8%BF%98%E5%BE%97%E5%AE%89%E8%A3%85%E4%B8%80%E4%BC%9A%0A%E5%AE%89%E8%A3%85command%20tools%0A%0Abrew%0A\-\-\-\-%0A%0A%3E%2Fbin%2Fzsh%20\-c%20%22%24\(curl%20\-fsSL%20https%3A%2F%2Fgitee.com%2Fcunkai%2FHomebrewCN%2Fraw%2Fmaster%2FHomebrew.sh\)%22%0A%0A%E8%BF%99%E4%B8%AA%E5%8F%AF%E4%BB%A5%EF%BC%8C%E4%BD%86%E4%BE%9D%E8%B5%96GIT%EF%BC%8C%E5%BB%BA%E8%AE%AE%E5%85%88%E5%AE%89%E8%A3%85%E5%A5%BDxcode%20%E5%92%8Ccommand%20tools%0A%0A%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B%E4%B8%AD%EF%BC%8C%E9%A3%8E%E6%89%87%E5%A5%BD%E5%93%8D%20%E3%80%82%E3%80%82%E3%80%82%E4%B8%94%E5%8F%88%E6%98%AF%E4%B8%80%E6%8E%92%20%E7%BA%A2%E5%AD%97%EF%BC%8C%E5%A4%A7%E6%A6%82%E6%84%8F%E6%80%9D%E6%98%AF%EF%BC%9A%E4%BB%A5ROOT%E8%BF%90%E8%A1%8Cbrew%E9%9D%9E%E5%B8%B8%E5%8D%B1%E9%99%A9%EF%BC%88%E5%8E%BB%E4%BD%A0%E5%A6%B9%E7%9A%84%EF%BC%8C%E8%80%81%E5%AD%90%E7%9A%84%E7%94%B5%E8%84%91%EF%BC%8C%E6%88%91%E6%83%B3%E6%80%8E%E4%B9%88%E7%94%A8ROOT%E5%B0%B1%E6%80%8E%E4%B9%88%E7%94%A8%EF%BC%81%EF%BC%89%0A%60%60%60%0Abrew%20update%0Abrew%20install%20wget%0Abrew%20install%20tree%0Abrew%20install%20ffmpeg%0A%60%60%60%0A%0A%E8%AE%BE%E7%BD%AEcDock%0A\-\-\-\-%0A%E5%AE%89%E8%A3%85%20cDock%20%0A%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE%20\-%3E%20%E7%A8%8B%E5%BA%8F%E5%9D%9E%20\-%E3%80%8B%20%E5%85%B3%E9%97%AD%20%E6%98%BE%E7%A4%BA%E6%9C%80%E8%BF%91%E6%89%93%E5%BC%80%E7%9A%84APP%0A%0A%0A%E7%BF%BB%E5%A2%99%E8%BD%AF%E4%BB%B6%0A\-\-\-%0Ahttps%3A%2F%2Fhkscloud.men%2F%23%2Flogin%0A%E5%AE%89%E8%A3%85%20%20clashX%0A%0Aalfred%0A\-\-\-\-\-%0A%5B%E8%AF%B7%E7%82%B9%E5%87%BB%5D\(evernote%3A%2F%2F%2Fview%2F35868219%2Fs61%2Fa4e076ff\-362d\-4655\-abc1\-993bc2e3c3d3%2Fa4e076ff\-362d\-4655\-abc1\-993bc2e3c3d3%2F\)%0A%0Agithub\-ssh%0A\-\-\-\-\-%0Assh\-keygen%20\-t%20rsa%20\-C%20%22mqzhifu%40sina.com%22%0A%E5%8D%B4%E6%98%AFgithub%20setting%20ssh%20%E4%B8%AD%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E5%85%AC%E9%92%A5%0A%0Aitem2%0A\-\-\-\-%0A%E5%85%B7%E4%BD%93%E5%B0%B1%E4%B8%8D%E5%86%99%E4%BA%86%EF%BC%8C%E5%8F%82%E7%85%A7%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%5B%E6%96%87%E6%A1%A3%5D\(evernote%3A%2F%2F%2Fview%2F35868219%2Fs61%2F3984602f\-1251\-4fac\-aae3\-318dd784839b%2F3984602f\-1251\-4fac\-aae3\-318dd784839b%2F\)%0A%E8%AE%B0%E5%BE%97%E9%85%8D%E7%BD%AElogin.sh%20os\_startup.sh%0A%0Avim%0A\-\-\-\-%0A%5B%E8%AF%B7%E7%82%B9%E5%87%BB%5D\(evernote%3A%2F%2F%2Fview%2F35868219%2Fs61%2Fb4d18af1\-3a87\-4b0d\-a234\-833734866d43%2Fb4d18af1\-3a87\-4b0d\-a234\-833734866d43%2F\)%0A%0A%0A%0A%0A%E8%AE%BE%E7%BD%AE%E5%BC%80%E6%9C%BA%E8%87%AA%E5%90%AF%0A\-\-\-%0Athor%0Amagent%0AHyper%20switch%0A%0A%0A%E5%85%B3%E9%97%ADspotlight%E7%B4%A2%E5%BC%95%0A\-\-\-\-%0Asudo%20launchctl%20unload%20\-w%20%2FSystem%2FLibrary%2FLaunchAgents%2Fcom.apple.Spotlight.plist%0Asudo%20launchctl%20load%20\-w%20%2FSystem%2FLibrary%2FLaunchAgents%2Fcom.apple.Spotlight.plist%0A%0Asudo%20launchctl%20unload%20\-w%20%2FSystem%2FLibrary%2FLaunchDaemons%2Fcom.apple.metadata.mds.plist%0Asudo%20launchctl%20load%20\-w%20%2FSystem%2FLibrary%2FLaunchDaemons%2Fcom.apple.metadata.mds.plist%0A%0A%E5%85%B3%E9%97%AD%E8%87%AA%E5%8A%A8%E7%B4%A2%E5%BC%95%0A%3Esudo%20mdutil%20\-a%20\-i%20off%20%0A%0A%E5%88%A0%E9%99%A4%E7%B4%A2%E5%BC%95%0A%3Esudo%20mdutil%20\-a%20\-E%20%0A%0A%E9%87%8D%E6%96%B0%E6%89%93%E5%BC%80%0A%3Esudo%20mdutil%20\-a%20\-i%20on%0A%0A%E9%87%8D%E5%BB%BA%E7%B4%A2%E5%BC%95%0A%3Esudo%20mdutil%20\-E%0A%0A%0A%E6%9C%80%E5%90%8E%E7%A1%AC%E7%9B%98%E7%A9%BA%E9%97%B4%0A\-\-\-%0A%E6%8A%98%E8%85%BE%E5%AE%8C%E5%89%A9%E4%B8%8B%EF%BC%9A422%E4%B8%AAG%0A%0Achown%20\-R%20%24\(whoami\)%20%24\(brew%20\-\-prefix\)%2F\*%0A%0A%0A%E7%A7%BB%E5%8A%A8%E7%A1%AC%E7%9B%98%E8%A2%AB%E5%9D%91%0A\-\-\-\-\-\-\-\-%0A%E7%A7%BB%E5%8A%A8%E7%A1%AC%E7%9B%98%E6%A0%BC%E5%BC%8F%E4%B8%8D%E5%90%8C%EF%BC%8CWIN%E4%B8%8B%E6%98%AFNTFS%EF%BC%8CMAC%E4%B8%8B%E6%98%AFHFS%2BEXFAT%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%AF%BB%EF%BC%8C%E4%BD%86%E4%B8%8D%E8%83%BD%E5%86%99%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%83%B3%E8%A6%81%E5%86%99ntfs%E9%9C%80%E8%A6%81%E9%A2%9D%E5%A4%96%E6%9C%80%E5%B0%91%E8%8A%B139%E5%91%98%E4%B9%B0%E4%B8%AA%E8%BD%AF%E4%BB%B6%EF%BC%8C%E4%B8%9439%E5%85%83%E5%8F%AA%E8%83%BD%E6%94%AF%E6%8C%81%E4%B8%80%E5%8F%B0%E8%AE%BE%E5%A4%87%E3%80%81%E8%AF%BB%E5%86%99%E9%80%9F%E5%BA%A6%E6%85%A2%E3%80%81%E6%97%A0%E6%B3%95%E4%BF%9D%E8%AF%81MAC%E6%9B%B4%E6%96%B0%E7%89%88%E6%9C%AC%E7%BB%A7%E7%BB%AD%E4%BD%BF%E7%94%A8%E3%80%82%E6%88%91TMD%E4%B9%9F%E6%98%AF%E8%B4%B1%EF%BC%8C%E9%9D%9E%E8%A6%81%E8%87%AA%E5%B7%B1%E5%B0%9D%E8%AF%95%E7%A0%B4%E8%A7%A3%EF%BC%8C%E7%BB%93%E6%9E%9C1.8T%E7%9A%84%E6%95%B0%E6%8D%AE%E5%85%A8%E4%B8%A2%EF%BC%8C%E4%B8%8D%E5%B0%91%E9%83%BD%E6%98%AFN%E5%B9%B4%E7%A7%AF%E7%B4%AF%E4%B8%8B%E6%9D%A5%E7%9A%84%E7%A6%81%E6%92%AD%E7%94%B5%E5%BD%B1%EF%BC%8C%E8%BF%98%E6%9C%89%E6%B5%B7%E4%BC%9F%E7%9A%84%E9%AB%98%E6%B8%85%E7%BE%8E%E5%89%A7%EF%BC%8C%E6%93%8D%E4%BA%86%EF%BC%81%0A%0A%E5%85%88%E4%B8%8D%E8%80%83%E8%AF%95%E4%B8%A2%E7%9A%84%E6%95%B0%E6%8D%AE%E4%BA%86%EF%BC%8C%E6%9F%A5%E6%9F%A5%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95%E5%90%A7%20%20%0A%0A%E5%85%B7%E4%BD%93%E6%96%B9%E6%B3%95%EF%BC%9A%20%20%0A1%E3%80%81MAC%E4%B8%8B%E7%A3%81%E7%9B%98%E5%B7%A5%E5%85%B7\-%3E%E6%8A%B9%E9%99%A4\-%3E%E5%86%8D%E5%88%86%E5%8C%BA%E6%88%90exFat%20%20%0A2%E3%80%81WIN%E4%B8%8B%E6%89%A7%E8%A1%8C%0A%3Echkdsk%20F%3A%20%2FF%20%20%0A%0A%E4%B8%A4%E7%AB%AF%E9%83%BD%E8%AF%95%E4%BA%86%E4%B8%8B%E6%B2%A1%E9%97%AE%E9%A2%98%EF%BC%8C%E5%A4%9A%E4%BA%863%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%AD%90%20%0A%3E.fseventsd%20%20%0A%3E.Spotlight\-V100%20%20%0A%3EFOUND.000%20%20%0A%0A%E6%80%BB%E7%BB%93%20%20%0A%E4%B9%8B%E5%89%8D%E6%98%AFMAC%E4%B8%8B%E5%85%BC%E5%AE%B9WIN%20%20%0A%E7%8E%B0%E5%9C%A8%E6%94%B9%E6%88%90WIN%E5%8E%BB%E5%85%BC%E5%AE%B9MAC%20%20%0A%0AWIN%E4%B8%8B%E8%B6%85%E7%AE%80%E5%8D%95%EF%BC%8C%E4%B8%80%E6%9D%A1%E6%8C%87%E4%BB%A4%EF%BC%8CMAC%E4%B8%8A%E5%B0%B1%E5%BE%97%E8%8A%B1%E9%92%B1%EF%BC%8C%E8%80%8C%E4%B8%94exFat%E5%A5%BD%E5%83%8F%E8%BF%98%E6%98%AFmicrosoft%20%E5%8F%91%E6%98%8E%E7%9A%84%EF%BC%8C%E5%8F%A6%E5%A4%96%EF%BC%8C%E5%B0%B1%E7%AE%97%E7%94%A8%E4%BB%98%E8%B4%B9%E8%BD%AF%E4%BB%B6%EF%BC%8C%E4%B8%94%E6%9C%80%E9%AB%98VIP%E4%BB%98%E8%B4%B9%EF%BC%8CMAC%E5%86%99NTFS%E7%9A%84%E9%80%9F%E5%BA%A6%E4%BE%9D%E7%84%B6%E6%85%A2%EF%BC%8C%E4%B8%94%E6%9C%89%E4%B8%A2%E6%95%B0%E6%8D%AE%E9%A3%8E%E9%99%A9%20%EF%BC%8CNND%EF%BC%8CSB%E7%9A%84%E8%8B%B9%E6%9E%9C%E3%80%82%0A%0A%0A%E8%99%BD%E7%84%B6%E6%95%B0%E6%8D%AE%E4%B8%A2%E4%BA%86%EF%BC%8C%E4%BD%86%E4%B8%A2%E5%B0%B1%E4%B8%A2%E4%BA%86%E5%90%A7%EF%BC%8C%E6%9C%89%E4%BA%9B%E4%B8%8D%E6%80%8E%E4%B9%88%E7%9C%8B%EF%BC%8C%E8%BF%98%E8%83%BD%E4%B8%8B%E7%9A%84%EF%BC%8C%E5%86%8D%E9%87%8D%E6%96%B0%E6%89%BE%E6%89%BE%E8%B5%84%E6%BA%90%EF%BC%8C%E4%B8%8B%E4%B8%8D%E4%BA%86%E7%9A%84%E5%B0%B1%E7%AE%97%E4%BA%86%E3%80%82%E5%BD%93%E5%B9%B4%E6%89%8B%E6%9D%80%E7%86%8A%E7%8C%AB%E7%AE%97%E6%98%AF%E7%86%9F%E6%82%89%E4%BA%86WIN%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84%EF%BC%8C%E4%B8%8D%E5%81%9C%E7%9A%84%E9%87%8D%E5%81%9A%E7%B3%BB%E7%BB%9F%E6%90%9E%E5%9D%8F%E7%A1%AC%E7%9B%98%EF%BC%8C%E7%AE%97%E6%98%AF%E7%86%9F%E6%82%89%E4%BA%86WIN%E7%9A%84%E7%A1%AC%E7%9B%98%E6%93%8D%E4%BD%9C%EF%BC%8CMAC%E4%B8%8B%E4%B9%9F%E5%BE%97%E6%8D%9F%E5%A4%B1%E7%82%B9%EF%BC%8C%E4%B8%8D%E7%84%B6%E5%93%AA%E9%87%8C%E6%9D%A5%E7%9A%84%E7%86%9F%E6%82%89%E5%91%A2~%E5%BD%93%E6%98%AF%E5%AD%A6%E8%B4%B9%E5%90%A7%E3%80%82%0A%0A%E5%8F%A6%E5%A4%96%EF%BC%8CWIN%E4%B8%8B%E9%BA%BB%E7%83%A6%E7%82%B9%EF%BC%8C%E6%AF%8F%E6%AC%A1%E9%83%BD%E5%BE%97%E6%89%A7%E8%A1%8C%E4%B8%80%E4%B8%8B%E6%8C%87%E4%BB%A4%E6%89%8D%E8%83%BD%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%BD%86%E6%98%AF%E6%80%BB%E6%AF%94%E8%8A%B1%E9%92%B1%E8%A6%81%E5%BC%BA%EF%BC%8C%E8%80%8C%E4%B8%94%EF%BC%8C%E7%8E%B0%E5%9C%A8%E5%A4%A7%E9%83%A8%E5%88%86%E6%97%B6%E9%97%B4%E4%BD%BF%E7%94%A8MAC%E5%8A%9E%E5%85%AC%EF%BC%8C%E7%BB%8F%E5%B8%B8%E4%BC%A0%E4%B8%9C%E8%A5%BF%E4%B9%9F%E6%98%AFMAC%20%EF%BC%8C%E6%95%88%E6%9E%9C%E6%9B%B4%E5%A5%BD%E4%BA%9B%E3%80%82%0A
