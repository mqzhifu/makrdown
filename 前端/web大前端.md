# 起因

公司同事离职，留了个项目给我，是 VUE 写的，且还得继续更新维护，既然又得重新搞前端，这次就好好看看吧，短时间肯定不能完全搞的很透，但是大概的流程可以理理。

# 目的

1. 宏观了解下现在的前端技术栈
2. 对比下早期与现在的前端技术区分
3. 将自己的旧时前端技术升级一下
4. 避免有人忽悠我
5. 做全局构架更得心应手

# 回顾

## 早期

从 09 年毕业开始，前端的布局刚刚从 TABLE 转成 DIV，速度确实比以前快了，但是前端确实也开始有那么点设计模式的感觉，同时也加大了程序员的工作量。

工作模式：JS+HTML+css/div

基本 phper 以上都得写，之后 HTML\+css/div 终于被分离给前端了，但 HTML\+css/div 只是少量写，同时是得帮助管理，把静态文件嵌入到项目中、发布布置等，实际上还是没有完全分离。

这之后出现了一些前端类库/框架:

1. Django
2. extjs
3. jquery
4. angular
   ...

新的工作模式：JS+HTML+css/div+jquery

## 之后发展史

- AJAX 的出现，单页面不用刷新即可动态获取数据，动态更新数据了.
- 前端提出 OOP 思想，做封装，像：闭包、原型链、作用域等开始出现。
- backbone 框架,终于有个类似 mvc 的前端框架出现，然而，我基本专注后端了。。。。

backbone:2010 年出品，它不是一个单独的东西，还得配合 template.js undecode.js。但是出生太早了，被后面的追赶着超越了，基本退出历史舞台了。
以 backbone 作为分割点吧，基本上我就告别前端，专注后端了，而思想就停留在：

js+jquery+ajax+oop+少量 html css div

个人对前端的核心点认知：浏览器如何解析 hml css，v8 如何执行代码，js 如何操纵 dom 节点,JS 的 OOP 原理。
这会对前端的总结：只能写点最最基础的交互，连 HTTP 协议都不懂，我更认为这会的前端就是个'转换器'，把 PS 转换成 HTML，借用后端、浏览器帮忙，才能开发一个项目。

# SOC

Separation of concerns ，直译 ：关注点分离

> 好像概念有点大，有个哥们出了本书就描述这个东西。

核心概念：分离，概念有点大，是 V 跟 M 分？ctrl 跟 model 分？还是前后端分享？我觉得：以上都得分离。

既然我能列出就证明以上这些，就证明确实可能得分离，所以都得分。那都得分，就得确定怎么分？另一个概念：关注点。如何分：个人感觉就是抽象...

1. 可抽象的
2. 变化较多
3. 基于以上两点，分享后，还不能影响到其它的.

映射到前端上就是：VUE 就是这个，只是它比较大。它把 view model ctrl 抽象成了 mvvm

VUE 内部可复用的代码，如：组件~
组件内部又可以嵌入其它组件，继续抽象，如：slot

再结合看：关注/专注点，这个词，它就是能把所有可抽象的东西都抽象出来，然后，再分别派发给某个专业的人，这个人就是专业的人。也就是这个人只:关注/专注 某个点。

两个词结合看：把项目代码结构，根据一定特性/关注点，进行抽象，然后分拆，最后就变成了，专业的人干专业的事儿。

# 大前端

ES

JAVASCRIPT:1995 年由 Netscape 公司开发，嵌入 WEB 中。1996 年提交给 ECMAS，希望国际化。

ECMA:欧洲计算机制造联合会

JS 被该协会定义为：ECMAScript

ES6

就是一个 ES 版本，2015 年正式发布。不过，2018 年 ES9 已经正式发布了，ES10 草案好像也正常了。
不过，主流能做到完全兼容的是 ES5，ES6 算是一个跨度比较大的，那就简单描述下，ES6 增加的东西

