# alfred

这个真你妈的是神器，就是一个搜索框，往里可以输入任何内容，还包括指令

不过有个硬伤：他是依赖mac自带的spotlight，而spotlight是依赖mds做索引，而mds这个吃CPU

## 初始化

1. 打开后，先申请下：系统权限
2. 设置下快捷键
3. 设置搜索目录
4. 添加百度web搜索条目

## 主要功能

1. 文件/目录 搜索
2. 快捷启动软件
3. 快速启动浏览器并搜索
4. 插件，嵌入各种插件方便日常使用，像 有道翻译、 UNIXSTAME时间转换、md5 等
5. 快速执行系统指令，如：shutdown restart gc quitAll等
6. 计算器

具体：

> 1、自定义搜索的目录，搜索任何东西
> 
> 
> 2、open 开头，可以搜索\+打开文件
> 
> 
> 3、find 开头，可以搜索 文件具体位置，在FIND中打开
> 
> 
> 4、baidu开头，可直接在默认浏览器中搜索 关键词
> 
> 
> 5、计算机，可直接在搜索框内 输入数字\+运算符号
> 
> 
> 6、define 开头，加任意单词，可翻译
> 
> 
> 7、spell 开头，想不起哪个单词，可帮助提示补全

## 日常 workflows 列表

|指令   |说明                              |
|-------|----------------------------------|
|yd     |快捷使用有道词典翻译              |
|hash   |md5 sha 加密串                    |
|time   |unix时间转换                      |
|github |快速在gt上查找项目或找开自己的项目|
|encode |url encode/decode 转换器          |
|NSC    |各种进制转换                      |
|codeVar|给变量起名                        |

## 日常OS操作指令

先：修改清空垃圾桶的快捷键为gc

|指令    |说明                  |
|--------|----------------------|
|gc      |清空回收站            |
|restart |重启                  |
|shutdown|关机                  |
|quitall |关闭当前所有已打开软件|
|about   |关于本机              |

设置搜索目录

```
/Applications
/Applications/Xcode.app/Contents/Applications
/Library/PreferencePanes
/System/Library/PreferencePanes
/usr/local/Cellar

/System/Applications
/data/www

```

## 有道

去有道云智能中，找配置信息

https://ai.youdao.com/\#/

手机号登陆

创建应用 文本翻译 勾选 接入方式：api

业务总览：

appid：

5b257a9c33bd6d51

应用密钥

1Yc6rq2LXuUSw83mMz2pEo8uorH6peN3

下载workflow:

https://github.com/wensonsmith/YoudaoTranslator/releases/download/3.1.0/YoudaoTranslator\-3.1.zip

拖入alfred中

点击右上角的 X 图标，选择 选项卡：环境变量，把appid 密钥 设置进去

## hash

https://github.com/willfarrell/alfred\-hash\-workflow

## TimeStamp

https://github.com/WiconWang/Alfred\-Workflows\-TimeStamp

time 2017\-03\-07 20:14:48

time 1488888888

time now

## github

https://github.com/gharlan/alfred\-github\-workflow/releases

## 创建文件

https://github.com/SteliosHa/Alfred\_File\_Creator

