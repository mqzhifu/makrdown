# hexo-博客

## 安装与体验

前置条件，需要已安装过：git 、node 、npm

先安装它的工具\-指令行

```
npm install -g hexo-cli
```

创建一个文件夹并进入：

```
>mkdir /data/www/golang/myblog
>cd /data/www/golang/myblog
```

安装：

```
>hexo init   #初始化，感觉就是下代码(必须得是一个空文件夹)
>npm install #安装，应该安装具体的库 改改配置文件之类的，不确定这一步是否可省略
```

生成静态文件：

> hexo g

开启守护进程:\(根目录下的 public 目录\)

> hexo s

打开网页，看看效果:

> http://localhost:4000/

创建一个篇文章（在 source/\_posts 目录下创建了一个文件）：

> hexo n 'my\-first\-blog'

查看刚刚创建的文章详细内容：

> vim /data/www/golang/myblog/source/\_posts/my\-first\-blog.md

清除 已生成的静态文件\(清空public目录\)：

> hexo clean

重启：\(3个指令的合集：先清空，再重新生成静态文件，最后启动服务\)

> hexo clean && hexo g && hexo s

## 创建分类与tag

创建一个分类:

> hexo new page categories

根据提示符打开文件：

> vim /data/www/golang/myblog/source/categories/index.md

```
type : categories
```

创建一个tags:

> hexo new page tags

根据提示符打开文件：

> vim /data/www/golang/myblog/source/tags/index.md

```
type: "tags"
```

修改配置文件：

> vim \_config.yml

```
category_dir: /categories/
tag_dir: /tags/
```

找一个文章修改，试试效果

> vim /data/www/golang/myblog/source/\_posts/my\-first\-blog.md

```
categories : web前端
tags : vue
```

## 创建\<关于\>页面

hexo new page "about"

vim config

```
menu:
  about: /about/ || fa fa-user
```

## hexo配置文件

vim \_config.yml

```
# Site
title: xiaoz - blog
subtitle: 'life detail'
description: 'Attentively records the good life'
keywords: "technology tech jobs job life worker internet golang php vue js"
author: xiaoz
#如果使用next 这个是旧版
language: zh-Hans 
#如果使用next 这个是新版
language: zh-CN 
timezone: 'Asia/Shanghai'
```

## 注意next版本

5.X版本

> https://github.com/iissnan/hexo\-theme\-next
> 
> 
> start=15.8

7.X版本

> https://github.com/theme\-next/hexo\-theme\-next
> 
> 
> start=7.7K

> 感觉变化挺大的，它直接把URL地址/版本库都给变了，不过新版的github星星略少

为什么我本次安装，会扯出来旧版？

从星星上看，旧版肯定用的人多。URL都换了，有点硬分叉的意思，那么可能就形成：各自版本都有使用的群体。而且旧版人多，多年积累下的文档肯定更多，所以我被节奏，全是旧版的安装方式了...

被坑

1. 文档，两个版本除了配置不一样，查文档时也容易混淆，造成配置不生效的情况
2. 因为两版差异大，第一次配置好的东西基本没用，还得再来一次

## next主题

hexo支持主题更换，官方主题地址：

> https://hexo.io/themes/

感觉这个官方主题地址有点形同虚设，因为：好的主题就那么2~3个，其中最火的就是：next......

这里我也选择:next主题，随大流不一定对，但肯定不会错，先下载主题，这里有两种方法：

方法1：

```
cd /data/www/golang/myblog
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

方法2：

```
cd /data/www/golang/myblog/themes/next
wget https://github.com/theme-next/hexo-theme-next/archive/refs/tags/v7.8.0.tar.gz
tar -zxvf v7.8.0.tar.gz
```

按说方法2更正规，但我看打包的时间 ：2020\-4月，而master的最后提交是：2021\-7月，算了正规的方法放弃了，用git clone吧，麻烦的就是最后统一提交GIT时：得把next的git删除了

修改主站配置:

vim \_config.yml

```
theme: next
```

修改next主题配置：

> vim /data/www/golang/myblog/themes/next/\_config.yml

貌似报错，得安装个：

npm i hexo\-renderer\-swig

hexo s \-\-debug

## next代码块配置

```
codeblock:
  highlight_theme: night bright #高亮 + 黑色背景
  copy_button:
    enable: true #复制按钮
```

## next添加头像

```
avatar:
  url: /images/avatar_fly2.jpeg
  rounded: true #圆形
```

## next社交链接\(侧边栏\)

```
social:
  GitHub: https://github.com/mqzhifu || fab fa-github
  E-Mail: mailto:mqzhifu@sina.com || fa fa-envelope
  Weibo: https://weibo.com/u/1806907257 || fab fa-weibo
```

## 给文章添加结尾话术

我看了看它的脚本：

> vim post.swig

貌似可配置的东西还挺多的，再回到主题配置文件，打开配置项

```
custom_file_path:
  postBodyEnd: source/_data/post-body-end.swig
```

mkdir /data/www/golang/myblog/source/\_data

touch /data/www/golang/myblog/source/\_data/post\-body\-end.swig

vim /data/www/golang/myblog/source/\_data/post\-body\-end.swig

```
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i ></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```

## 支持流程/甘特图

npm install hexo\-filter\-mermaid\-diagrams \-\-save

修改next主题配置

enable : true

## 添加统计

> vim /data/www/golang/myblog/themes/next/\_config.yml

```
busuanzi_count:
  enable: true
```

旧版的还需要改，新版的就改个配置参数就行了

vim /data/www/golang/myblog/themes/next/layout/\_third\-party/analytics/busuanzi\-counter.swig

```
<script async src="https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

## next添加搜索

```
cd /data/www/golang/myblog/
npm install hexo-generator-searchdb --save
```

vim themes/next/\_config.yml

```
local_search:
    enable: true
```

## 错误

旧版才有这个问题

转到next主题后，发现左侧菜单所有的链接多了一个：%20

vim themes/next/\_config.yml

搜索menu

把每个连接的空格删除了

## 隐藏网页底部无用信息

vim themes/next/layout/\_partials/footer.swig

```
footer:
    powered: false
  theme:
    enable: false
    version: false
```

## 右上角打开GITHUB连接

github\_banner:

enable: true

## next\-设置浏览器 favicon

favicon:

\#small: /images/favicon\-16x16\-next.png

\#medium: /images/favicon\-32x32\-next.png

## 动态背景

cd /Users/mayanyan/data/www/golang/zblog/themes/next

git clone https://github.com/theme\-next/theme\-next\-three source/lib/three

rm \-rf .git

```
three:
  enable: true
  three_waves: true
  canvas_lines: false
  canvas_sphere: false
```

坑B的，3个效果都死难看

cd /data/www/golang/zblog/themes/next

git clone https://github.com/theme\-next/theme\-next\-canvas\-nest source/lib/canvas\-nest

```
canvas_nest:
  enable: true
```

## 分享功能

npm install theme\-next/hexo\-next\-share

npm install \-g grunt \#这个我不确定是不是一定得安装

旧版：安装完成后，配置文件会多出一些配置，改成true 即可

新版：得手动添加一下

```
needmoreshare:
  enable: true
  cdn:
    js: //cdn.jsdelivr.net/gh/theme-next/theme-next-needmoreshare2@1/needsharebutton.min.js
    css: //cdn.jsdelivr.net/gh/theme-next/theme-next-needmoreshare2@1/needsharebutton.min.css
  postbottom:
    enable: true
    options:
      iconStyle: box
      boxForm: horizontal
      position: bottomCenter
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
  float:
    enable: false
    options:
      iconStyle: box
      boxForm: horizontal
      position: middleRight
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
```

## 打开收缩功能

旧版 直接修改配置即可

```
auto_excerpt:
  enable: true
  length: 150
```

> 就是MD格式没了，特别乱

新版

方法1：最简单的方式是用它默认的，改改配置文件即可

```
excerpt_description: true
read_more_btn: true
```

在文章的头部中的desc标签中：添加信息

或在内容中添加标签：

```
<!-- more -->
```

方法2：得安插插件 ：

> npm install hexo\-excerpt \-\-save