1. let
2. promise async away\(awync wait\)
3. const
4. import、export
5. class、extends、super
6. =\> arrow functions
7. template string
8. destructuring
9. default
10. Generators

看着加了不少东西，但感觉咋说呢，就是把 JS 后端化、复杂化。也就是：JS 由嵌入型脚本，想转正为一门正式的后端语言的语言。像：异步编程、OOP 、包管理。
个人觉得：有那么点 PHP 想要写后端的意思，然后，各种写类库，加扩展，但结果被 GO 给取代了。前端真是瞎折腾。

# type script

又是个逗比的东西，ts 代码最后还是被编译成 js，但是他里面做了一些强类型的操作，实际上，是在 ES6 的基础上，又加强了 JS 向后端进化的一个过程。

# node.js

可以写后端的 js，那从宏观看，JS 真是完全可以写后端了。
基于 V8，简单说就是：用 C\+\+ 把 V8 引擎又封装了一层，专供执行 JS 代码的。
内部原理：event loop 模型，详细的我也不熟，空了可以看看，但感觉得先看 V8 引擎。
但讽刺的是，作者好像放弃更新了，因为太臃肿，随便写个东西，得先下载 2w 个文件....新的好像叫什么：deno......

# NPM

JS 类包管理，跟 maven composer 一个鸟东西，也有个 package.json 文件用来配置源

> 好像是用 node.js 编写的...安装 node 会连带着一并给安装了

还有个 cnpm，估计跟 NPM 一样，淘宝的一个代理 NPM 的东西，我猜就是改了个源地址...

> 有趣的是 npm instal cnmp 哈哈

然后，就发现跟我猜的一样，cnmp 果然就是改了个源，因为可以：

> npm instal xxxx \-\-registry=http://wwwww

这东西居然，还能执行脚本... 有点刷新认知，干了一些程序员或者 shell docker 的活。

> npm run xxxx

原理：package.json 中有个 key 叫： scripts ，它里面包含的每一项 就是需要执行的脚本

> npm run xxxx 其中的 xxxx 会当做参数，传到 scripts 中

JS 有了 node 和 npm，就有玩头了~可以上传/下载包，还能通过指令行来执行包~

像实力强的大厂(vue)，完全可以用 NODE 编写脚手架代码，然后封装成包上传，使用者先用 NPM 把包下载下来，再执行 node.js，接着这个包就可以在使用者的机器上动态生成构架文件了~

基本上后端的一套玩法，JS 都能玩儿了。

> 不过，就一个 VUE 最最最简单的项目，要下载 2W 多个文件,也难怪 NODE 创始要放弃 NODE

# webpack

> npm install webpack \-g
> npm install webpack\-cli \-g

打包工具 , 把你项目中的 JS 代码，打包成一个 JS 文件，再压缩，然后生成一个新的文件。

> 不过，你得得按照它的方式来写代码，如：得有个 modules，得有 main 入口

最后定义一个 webpack.config.js 定义一下 modules 目录及输出位置。

> 另外，他能把你的 ES6 打包成 ES5 格式

先不看各种框架，单从这几个基础的软件：es6 ts node webpack ，JS 已经由我最初的认知：只能写写前端，现在真的能根本上实现了，JS 也可以写后端了。尤其再配合它的包管理工具，大体上就跟后端的模式 90%都相似了。

看看 webpack 的大体语法

1. entry:一个项目的入口文件,有点 main 函数的意思。这里可以有多个，一般是一个，因为是单面应用。webpack 开始打包时，得找到某些项目的入口，由此入口再依次编译所依赖的东西。
2. output:打包好的文件，输出的位置

```
    path: path.resolve(__dirname, '../dist'),
    publicPath: '/assets/',
    filename: 'static/js/[name].js',
    chunkFilename: '[id].bundle.js',
```

1. module:自定义要加载哪些文件，不加载哪些文件，使用什么加载器 loader
2. devServer:开启一个 http 守护进程~dev=开发，server:服务器，给前端开发者启动一个后端 HTTP 监听进程。