%E8%BF%99%E4%B8%AA%E7%9C%9F%E4%BD%A0%E5%A6%88%E7%9A%84%E6%98%AF%E7%A5%9E%E5%99%A8%EF%BC%8C%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E6%90%9C%E7%B4%A2%E6%A1%86%EF%BC%8C%E5%BE%80%E9%87%8C%E5%8F%AF%E4%BB%A5%E8%BE%93%E5%85%A5%E4%BB%BB%E4%BD%95%E5%86%85%E5%AE%B9%EF%BC%8C%E8%BF%98%E5%8C%85%E6%8B%AC%E6%8C%87%E4%BB%A4%0A%E4%B8%8D%E8%BF%87%E6%9C%89%E4%B8%AA%E7%A1%AC%E4%BC%A4%EF%BC%9A%E4%BB%96%E6%98%AF%E4%BE%9D%E8%B5%96mac%E8%87%AA%E5%B8%A6%E7%9A%84spotlight%EF%BC%8C%E8%80%8Cspotlight%E6%98%AF%E4%BE%9D%E8%B5%96mds%E5%81%9A%E7%B4%A2%E5%BC%95%EF%BC%8C%E8%80%8Cmds%E8%BF%99%E4%B8%AA%E5%90%83CPU%20%0A%0A%E5%88%9D%E5%A7%8B%E5%8C%96%0A\-\-\-\-%0A1.%20%E6%89%93%E5%BC%80%E5%90%8E%EF%BC%8C%E5%85%88%E7%94%B3%E8%AF%B7%E4%B8%8B%EF%BC%9A%E7%B3%BB%E7%BB%9F%E6%9D%83%E9%99%90%0A2.%20%E8%AE%BE%E7%BD%AE%E4%B8%8B%E5%BF%AB%E6%8D%B7%E9%94%AE%0A3.%20%E8%AE%BE%E7%BD%AE%E6%90%9C%E7%B4%A2%E7%9B%AE%E5%BD%95%0A4.%20%E6%B7%BB%E5%8A%A0%E7%99%BE%E5%BA%A6web%E6%90%9C%E7%B4%A2%E6%9D%A1%E7%9B%AE%0A%0A%E4%B8%BB%E8%A6%81%E5%8A%9F%E8%83%BD%0A\-\-\-\-%0A%0A1.%20%E6%96%87%E4%BB%B6%2F%E7%9B%AE%E5%BD%95%20%E6%90%9C%E7%B4%A2%0A2.%20%E5%BF%AB%E6%8D%B7%E5%90%AF%E5%8A%A8%E8%BD%AF%E4%BB%B6%0A4.%20%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B9%B6%E6%90%9C%E7%B4%A2%0A5.%20%E6%8F%92%E4%BB%B6%EF%BC%8C%E5%B5%8C%E5%85%A5%E5%90%84%E7%A7%8D%E6%8F%92%E4%BB%B6%E6%96%B9%E4%BE%BF%E6%97%A5%E5%B8%B8%E4%BD%BF%E7%94%A8%EF%BC%8C%E5%83%8F%20%E6%9C%89%E9%81%93%E7%BF%BB%E8%AF%91%E3%80%81%20UNIXSTAME%E6%97%B6%E9%97%B4%E8%BD%AC%E6%8D%A2%E3%80%81md5%20%E7%AD%89%0A6.%20%E5%BF%AB%E9%80%9F%E6%89%A7%E8%A1%8C%E7%B3%BB%E7%BB%9F%E6%8C%87%E4%BB%A4%EF%BC%8C%E5%A6%82%EF%BC%9Ashutdown%20restart%20gc%20quitAll%E7%AD%89%0A7.%20%E8%AE%A1%E7%AE%97%E5%99%A8%0A%0A%E5%85%B7%E4%BD%93%EF%BC%9A%0A%0A%3E1%E3%80%81%E8%87%AA%E5%AE%9A%E4%B9%89%E6%90%9C%E7%B4%A2%E7%9A%84%E7%9B%AE%E5%BD%95%EF%BC%8C%E6%90%9C%E7%B4%A2%E4%BB%BB%E4%BD%95%E4%B8%9C%E8%A5%BF%20%20%0A%3E2%E3%80%81open%20%E5%BC%80%E5%A4%B4%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%90%9C%E7%B4%A2%2B%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%20%20%0A%3E3%E3%80%81find%20%E5%BC%80%E5%A4%B4%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%90%9C%E7%B4%A2%20%E6%96%87%E4%BB%B6%E5%85%B7%E4%BD%93%E4%BD%8D%E7%BD%AE%EF%BC%8C%E5%9C%A8FIND%E4%B8%AD%E6%89%93%E5%BC%80%20%20%0A%3E4%E3%80%81baidu%E5%BC%80%E5%A4%B4%EF%BC%8C%E5%8F%AF%E7%9B%B4%E6%8E%A5%E5%9C%A8%E9%BB%98%E8%AE%A4%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%AD%E6%90%9C%E7%B4%A2%20%E5%85%B3%E9%94%AE%E8%AF%8D%20%20%0A%3E5%E3%80%81%E8%AE%A1%E7%AE%97%E6%9C%BA%EF%BC%8C%E5%8F%AF%E7%9B%B4%E6%8E%A5%E5%9C%A8%E6%90%9C%E7%B4%A2%E6%A1%86%E5%86%85%20%E8%BE%93%E5%85%A5%E6%95%B0%E5%AD%97%2B%E8%BF%90%E7%AE%97%E7%AC%A6%E5%8F%B7%20%20%0A%3E6%E3%80%81define%20%E5%BC%80%E5%A4%B4%EF%BC%8C%E5%8A%A0%E4%BB%BB%E6%84%8F%E5%8D%95%E8%AF%8D%EF%BC%8C%E5%8F%AF%E7%BF%BB%E8%AF%91%20%20%0A%3E7%E3%80%81spell%20%20%E5%BC%80%E5%A4%B4%EF%BC%8C%E6%83%B3%E4%B8%8D%E8%B5%B7%E5%93%AA%E4%B8%AA%E5%8D%95%E8%AF%8D%EF%BC%8C%E5%8F%AF%E5%B8%AE%E5%8A%A9%E6%8F%90%E7%A4%BA%E8%A1%A5%E5%85%A8%20%20%0A%0A%0A%E6%97%A5%E5%B8%B8%20workflows%20%E5%88%97%E8%A1%A8%0A\-\-\-\-%0A%7C%E6%8C%87%E4%BB%A4%7C%E8%AF%B4%E6%98%8E%7C%0A%7C\-\-\-\-%7C\-\-\-\-\-%7C%0A%7Cyd%7C%E5%BF%AB%E6%8D%B7%E4%BD%BF%E7%94%A8%E6%9C%89%E9%81%93%E8%AF%8D%E5%85%B8%E7%BF%BB%E8%AF%91%7C%0A%7Chash%7Cmd5%20sha%20%E5%8A%A0%E5%AF%86%E4%B8%B2%7C%0A%7Ctime%7Cunix%E6%97%B6%E9%97%B4%E8%BD%AC%E6%8D%A2%7C%0A%7Cgithub%7C%E5%BF%AB%E9%80%9F%E5%9C%A8gt%E4%B8%8A%E6%9F%A5%E6%89%BE%E9%A1%B9%E7%9B%AE%E6%88%96%E6%89%BE%E5%BC%80%E8%87%AA%E5%B7%B1%E7%9A%84%E9%A1%B9%E7%9B%AE%7C%0A%7Cencode%7Curl%20encode%2Fdecode%20%E8%BD%AC%E6%8D%A2%E5%99%A8%7C%0A%7CNSC%7C%E5%90%84%E7%A7%8D%E8%BF%9B%E5%88%B6%E8%BD%AC%E6%8D%A2%7C%0A%7CcodeVar%7C%E7%BB%99%E5%8F%98%E9%87%8F%E8%B5%B7%E5%90%8D%7C%0A%0A%0A%E6%97%A5%E5%B8%B8OS%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4%0A\-\-\-\-\-%0A%0A%E5%85%88%EF%BC%9A%E4%BF%AE%E6%94%B9%E6%B8%85%E7%A9%BA%E5%9E%83%E5%9C%BE%E6%A1%B6%E7%9A%84%E5%BF%AB%E6%8D%B7%E9%94%AE%E4%B8%BAgc%0A%0A%7C%E6%8C%87%E4%BB%A4%7C%E8%AF%B4%E6%98%8E%7C%0A%7C\-\-\-\-%7C\-\-\-\-\-%7C%0A%7Cgc%7C%E6%B8%85%E7%A9%BA%E5%9B%9E%E6%94%B6%E7%AB%99%7C%0A%7Crestart%7C%E9%87%8D%E5%90%AF%7C%0A%7Cshutdown%7C%E5%85%B3%E6%9C%BA%7C%0A%7Cquitall%7C%E5%85%B3%E9%97%AD%E5%BD%93%E5%89%8D%E6%89%80%E6%9C%89%E5%B7%B2%E6%89%93%E5%BC%80%E8%BD%AF%E4%BB%B6%7C%0A%7Cabout%7C%E5%85%B3%E4%BA%8E%E6%9C%AC%E6%9C%BA%7C%0A%0A%E8%AE%BE%E7%BD%AE%E6%90%9C%E7%B4%A2%E7%9B%AE%E5%BD%95%20%20%20%0A%60%60%60%0A%2FApplications%0A%2FApplications%2FXcode.app%2FContents%2FApplications%0A%2FLibrary%2FPreferencePanes%0A%2FSystem%2FLibrary%2FPreferencePanes%0A%2Fusr%2Flocal%2FCellar%0A%2FSystem%2FApplications%0A%2Fdata%2Fwww%0A%0A%0A%60%60%60%0A%0A%E6%9C%89%E9%81%93%0A\-\-\-%0A%E5%8E%BB%E6%9C%89%E9%81%93%E4%BA%91%E6%99%BA%E8%83%BD%E4%B8%AD%EF%BC%8C%E6%89%BE%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%0Ahttps%3A%2F%2Fai.youdao.com%2F%23%2F%0A%E6%89%8B%E6%9C%BA%E5%8F%B7%E7%99%BB%E9%99%86%0A%0A%E5%88%9B%E5%BB%BA%E5%BA%94%E7%94%A8%20%E6%96%87%E6%9C%AC%E7%BF%BB%E8%AF%91%20%E5%8B%BE%E9%80%89%20%E6%8E%A5%E5%85%A5%E6%96%B9%E5%BC%8F%EF%BC%9Aapi%0A%0A%E4%B8%9A%E5%8A%A1%E6%80%BB%E8%A7%88%EF%BC%9A%0Aappid%EF%BC%9A%0A5b257a9c33bd6d51%0A%E5%BA%94%E7%94%A8%E5%AF%86%E9%92%A5%0A1Yc6rq2LXuUSw83mMz2pEo8uorH6peN3%0A%0A%E4%B8%8B%E8%BD%BDworkflow%3A%0Ahttps%3A%2F%2Fgithub.com%2Fwensonsmith%2FYoudaoTranslator%2Freleases%2Fdownload%2F3.1.0%2FYoudaoTranslator\-3.1.zip%0A%0A%E6%8B%96%E5%85%A5alfred%E4%B8%AD%0A%E7%82%B9%E5%87%BB%E5%8F%B3%E4%B8%8A%E8%A7%92%E7%9A%84%20X%20%E5%9B%BE%E6%A0%87%EF%BC%8C%E9%80%89%E6%8B%A9%20%E9%80%89%E9%A1%B9%E5%8D%A1%EF%BC%9A%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%EF%BC%8C%E6%8A%8Aappid%20%E5%AF%86%E9%92%A5%20%E8%AE%BE%E7%BD%AE%E8%BF%9B%E5%8E%BB%0A%0Ahash%0A\-\-\-\-%0Ahttps%3A%2F%2Fgithub.com%2Fwillfarrell%2Falfred\-hash\-workflow%0A%0A%0A%0ATimeStamp%0A\-\-\-\-\-%0Ahttps%3A%2F%2Fgithub.com%2FWiconWang%2FAlfred\-Workflows\-TimeStamp%0A%0Atime%202017\-03\-07%2020%3A14%3A48%0Atime%201488888888%0Atime%20now%0A%0Agithub%0A\-\-\-%0Ahttps%3A%2F%2Fgithub.com%2Fgharlan%2Falfred\-github\-workflow%2Freleases%0A%0A%0A%E5%88%9B%E5%BB%BA%E6%96%87%E4%BB%B6%0A\-\-%0Ahttps%3A%2F%2Fgithub.com%2FSteliosHa%2FAlfred\_File\_Creator%0A%0A%0A