修改配置信息：

```
excerpt:
  depth: 5  
  excerpt_excludes: []
  more_excludes: []
  hideWholePostExcerpts: true
```

> 推荐方法2，主要改一个地方全文生效，省去麻烦

## 彩色标签

文章详情，尾部的标签加上icon图标\(默认是个\#号\)

tag\_icon: true

tag页面增加彩色：

> https://blog.csdn.net/qq\_39974578/article/details/114172260

创建一个配色文件：

> touch /data/www/golang/zblog/themes/next/layout/tag\-color.swig

添加配色代码：

> vim /data/www/golang/zblog/themes/next/layout/tag\-color.swig

vim themes/next/layout/page.swig

```
{% include 'tag-color.swig' %}
```

侧边栏增加tags：

npm install hexo\-tag\-cloud@^2.1.\* \-\-save

vim /data/www/golang/zblog/themes/next/layout/\_macro/sidebar.swig

```
{% if site.tags.length > 1 %}
<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcloud.js') }}"></script>
<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcanvas.js') }}"></script>
<div >
    <h3 >Tag Cloud</h3>
    <div >
        <canvas width="250" height="250" style="width:100%">
            {{ list_tags() }}
        </canvas>
    </div>
</div>
{% endif %}
```

```
# hexo-tag-cloud
tag_cloud:
    textFont: Trebuchet MS, Helvetica
    textColor: '#333'
    textHeight: 25
    outlineColor: '#E2E1D1'
    maxSpeed: 0.1
```

给文章底部tag 添加彩色:

vim /data/www/golang/zblog/themes/next/layout/\_macro/post.swig

```
 <div >
            {%- for tag in post.tags.toArray() %}
              <a   rel="tag"><i ></i> {{ tag.name }}</a>
            {%- endfor %}
          </div>
          <script type="text/javascript">
            var tagsall=document.getElementsByClassName("post-tags")
            for (var i = tagsall.length - 1; i >= 0; i--){
                var tags=tagsall[i].getElementsByTagName("a");
                for (var j = tags.length - 1; j >= 0; j--) {
                    var golden_ratio = 0.618033988749895;
                    var s = 0.5;
                    var v = 0.999;
                    var h = golden_ratio + Math.random()*0.8 - 0.5;
                    var h_i = parseInt(h * 6);
                    var f = h * 6 - h_i;
                    var p = v * (1 - s);
                    var q = v * (1 - f * s);
                    var t = v * (1 - (1 - f) * s);
                    var r, g, b;
                    switch (h_i) {
                        case 0:
                            r = v;
                            g = t;
                            b = p;
                            break;
                        case 1:
                            r = q;
                            g = v;
                            b = p;
                            break;
                        case 2:
                            r = p;
                            g = v;
                            b = t;
                            break;
                        case 3 :
                            r = p;
                            g = q;
                            b = v;
                            break;
                        case 4:
                            r = t;
                            g = p;
                            b = v;
                            break;
                        case 5:
                            r = v;
                            g = p;
                            b = q;
                            break;
                        default:
                            r = 1;
                            g = 1;
                            b = 1;
                      }
                    tags[j].style.background = "rgba("+parseInt(r*255)+","+parseInt(g*255)+","+parseInt(b*255)+","+0.5+")";
                }
            }                        
            </script>

```

```
<style>

.posts-expand .post-tags a {
    display: inline-block;
    font-size: 0.8em;
    padding: 0px 10px;
    border-radius: 8px;
    color: rgb(85, 85, 85);
    border: 0px;
}

</style>
```

vim /data/www/golang/zblog/themes/next/\_config.yml

```
<span >
    ${iconText('far fa-comment', 'valine','评论数')}
    <a title="valine"   itemprop="discussionUrl">
      <span data-xitemprop="commentCount"></span>
    </a>
  </span>
```

## 侧边栏添加分隔线

/Users/mayanyan/data/www/golang/zblog/themes/next/source/css/\_common/components/post/post\-eof.styl

```
.sidebar-eof {
  background: $grey-light;
  
  text-align: center;
  width: 100%;
}
```

## 浏览进度

scrollpercent: true

## 评论

先去LeanCloud注册个账号，再创建个应用，拿到appId appKey，设置到配置文件中

valine

文章详情页：标题下面的评论数非中文，有点BUG，改一下

vim themes/next/scripts/filters/comment/valine.js

## 添加图片列表功能

创建一个文件：/data/www/golang/zblog/themes/next/scripts/show\_some.js

```
var fs = require("fs");

hexo.extend.tag.register('show_some', function(args){

    if(!args[0]){
	return false;
    }

    var listStr = "";
    const files = fs.readdirSync(args[0]);
    const prefix = args[1];

    files.forEach(function (item, index) {
        var thisPicLink = prefix+item;
        var divStyle = "float:left";
        var oneRow = "<div  style='margin-right:10px;"+divStyle+"'><div>"+item+"</div>"

        oneRow += "<div><a href='"+thisPicLink+"' target='_blank' >"  +"<img src='" + thisPicLink + "' width=200 height=400 > </a> </div>";
        //display: inline-block
        oneRow += "</div>";
        listStr += oneRow;
    })

  listStr += "<div style='clear: left'></div>";

    return listStr ;
});

```

图片排队样式

vim \_common/scaffolding/tags/group\-pictures.styl

```
.page-post-detail .post-body .group-picture-column {
  // float: none;
  margin-top: 10px;
  // width: auto !important;
  img { margin: 0 auto; }
}

```

最后哪里使用，哪里引用 ：

> {% show\_some /data/www/golang/zblog/source/images/脑图/ /images/脑图/ %}

## 快速重启

hexo clean && hexo g && hexo s

## 尾部设置

```
footer:
  # Specify the date when the site was setup. If not defined, current year will be used.
  since: 2022

  # Icon between year and copyright info.
  icon:
    # Icon name in Font Awesome. See: https://fontawesome.com/icons
    #name: fa fa-heart
    name: fa fa-bug
    # If you want to animate the icon, set it to true.
    animated: true
    # Change the color of icon, using Hex Code.
    color: "#ff0000"
    #color: "#000000"
```

## 收尾，加入git仓

发现next主题是带着git版本的，不能直接把全部代码加入到git里

git status 看看是什么情况？

发现：除了配置文件，还有一个：头像图片有变动

先把配置文件备份下，再备份下图片

现在

git上创建一个项目,clone 一下代码：

> cd /data/www/golang
> 
> 
> git clone git@github.com:mqzhifu/zblog.git

## 总结

虽然说是hexo博客，但是对hexo的配置很少，大部分都是在调整next主题。而修改next主题就是修改页面

%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%93%E9%AA%8C%0A\-\-\-\-%0A%E5%89%8D%E7%BD%AE%E6%9D%A1%E4%BB%B6%EF%BC%8C%E9%9C%80%E8%A6%81%E5%B7%B2%E5%AE%89%E8%A3%85%E8%BF%87%EF%BC%9Agit%20%E3%80%81node%20%E3%80%81npm%0A%0A%E5%85%88%E5%AE%89%E8%A3%85%E5%AE%83%E7%9A%84%E5%B7%A5%E5%85%B7\-%E6%8C%87%E4%BB%A4%E8%A1%8C%0A%60%60%60%0Anpm%20install%20\-g%20hexo\-cli%0A%60%60%60%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%B9%B6%E8%BF%9B%E5%85%A5%EF%BC%9A%0A%60%60%60%0A%3Emkdir%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%0A%3Ecd%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%0A%60%60%60%0A%E5%AE%89%E8%A3%85%EF%BC%9A%0A%60%60%60%0A%3Ehexo%20init%20%20%20%23%E5%88%9D%E5%A7%8B%E5%8C%96%EF%BC%8C%E6%84%9F%E8%A7%89%E5%B0%B1%E6%98%AF%E4%B8%8B%E4%BB%A3%E7%A0%81\(%E5%BF%85%E9%A1%BB%E5%BE%97%E6%98%AF%E4%B8%80%E4%B8%AA%E7%A9%BA%E6%96%87%E4%BB%B6%E5%A4%B9\)%0A%3Enpm%20install%20%23%E5%AE%89%E8%A3%85%EF%BC%8C%E5%BA%94%E8%AF%A5%E5%AE%89%E8%A3%85%E5%85%B7%E4%BD%93%E7%9A%84%E5%BA%93%20%E6%94%B9%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B9%8B%E7%B1%BB%E7%9A%84%EF%BC%8C%E4%B8%8D%E7%A1%AE%E5%AE%9A%E8%BF%99%E4%B8%80%E6%AD%A5%E6%98%AF%E5%90%A6%E5%8F%AF%E7%9C%81%E7%95%A5%0A%60%60%60%0A%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%EF%BC%9A%0A%3Ehexo%20g%20%20%0A%0A%E5%BC%80%E5%90%AF%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%3A\(%E6%A0%B9%E7%9B%AE%E5%BD%95%E4%B8%8B%E7%9A%84%20public%20%E7%9B%AE%E5%BD%95\)%0A%3Ehexo%20s%20%20%0A%0A%E6%89%93%E5%BC%80%E7%BD%91%E9%A1%B5%EF%BC%8C%E7%9C%8B%E7%9C%8B%E6%95%88%E6%9E%9C%3A%0A%3Ehttp%3A%2F%2Flocalhost%3A4000%2F%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%AF%87%E6%96%87%E7%AB%A0%EF%BC%88%E5%9C%A8%20source%2F\_posts%20%E7%9B%AE%E5%BD%95%E4%B8%8B%E5%88%9B%E5%BB%BA%E4%BA%86%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%EF%BC%89%EF%BC%9A%0A%3Ehexo%20n%20'my\-first\-blog'%0A%0A%E6%9F%A5%E7%9C%8B%E5%88%9A%E5%88%9A%E5%88%9B%E5%BB%BA%E7%9A%84%E6%96%87%E7%AB%A0%E8%AF%A6%E7%BB%86%E5%86%85%E5%AE%B9%EF%BC%9A%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2F\_posts%2Fmy\-first\-blog.md%0A%0A%E6%B8%85%E9%99%A4%20%E5%B7%B2%E7%94%9F%E6%88%90%E7%9A%84%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6\(%E6%B8%85%E7%A9%BApublic%E7%9B%AE%E5%BD%95\)%EF%BC%9A%0A%3Ehexo%20clean%0A%0A%E9%87%8D%E5%90%AF%EF%BC%9A\(3%E4%B8%AA%E6%8C%87%E4%BB%A4%E7%9A%84%E5%90%88%E9%9B%86%EF%BC%9A%E5%85%88%E6%B8%85%E7%A9%BA%EF%BC%8C%E5%86%8D%E9%87%8D%E6%96%B0%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%EF%BC%8C%E6%9C%80%E5%90%8E%E5%90%AF%E5%8A%A8%E6%9C%8D%E5%8A%A1\)%0A%3Ehexo%20clean%20%26%26%20hexo%20g%20%26%26%20hexo%20s%0A%0A%0A%0A%E5%88%9B%E5%BB%BA%E5%88%86%E7%B1%BB%E4%B8%8Etag%0A\-\-\-\-%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%88%86%E7%B1%BB%3A%0A%3Ehexo%20new%20page%20categories%0A%0A%E6%A0%B9%E6%8D%AE%E6%8F%90%E7%A4%BA%E7%AC%A6%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%EF%BC%9A%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2Fcategories%2Findex.md%0A%60%60%60%0Atype%20%3A%20categories%0A%60%60%60%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAtags%3A%0A%3Ehexo%20new%20page%20tags%0A%0A%E6%A0%B9%E6%8D%AE%E6%8F%90%E7%A4%BA%E7%AC%A6%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%EF%BC%9A%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2Ftags%2Findex.md%0A%60%60%60%0Atype%3A%20%22tags%22%0A%60%60%60%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%9A%0A%3Evim%20\_config.yml%0A%60%60%60%0Acategory\_dir%3A%20%2Fcategories%2F%0Atag\_dir%3A%20%2Ftags%2F%0A%60%60%60%0A%E6%89%BE%E4%B8%80%E4%B8%AA%E6%96%87%E7%AB%A0%E4%BF%AE%E6%94%B9%EF%BC%8C%E8%AF%95%E8%AF%95%E6%95%88%E6%9E%9C%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2F\_posts%2Fmy\-first\-blog.md%0A%60%60%60%0Acategories%20%3A%20web%E5%89%8D%E7%AB%AF%0Atags%20%3A%20vue%0A%60%60%60%0A%0A%0A%E5%88%9B%E5%BB%BA%3C%E5%85%B3%E4%BA%8E%3E%E9%A1%B5%E9%9D%A2%0A\-\-\-%0Ahexo%20new%20page%20%22about%22%0Avim%20config%0A%60%60%60%0Amenu%3A%0A%20%20about%3A%20%2Fabout%2F%20%7C%7C%20fa%20fa\-user%0A%60%60%60%0A%0A%0A%0Ahexo%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A\-\-\-%0Avim%20\_config.yml%0A%0A%60%60%60%0A%23%20Site%0Atitle%3A%20xiaoz%20\-%20blog%0Asubtitle%3A%20'life%20detail'%0Adescription%3A%20'Attentively%20records%20the%20good%20life'%0Akeywords%3A%20%22technology%20tech%20jobs%20job%20life%20worker%20internet%20golang%20php%20vue%20js%22%0Aauthor%3A%20xiaoz%0A%23%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8next%20%E8%BF%99%E4%B8%AA%E6%98%AF%E6%97%A7%E7%89%88%0Alanguage%3A%20zh\-Hans%20%0A%23%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8next%20%E8%BF%99%E4%B8%AA%E6%98%AF%E6%96%B0%E7%89%88%0Alanguage%3A%20zh\-CN%20%0Atimezone%3A%20'Asia%2FShanghai'%0A%60%60%60%0A%0A%E6%B3%A8%E6%84%8Fnext%E7%89%88%E6%9C%AC%0A\-\-\-\-%0A5.X%E7%89%88%E6%9C%AC%0A%3Ehttps%3A%2F%2Fgithub.com%2Fiissnan%2Fhexo\-theme\-next%0A%3Estart%3D15.8%0A%0A7.X%E7%89%88%E6%9C%AC%0A%3Ehttps%3A%2F%2Fgithub.com%2Ftheme\-next%2Fhexo\-theme\-next%0A%3Estart%3D7.7K%0A%0A%3E%E6%84%9F%E8%A7%89%E5%8F%98%E5%8C%96%E6%8C%BA%E5%A4%A7%E7%9A%84%EF%BC%8C%E5%AE%83%E7%9B%B4%E6%8E%A5%E6%8A%8AURL%E5%9C%B0%E5%9D%80%2F%E7%89%88%E6%9C%AC%E5%BA%93%E9%83%BD%E7%BB%99%E5%8F%98%E4%BA%86%EF%BC%8C%E4%B8%8D%E8%BF%87%E6%96%B0%E7%89%88%E7%9A%84github%E6%98%9F%E6%98%9F%E7%95%A5%E5%B0%91%0A%0A%E4%B8%BA%E4%BB%80%E4%B9%88%E6%88%91%E6%9C%AC%E6%AC%A1%E5%AE%89%E8%A3%85%EF%BC%8C%E4%BC%9A%E6%89%AF%E5%87%BA%E6%9D%A5%E6%97%A7%E7%89%88%EF%BC%9F%0A%E4%BB%8E%E6%98%9F%E6%98%9F%E4%B8%8A%E7%9C%8B%EF%BC%8C%E6%97%A7%E7%89%88%E8%82%AF%E5%AE%9A%E7%94%A8%E7%9A%84%E4%BA%BA%E5%A4%9A%E3%80%82URL%E9%83%BD%E6%8D%A2%E4%BA%86%EF%BC%8C%E6%9C%89%E7%82%B9%E7%A1%AC%E5%88%86%E5%8F%89%E7%9A%84%E6%84%8F%E6%80%9D%EF%BC%8C%E9%82%A3%E4%B9%88%E5%8F%AF%E8%83%BD%E5%B0%B1%E5%BD%A2%E6%88%90%EF%BC%9A%E5%90%84%E8%87%AA%E7%89%88%E6%9C%AC%E9%83%BD%E6%9C%89%E4%BD%BF%E7%94%A8%E7%9A%84%E7%BE%A4%E4%BD%93%E3%80%82%E8%80%8C%E4%B8%94%E6%97%A7%E7%89%88%E4%BA%BA%E5%A4%9A%EF%BC%8C%E5%A4%9A%E5%B9%B4%E7%A7%AF%E7%B4%AF%E4%B8%8B%E7%9A%84%E6%96%87%E6%A1%A3%E8%82%AF%E5%AE%9A%E6%9B%B4%E5%A4%9A%EF%BC%8C%E6%89%80%E4%BB%A5%E6%88%91%E8%A2%AB%E8%8A%82%E5%A5%8F%EF%BC%8C%E5%85%A8%E6%98%AF%E6%97%A7%E7%89%88%E7%9A%84%E5%AE%89%E8%A3%85%E6%96%B9%E5%BC%8F%E4%BA%86...%0A%0A%E8%A2%AB%E5%9D%91%0A1.%20%E6%96%87%E6%A1%A3%EF%BC%8C%E4%B8%A4%E4%B8%AA%E7%89%88%E6%9C%AC%E9%99%A4%E4%BA%86%E9%85%8D%E7%BD%AE%E4%B8%8D%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%9F%A5%E6%96%87%E6%A1%A3%E6%97%B6%E4%B9%9F%E5%AE%B9%E6%98%93%E6%B7%B7%E6%B7%86%EF%BC%8C%E9%80%A0%E6%88%90%E9%85%8D%E7%BD%AE%E4%B8%8D%E7%94%9F%E6%95%88%E7%9A%84%E6%83%85%E5%86%B5%0A2.%20%E5%9B%A0%E4%B8%BA%E4%B8%A4%E7%89%88%E5%B7%AE%E5%BC%82%E5%A4%A7%EF%BC%8C%E7%AC%AC%E4%B8%80%E6%AC%A1%E9%85%8D%E7%BD%AE%E5%A5%BD%E7%9A%84%E4%B8%9C%E8%A5%BF%E5%9F%BA%E6%9C%AC%E6%B2%A1%E7%94%A8%EF%BC%8C%E8%BF%98%E5%BE%97%E5%86%8D%E6%9D%A5%E4%B8%80%E6%AC%A1%0A%0A%0A%0A%0Anext%E4%B8%BB%E9%A2%98%0A\-\-\-\-%0Ahexo%E6%94%AF%E6%8C%81%E4%B8%BB%E9%A2%98%E6%9B%B4%E6%8D%A2%EF%BC%8C%E5%AE%98%E6%96%B9%E4%B8%BB%E9%A2%98%E5%9C%B0%E5%9D%80%EF%BC%9A%0A%3Ehttps%3A%2F%2Fhexo.io%2Fthemes%2F%0A%0A%E6%84%9F%E8%A7%89%E8%BF%99%E4%B8%AA%E5%AE%98%E6%96%B9%E4%B8%BB%E9%A2%98%E5%9C%B0%E5%9D%80%E6%9C%89%E7%82%B9%E5%BD%A2%E5%90%8C%E8%99%9A%E8%AE%BE%EF%BC%8C%E5%9B%A0%E4%B8%BA%EF%BC%9A%E5%A5%BD%E7%9A%84%E4%B8%BB%E9%A2%98%E5%B0%B1%E9%82%A3%E4%B9%882~3%E4%B8%AA%EF%BC%8C%E5%85%B6%E4%B8%AD%E6%9C%80%E7%81%AB%E7%9A%84%E5%B0%B1%E6%98%AF%EF%BC%9Anext......%0A%0A%E8%BF%99%E9%87%8C%E6%88%91%E4%B9%9F%E9%80%89%E6%8B%A9%3Anext%E4%B8%BB%E9%A2%98%EF%BC%8C%E9%9A%8F%E5%A4%A7%E6%B5%81%E4%B8%8D%E4%B8%80%E5%AE%9A%E5%AF%B9%EF%BC%8C%E4%BD%86%E8%82%AF%E5%AE%9A%E4%B8%8D%E4%BC%9A%E9%94%99%EF%BC%8C%E5%85%88%E4%B8%8B%E8%BD%BD%E4%B8%BB%E9%A2%98%EF%BC%8C%E8%BF%99%E9%87%8C%E6%9C%89%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%B3%95%EF%BC%9A%0A%E6%96%B9%E6%B3%951%EF%BC%9A%0A%60%60%60%0Acd%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%0Agit%20clone%20https%3A%2F%2Fgithub.com%2Ftheme\-next%2Fhexo\-theme\-next%20themes%2Fnext%0A%60%60%60%0A%E6%96%B9%E6%B3%952%EF%BC%9A%0A%60%60%60%0Acd%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fthemes%2Fnext%0Awget%20https%3A%2F%2Fgithub.com%2Ftheme\-next%2Fhexo\-theme\-next%2Farchive%2Frefs%2Ftags%2Fv7.8.0.tar.gz%0Atar%20\-zxvf%20v7.8.0.tar.gz%0A%60%60%60%0A%0A%E6%8C%89%E8%AF%B4%E6%96%B9%E6%B3%952%E6%9B%B4%E6%AD%A3%E8%A7%84%EF%BC%8C%E4%BD%86%E6%88%91%E7%9C%8B%E6%89%93%E5%8C%85%E7%9A%84%E6%97%B6%E9%97%B4%20%EF%BC%9A2020\-4%E6%9C%88%EF%BC%8C%E8%80%8Cmaster%E7%9A%84%E6%9C%80%E5%90%8E%E6%8F%90%E4%BA%A4%E6%98%AF%EF%BC%9A2021\-7%E6%9C%88%EF%BC%8C%E7%AE%97%E4%BA%86%E6%AD%A3%E8%A7%84%E7%9A%84%E6%96%B9%E6%B3%95%E6%94%BE%E5%BC%83%E4%BA%86%EF%BC%8C%E7%94%A8git%20clone%E5%90%A7%EF%BC%8C%E9%BA%BB%E7%83%A6%E7%9A%84%E5%B0%B1%E6%98%AF%E6%9C%80%E5%90%8E%E7%BB%9F%E4%B8%80%E6%8F%90%E4%BA%A4GIT%E6%97%B6%EF%BC%9A%E5%BE%97%E6%8A%8Anext%E7%9A%84git%E5%88%A0%E9%99%A4%E4%BA%86%0A%0A%0A%0A%E4%BF%AE%E6%94%B9%E4%B8%BB%E7%AB%99%E9%85%8D%E7%BD%AE%3A%0Avim%20\_config.yml%0A%60%60%60%0Atheme%3A%20next%0A%60%60%60%0A%0A%E4%BF%AE%E6%94%B9next%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%EF%BC%9A%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fthemes%2Fnext%2F\_config.yml%0A%0A%E8%B2%8C%E4%BC%BC%E6%8A%A5%E9%94%99%EF%BC%8C%E5%BE%97%E5%AE%89%E8%A3%85%E4%B8%AA%EF%BC%9A%0Anpm%20i%20hexo\-renderer\-swig%0A%20%0Ahexo%20s%20\-\-debug%0A%0A%0A%0Anext%E4%BB%A3%E7%A0%81%E5%9D%97%E9%85%8D%E7%BD%AE%0A\-\-\-\-%0A%60%60%60%0Acodeblock%3A%0A%20%20highlight\_theme%3A%20night%20bright%20%23%E9%AB%98%E4%BA%AE%20%2B%20%E9%BB%91%E8%89%B2%E8%83%8C%E6%99%AF%0A%20%20copy\_button%3A%0A%20%20%20%20enable%3A%20true%20%23%E5%A4%8D%E5%88%B6%E6%8C%89%E9%92%AE%0A%60%60%60%0A%0A%0Anext%E6%B7%BB%E5%8A%A0%E5%A4%B4%E5%83%8F%0A\-\-\-%0A%60%60%60%0Aavatar%3A%0A%20%20url%3A%20%2Fimages%2Favatar\_fly2.jpeg%0A%20%20rounded%3A%20true%20%23%E5%9C%86%E5%BD%A2%0A%60%60%60%0A%0Anext%E7%A4%BE%E4%BA%A4%E9%93%BE%E6%8E%A5\(%E4%BE%A7%E8%BE%B9%E6%A0%8F\)%0A\-\-\-\-%0A%60%60%60%0Asocial%3A%0A%20%20GitHub%3A%20https%3A%2F%2Fgithub.com%2Fmqzhifu%20%7C%7C%20fab%20fa\-github%0A%20%20E\-Mail%3A%20mailto%3Amqzhifu%40sina.com%20%7C%7C%20fa%20fa\-envelope%0A%20%20Weibo%3A%20https%3A%2F%2Fweibo.com%2Fu%2F1806907257%20%7C%7C%20fab%20fa\-weibo%0A%60%60%60%0A%0A%E7%BB%99%E6%96%87%E7%AB%A0%E6%B7%BB%E5%8A%A0%E7%BB%93%E5%B0%BE%E8%AF%9D%E6%9C%AF%0A\-\-\-%0A%E6%88%91%E7%9C%8B%E4%BA%86%E7%9C%8B%E5%AE%83%E7%9A%84%E8%84%9A%E6%9C%AC%EF%BC%9A%0A%3Evim%20post.swig%0A%0A%E8%B2%8C%E4%BC%BC%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%9A%84%E4%B8%9C%E8%A5%BF%E8%BF%98%E6%8C%BA%E5%A4%9A%E7%9A%84%EF%BC%8C%E5%86%8D%E5%9B%9E%E5%88%B0%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E6%89%93%E5%BC%80%E9%85%8D%E7%BD%AE%E9%A1%B9%0A%60%60%60%0Acustom\_file\_path%3A%0A%20%20postBodyEnd%3A%20source%2F\_data%2Fpost\-body\-end.swig%0A%60%60%60%0A%0Amkdir%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2F\_data%0Atouch%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2F\_data%2Fpost\-body\-end.swig%0Avim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fsource%2F\_data%2Fpost\-body\-end.swig%0A%60%60%60%0A%3Cdiv%3E%0A%20%20%20%20%7B%25%20if%20not%20is\_index%20%25%7D%0A%20%20%20%20%20%20%20%20%3Cdiv%20style%3D%22text\-align%3Acenter%3Bcolor%3A%20%23ccc%3Bfont\-size%3A14px%3B%22%3E\-\-\-\-\-\-\-\-\-\-\-\-\-%E6%9C%AC%E6%96%87%E7%BB%93%E6%9D%9F%3Ci%20class%3D%22fa%20fa\-hand\-peace\-o%22%3E%3C%2Fi%3E%E6%84%9F%E8%B0%A2%E6%82%A8%E7%9A%84%E9%98%85%E8%AF%BB\-\-\-\-\-\-\-\-\-\-\-\-\-%3C%2Fdiv%3E%0A%20%20%20%20%7B%25%20endif%20%25%7D%0A%3C%2Fdiv%3E%0A%60%60%60%0A%0A%E6%94%AF%E6%8C%81%E6%B5%81%E7%A8%8B%2F%E7%94%98%E7%89%B9%E5%9B%BE%0A\-\-\-%0Anpm%20install%20hexo\-filter\-mermaid\-diagrams%20\-\-save%0A%0A%E4%BF%AE%E6%94%B9next%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%0Aenable%20%3A%20true%0A%0A%E6%B7%BB%E5%8A%A0%E7%BB%9F%E8%AE%A1%0A\-\-\-\-%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fthemes%2Fnext%2F\_config.yml%0A%60%60%60%0Abusuanzi\_count%3A%0A%20%20enable%3A%20true%0A%60%60%60%0A%0A%E6%97%A7%E7%89%88%E7%9A%84%E8%BF%98%E9%9C%80%E8%A6%81%E6%94%B9%EF%BC%8C%E6%96%B0%E7%89%88%E7%9A%84%E5%B0%B1%E6%94%B9%E4%B8%AA%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%B0%B1%E8%A1%8C%E4%BA%86%0Avim%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2Fthemes%2Fnext%2Flayout%2F\_third\-party%2Fanalytics%2Fbusuanzi\-counter.swig%0A%60%60%60%0A%3Cscript%20async%20src%3D%22https%3A%2F%2Fbusuanzi.ibruce.info%2Fbusuanzi%2F2.3%2Fbusuanzi.pure.mini.js%22%3E%3C%2Fscript%3E%0A%60%60%60%0A%0Anext%E6%B7%BB%E5%8A%A0%E6%90%9C%E7%B4%A2%0A\-\-\-%0A%60%60%60%0Acd%20%2Fdata%2Fwww%2Fgolang%2Fmyblog%2F%0Anpm%20install%20hexo\-generator\-searchdb%20\-\-save%0A%60%60%60%0Avim%20themes%2Fnext%2F\_config.yml%0A%60%60%60%0Alocal\_search%3A%0A%20%20%20%20enable%3A%20true%0A%60%60%60%0A%0A%E9%94%99%E8%AF%AF%0A\-\-\-\-%0A%E6%97%A7%E7%89%88%E6%89%8D%E6%9C%89%E8%BF%99%E4%B8%AA%E9%97%AE%E9%A2%98%0A%0A%E8%BD%AC%E5%88%B0next%E4%B8%BB%E9%A2%98%E5%90%8E%EF%BC%8C%E5%8F%91%E7%8E%B0%E5%B7%A6%E4%BE%A7%E8%8F%9C%E5%8D%95%E6%89%80%E6%9C%89%E7%9A%84%E9%93%BE%E6%8E%A5%E5%A4%9A%E4%BA%86%E4%B8%80%E4%B8%AA%EF%BC%9A%2520%0Avim%20themes%2Fnext%2F\_config.yml%0A%E6%90%9C%E7%B4%A2menu%0A%E6%8A%8A%E6%AF%8F%E4%B8%AA%E8%BF%9E%E6%8E%A5%E7%9A%84%E7%A9%BA%E6%A0%BC%E5%88%A0%E9%99%A4%E4%BA%86%0A%0A%0A%E9%9A%90%E8%97%8F%E7%BD%91%E9%A1%B5%E5%BA%95%E9%83%A8%E6%97%A0%E7%94%A8%E4%BF%A1%E6%81%AF%0A\-\-\-\-%0Avim%20themes%2Fnext%2Flayout%2F\_partials%2Ffooter.swig%0A%60%60%60%0Afooter%3A%0A%20%20%20%20powered%3A%20false%0A%20%20theme%3A%0A%20%20%20%20enable%3A%20false%0A%20%20%20%20version%3A%20false%0A%60%60%60%0A%0A%E5%8F%B3%E4%B8%8A%E8%A7%92%E6%89%93%E5%BC%80GITHUB%E8%BF%9E%E6%8E%A5%0A\-\-\-%0Agithub\_banner%3A%0A%20%20enable%3A%20true%0A%20%20%0Anext\-%E8%AE%BE%E7%BD%AE%E6%B5%8F%E8%A7%88%E5%99%A8%20favicon%0A\-\-\-\-%0Afavicon%3A%0A%20%20%23small%3A%20%2Fimages%2Ffavicon\-16x16\-next.png%0A%20%20%23medium%3A%20%2Fimages%2Ffavicon\-32x32\-next.png%20%20%0A%0A%0A%E5%8A%A8%E6%80%81%E8%83%8C%E6%99%AF%0A\-\-\-\-%0Acd%20%2FUsers%2Fmayanyan%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%0Agit%20clone%20https%3A%2F%2Fgithub.com%2Ftheme\-next%2Ftheme\-next\-three%20source%2Flib%2Fthree%0Arm%20\-rf%20.git%0A%0A%60%60%60%0Athree%3A%0A%20%20enable%3A%20true%0A%20%20three\_waves%3A%20true%0A%20%20canvas\_lines%3A%20false%0A%20%20canvas\_sphere%3A%20false%0A%60%60%60%0A%0A%E5%9D%91B%E7%9A%84%EF%BC%8C3%E4%B8%AA%E6%95%88%E6%9E%9C%E9%83%BD%E6%AD%BB%E9%9A%BE%E7%9C%8B%0A%0Acd%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%0Agit%20clone%20https%3A%2F%2Fgithub.com%2Ftheme\-next%2Ftheme\-next\-canvas\-nest%20source%2Flib%2Fcanvas\-nest%0A%60%60%60%0Acanvas\_nest%3A%0A%20%20enable%3A%20true%0A%60%60%60%20%20%0A%0A%E5%88%86%E4%BA%AB%E5%8A%9F%E8%83%BD%0A\-\-\-%0Anpm%20install%20theme\-next%2Fhexo\-next\-share%0Anpm%20install%20\-g%20grunt%20%23%E8%BF%99%E4%B8%AA%E6%88%91%E4%B8%8D%E7%A1%AE%E5%AE%9A%E6%98%AF%E4%B8%8D%E6%98%AF%E4%B8%80%E5%AE%9A%E5%BE%97%E5%AE%89%E8%A3%85%0A%0A%E6%97%A7%E7%89%88%EF%BC%9A%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BC%9A%E5%A4%9A%E5%87%BA%E4%B8%80%E4%BA%9B%E9%85%8D%E7%BD%AE%EF%BC%8C%E6%94%B9%E6%88%90true%20%E5%8D%B3%E5%8F%AF%0A%E6%96%B0%E7%89%88%EF%BC%9A%E5%BE%97%E6%89%8B%E5%8A%A8%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%8B%0A%60%60%60%0Aneedmoreshare%3A%0A%20%20enable%3A%20true%0A%20%20cdn%3A%0A%20%20%20%20js%3A%20%2F%2Fcdn.jsdelivr.net%2Fgh%2Ftheme\-next%2Ftheme\-next\-needmoreshare2%401%2Fneedsharebutton.min.js%0A%20%20%20%20css%3A%20%2F%2Fcdn.jsdelivr.net%2Fgh%2Ftheme\-next%2Ftheme\-next\-needmoreshare2%401%2Fneedsharebutton.min.css%0A%20%20postbottom%3A%0A%20%20%20%20enable%3A%20true%0A%20%20%20%20options%3A%0A%20%20%20%20%20%20iconStyle%3A%20box%0A%20%20%20%20%20%20boxForm%3A%20horizontal%0A%20%20%20%20%20%20position%3A%20bottomCenter%0A%20%20%20%20%20%20networks%3A%20Weibo%2CWechat%2CDouban%2CQQZone%2CTwitter%2CFacebook%0A%20%20float%3A%0A%20%20%20%20enable%3A%20false%0A%20%20%20%20options%3A%0A%20%20%20%20%20%20iconStyle%3A%20box%0A%20%20%20%20%20%20boxForm%3A%20horizontal%0A%20%20%20%20%20%20position%3A%20middleRight%0A%20%20%20%20%20%20networks%3A%20Weibo%2CWechat%2CDouban%2CQQZone%2CTwitter%2CFacebook%0A%60%60%60%0A%0A%0A%E6%89%93%E5%BC%80%E6%94%B6%E7%BC%A9%E5%8A%9F%E8%83%BD%0A\-\-\-\-\-%0A%E6%97%A7%E7%89%88%20%E7%9B%B4%E6%8E%A5%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E5%8D%B3%E5%8F%AF%0A%60%60%60%0Aauto\_excerpt%3A%0A%20%20enable%3A%20true%0A%20%20length%3A%20150%0A%60%60%60%0A%3E%E5%B0%B1%E6%98%AFMD%E6%A0%BC%E5%BC%8F%E6%B2%A1%E4%BA%86%EF%BC%8C%E7%89%B9%E5%88%AB%E4%B9%B1%0A%0A%E6%96%B0%E7%89%88%0A%0A%E6%96%B9%E6%B3%951%EF%BC%9A%E6%9C%80%E7%AE%80%E5%8D%95%E7%9A%84%E6%96%B9%E5%BC%8F%E6%98%AF%E7%94%A8%E5%AE%83%E9%BB%98%E8%AE%A4%E7%9A%84%EF%BC%8C%E6%94%B9%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%8D%B3%E5%8F%AF%0A%60%60%60%0Aexcerpt\_description%3A%20true%0Aread\_more\_btn%3A%20true%0A%60%60%60%0A%E5%9C%A8%E6%96%87%E7%AB%A0%E7%9A%84%E5%A4%B4%E9%83%A8%E4%B8%AD%E7%9A%84desc%E6%A0%87%E7%AD%BE%E4%B8%AD%EF%BC%9A%E6%B7%BB%E5%8A%A0%E4%BF%A1%E6%81%AF%0A%E6%88%96%E5%9C%A8%E5%86%85%E5%AE%B9%E4%B8%AD%E6%B7%BB%E5%8A%A0%E6%A0%87%E7%AD%BE%EF%BC%9A%0A%60%60%60%0A%3C\!\-\-%20more%20\-\-%3E%0A%60%60%60%0A%0A%0A%E6%96%B9%E6%B3%952%EF%BC%9A%E5%BE%97%E5%AE%89%E6%8F%92%E6%8F%92%E4%BB%B6%20%EF%BC%9A%0A%3Enpm%20install%20hexo\-excerpt%20\-\-save%0A%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%9A%0A%60%60%60%0Aexcerpt%3A%0A%20%20depth%3A%205%20%20%0A%20%20excerpt\_excludes%3A%20%5B%5D%0A%20%20more\_excludes%3A%20%5B%5D%0A%20%20hideWholePostExcerpts%3A%20true%0A%60%60%60%0A%0A%3E%E6%8E%A8%E8%8D%90%E6%96%B9%E6%B3%952%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%94%B9%E4%B8%80%E4%B8%AA%E5%9C%B0%E6%96%B9%E5%85%A8%E6%96%87%E7%94%9F%E6%95%88%EF%BC%8C%E7%9C%81%E5%8E%BB%E9%BA%BB%E7%83%A6%0A%0A%0A%E5%BD%A9%E8%89%B2%E6%A0%87%E7%AD%BE%0A\-\-\-\-%0A%E6%96%87%E7%AB%A0%E8%AF%A6%E6%83%85%EF%BC%8C%E5%B0%BE%E9%83%A8%E7%9A%84%E6%A0%87%E7%AD%BE%E5%8A%A0%E4%B8%8Aicon%E5%9B%BE%E6%A0%87\(%E9%BB%98%E8%AE%A4%E6%98%AF%E4%B8%AA%23%E5%8F%B7\)%0Atag\_icon%3A%20true%0A%0A%0Atag%E9%A1%B5%E9%9D%A2%E5%A2%9E%E5%8A%A0%E5%BD%A9%E8%89%B2%EF%BC%9A%0A%0A%3Ehttps%3A%2F%2Fblog.csdn.net%2Fqq\_39974578%2Farticle%2Fdetails%2F114172260%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%85%8D%E8%89%B2%E6%96%87%E4%BB%B6%EF%BC%9A%0A%3Etouch%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Flayout%2Ftag\-color.swig%0A%0A%E6%B7%BB%E5%8A%A0%E9%85%8D%E8%89%B2%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%3Evim%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Flayout%2Ftag\-color.swig%0A%0Avim%20themes%2Fnext%2Flayout%2Fpage.swig%0A%60%60%60%0A%7B%25%20include%20'tag\-color.swig'%20%25%7D%0A%60%60%60%0A%E4%BE%A7%E8%BE%B9%E6%A0%8F%E5%A2%9E%E5%8A%A0tags%EF%BC%9A%0Anpm%20install%20hexo\-tag\-cloud%40%5E2.1.\*%20\-\-save%0A%0Avim%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Flayout%2F\_macro%2Fsidebar.swig%0A%60%60%60%0A%7B%25%20if%20site.tags.length%20%3E%201%20%25%7D%0A%3Cscript%20type%3D%22text%2Fjavascript%22%20charset%3D%22utf\-8%22%20src%3D%22%7B%7B%20url\_for\('%2Fjs%2Ftagcloud.js'\)%20%7D%7D%22%3E%3C%2Fscript%3E%0A%3Cscript%20type%3D%22text%2Fjavascript%22%20charset%3D%22utf\-8%22%20src%3D%22%7B%7B%20url\_for\('%2Fjs%2Ftagcanvas.js'\)%20%7D%7D%22%3E%3C%2Fscript%3E%0A%3Cdiv%20class%3D%22widget\-wrap%22%3E%0A%20%20%20%20%3Ch3%20class%3D%22widget\-title%22%3ETag%20Cloud%3C%2Fh3%3E%0A%20%20%20%20%3Cdiv%20id%3D%22myCanvasContainer%22%20class%3D%22widget%20tagcloud%22%3E%0A%20%20%20%20%20%20%20%20%3Ccanvas%20width%3D%22250%22%20height%3D%22250%22%20id%3D%22resCanvas%22%20style%3D%22width%3A100%25%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%7B%7B%20list\_tags\(\)%20%7D%7D%0A%20%20%20%20%20%20%20%20%3C%2Fcanvas%3E%0A%20%20%20%20%3C%2Fdiv%3E%0A%3C%2Fdiv%3E%0A%7B%25%20endif%20%25%7D%0A%60%60%60%0A%60%60%60%0A%23%20hexo\-tag\-cloud%0Atag\_cloud%3A%0A%20%20%20%20textFont%3A%20Trebuchet%20MS%2C%20Helvetica%0A%20%20%20%20textColor%3A%20'%23333'%0A%20%20%20%20textHeight%3A%2025%0A%20%20%20%20outlineColor%3A%20'%23E2E1D1'%0A%20%20%20%20maxSpeed%3A%200.1%0A%60%60%60%0A%0A%E7%BB%99%E6%96%87%E7%AB%A0%E5%BA%95%E9%83%A8tag%20%E6%B7%BB%E5%8A%A0%E5%BD%A9%E8%89%B2%3A%0Avim%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Flayout%2F\_macro%2Fpost.swig%0A%0A%60%60%60%0A%20%3Cdiv%20class%3D%22post\-tags%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%7B%25\-%20for%20tag%20in%20post.tags.toArray\(\)%20%25%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Ca%20href%3D%22%7B%7B%20url\_for\(tag.path\)%20%7D%7D%22%20rel%3D%22tag%22%3E%3Ci%20class%3D%22fa%20fa\-tag%22%3E%3C%2Fi%3E%20%7B%7B%20tag.name%20%7D%7D%3C%2Fa%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%7B%25\-%20endfor%20%25%7D%0A%20%20%20%20%20%20%20%20%20%20%3C%2Fdiv%3E%0A%20%20%20%20%20%20%20%20%20%20%3Cscript%20type%3D%22text%2Fjavascript%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tagsall%3Ddocument.getElementsByClassName\(%22post\-tags%22\)%0A%20%20%20%20%20%20%20%20%20%20%20%20for%20\(var%20i%20%3D%20tagsall.length%20\-%201%3B%20i%20%3E%3D%200%3B%20i\-\-\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20tags%3Dtagsall%5Bi%5D.getElementsByTagName\(%22a%22\)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20for%20\(var%20j%20%3D%20tags.length%20\-%201%3B%20j%20%3E%3D%200%3B%20j\-\-\)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20golden\_ratio%20%3D%200.618033988749895%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20s%20%3D%200.5%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20v%20%3D%200.999%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20h%20%3D%20golden\_ratio%20%2B%20Math.random\(\)\*0.8%20\-%200.5%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20h\_i%20%3D%20parseInt\(h%20\*%206\)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20f%20%3D%20h%20\*%206%20\-%20h\_i%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20p%20%3D%20v%20\*%20\(1%20\-%20s\)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20q%20%3D%20v%20\*%20\(1%20\-%20f%20\*%20s\)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20t%20%3D%20v%20\*%20\(1%20\-%20\(1%20\-%20f\)%20\*%20s\)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20var%20r%2C%20g%2C%20b%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20switch%20\(h\_i\)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%200%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20t%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%201%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20q%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%202%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20t%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%203%20%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20q%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%204%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20t%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%205%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%20v%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%20p%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%20q%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20r%20%3D%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20g%20%3D%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20b%20%3D%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20tags%5Bj%5D.style.background%20%3D%20%22rgba\(%22%2BparseInt\(r\*255\)%2B%22%2C%22%2BparseInt\(g\*255\)%2B%22%2C%22%2BparseInt\(b\*255\)%2B%22%2C%22%2B0.5%2B%22\)%22%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fscript%3E%0A%0A%0A%60%60%60%0A%0A%60%60%60%0A%3Cstyle%3E%0A%0A.posts\-expand%20.post\-tags%20a%20%7B%0A%20%20%20%20display%3A%20inline\-block%3B%0A%20%20%20%20font\-size%3A%200.8em%3B%0A%20%20%20%20padding%3A%200px%2010px%3B%0A%20%20%20%20border\-radius%3A%208px%3B%0A%20%20%20%20color%3A%20rgb\(85%2C%2085%2C%2085\)%3B%0A%20%20%20%20border%3A%200px%3B%0A%7D%0A%0A%3C%2Fstyle%3E%0A%60%60%60%0A%0A%0Avim%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2F\_config.yml%0A%60%60%60%0A%3Cspan%20class%3D%22post\-meta\-item%22%3E%0A%20%20%20%20%24%7BiconText\('far%20fa\-comment'%2C%20'valine'%2C'%E8%AF%84%E8%AE%BA%E6%95%B0'\)%7D%0A%20%20%20%20%3Ca%20title%3D%22valine%22%20href%3D%22%7B%7B%20url\_for\(post.path\)%20%7D%7D%23valine\-comments%22%20itemprop%3D%22discussionUrl%22%3E%0A%20%20%20%20%20%20%3Cspan%20class%3D%22post\-comments\-count%20valine\-comment\-count%22%20data\-xid%3D%22%7B%7B%20url\_for\(post.path\)%20%7D%7D%22%20itemprop%3D%22commentCount%22%3E%3C%2Fspan%3E%0A%20%20%20%20%3C%2Fa%3E%0A%20%20%3C%2Fspan%3E%0A%60%60%60%0A%0A%E4%BE%A7%E8%BE%B9%E6%A0%8F%E6%B7%BB%E5%8A%A0%E5%88%86%E9%9A%94%E7%BA%BF%0A\-\-\-%0A%2FUsers%2Fmayanyan%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Fsource%2Fcss%2F\_common%2Fcomponents%2Fpost%2Fpost\-eof.styl%0A%60%60%60%0A.sidebar\-eof%20%7B%0A%20%20background%3A%20%24grey\-light%3B%0A%20%20height%3A%201px%3B%0A%20%20text\-align%3A%20center%3B%0A%20%20width%3A%20100%25%3B%0A%7D%0A%60%60%60%0A%E6%B5%8F%E8%A7%88%E8%BF%9B%E5%BA%A6%0A\-\-\-%0Ascrollpercent%3A%20true%0A%0A%0A%E8%AF%84%E8%AE%BA%0A\-\-\-%0A%E5%85%88%E5%8E%BBLeanCloud%E6%B3%A8%E5%86%8C%E4%B8%AA%E8%B4%A6%E5%8F%B7%EF%BC%8C%E5%86%8D%E5%88%9B%E5%BB%BA%E4%B8%AA%E5%BA%94%E7%94%A8%EF%BC%8C%E6%8B%BF%E5%88%B0appId%20appKey%EF%BC%8C%E8%AE%BE%E7%BD%AE%E5%88%B0%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%0Avaline%0A%0A%E6%96%87%E7%AB%A0%E8%AF%A6%E6%83%85%E9%A1%B5%EF%BC%9A%E6%A0%87%E9%A2%98%E4%B8%8B%E9%9D%A2%E7%9A%84%E8%AF%84%E8%AE%BA%E6%95%B0%E9%9D%9E%E4%B8%AD%E6%96%87%EF%BC%8C%E6%9C%89%E7%82%B9BUG%EF%BC%8C%E6%94%B9%E4%B8%80%E4%B8%8B%0Avim%20themes%2Fnext%2Fscripts%2Ffilters%2Fcomment%2Fvaline.js%0A%0A%E6%B7%BB%E5%8A%A0%E5%9B%BE%E7%89%87%E5%88%97%E8%A1%A8%E5%8A%9F%E8%83%BD%0A\-\-\-%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%EF%BC%9A%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fthemes%2Fnext%2Fscripts%2Fshow\_some.js%0A%60%60%60%0Avar%20fs%20%3D%20require\(%22fs%22\)%3B%0A%0Ahexo.extend.tag.register\('show\_some'%2C%20function\(args\)%7B%0A%0A%20%20%20%20if\(\!args%5B0%5D\)%7B%0A%09return%20false%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20var%20listStr%20%3D%20%22%22%3B%0A%20%20%20%20const%20files%20%3D%20fs.readdirSync\(args%5B0%5D\)%3B%0A%20%20%20%20const%20prefix%20%3D%20args%5B1%5D%3B%0A%0A%20%20%20%20files.forEach\(function%20\(item%2C%20index\)%20%7B%0A%20%20%20%20%20%20%20%20var%20thisPicLink%20%3D%20prefix%2Bitem%3B%0A%20%20%20%20%20%20%20%20var%20divStyle%20%3D%20%22float%3Aleft%22%3B%0A%20%20%20%20%20%20%20%20var%20oneRow%20%3D%20%22%3Cdiv%20%20style%3D'margin\-right%3A10px%3B%22%2BdivStyle%2B%22'%3E%3Cdiv%3E%22%2Bitem%2B%22%3C%2Fdiv%3E%22%0A%0A%20%20%20%20%20%20%20%20oneRow%20%2B%3D%20%22%3Cdiv%3E%3Ca%20href%3D'%22%2BthisPicLink%2B%22'%20target%3D'\_blank'%20%3E%22%20%20%2B%22%3Cimg%20src%3D'%22%20%2B%20thisPicLink%20%2B%20%22'%20alt%3D'%22%2Bitem%2B%22'%20width%3D200%20height%3D400%20%3E%20%3C%2Fa%3E%20%3C%2Fdiv%3E%22%3B%0A%20%20%20%20%20%20%20%20%2F%2Fdisplay%3A%20inline\-block%0A%20%20%20%20%20%20%20%20oneRow%20%2B%3D%20%22%3C%2Fdiv%3E%22%3B%0A%20%20%20%20%20%20%20%20listStr%20%2B%3D%20oneRow%3B%0A%20%20%20%20%7D\)%0A%0A%20%20listStr%20%2B%3D%20%22%3Cdiv%20style%3D'clear%3A%20left'%3E%3C%2Fdiv%3E%22%3B%0A%0A%0A%20%20%20%20return%20listStr%20%3B%0A%7D\)%3B%0A%0A%0A%0A%60%60%60%0A%0A%E5%9B%BE%E7%89%87%E6%8E%92%E9%98%9F%E6%A0%B7%E5%BC%8F%0Avim%20\_common%2Fscaffolding%2Ftags%2Fgroup\-pictures.styl%0A%60%60%60%0A.page\-post\-detail%20.post\-body%20.group\-picture\-column%20%7B%0A%20%20%2F%2F%20float%3A%20none%3B%0A%20%20margin\-top%3A%2010px%3B%0A%20%20%2F%2F%20width%3A%20auto%20\!important%3B%0A%20%20img%20%7B%20margin%3A%200%20auto%3B%20%7D%0A%7D%0A%0A%60%60%60%0A%0A%E6%9C%80%E5%90%8E%E5%93%AA%E9%87%8C%E4%BD%BF%E7%94%A8%EF%BC%8C%E5%93%AA%E9%87%8C%E5%BC%95%E7%94%A8%20%EF%BC%9A%0A%3E%7B%25%20show\_some%20%2Fdata%2Fwww%2Fgolang%2Fzblog%2Fsource%2Fimages%2F%E8%84%91%E5%9B%BE%2F%20%2Fimages%2F%E8%84%91%E5%9B%BE%2F%20%25%7D%0A%0A%E5%BF%AB%E9%80%9F%E9%87%8D%E5%90%AF%0A\-\-\-\-%0Ahexo%20clean%20%26%26%20hexo%20g%20%26%26%20hexo%20s%0A%0A%E5%B0%BE%E9%83%A8%E8%AE%BE%E7%BD%AE%0A\-\-\-%0A%60%60%60%0Afooter%3A%0A%20%20%23%20Specify%20the%20date%20when%20the%20site%20was%20setup.%20If%20not%20defined%2C%20current%20year%20will%20be%20used.%0A%20%20since%3A%202022%0A%0A%20%20%23%20Icon%20between%20year%20and%20copyright%20info.%0A%20%20icon%3A%0A%20%20%20%20%23%20Icon%20name%20in%20Font%20Awesome.%20See%3A%20https%3A%2F%2Ffontawesome.com%2Ficons%0A%20%20%20%20%23name%3A%20fa%20fa\-heart%0A%20%20%20%20name%3A%20fa%20fa\-bug%0A%20%20%20%20%23%20If%20you%20want%20to%20animate%20the%20icon%2C%20set%20it%20to%20true.%0A%20%20%20%20animated%3A%20true%0A%20%20%20%20%23%20Change%20the%20color%20of%20icon%2C%20using%20Hex%20Code.%0A%20%20%20%20color%3A%20%22%23ff0000%22%0A%20%20%20%20%23color%3A%20%22%23000000%22%0A%60%60%60%0A%0A%E6%94%B6%E5%B0%BE%EF%BC%8C%E5%8A%A0%E5%85%A5git%E4%BB%93%0A\-\-\-\-\-%0A%E5%8F%91%E7%8E%B0next%E4%B8%BB%E9%A2%98%E6%98%AF%E5%B8%A6%E7%9D%80git%E7%89%88%E6%9C%AC%E7%9A%84%EF%BC%8C%E4%B8%8D%E8%83%BD%E7%9B%B4%E6%8E%A5%E6%8A%8A%E5%85%A8%E9%83%A8%E4%BB%A3%E7%A0%81%E5%8A%A0%E5%85%A5%E5%88%B0git%E9%87%8C%0Agit%20status%20%E7%9C%8B%E7%9C%8B%E6%98%AF%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%EF%BC%9F%0A%E5%8F%91%E7%8E%B0%EF%BC%9A%E9%99%A4%E4%BA%86%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BF%98%E6%9C%89%E4%B8%80%E4%B8%AA%EF%BC%9A%E5%A4%B4%E5%83%8F%E5%9B%BE%E7%89%87%E6%9C%89%E5%8F%98%E5%8A%A8%0A%E5%85%88%E6%8A%8A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%A4%87%E4%BB%BD%E4%B8%8B%EF%BC%8C%E5%86%8D%E5%A4%87%E4%BB%BD%E4%B8%8B%E5%9B%BE%E7%89%87%0A%E7%8E%B0%E5%9C%A8%0A%0Agit%E4%B8%8A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%2Cclone%20%E4%B8%80%E4%B8%8B%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%3Ecd%20%2Fdata%2Fwww%2Fgolang%0A%3Egit%20clone%20git%40github.com%3Amqzhifu%2Fzblog.git%0A%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-%0A%E8%99%BD%E7%84%B6%E8%AF%B4%E6%98%AFhexo%E5%8D%9A%E5%AE%A2%EF%BC%8C%E4%BD%86%E6%98%AF%E5%AF%B9hexo%E7%9A%84%E9%85%8D%E7%BD%AE%E5%BE%88%E5%B0%91%EF%BC%8C%E5%A4%A7%E9%83%A8%E5%88%86%E9%83%BD%E6%98%AF%E5%9C%A8%E8%B0%83%E6%95%B4next%E4%B8%BB%E9%A2%98%E3%80%82%E8%80%8C%E4%BF%AE%E6%94%B9next%E4%B8%BB%E9%A2%98%E5%B0%B1%E6%98%AF%E4%BF%AE%E6%94%B9%E9%A1%B5%E9%9D%A2%0A%0A