> 它内部是另一包，webpack\-dev\-server，开启后，output 参数基本失效，但支持热打包，即：你 JS 有修改，立刻可以查看到结果。其实也挺好理解，它是个守护进程，能时时监听到所有，那么，output 的打包文件，被加载到内存中，同时监听文件变化。

```
port:
proxy:
```

配置好 webpack.config.js 后，开始打包：

> webpack

会在 output 指定的位置生成打包好的文件。

> 之前我跟前端联调，他们居然还要打包编译... 我就不太理解，一个破前端还用编译打包？js 文件编译个毛线，JS 得加载到浏览器才能运行，这不逗我呢？现在看完，也能理解了。

整体看现在的前端的模式：用 node.js 日常开发，配合 NPM 包管理，最后把这些 JS\\CSS\\IMG，统一用 webpack 打包成一个新的浏览器可执行的文件，最后开启 web\-service 监听。

基本上后端的开发流程，前端都可以串起来了。

感觉还是有点绕，node.js 能写点后端，如：webservice，同时支持一些后端语言的语法，如：强类型、import/export 等，最后打包。但问题是打包完了还是.js 文件，这个就有点讽刺。不过也可能是因为它得内嵌到浏览器，资源也比较碎片化，加上这个流程确实是方便些。

这里有篇文章，可以细细品品:

> https://zhuanlan.zhihu.com/p/32148338

感觉前端真的是瞎 JB 折腾，看似是想后端体系挣脱出来，想照搬后端模式，想要模块化、OOP，又想简单化，最后搞的真是复杂到你想放弃。我依然觉得：JS\+JQUERY 更好，因为复杂过度后，就一个问题你无法解决：如何排查错误？所有的代码都是经过框架打包组合后生成，尤其 JS 被打包成一个大大的 JS 文件。

# 前端三大框架

Angular:2012 年，google 的几个 JAVA 后端搞得一个框架，基于 MVC，肯定是后端使用非常顺，对前端不太友好，另外它也有点重，版本兼容也不太好。但它的功劳是 比较早的整合了 MVC 模式\(个人感觉 backbone 更早\)。

react:facebook 出品，2013，首次提出虚拟 DOM。

vue:2014 年，国人设计，有 mvvm 设计模式，也有虚拟 dom 模式，同时又非常小，又有组件化且专注于 view 层。

总的来说：年代越新，融合的思想也就越好，用的人也最多。但更新真是太快太乱了，早期的 JUQERY\+JQUERY\-UI，再到，前几年感觉 react 一统天下的意思，这 2 年风向就变了，完全转身 VUE 了。

# mvvm

model view view\-model：跟 mvc 差不多，就是个框架的设计模式。它的诞生，算是终于给前端统一了设计模式，可持续发展的模式。

> 感觉前端发展就是当时后端的发展过程，但感觉真的是一阵乱射，最终有几支箭射中了，大家再回到这个正确的方向，之后继续乱射....还是基础不行啊~

# 虚拟 dom

前置说明:前端的日常开发，基本上就是修改 dom 树，然后，浏览器根据修改内容再重新做渲染。好像现在有工具能直接把 PS 生成 HTML\+CSS/DIV.

vdom:用 js 虚拟出一层 dom

原理：先创建一个 JS 对象，然后，把某个\(也可能包含多个子 DOM 节点\)真实的 dom 节点绑定到这个 js 对象上，同时再给这个 JS 对象绑定上自己的日常操作函数，这样，所有的操作都是操作这个 JS 对象，而不直接操作 dom，最后虚拟 dom 可能会修改 dom 也可能不改。

> 相比之下，jquery 就是直接操作 DOM

理论上：比一直修改 dom 树造成浏览器平凡重新渲染，效率更高些。

所以，我们看上面的原理，核心点就是这个 JS 对象，其实这个对象就是 VUE...react

## 优点

1. 性能可能会加强，避免浏览器一直重复渲染
2. 程序员可能更专注于在 JS 上，而不是 HTML 标签上
3. 有组件思想，代码利用率更高

## 缺点

1. 学习成本、使用成本
2. 复杂化，造成更大的维护成本

虚拟 dom 的诞生也就是 mvvm 中的 vm，其结果就是 vue react 引领的这场变革，所有的前端代码就都得开始玩 vdom 了

基本上理解了 vm，能够套用到 mvvm 模式中，大体上也就理解了现在的前端开发模式的核心了，剩下的就是围绕 vm 的各种操作了。

# 单页应用

# VUE

因为它现在最火，就拿出它来学习下。
它其实就是用 JS 写的一个库，然后实例化成一个 JS\-OBJ，就是 V\-DOM 类型.

> 同类型对比 LINUX 系统：用户态与内核态，内核态就是 VUE

原本可能松散的 JS 代码，被统一整理到 VUE 中，如：前端程序员把各种回调函数注册到这个 JS\-OBJ 中，用户的操作第一时间也是操作这个 JS\-OBJ，等于中间加了整整一层而以。

这种 JS\-OBJ，可能一个页面就有 N 个，所以，它也有生命周期的概念：

beforeCreate created beforeMount mounted beforeUpdate udpated destory

> 感觉又是安卓 IOS 那一套.....问题是：安卓 IOS 那一套是真的给一个完整的 APP 在上面运行的，你的 APP 代安装时，得经过编译成可执行文件，放在硬盘上，然后 OS 直接可以执行了... 而 JS 这垃圾，并不是直接运行在 OS，是运行在浏览器里，浏览器还不能直接执行，还得再给 V8 引擎，最后 VUE 又强行模拟出来这么一层....

这个生命周期，其实也能更好的解释什么是：V\-DOM
生命周期这一堆函数，基本上可以统一定义为：钩子，如：

创建一个 APP，在最终挂载成功之前，要用 AJAX 获取到后端数据才行，那用 mound 函数。

## vue 七大 vm 属性

data:view 层需要的数据均从这里拿
methods:前端程序用来控制 DOM 变化的一些自定义
el:绑定到真实的某个 DOM 节点 ID
template:模板
compute:跟 methords 有点像，也是一些自定义函数，但它是属性，常驻内存\(前提不发生改变\)，有那么点缓存的意思。
watch:
render:

另外还有一些其它的属性：

slot:插槽,类似页面布局，定好布局后，就是一个槽一个槽，然后你再动态的替换.
props:接收外层 bind 的数据

接上面的总结，理解了 VM 的原理，接着就是在 VM 上做各种操作，日常的开发基本就是在这 7 个属性中了.

> 其实高级 VUE 我猜，就是 VUE 内部的这种 V\-DOM 间的操作，可能是同级，也可能是父子组，也也可能是组件之间

vue 确实够小巧，并且保持专注，其他的一概不管，如果想有更多功能，那就引入其他包！！！

> 感觉 vue 跟我当年的 jquery 挺像，任何简单的东西都可能会火~

## vuex

类库，状态管理

它更像是一个仓库，全局仓库，或者叫全局本地仓库~

每个组件之间：一但某一个 model 层的数据发生变化，连带着可能所有组件都得跟着变，那么组件间的数据通信就是个问题。这个库就是解决这个问题，将各种数据的变化定义成状态，然后存到仓库中，并设置当发生改变后的回调函数。

1. state:可以理解为仓库里的具体数据
2. mutations:有一些方法，可以上面 state 里的数据
3. action:做异常的处理，有那么点钩子的意思
4. getter:读取 state 里的数据，是个函数，也有那么点钩子的意思
5. module:如果这种全局状态数据过多，就把一些特定的数据绑定到固定的模块上

外部类，想要修改 state 里的数据，必须得调用它的 mustations 里的方法修改。

> $store.commit\("mutations 里的函数名"\)

mutations 里的函数就是真正修改 state 里的数据。

action 的使用

> $store.dispach

咋个形容呢，这东西的初衷我能理解...包括 各种钩子，可能出现锁等等... 初衷是好的，但 小项目，你就定义个全局变量不就得了...

## VuexPersistence

配合 vuex 使用。

vuex 存储了全局数据后，还是在内存中~刷新后，数据丢失，如果能存储，那就完美了，于是有了 VuexPersistence，帮助 vuwx 持久化数据的。

还得先讲下：localStorage

## localStorage

HTML5 中，新加入了一个 localStorage 特性，本地仓库。解决了 cookie 存储空间不足的问题\(cookie 中每条 cookie 的存储空间为 4k\)，localStorage 中一般浏览器支持的是 5M 大小，这个在不同的浏览器中 localStorage 会有所不同。

相同域名/端口，多 TAB 可以共享。

这个东西挺好，等于 WEB 前端有本地数据库了，缓存点后端 API 数据、存点自己的东西，可方便多了。

还有个 sessionStorage，跟它有点像，但是页面关闭后即消失，也是正常，为了安全嘛~

VuexPersistence 就是基于 localStorage，感觉也没啥创新，估计就是封装一层或做点兼容之类的吧。

## vue-router

类库，路由器

这东西主要是组件之间的跳转

后端对路由的理解：做分发\+真的跳转，还是模式不一样，前端：是由用户驱动，N 个组件响应模式，就是组件之间的跳转，而后端有点像是'被动式'接收前端请求，然后路由主要是做分发功能

组件的对比：后端的组件直接就挂在容器或全局变量上了，复杂一点的就走 RPC 了。

## vue-cli

一个指令行控制的插件，帮助开发者快速搭建、打包 VUE。

> npm install vue\-cli

安装好后，我们找个目录，初始化一个 VUE 的项目

> vue init webpack xxxxxx

这个时候，目录下会有一堆 vue\-cli 脚本创建的新目录和新文件

```
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── logo.png
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── index.html
├── package.json
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── HelloWorld.vue
│   └── main.js
└── static
```

> 初始化只是帮助建立好结构，其重点是：package.json，这个是依赖包

从这个结构看，之前所有的 VUE 学习 DEMO、视频的写法，均是用于演示，而真实的项目开发，不可能那么零散。还有实际项目中可能会用到 N 多个类库，让你随便放也有点 LOW，所以，它就有个 cli 这个鬼东西，类似代码结构目录生成器，来帮助使用者，快速按照一定结构创建一个项目.

接着开始下载安装各种依赖包：

> npm install
> 哎，这东西不挂 VPN，基本上是会挂....

启动

> npm run dev

问题：它没有 webpack.config.js ，那它是如何打包能运行的呢？如：

1. 没有 entry，那它如何找到 src/main.js 的？
2. output 虽然在 vue.config.js 中有定义，可为啥不输出呢？
3. index.html 不在根目录，它是如何找到的呢？

解决：

1. node_modules/@vue/cli\-service/lib/config/base.js 中找见到了... entry:/src/main.js
2. webpack 如果配置文件中开启了 devServer ，就不再输出了
3. 没找到...

**启动原理**

> 从 package.json 中找 script\-\>vue\-cli\-service serve

vue\-cli\-service 居然在 node_modules/.bin 下，是个软连接，估计是 npm 的快捷方式

打开后，发现包含了 ../lib/Service ，咋也找不见...最后：

> node_modules/@vue/cli\-service/lib/Service.js 找见了....

@vue 这是什么鬼？

vue 整整包了一层 webpack....

## VUE 创建新项目流程

1. 安装 NODE.JS
2. 安装 NPM
3. 安装包：vue\-cli webpack
4. 初始化一个新项目:vue init webpack XXXX
5. 安装这个新项目的依赖包 npm install
6. 开始正常编写代码,src 目录
7. 项目编写完成，启动/测试 npm run server
8. 启动过程\-\>package.json\-\>script\-\>build.js
9. build\-\>检查当前环境变量\-\>根据指令行参数，决定选择哪个环境变量的 buile 脚本\(就是配置不一样\)\-\>webpack 打包 JS 文件\-\>再找到项目的配置文件\-\>启动 webservice~

## react

因为项目就是 VUE，精力全花在了 VUE 上面，但我还是简单查了查这两个的对比，大致：

1. react 比较开放，有那么点原生 JS 的感觉，并不会像 VUE 插手那么多。
2. vue 有点框死了，就往 V\-DOM OBJ 上挂各种函数，有那么点 闭包的感觉。

区别：

> 前置：没有哪个好哪个坏，一切看使用场景。

1. vue 因为封装的好，学习成本低，上手容易，好招人，也不容易多强的技术，适合中小项目。
2. react 比较开放，需要各种造轮子。适合大厂，有牛 B 的技术。学习成本有点高。

另外，react 好像插件比较多，有那么点 JAVA SPRING 的意思，对 TS 兼容好，它还有 react native 加持，在移动客户端也有所建树。

## flutter

之前也只是听说，现在依然也只是听说，但看了看文档，感觉这个东西才是比较牛 B 的呢。

vue 我是没找到能统一三端的插件。

react 可以，但他必须依赖 V8

fluter google 出品，单独有个引擎，能支持 3 端

## axios

类库，网络管理

http XMLHttpRequests ajax

就是把上面的，有网络请求相关的操作，统一封装了一层，把原本要程序员写的一堆代码统一抽离封装，并且有一定的数据映射功能（如：ORM）

基本上 vue\+vuex\+router\+axios 就等于后端的一个比较完整的框架了。

反正做为后端的我比较喜欢这种模式，大而全的东西，注定过重，且学习成本太高。反而由程序员手动组建一个框架更舒服。

> 从 VUE 的整体模式来看，就是把前端以前各种松散的代码，进行 抽象、抽离、封装成统一的类库，再给程序员使用。TMD，怎么看都是在跟后端学...

# 前端 ui 框架

ant design:阿里，react
elementui iview ice :vue
amazeui:
layui
docsify

## elementui

> npm I element\-ui \-S
> cnpm install sass\-loader node\-sass

至此：大前端的整体流程就都穿起来了。需要掌握的知识点也就如上所述呀。

# 多端

早期的开发基本就一端：PC，而现在要考虑 PC H5 andrio IOS，另外：PC H5 在我那会儿有个很头疼的问题：浏览器兼容。
所以，从这个角度看，前端的发展与进化，多多少少也是被市场需求所逼，它如果再不进化，那么这 N 端及浏览器兼容真的会大大影响效率。
结合看 VUE 好像在移动端没有 react native、fluter 更强，那么，在做构架的时候就得好好想想了

> ps:从这里也能理解出，确实 VUE 可能对大项目不适合

如果所有公司的想法都是：兼容多端，那么可悲的就是现在的 android ios 的开发程序员了，只会单一载体开发，被 JS 降维打击，必然是大批失业潮。\(感觉这里 JS 前端又得迷之自信一会儿...\)

# 总结

猛一看，前端这几年：一堆新鲜的东西，backbone react vue angular node.js npm 等等， 但是一地鸡毛...其核心的发展史还是在向后端学习，而之所以一地鸡毛，根本原因：还是基础太差，造成想法飘逸而不且实际，框架百出，但没有能力去认识思考底层的原理。

但是，前端确实是在进步,大前端的概念抛出来，前端确实比以前要学习很多知识。尤其不能光光盯着前端那点东西，还得会点 nodejs，懂点后端知识。也让后端不再那么孤独（拿一样的薪水，需要学的知识却比前端多一堆!!!）。另外，也能驱散一些前端混子。

从后端看这种趋势，喜剧参半：

1. 好的是，因为现在的前端：已经把后端大部分的工作都拿过去了，如：action 路由、本地缓存、权限验证等。
2. 不好的是：后端的生存空间被压缩。

这就挺纠结，跟前端对接口，总觉得他们笨，要后端做很多，但真的被拿过去后，发现：自己不再是最重要的那个后端了....

## 冲突

按照总结后的看，前端在整体后端的东西，那么，以 node 诞生来看，前端未来会不会无聊的又搞出后端的东西？如：队列系统、日志系统等。我觉得他们干的出来。那么，行业又得出现一次混乱期，有那么点：分分合合，合合分分的意思呢。这也是有些前端迷之自信的原因。

## 合格的前端

基础：node.js npm webpack vue react react\-native ios android fluter 数据库

概念：v\-dom 插件 callback es6 基础 异步编程 缓存

设计：路由器 权限验证

语言：js ts java oc sql\-query dart html xml yaml css/div

## 行业分析

从游戏行业的前端看，后端正在被弱化，基本可能一个游戏公司一堆前端，后端也就几个，而且，可能几个项目共享这几个后端。估计后面可能互联网公司也会这样。也就是：以后前端要做全栈工程师，有那么点风水轮流转的感觉。

那后端慢慢可能慢慢变成：

1. api 接口，更关注业务的具体逻辑。
2. 数据库存储，
3. 服务器，如：安全和分布式部署
4. 框架，如：微服务
5. 大数据

但是，前端的发展史完全是照抄后端，那么，后端的这个'过程'，其实，前端应该也是这样，因为当前端进化到一定程度，如：完整期、平衡期后，各种框架及设计模式、思想被统一后，那么前端也会大批量的下岗，如：unity ue 对比=\> java\-spring

## 交流

跟同事交流，他提到一点现在的 WEB 前端正慢慢转换成前 10 年的 CS 模式，我觉得这个总结挺有道理。

VUE\+NODE.JS 还真是干了以前 CS 模式的 client 端，很重。过去的 10 年是，可能是重后轻前，现在又反过来是重前轻后。讽刺的是：现在转变成前 10 年的模式，未来 10 年没准可能又换成现在的模式。变来变去，其核心就那么点东西，像我记这篇笔记，满打满算就一天的时间，之所以快，是底层的原理都懂....

我想起我领导的一句话：UNIX LINUX 这都老古董级别的东西，为啥现在还在用？而且全世界都在用？

## 吐槽

大前端的兴起，让一些前端开始了迷之自信。

1. 会一个 JS/TS\+NODE.JS 就完全够行走天下了。
2. VUE 是非常牛逼的一个东西，是神~完全把 VUE 给神化了...

对于这种想法，我是真无语啊。再牛逼的 JS 他也只是一个前端，而一个公司的整体架构，是多语言整合\(配合着完成所有任务\)

比如：

PHP：适合写 WEB 端接口，适合中小型项目，如：变化快 迭代快 开始周期短
PY 适合中小型项目的 BI 分析
JAVA-SPIRNG：适合中大型 项目
JAVA-SPRING-CLOUE:适合大型项目
GO:基于 JAVA 跟 PHP 中间的语言，小而美，但还可以性能高，并写后端守护进程的模式的项目
C\# .NET :在 WIN 的软件开发，还有游戏行业，还是非常 好的。
C/C\+\+:超底层的东西，非常讲究性能的时候，这个非常 适合。

还有各种开源的软件/中间件：

1. 队列系统
2. 负载均衡器
3. 连接管理器
4. 链路追踪器
5. 搜索引擎
6. 日志管理体系
7. gitlab
8. 发布/布置
   ......

退一步讲，都不说这些复杂的，就随随便便说个算法：读时写 二叉树 hashMap 八皇后 动态规划 bitmap geo 递归 TCP 拥塞策略，真是分分钟秒了你啊....

哎，反正总结下来吧，归根结底：还是前端的基础知识储备太差。
