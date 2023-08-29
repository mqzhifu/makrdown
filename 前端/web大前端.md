# web大前端

## 原因

公司同事离职，留了个项目给我，是VUE写的，且还得继续更新维护，既然又得重新搞前端，这次就好好看看吧，短时间肯定不能完全搞的很透，但是大概的流程可以理理。

## 目的

1. 宏观了解下现在的前端技术栈
2. 对比下早期与现在的前端技术区分
3. 将自己的旧时前端技术升级一下
4. 避免有人忽悠我
5. 做全局构架更得心应手

## 回顾

从09年毕业开始，前端的布局刚刚从TABLE转成DIV，速度确实比以前快了，但是前端确实也开始有那么点设计模式的感觉，同时也加大了程序员的工作量。

工作点：JS\+HTML\+css/div

基本phper以上都得写，之后HTML\+css/div终于被分离给前端了，但HTML\+css/div只是少量写，同时是得帮助管理，把静态文件嵌入到项目中、发布布置等，实际上还是没有完全分离。

这之后出现了一些前端类库/框架:

1. Django
2. extjs
3. jquery
4. angular
    ...

一堆吧，反正那个时候感觉也不少，但还是jquery使用最多，主要是小巧，用起来简单方便。

再之后就是AJAX的出现，单页面不用刷新即可动态获取数据，动态更新数据了.

再往后，前端提出OOP思想，做封装，像：闭包、原型链、作用域等开始出现。

再往后，有个：backbone框架,终于有个类似mvc的前端框架出现，然而，我基本专注后端了。。。。

backbone:2010年出品，它不是一个单独的东西，还得配合template.js undecode.js。但是出生太早了，被后面的追赶着超越了，基本退出历史舞台了。

以backbone作为分割点吧，基本上我就告别前端，专注后端了，而思想就停留在：

js\+jquery\+ajax\+oo\+少量html css div

个人对前端的核心点认知：浏览器如何解析hml css，v8如何执行代码，js如何操纵dom节点,JS的OOP原理。

这会对前端的总结：只能写点最最基础的交互，连HTTP协议都不懂，我更认为这会的前端就是个'转换器'，把PS转换成HTML，借用后端、浏览器帮忙，才能开发一个项目。

## SOC

先看一下这个单词：Separation of concerns ，直译 ：关注点分离

> 好像概念有点大，有个哥们出了本书就描述这个东西。

核心概念：分离，概念有点大，是V跟M分？ctrl跟model分？还是前后端分享？我觉得：以上都得分离。

既然我能列出就证明以上这些，就证明确实可能得分离，所以都得分。那都得分，就得确定怎么分？另一个概念：关注点。如何分：个人感觉就是抽象...

1. 可抽象的
2. 变化较多
3. 基于以上两点，分享后，还不能影响到其它的.

映射到前端上就是：VUE就是这个，只是它比较大。它把view model ctrl 抽象成了mvvm

VUE内部可复用的代码，如：组件~

组件内部又可以嵌入其它组件，继续抽象，如：slot

再结合看：关注/专注点，这个词，它就是能把所有可抽象的东西都抽象出来，然后，再分别派发给某个专业的人，这个人就是专业的人。也就是这个人只:关注/专注 某个点。

两个词结合看：把项目代码结构，根据一定特性/关注点，进行抽象，然后分拆，最后就变成了，专业的人干专业的事儿。

## 大前端

**ES**

JAVASCRIPT:1995年由Netscape公司开发，嵌入WEB中。1996年提交给ECMAS，希望国际化。

ECMA:欧洲计算机制造联合会

JS被该协会定义为：ECMAScript

**ES6**

就是一个ES版本，2015年正式发布。不过，2018年ES9已经正式发布了，ES10草案好像也正常了。

不过，主流能做到完全兼容的是ES5，ES6算是一个跨度比较大的，那就简单描述下，ES6增加的东西

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

看着加了不少东西，但感觉咋说呢，就是把JS后端化、复杂化。也就是：JS由嵌入型脚本，想转正为一门正式的后端语言的语言。像：异步编程、OOP 、包管理。

个人觉得：有那么点PHP想要写后端的意思，然后，各种写类库，加扩展，但结果被GO给取代了。前端真是瞎折腾。

## type script

又是个逗比的东西，ts代码最后还是被编译成js，但是他里面做了一些强类型的操作，实际上，是在ES6的基础上，又加强了JS向后端进化的一个过程。

## node.js

可以写后端的js，那从宏观看，JS真是完全可以写后端了。

基于V8，简单说就是：用 C\+\+ 把V8引擎又封装了一层，专供执行JS代码的。

内部原理：event loop模型，详细的我也不熟，空了可以看看，但感觉得先看V8引擎。

但讽刺的是，作者好像放弃更新了，因为太臃肿，随便写个东西，得先下载2w个文件....新的好像叫什么：deno......

## NPM

JS 类包管理，跟maven composer 一个鸟东西，也有个package.json文件用来配置源

> 好像是用node.js编写的...安装node会连带着一并给安装了

还有个cnpm，估计跟NPM一样，淘宝的一个代理NPM的东西，我猜就是改了个源地址...

> 有趣的是 npm instal cnmp 哈哈

然后，就发现跟我猜的一样，cnmp果然就是改了个源，因为可以：

> npm instal xxxx \-\-registry=http://wwwww

这东西居然，还能执行脚本... 有点刷新认知，干了一些程序员或者shell docker的活。

> npm run xxxx

原理：package.json 中有个key叫： scripts ，它里面包含的每一项 就是需要执行的脚本

> npm run xxxx 其中的xxxx 会当做参数，传到 scripts 中

JS有了node 和 npm，就有玩头了~可以上传/下载包，还能通过指令行来执行包~

像实力强的大厂\(vue\)，完全可以用NODE编写脚手架代码，然后封装成包上传，使用者先用NPM把包下载下来，再执行node.js，接着这个包就可以在使用者的机器上动态生成构架文件了~

基本上后端的一套玩法，JS都能玩儿了。

> 不过，就一个VUE最最最简单的项目，要下载2W多个文件,也难怪NODE创始要放弃NODE

## webpack

> npm install webpack \-g
> 
> 
> npm install webpack\-cli \-g

打包工具 , 把你项目中的JS代码，打包成一个JS文件，再压缩，然后生成一个新的文件。

> 不过，你得得按照它的方式来写代码，如：得有个modules，得有main入口

最后定义一个webpack.config.js定义一下modules目录及输出位置。

> 另外，他能把你的ES6打包成ES5格式

先不看各种框架，单从这几个基础的软件：es6 ts node webpack ，JS已经由我最初的认知：只能写写前端，现在真的能根本上实现了，JS也可以写后端了。尤其再配合它的包管理工具，大体上就跟后端的模式90%都相似了。

看看webpack的大体语法

1. entry:一个项目的入口文件,有点main函数的意思。这里可以有多个，一般是一个，因为是单面应用。webpack开始打包时，得找到某些项目的入口，由此入口再依次编译所依赖的东西。
2. output:打包好的文件，输出的位置

```
    path: path.resolve(__dirname, '../dist'),
    publicPath: '/assets/',
    filename: 'static/js/[name].js',
    chunkFilename: '[id].bundle.js',
```

1. module:自定义要加载哪些文件，不加载哪些文件，使用什么加载器loader
2. devServer:开启一个http守护进程~dev=开发，server:服务器，给前端开发者启动一个后端HTTP监听进程。

> 它内部是另一包，webpack\-dev\-server，开启后，output参数基本失效，但支持热打包，即：你JS有修改，立刻可以查看到结果。其实也挺好理解，它是个守护进程，能时时监听到所有，那么，output的打包文件，被加载到内存中，同时监听文件变化。

```
port:
proxy:
```

配置好webpack.config.js后，开始打包：

> webpack

会在output指定的位置生成打包好的文件。

> 之前我跟前端联调，他们居然还要打包编译... 我就不太理解，一个破前端还用编译打包？js文件编译个毛线，JS得加载到浏览器才能运行，这不逗我呢？现在看完，也能理解了。

整体看现在的前端的模式：用node.js日常开发，配合NPM包管理，最后把这些JS\\CSS\\IMG，统一用webpack打包成一个新的浏览器可执行的文件，最后开启web\-service监听。

基本上后端的开发流程，前端都可以串起来了。

感觉还是有点绕，node.js能写点后端，如：webservice，同时支持一些后端语言的语法，如：强类型、import/export等，最后打包。但问题是打包完了还是.js文件，这个就有点讽刺。不过也可能是因为它得内嵌到浏览器，资源也比较碎片化，加上这个流程确实是方便些。

这里有篇文章，可以细细品品:

> https://zhuanlan.zhihu.com/p/32148338

感觉前端真的是瞎JB折腾，看似是想后端体系挣脱出来，想照搬后端模式，想要模块化、OOP，又想简单化，最后搞的真是复杂到你想放弃。我依然觉得：JS\+JQUERY更好，因为复杂过度后，就一个问题你无法解决：如何排查错误？所有的代码都是经过框架打包组合后生成，尤其JS被打包成一个大大的JS文件。

## 前端三大框架

Angular:2012年，google的几个JAVA后端搞得一个框架，基于MVC，肯定是后端使用非常顺，对前端不太友好，另外它也有点重，版本兼容也不太好。但它的功劳是 比较早的整合了MVC模式\(个人感觉backbone更早\)。

react:facebook出品，2013，首次提出虚拟DOM。

vue:2014年，国人设计，有mvvm设计模式，也有虚拟dom模式，同时又非常小，又有组件化且专注于view层。

总的来说：年代越新，融合的思想也就越好，用的人也最多。但更新真是太快太乱了，早期的JUQERY\+JQUERY\-UI，再到，前几年感觉react一统天下的意思，这2年风向就变了，完全转身VUE了。

## mvvm

model view view\-model：跟mvc差不多，就是个框架的设计模式。它的诞生，算是终于给前端统一了设计模式，可持续发展的模式。

> 感觉前端发展就是当时后端的发展过程，但感觉真的是一阵乱射，最终有几支箭射中了，大家再回到这个正确的方向，之后继续乱射....还是基础不行啊~

## 虚拟dom

前置说明:前端的日常开发，基本上就是修改dom树，然后，浏览器根据修改内容再重新做渲染。好像现在有工具能直接把PS生成HTML\+CSS/DIV.

vdom:用js虚拟出一层dom

原理：先创建一个JS对象，然后，把某个\(也可能包含多个子DOM节点\)真实的dom节点绑定到这个js对象上，同时再给这个JS对象绑定上自己的日常操作函数，这样，所有的操作都是操作这个JS对象，而不直接操作dom，最后虚拟dom可能会修改dom也可能不改。

> 相比之下，jquery就是直接操作DOM

理论上：比一直修改dom树造成浏览器平凡重新渲染，效率更高些。

所以，我们看上面的原理，核心点就是这个JS对象，其实这个对象就是VUE...react

## 优点

1. 性能可能会加强，避免浏览器一直重复渲染
2. 程序员可能更专注于在JS上，而不是HTML标签上
3. 有组件思想，代码利用率更高

## 缺点

1. 学习成本、使用成本
2. 复杂化，造成更大的维护成本

虚拟dom的诞生也就是mvvm中的vm，其结果就是vue react引领的这场变革，所有的前端代码就都得开始玩vdom了

基本上理解了vm，能够套用到mvvm模式中，大体上也就理解了现在的前端开发模式的核心了，剩下的就是围绕vm的各种操作了。

## 单页应用

## VUE

因为它现在最火，就拿出它来学习下。

它其实就是用JS写的一个库，然后实例化成一个JS\-OBJ，就是V\-DOM类型.

> 同类型对比LINUX系统：用户态与内核态，内核态就是VUE

原本可能松散的JS代码，被统一整理到VUE中，如：前端程序员把各种回调函数注册到这个JS\-OBJ中，用户的操作第一时间也是操作这个JS\-OBJ，等于中间加了整整一层而以。

这种JS\-OBJ，可能一个页面就有N个，所以，它也有生命周期的概念：

beforeCreate created beforeMount mounted beforeUpdate udpated destory

> 感觉又是安卓 IOS 那一套.....问题是：安卓 IOS 那一套是真的给一个完整的APP在上面运行的，你的APP代安装时，得经过编译成可执行文件，放在硬盘上，然后OS直接可以执行了... 而JS这垃圾，并不是直接运行在OS，是运行在浏览器里，浏览器还不能直接执行，还得再给V8引擎，最后VUE又强行模拟出来这么一层....

这个生命周期，其实也能更好的解释什么是：V\-DOM

生命周期这一堆函数，基本上可以统一定义为：钩子，如：

创建一个APP，在最终挂载成功之前，要用AJAX获取到后端数据才行，那用mound函数。

**vue七大vm属性**

data:view 层需要的数据均从这里拿

methods:前端程序用来控制DOM变化的一些自定义

el:绑定到真实的某个DOM节点ID

template:模板

compute:跟methords有点像，也是一些自定义函数，但它是属性，常驻内存\(前提不发生改变\)，有那么点缓存的意思。

watch:

render:

另外还有一些其它的属性：

slot:插槽,类似页面布局，定好布局后，就是一个槽一个槽，然后你再动态的替换.

props:接收外层bind的数据

接上面的总结，理解了VM的原理，接着就是在VM上做各种操作，日常的开发基本就是在这7个属性中了.

> 其实高级VUE我猜，就是VUE内部的这种V\-DOM间的操作，可能是同级，也可能是父子组，也也可能是组件之间

vue确实够小巧，并且保持专注，其他的一概不管，如果想有更多功能，那就引入其他包！！！

> 感觉vue跟我当年的jquery挺像，任何简单的东西都可能会火~

## vuex

类库，状态管理

它更像是一个仓库，全局仓库，或者叫全局本地仓库~

每个组件之间：一但某一个model层的数据发生变化，连带着可能所有组件都得跟着变，那么组件间的数据通信就是个问题。这个库就是解决这个问题，将各种数据的变化定义成状态，然后存到仓库中，并设置当发生改变后的回调函数。

1. state:可以理解为仓库里的具体数据
2. mutations:有一些方法，可以上面state里的数据
3. action:做异常的处理，有那么点钩子的意思
4. getter:读取state里的数据，是个函数，也有那么点钩子的意思
5. module:如果这种全局状态数据过多，就把一些特定的数据绑定到固定的模块上

外部类，想要修改state里的数据，必须得调用它的mustations里的方法修改。

> $store.commit\("mutations里的函数名"\)

mutations里的函数就是真正修改state里的数据。

action的使用

> $store.dispach

咋个形容呢，这东西的初衷我能理解...包括 各种钩子，可能出现锁等等... 初衷是好的，但 小项目，你就定义个全局变量不就得了...

## VuexPersistence

配合vuex使用。

vuex存储了全局数据后，还是在内存中~刷新后，数据丢失，如果能存储，那就完美了，于是有了VuexPersistence，帮助vuwx持久化数据的。

还得先讲下：localStorage

**localStorage**

HTML5中，新加入了一个localStorage特性，本地仓库。解决了cookie存储空间不足的问题\(cookie中每条cookie的存储空间为4k\)，localStorage中一般浏览器支持的是5M大小，这个在不同的浏览器中localStorage会有所不同。

相同域名/端口，多TAB可以共享。

这个东西挺好，等于WEB前端有本地数据库了，缓存点后端API数据、存点自己的东西，可方便多了。

还有个sessionStorage，跟它有点像，但是页面关闭后即消失，也是正常，为了安全嘛~

VuexPersistence就是基于localStorage，感觉也没啥创新，估计就是封装一层或做点兼容之类的吧。

## vue\-router

类库，路由器

这东西主要是组件之间的跳转

后端对路由的理解：做分发\+真的跳转，还是模式不一样，前端：是由用户驱动，N个组件响应模式，就是组件之间的跳转，而后端有点像是'被动式'接收前端请求，然后路由主要是做分发功能

组件的对比：后端的组件直接就挂在容器或全局变量上了，复杂一点的就走RPC了。

## vue\-cli

一个指令行控制的插件，帮助开发者快速搭建、打包VUE。

> npm install vue\-cli

安装好后，我们找个目录，初始化一个VUE的项目

> vue init webpack xxxxxx

这个时候，目录下会有一堆vue\-cli脚本创建的新目录和新文件

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

从这个结构看，之前所有的VUE学习DEMO、视频的写法，均是用于演示，而真实的项目开发，不可能那么零散。还有实际项目中可能会用到N多个类库，让你随便放也有点LOW，所以，它就有个cli 这个鬼东西，类似代码结构目录生成器，来帮助使用者，快速按照一定结构创建一个项目.

接着开始下载安装各种依赖包：

> npm install

哎，这东西不挂 VPN，基本上是会挂....

启动

> npm run dev

问题：它没有webpack.config.js ，那它是如何打包能运行的呢？如：

1. 没有entry，那它如何找到src/main.js 的？
2. output 虽然在vue.config.js 中有定义，可为啥不输出呢？
3. index.html不在根目录，它是如何找到的呢？

解决：

1. node\_modules/@vue/cli\-service/lib/config/base.js中找见到了... entry:/src/main.js
2. webpack如果配置文件中开启了devServer ，就不再输出了
3. 没找到...

**启动原理**

> 从package.json中找script\-\>vue\-cli\-service serve

vue\-cli\-service 居然在 node\_modules/.bin 下，是个软连接，估计是npm 的快捷方式

打开后，发现包含了 ../lib/Service ，咋也找不见...最后：

> node\_modules/@vue/cli\-service/lib/Service.js 找见了....

@vue 这是什么鬼？

vue 整整包了一层webpack....

## VUE创建新项目流程

1. 安装NODE.JS
2. 安装NPM
3. 安装包：vue\-cli webpack
4. 初始化一个新项目:vue init webpack XXXX
5. 安装这个新项目的依赖包 npm install
6. 开始正常编写代码,src 目录
7. 项目编写完成，启动/测试 npm run server
8. 启动过程\-\>package.json\-\>script\-\>build.js
9. build\-\>检查当前环境变量\-\>根据指令行参数，决定选择哪个环境变量的buile脚本\(就是配置不一样\)\-\>webpack打包JS文件\-\>再找到项目的配置文件\-\>启动webservice~

## react

因为项目就是VUE，精力全花在了VUE上面，但我还是简单查了查这两个的对比，大致：

1. react比较开放，有那么点原生JS的感觉，并不会像VUE插手那么多。
2. vue有点框死了，就往V\-DOM OBJ上挂各种函数，有那么点 闭包的感觉。

区别：

> 前置：没有哪个好哪个坏，一切看使用场景。

1. vue因为封装的好，学习成本低，上手容易，好招人，也不容易多强的技术，适合中小项目。
2. react比较开放，需要各种造轮子。适合大厂，有牛B的技术。学习成本有点高。

另外，react好像插件比较多，有那么点JAVA SPRING的意思，对TS兼容好，它还有react native加持，在移动客户端也有所建树。

## flutter

之前也只是听说，现在依然也只是听说，但看了看文档，感觉这个东西才是比较牛B的呢。

vue 我是没找到能统一三端的插件。

react 可以，但他必须依赖V8

fluter google出品，单独有个引擎，能支持3端

## axios

类库，网络管理

http XMLHttpRequests ajax

就是把上面的，有网络请求相关的操作，统一封装了一层，把原本要程序员写的一堆代码统一抽离封装，并且有一定的数据映射功能（如：ORM）

基本上vue\+vuex\+router\+axios就等于后端的一个比较完整的框架了。

反正做为后端的我比较喜欢这种模式，大而全的东西，注定过重，且学习成本太高。反而由程序员手动组建一个框架更舒服。

> 从VUE的整体模式来看，就是把前端以前各种松散的代码，进行 抽象、抽离、封装成统一的类库，再给程序员使用。TMD，怎么看都是在跟后端学...

## 前端ui框架

ant design:阿里，react

elementui iview ice :vue

amazeui:

layui

docsify

## elementui

> npm I element\-ui \-S
> 
> 
> cnpm install sass\-loader node\-sass

至此：大前端的整体流程就都穿起来了。需要掌握的知识点也就如上所述呀。

## 多端

早期的开发基本就一端：PC，而现在要考虑PC H5 andrio IOS，另外：PC H5 在我那会儿有个很头疼的问题：浏览器兼容。

所以，从这个角度看，前端的发展与进化，多多少少也是被市场需求所逼，它如果再不进化，那么这N端及浏览器兼容真的会大大影响效率。

结合看VUE好像在移动端没有react native、fluter更强，那么，在做构架的时候就得好好想想了

> ps:从这里也能理解出，确实VUE可能对大项目不适合

如果所有公司的想法都是：兼容多端，那么可悲的就是现在的android ios 的开发程序员了，只会单一载体开发，被JS降维打击，必然是大批失业潮。\(感觉这里JS前端又得迷之自信一会儿...\)

## 总结

猛一看，前端这几年：一堆新鲜的东西，backbone react vue angular node.js npm 等等， 但是一地鸡毛...其核心的发展史还是在向后端学习，而之所以一地鸡毛，根本原因：还是基础太差，造成想法飘逸而不且实际，框架百出，但没有能力去认识思考底层的原理。

但是，前端确实是在进步,大前端的概念抛出来，前端确实比以前要学习很多知识。尤其不能光光盯着前端那点东西，还得会点nodejs，懂点后端知识。也让后端不再那么孤独（拿一样的薪水，需要学的知识却比前端多一堆\!\!\!）。另外，也能驱散一些前端混子。

从后端看这种趋势，喜剧参半：

1. 好的是，因为现在的前端：已经把后端大部分的工作都拿过去了，如：action路由、本地缓存、权限验证等。
2. 不好的是：后端的生存空间被压缩。

这就挺纠结，跟前端对接口，总觉得他们笨，要后端做很多，但真的被拿过去后，发现：自己不再是最重要的那个后端了....

## 冲突

按照总结后的看，前端在整体后端的东西，那么，以node诞生来看，前端未来会不会无聊的又搞出后端的东西？如：队列系统、日志系统等。我觉得他们干的出来。那么，行业又得出现一次混乱期，有那么点：分分合合，合合分分的意思呢。这也是有些前端迷之自信的原因。

## 合格的前端

基础：node.js npm webpack vue react react\-native ios android fluter 数据库

概念：v\-dom 插件 callback es6基础 异步编程 缓存

设计：路由器 权限验证

语言：js ts java oc sql\-query dart html xml yaml css/div

## 行业分析

从游戏行业的前端看，后端正在被弱化，基本可能一个游戏公司一堆前端，后端也就几个，而且，可能几个项目共享这几个后端。估计后面可能互联网公司也会这样。也就是：以后前端要做全栈工程师，有那么点风水轮流转的感觉。

那后端慢慢可能慢慢变成：

1. api接口，更关注业务的具体逻辑。
2. 数据库存储，
3. 服务器，如：安全和分布式部署
4. 框架，如：微服务
5. 大数据

但是，前端的发展史完全是照抄后端，那么，后端的这个'过程'，其实，前端应该也是这样，因为当前端进化到一定程度，如：完整期、平衡期后，各种框架及设计模式、思想被统一后，那么前端也会大批量的下岗，如：unity ue 对比=\> java\-spring

## 交流

跟同事交流，他提到一点现在的WEB前端正慢慢转换成前10年的CS模式，我觉得这个总结挺有道理。

VUE\+NODE.JS 还真是干了以前CS模式的client端，很重。过去的10年是，可能是重后轻前，现在又反过来是重前轻后。讽刺的是：现在转变成前10年的模式，未来10年没准可能又换成现在的模式。变来变去，其核心就那么点东西，像我记这篇笔记，满打满算就一天的时间，之所以快，是底层的原理都懂....

我想起我领导的一句话：UNIX LINUX 这都老古董级别的东西，为啥现在还在用？而且全世界都在用？

## 吐槽

大前端的兴起，让一些前端开始了迷之自信。

1. 会一个JS/TS\+NODE.JS 就完全够行走天下了。
2. VUE是非常牛逼的一个东西，是神~完全把VUE给神化了...

对于这种想法，我是真无语啊。再牛逼的JS他也只是一个前端，而一个公司的整体架构，是多语言整合\(配合着完成所有任务\)

比如：

PHP：适合写WEB端接口，适合中小型项目，如：变化快 迭代快 开始周期短

PY 适合中小型项目的BI分析

JAVA\-SPIRNG：适合中大型 项目

JAVA\-SPRING\-CLOUE:适合大型项目

GO:基于JAVA跟PHP中间的语言，小而美，但还可以性能高，并写后端守护进程的模式的项目

C\# .NET :在WIN的软件开发，还有游戏行业，还是非常 好的。

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

退一步讲，都不说这些复杂的，就随随便便说个算法：读时写 二叉树 hashMap 八皇后 动态规划 bitmap geo 递归 TCP拥塞策略，真是分分钟秒了你啊....

哎，反正总结下来吧，归根结底：还是前端的基础知识储备太差。

%E5%8E%9F%E5%9B%A0%0A\-\-\-\-%0A%E5%85%AC%E5%8F%B8%E5%90%8C%E4%BA%8B%E7%A6%BB%E8%81%8C%EF%BC%8C%E7%95%99%E4%BA%86%E4%B8%AA%E9%A1%B9%E7%9B%AE%E7%BB%99%E6%88%91%EF%BC%8C%E6%98%AFVUE%E5%86%99%E7%9A%84%EF%BC%8C%E4%B8%94%E8%BF%98%E5%BE%97%E7%BB%A7%E7%BB%AD%E6%9B%B4%E6%96%B0%E7%BB%B4%E6%8A%A4%EF%BC%8C%E6%97%A2%E7%84%B6%E5%8F%88%E5%BE%97%E9%87%8D%E6%96%B0%E6%90%9E%E5%89%8D%E7%AB%AF%EF%BC%8C%E8%BF%99%E6%AC%A1%E5%B0%B1%E5%A5%BD%E5%A5%BD%E7%9C%8B%E7%9C%8B%E5%90%A7%EF%BC%8C%E7%9F%AD%E6%97%B6%E9%97%B4%E8%82%AF%E5%AE%9A%E4%B8%8D%E8%83%BD%E5%AE%8C%E5%85%A8%E6%90%9E%E7%9A%84%E5%BE%88%E9%80%8F%EF%BC%8C%E4%BD%86%E6%98%AF%E5%A4%A7%E6%A6%82%E7%9A%84%E6%B5%81%E7%A8%8B%E5%8F%AF%E4%BB%A5%E7%90%86%E7%90%86%E3%80%82%0A%0A%E7%9B%AE%E7%9A%84%0A\-\-\-\-%0A1.%20%E5%AE%8F%E8%A7%82%E4%BA%86%E8%A7%A3%E4%B8%8B%E7%8E%B0%E5%9C%A8%E7%9A%84%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF%E6%A0%88%0A2.%20%E5%AF%B9%E6%AF%94%E4%B8%8B%E6%97%A9%E6%9C%9F%E4%B8%8E%E7%8E%B0%E5%9C%A8%E7%9A%84%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF%E5%8C%BA%E5%88%86%0A3.%20%E5%B0%86%E8%87%AA%E5%B7%B1%E7%9A%84%E6%97%A7%E6%97%B6%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF%E5%8D%87%E7%BA%A7%E4%B8%80%E4%B8%8B%0A4.%20%E9%81%BF%E5%85%8D%E6%9C%89%E4%BA%BA%E5%BF%BD%E6%82%A0%E6%88%91%0A5.%20%E5%81%9A%E5%85%A8%E5%B1%80%E6%9E%84%E6%9E%B6%E6%9B%B4%E5%BE%97%E5%BF%83%E5%BA%94%E6%89%8B%0A%0A%E5%9B%9E%E9%A1%BE%0A\-\-\-\-\-\-%0A%E4%BB%8E09%E5%B9%B4%E6%AF%95%E4%B8%9A%E5%BC%80%E5%A7%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E7%9A%84%E5%B8%83%E5%B1%80%E5%88%9A%E5%88%9A%E4%BB%8ETABLE%E8%BD%AC%E6%88%90DIV%EF%BC%8C%E9%80%9F%E5%BA%A6%E7%A1%AE%E5%AE%9E%E6%AF%94%E4%BB%A5%E5%89%8D%E5%BF%AB%E4%BA%86%EF%BC%8C%E4%BD%86%E6%98%AF%E5%89%8D%E7%AB%AF%E7%A1%AE%E5%AE%9E%E4%B9%9F%E5%BC%80%E5%A7%8B%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E6%84%9F%E8%A7%89%EF%BC%8C%E5%90%8C%E6%97%B6%E4%B9%9F%E5%8A%A0%E5%A4%A7%E4%BA%86%E7%A8%8B%E5%BA%8F%E5%91%98%E7%9A%84%E5%B7%A5%E4%BD%9C%E9%87%8F%E3%80%82%0A%0A%E5%B7%A5%E4%BD%9C%E7%82%B9%EF%BC%9AJS%2BHTML%2Bcss%2Fdiv%0A%0A%E5%9F%BA%E6%9C%ACphper%E4%BB%A5%E4%B8%8A%E9%83%BD%E5%BE%97%E5%86%99%EF%BC%8C%E4%B9%8B%E5%90%8EHTML%2Bcss%2Fdiv%E7%BB%88%E4%BA%8E%E8%A2%AB%E5%88%86%E7%A6%BB%E7%BB%99%E5%89%8D%E7%AB%AF%E4%BA%86%EF%BC%8C%E4%BD%86HTML%2Bcss%2Fdiv%E5%8F%AA%E6%98%AF%E5%B0%91%E9%87%8F%E5%86%99%EF%BC%8C%E5%90%8C%E6%97%B6%E6%98%AF%E5%BE%97%E5%B8%AE%E5%8A%A9%E7%AE%A1%E7%90%86%EF%BC%8C%E6%8A%8A%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%E5%B5%8C%E5%85%A5%E5%88%B0%E9%A1%B9%E7%9B%AE%E4%B8%AD%E3%80%81%E5%8F%91%E5%B8%83%E5%B8%83%E7%BD%AE%E7%AD%89%EF%BC%8C%E5%AE%9E%E9%99%85%E4%B8%8A%E8%BF%98%E6%98%AF%E6%B2%A1%E6%9C%89%E5%AE%8C%E5%85%A8%E5%88%86%E7%A6%BB%E3%80%82%0A%0A%E8%BF%99%E4%B9%8B%E5%90%8E%E5%87%BA%E7%8E%B0%E4%BA%86%E4%B8%80%E4%BA%9B%E5%89%8D%E7%AB%AF%E7%B1%BB%E5%BA%93%2F%E6%A1%86%E6%9E%B6%3A%0A1.%20Django%20%0A2.%20extjs%20%0A3.%20jquery%20%0A4.%20angular%0A...%0A%0A%E4%B8%80%E5%A0%86%E5%90%A7%EF%BC%8C%E5%8F%8D%E6%AD%A3%E9%82%A3%E4%B8%AA%E6%97%B6%E5%80%99%E6%84%9F%E8%A7%89%E4%B9%9F%E4%B8%8D%E5%B0%91%EF%BC%8C%E4%BD%86%E8%BF%98%E6%98%AFjquery%E4%BD%BF%E7%94%A8%E6%9C%80%E5%A4%9A%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E5%B0%8F%E5%B7%A7%EF%BC%8C%E7%94%A8%E8%B5%B7%E6%9D%A5%E7%AE%80%E5%8D%95%E6%96%B9%E4%BE%BF%E3%80%82%0A%0A%E5%86%8D%E4%B9%8B%E5%90%8E%E5%B0%B1%E6%98%AFAJAX%E7%9A%84%E5%87%BA%E7%8E%B0%EF%BC%8C%E5%8D%95%E9%A1%B5%E9%9D%A2%E4%B8%8D%E7%94%A8%E5%88%B7%E6%96%B0%E5%8D%B3%E5%8F%AF%E5%8A%A8%E6%80%81%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%8A%A8%E6%80%81%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E4%BA%86.%0A%0A%E5%86%8D%E5%BE%80%E5%90%8E%EF%BC%8C%E5%89%8D%E7%AB%AF%E6%8F%90%E5%87%BAOOP%E6%80%9D%E6%83%B3%EF%BC%8C%E5%81%9A%E5%B0%81%E8%A3%85%EF%BC%8C%E5%83%8F%EF%BC%9A%E9%97%AD%E5%8C%85%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E4%BD%9C%E7%94%A8%E5%9F%9F%E7%AD%89%E5%BC%80%E5%A7%8B%E5%87%BA%E7%8E%B0%E3%80%82%0A%0A%E5%86%8D%E5%BE%80%E5%90%8E%EF%BC%8C%E6%9C%89%E4%B8%AA%EF%BC%9Abackbone%E6%A1%86%E6%9E%B6%2C%E7%BB%88%E4%BA%8E%E6%9C%89%E4%B8%AA%E7%B1%BB%E4%BC%BCmvc%E7%9A%84%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6%E5%87%BA%E7%8E%B0%EF%BC%8C%E7%84%B6%E8%80%8C%EF%BC%8C%E6%88%91%E5%9F%BA%E6%9C%AC%E4%B8%93%E6%B3%A8%E5%90%8E%E7%AB%AF%E4%BA%86%E3%80%82%E3%80%82%E3%80%82%E3%80%82%0A%0Abackbone%3A2010%E5%B9%B4%E5%87%BA%E5%93%81%EF%BC%8C%E5%AE%83%E4%B8%8D%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8D%95%E7%8B%AC%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E8%BF%98%E5%BE%97%E9%85%8D%E5%90%88template.js%20undecode.js%E3%80%82%E4%BD%86%E6%98%AF%E5%87%BA%E7%94%9F%E5%A4%AA%E6%97%A9%E4%BA%86%EF%BC%8C%E8%A2%AB%E5%90%8E%E9%9D%A2%E7%9A%84%E8%BF%BD%E8%B5%B6%E7%9D%80%E8%B6%85%E8%B6%8A%E4%BA%86%EF%BC%8C%E5%9F%BA%E6%9C%AC%E9%80%80%E5%87%BA%E5%8E%86%E5%8F%B2%E8%88%9E%E5%8F%B0%E4%BA%86%E3%80%82%0A%E4%BB%A5backbone%E4%BD%9C%E4%B8%BA%E5%88%86%E5%89%B2%E7%82%B9%E5%90%A7%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%88%91%E5%B0%B1%E5%91%8A%E5%88%AB%E5%89%8D%E7%AB%AF%EF%BC%8C%E4%B8%93%E6%B3%A8%E5%90%8E%E7%AB%AF%E4%BA%86%EF%BC%8C%E8%80%8C%E6%80%9D%E6%83%B3%E5%B0%B1%E5%81%9C%E7%95%99%E5%9C%A8%EF%BC%9A%0Ajs%2Bjquery%2Bajax%2Boo%2B%E5%B0%91%E9%87%8Fhtml%20css%20div%0A%0A%E4%B8%AA%E4%BA%BA%E5%AF%B9%E5%89%8D%E7%AB%AF%E7%9A%84%E6%A0%B8%E5%BF%83%E7%82%B9%E8%AE%A4%E7%9F%A5%EF%BC%9A%E6%B5%8F%E8%A7%88%E5%99%A8%E5%A6%82%E4%BD%95%E8%A7%A3%E6%9E%90hml%20css%EF%BC%8Cv8%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E4%BB%A3%E7%A0%81%EF%BC%8Cjs%E5%A6%82%E4%BD%95%E6%93%8D%E7%BA%B5dom%E8%8A%82%E7%82%B9%2CJS%E7%9A%84OOP%E5%8E%9F%E7%90%86%E3%80%82%0A%0A%E8%BF%99%E4%BC%9A%E5%AF%B9%E5%89%8D%E7%AB%AF%E7%9A%84%E6%80%BB%E7%BB%93%EF%BC%9A%E5%8F%AA%E8%83%BD%E5%86%99%E7%82%B9%E6%9C%80%E6%9C%80%E5%9F%BA%E7%A1%80%E7%9A%84%E4%BA%A4%E4%BA%92%EF%BC%8C%E8%BF%9EHTTP%E5%8D%8F%E8%AE%AE%E9%83%BD%E4%B8%8D%E6%87%82%EF%BC%8C%E6%88%91%E6%9B%B4%E8%AE%A4%E4%B8%BA%E8%BF%99%E4%BC%9A%E7%9A%84%E5%89%8D%E7%AB%AF%E5%B0%B1%E6%98%AF%E4%B8%AA'%E8%BD%AC%E6%8D%A2%E5%99%A8'%EF%BC%8C%E6%8A%8APS%E8%BD%AC%E6%8D%A2%E6%88%90HTML%EF%BC%8C%E5%80%9F%E7%94%A8%E5%90%8E%E7%AB%AF%E3%80%81%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B8%AE%E5%BF%99%EF%BC%8C%E6%89%8D%E8%83%BD%E5%BC%80%E5%8F%91%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%E3%80%82%0A%0A%0ASOC%0A\-\-\-\-\-\-%0A%E5%85%88%E7%9C%8B%E4%B8%80%E4%B8%8B%E8%BF%99%E4%B8%AA%E5%8D%95%E8%AF%8D%EF%BC%9ASeparation%20of%20concerns%20%EF%BC%8C%E7%9B%B4%E8%AF%91%20%EF%BC%9A%E5%85%B3%E6%B3%A8%E7%82%B9%E5%88%86%E7%A6%BB%0A%3E%E5%A5%BD%E5%83%8F%E6%A6%82%E5%BF%B5%E6%9C%89%E7%82%B9%E5%A4%A7%EF%BC%8C%E6%9C%89%E4%B8%AA%E5%93%A5%E4%BB%AC%E5%87%BA%E4%BA%86%E6%9C%AC%E4%B9%A6%E5%B0%B1%E6%8F%8F%E8%BF%B0%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%E3%80%82%0A%0A%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5%EF%BC%9A%E5%88%86%E7%A6%BB%EF%BC%8C%E6%A6%82%E5%BF%B5%E6%9C%89%E7%82%B9%E5%A4%A7%EF%BC%8C%E6%98%AFV%E8%B7%9FM%E5%88%86%EF%BC%9Fctrl%E8%B7%9Fmodel%E5%88%86%EF%BC%9F%E8%BF%98%E6%98%AF%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E4%BA%AB%EF%BC%9F%E6%88%91%E8%A7%89%E5%BE%97%EF%BC%9A%E4%BB%A5%E4%B8%8A%E9%83%BD%E5%BE%97%E5%88%86%E7%A6%BB%E3%80%82%0A%E6%97%A2%E7%84%B6%E6%88%91%E8%83%BD%E5%88%97%E5%87%BA%E5%B0%B1%E8%AF%81%E6%98%8E%E4%BB%A5%E4%B8%8A%E8%BF%99%E4%BA%9B%EF%BC%8C%E5%B0%B1%E8%AF%81%E6%98%8E%E7%A1%AE%E5%AE%9E%E5%8F%AF%E8%83%BD%E5%BE%97%E5%88%86%E7%A6%BB%EF%BC%8C%E6%89%80%E4%BB%A5%E9%83%BD%E5%BE%97%E5%88%86%E3%80%82%E9%82%A3%E9%83%BD%E5%BE%97%E5%88%86%EF%BC%8C%E5%B0%B1%E5%BE%97%E7%A1%AE%E5%AE%9A%E6%80%8E%E4%B9%88%E5%88%86%EF%BC%9F%E5%8F%A6%E4%B8%80%E4%B8%AA%E6%A6%82%E5%BF%B5%EF%BC%9A%E5%85%B3%E6%B3%A8%E7%82%B9%E3%80%82%E5%A6%82%E4%BD%95%E5%88%86%EF%BC%9A%E4%B8%AA%E4%BA%BA%E6%84%9F%E8%A7%89%E5%B0%B1%E6%98%AF%E6%8A%BD%E8%B1%A1...%0A1.%20%E5%8F%AF%E6%8A%BD%E8%B1%A1%E7%9A%84%0A2.%20%E5%8F%98%E5%8C%96%E8%BE%83%E5%A4%9A%0A3.%20%E5%9F%BA%E4%BA%8E%E4%BB%A5%E4%B8%8A%E4%B8%A4%E7%82%B9%EF%BC%8C%E5%88%86%E4%BA%AB%E5%90%8E%EF%BC%8C%E8%BF%98%E4%B8%8D%E8%83%BD%E5%BD%B1%E5%93%8D%E5%88%B0%E5%85%B6%E5%AE%83%E7%9A%84.%0A%0A%E6%98%A0%E5%B0%84%E5%88%B0%E5%89%8D%E7%AB%AF%E4%B8%8A%E5%B0%B1%E6%98%AF%EF%BC%9AVUE%E5%B0%B1%E6%98%AF%E8%BF%99%E4%B8%AA%EF%BC%8C%E5%8F%AA%E6%98%AF%E5%AE%83%E6%AF%94%E8%BE%83%E5%A4%A7%E3%80%82%E5%AE%83%E6%8A%8Aview%20model%20ctrl%20%E6%8A%BD%E8%B1%A1%E6%88%90%E4%BA%86mvvm%0AVUE%E5%86%85%E9%83%A8%E5%8F%AF%E5%A4%8D%E7%94%A8%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%A6%82%EF%BC%9A%E7%BB%84%E4%BB%B6~%0A%E7%BB%84%E4%BB%B6%E5%86%85%E9%83%A8%E5%8F%88%E5%8F%AF%E4%BB%A5%E5%B5%8C%E5%85%A5%E5%85%B6%E5%AE%83%E7%BB%84%E4%BB%B6%EF%BC%8C%E7%BB%A7%E7%BB%AD%E6%8A%BD%E8%B1%A1%EF%BC%8C%E5%A6%82%EF%BC%9Aslot%0A%0A%E5%86%8D%E7%BB%93%E5%90%88%E7%9C%8B%EF%BC%9A%E5%85%B3%E6%B3%A8%2F%E4%B8%93%E6%B3%A8%E7%82%B9%EF%BC%8C%E8%BF%99%E4%B8%AA%E8%AF%8D%EF%BC%8C%E5%AE%83%E5%B0%B1%E6%98%AF%E8%83%BD%E6%8A%8A%E6%89%80%E6%9C%89%E5%8F%AF%E6%8A%BD%E8%B1%A1%E7%9A%84%E4%B8%9C%E8%A5%BF%E9%83%BD%E6%8A%BD%E8%B1%A1%E5%87%BA%E6%9D%A5%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E5%86%8D%E5%88%86%E5%88%AB%E6%B4%BE%E5%8F%91%E7%BB%99%E6%9F%90%E4%B8%AA%E4%B8%93%E4%B8%9A%E7%9A%84%E4%BA%BA%EF%BC%8C%E8%BF%99%E4%B8%AA%E4%BA%BA%E5%B0%B1%E6%98%AF%E4%B8%93%E4%B8%9A%E7%9A%84%E4%BA%BA%E3%80%82%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%BF%99%E4%B8%AA%E4%BA%BA%E5%8F%AA%3A%E5%85%B3%E6%B3%A8%2F%E4%B8%93%E6%B3%A8%20%E6%9F%90%E4%B8%AA%E7%82%B9%E3%80%82%0A%0A%E4%B8%A4%E4%B8%AA%E8%AF%8D%E7%BB%93%E5%90%88%E7%9C%8B%EF%BC%9A%E6%8A%8A%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84%EF%BC%8C%E6%A0%B9%E6%8D%AE%E4%B8%80%E5%AE%9A%E7%89%B9%E6%80%A7%2F%E5%85%B3%E6%B3%A8%E7%82%B9%EF%BC%8C%E8%BF%9B%E8%A1%8C%E6%8A%BD%E8%B1%A1%EF%BC%8C%E7%84%B6%E5%90%8E%E5%88%86%E6%8B%86%EF%BC%8C%E6%9C%80%E5%90%8E%E5%B0%B1%E5%8F%98%E6%88%90%E4%BA%86%EF%BC%8C%E4%B8%93%E4%B8%9A%E7%9A%84%E4%BA%BA%E5%B9%B2%E4%B8%93%E4%B8%9A%E7%9A%84%E4%BA%8B%E5%84%BF%E3%80%82%0A%0A%0A%0A%0A%E5%A4%A7%E5%89%8D%E7%AB%AF%0A\-\-\-\-%0A%0A\*\*ES\*\*%0AJAVASCRIPT%3A1995%E5%B9%B4%E7%94%B1Netscape%E5%85%AC%E5%8F%B8%E5%BC%80%E5%8F%91%EF%BC%8C%E5%B5%8C%E5%85%A5WEB%E4%B8%AD%E3%80%821996%E5%B9%B4%E6%8F%90%E4%BA%A4%E7%BB%99ECMAS%EF%BC%8C%E5%B8%8C%E6%9C%9B%E5%9B%BD%E9%99%85%E5%8C%96%E3%80%82%0AECMA%3A%E6%AC%A7%E6%B4%B2%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%88%B6%E9%80%A0%E8%81%94%E5%90%88%E4%BC%9A%0AJS%E8%A2%AB%E8%AF%A5%E5%8D%8F%E4%BC%9A%E5%AE%9A%E4%B9%89%E4%B8%BA%EF%BC%9AECMAScript%0A%0A\*\*ES6\*\*%0A%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AAES%E7%89%88%E6%9C%AC%EF%BC%8C2015%E5%B9%B4%E6%AD%A3%E5%BC%8F%E5%8F%91%E5%B8%83%E3%80%82%E4%B8%8D%E8%BF%87%EF%BC%8C2018%E5%B9%B4ES9%E5%B7%B2%E7%BB%8F%E6%AD%A3%E5%BC%8F%E5%8F%91%E5%B8%83%E4%BA%86%EF%BC%8CES10%E8%8D%89%E6%A1%88%E5%A5%BD%E5%83%8F%E4%B9%9F%E6%AD%A3%E5%B8%B8%E4%BA%86%E3%80%82%0A%E4%B8%8D%E8%BF%87%EF%BC%8C%E4%B8%BB%E6%B5%81%E8%83%BD%E5%81%9A%E5%88%B0%E5%AE%8C%E5%85%A8%E5%85%BC%E5%AE%B9%E7%9A%84%E6%98%AFES5%EF%BC%8CES6%E7%AE%97%E6%98%AF%E4%B8%80%E4%B8%AA%E8%B7%A8%E5%BA%A6%E6%AF%94%E8%BE%83%E5%A4%A7%E7%9A%84%EF%BC%8C%E9%82%A3%E5%B0%B1%E7%AE%80%E5%8D%95%E6%8F%8F%E8%BF%B0%E4%B8%8B%EF%BC%8CES6%E5%A2%9E%E5%8A%A0%E7%9A%84%E4%B8%9C%E8%A5%BF%0A%0A%0A1.%20let%0A2.%20promise%20async%20away\(awync%20wait\)%0A3.%20const%0A4.%20import%E3%80%81export%0A5.%20class%E3%80%81extends%E3%80%81super%0A5.%20%3D%3E%20arrow%20functions%20%0A6.%20template%20string%0A6.%20destructuring%0A7.%20default%0A8.%20Generators%0A%0A%E7%9C%8B%E7%9D%80%E5%8A%A0%E4%BA%86%E4%B8%8D%E5%B0%91%E4%B8%9C%E8%A5%BF%EF%BC%8C%E4%BD%86%E6%84%9F%E8%A7%89%E5%92%8B%E8%AF%B4%E5%91%A2%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%8A%8AJS%E5%90%8E%E7%AB%AF%E5%8C%96%E3%80%81%E5%A4%8D%E6%9D%82%E5%8C%96%E3%80%82%E4%B9%9F%E5%B0%B1%E6%98%AF%EF%BC%9AJS%E7%94%B1%E5%B5%8C%E5%85%A5%E5%9E%8B%E8%84%9A%E6%9C%AC%EF%BC%8C%E6%83%B3%E8%BD%AC%E6%AD%A3%E4%B8%BA%E4%B8%80%E9%97%A8%E6%AD%A3%E5%BC%8F%E7%9A%84%E5%90%8E%E7%AB%AF%E8%AF%AD%E8%A8%80%E7%9A%84%E8%AF%AD%E8%A8%80%E3%80%82%E5%83%8F%EF%BC%9A%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E3%80%81OOP%20%E3%80%81%E5%8C%85%E7%AE%A1%E7%90%86%E3%80%82%0A%0A%E4%B8%AA%E4%BA%BA%E8%A7%89%E5%BE%97%EF%BC%9A%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9PHP%E6%83%B3%E8%A6%81%E5%86%99%E5%90%8E%E7%AB%AF%E7%9A%84%E6%84%8F%E6%80%9D%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E5%90%84%E7%A7%8D%E5%86%99%E7%B1%BB%E5%BA%93%EF%BC%8C%E5%8A%A0%E6%89%A9%E5%B1%95%EF%BC%8C%E4%BD%86%E7%BB%93%E6%9E%9C%E8%A2%ABGO%E7%BB%99%E5%8F%96%E4%BB%A3%E4%BA%86%E3%80%82%E5%89%8D%E7%AB%AF%E7%9C%9F%E6%98%AF%E7%9E%8E%E6%8A%98%E8%85%BE%E3%80%82%0A%0A%0A%0Atype%20script%0A\-\-\-\-%0A%E5%8F%88%E6%98%AF%E4%B8%AA%E9%80%97%E6%AF%94%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8Cts%E4%BB%A3%E7%A0%81%E6%9C%80%E5%90%8E%E8%BF%98%E6%98%AF%E8%A2%AB%E7%BC%96%E8%AF%91%E6%88%90js%EF%BC%8C%E4%BD%86%E6%98%AF%E4%BB%96%E9%87%8C%E9%9D%A2%E5%81%9A%E4%BA%86%E4%B8%80%E4%BA%9B%E5%BC%BA%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%93%8D%E4%BD%9C%EF%BC%8C%E5%AE%9E%E9%99%85%E4%B8%8A%EF%BC%8C%E6%98%AF%E5%9C%A8ES6%E7%9A%84%E5%9F%BA%E7%A1%80%E4%B8%8A%EF%BC%8C%E5%8F%88%E5%8A%A0%E5%BC%BA%E4%BA%86JS%E5%90%91%E5%90%8E%E7%AB%AF%E8%BF%9B%E5%8C%96%E7%9A%84%E4%B8%80%E4%B8%AA%E8%BF%87%E7%A8%8B%E3%80%82%0A%0Anode.js%0A\-\-\-\-\-%0A%E5%8F%AF%E4%BB%A5%E5%86%99%E5%90%8E%E7%AB%AF%E7%9A%84js%EF%BC%8C%E9%82%A3%E4%BB%8E%E5%AE%8F%E8%A7%82%E7%9C%8B%EF%BC%8CJS%E7%9C%9F%E6%98%AF%E5%AE%8C%E5%85%A8%E5%8F%AF%E4%BB%A5%E5%86%99%E5%90%8E%E7%AB%AF%E4%BA%86%E3%80%82%0A%E5%9F%BA%E4%BA%8EV8%EF%BC%8C%E7%AE%80%E5%8D%95%E8%AF%B4%E5%B0%B1%E6%98%AF%EF%BC%9A%E7%94%A8%20C%2B%2B%20%E6%8A%8AV8%E5%BC%95%E6%93%8E%E5%8F%88%E5%B0%81%E8%A3%85%E4%BA%86%E4%B8%80%E5%B1%82%EF%BC%8C%E4%B8%93%E4%BE%9B%E6%89%A7%E8%A1%8CJS%E4%BB%A3%E7%A0%81%E7%9A%84%E3%80%82%0A%0A%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86%EF%BC%9Aevent%20loop%E6%A8%A1%E5%9E%8B%EF%BC%8C%E8%AF%A6%E7%BB%86%E7%9A%84%E6%88%91%E4%B9%9F%E4%B8%8D%E7%86%9F%EF%BC%8C%E7%A9%BA%E4%BA%86%E5%8F%AF%E4%BB%A5%E7%9C%8B%E7%9C%8B%EF%BC%8C%E4%BD%86%E6%84%9F%E8%A7%89%E5%BE%97%E5%85%88%E7%9C%8BV8%E5%BC%95%E6%93%8E%E3%80%82%0A%0A%E4%BD%86%E8%AE%BD%E5%88%BA%E7%9A%84%E6%98%AF%EF%BC%8C%E4%BD%9C%E8%80%85%E5%A5%BD%E5%83%8F%E6%94%BE%E5%BC%83%E6%9B%B4%E6%96%B0%E4%BA%86%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%A4%AA%E8%87%83%E8%82%BF%EF%BC%8C%E9%9A%8F%E4%BE%BF%E5%86%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%BE%97%E5%85%88%E4%B8%8B%E8%BD%BD2w%E4%B8%AA%E6%96%87%E4%BB%B6....%E6%96%B0%E7%9A%84%E5%A5%BD%E5%83%8F%E5%8F%AB%E4%BB%80%E4%B9%88%EF%BC%9Adeno......%0A%0ANPM%0A\-\-\-\-\-%0AJS%20%E7%B1%BB%E5%8C%85%E7%AE%A1%E7%90%86%EF%BC%8C%E8%B7%9Fmaven%20composer%20%E4%B8%80%E4%B8%AA%E9%B8%9F%E4%B8%9C%E8%A5%BF%EF%BC%8C%E4%B9%9F%E6%9C%89%E4%B8%AApackage.json%E6%96%87%E4%BB%B6%E7%94%A8%E6%9D%A5%E9%85%8D%E7%BD%AE%E6%BA%90%0A%3E%E5%A5%BD%E5%83%8F%E6%98%AF%E7%94%A8node.js%E7%BC%96%E5%86%99%E7%9A%84...%E5%AE%89%E8%A3%85node%E4%BC%9A%E8%BF%9E%E5%B8%A6%E7%9D%80%E4%B8%80%E5%B9%B6%E7%BB%99%E5%AE%89%E8%A3%85%E4%BA%86%0A%0A%E8%BF%98%E6%9C%89%E4%B8%AAcnpm%EF%BC%8C%E4%BC%B0%E8%AE%A1%E8%B7%9FNPM%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%B7%98%E5%AE%9D%E7%9A%84%E4%B8%80%E4%B8%AA%E4%BB%A3%E7%90%86NPM%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E6%88%91%E7%8C%9C%E5%B0%B1%E6%98%AF%E6%94%B9%E4%BA%86%E4%B8%AA%E6%BA%90%E5%9C%B0%E5%9D%80...%0A%3E%E6%9C%89%E8%B6%A3%E7%9A%84%E6%98%AF%20npm%20instal%20cnmp%20%E5%93%88%E5%93%88%0A%0A%E7%84%B6%E5%90%8E%EF%BC%8C%E5%B0%B1%E5%8F%91%E7%8E%B0%E8%B7%9F%E6%88%91%E7%8C%9C%E7%9A%84%E4%B8%80%E6%A0%B7%EF%BC%8Ccnmp%E6%9E%9C%E7%84%B6%E5%B0%B1%E6%98%AF%E6%94%B9%E4%BA%86%E4%B8%AA%E6%BA%90%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%8F%AF%E4%BB%A5%EF%BC%9A%0A%3Enpm%20instal%20%20xxxx%20\-\-registry%3Dhttp%3A%2F%2Fwwwww%0A%0A%0A%E8%BF%99%E4%B8%9C%E8%A5%BF%E5%B1%85%E7%84%B6%EF%BC%8C%E8%BF%98%E8%83%BD%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC...%20%E6%9C%89%E7%82%B9%E5%88%B7%E6%96%B0%E8%AE%A4%E7%9F%A5%EF%BC%8C%E5%B9%B2%E4%BA%86%E4%B8%80%E4%BA%9B%E7%A8%8B%E5%BA%8F%E5%91%98%E6%88%96%E8%80%85shell%20docker%E7%9A%84%E6%B4%BB%E3%80%82%0A%3Enpm%20run%20xxxx%0A%0A%E5%8E%9F%E7%90%86%EF%BC%9Apackage.json%20%E4%B8%AD%E6%9C%89%E4%B8%AAkey%E5%8F%AB%EF%BC%9A%20scripts%20%EF%BC%8C%E5%AE%83%E9%87%8C%E9%9D%A2%E5%8C%85%E5%90%AB%E7%9A%84%E6%AF%8F%E4%B8%80%E9%A1%B9%20%E5%B0%B1%E6%98%AF%E9%9C%80%E8%A6%81%E6%89%A7%E8%A1%8C%E7%9A%84%E8%84%9A%E6%9C%AC%0A%3Enpm%20run%20xxxx%20%E5%85%B6%E4%B8%AD%E7%9A%84xxxx%20%E4%BC%9A%E5%BD%93%E5%81%9A%E5%8F%82%E6%95%B0%EF%BC%8C%E4%BC%A0%E5%88%B0%20scripts%20%E4%B8%AD%0A%0A%0AJS%E6%9C%89%E4%BA%86node%20%E5%92%8C%20npm%EF%BC%8C%E5%B0%B1%E6%9C%89%E7%8E%A9%E5%A4%B4%E4%BA%86~%E5%8F%AF%E4%BB%A5%E4%B8%8A%E4%BC%A0%2F%E4%B8%8B%E8%BD%BD%E5%8C%85%EF%BC%8C%E8%BF%98%E8%83%BD%E9%80%9A%E8%BF%87%E6%8C%87%E4%BB%A4%E8%A1%8C%E6%9D%A5%E6%89%A7%E8%A1%8C%E5%8C%85~%0A%E5%83%8F%E5%AE%9E%E5%8A%9B%E5%BC%BA%E7%9A%84%E5%A4%A7%E5%8E%82\(vue\)%EF%BC%8C%E5%AE%8C%E5%85%A8%E5%8F%AF%E4%BB%A5%E7%94%A8NODE%E7%BC%96%E5%86%99%E8%84%9A%E6%89%8B%E6%9E%B6%E4%BB%A3%E7%A0%81%EF%BC%8C%E7%84%B6%E5%90%8E%E5%B0%81%E8%A3%85%E6%88%90%E5%8C%85%E4%B8%8A%E4%BC%A0%EF%BC%8C%E4%BD%BF%E7%94%A8%E8%80%85%E5%85%88%E7%94%A8NPM%E6%8A%8A%E5%8C%85%E4%B8%8B%E8%BD%BD%E4%B8%8B%E6%9D%A5%EF%BC%8C%E5%86%8D%E6%89%A7%E8%A1%8Cnode.js%EF%BC%8C%E6%8E%A5%E7%9D%80%E8%BF%99%E4%B8%AA%E5%8C%85%E5%B0%B1%E5%8F%AF%E4%BB%A5%E5%9C%A8%E4%BD%BF%E7%94%A8%E8%80%85%E7%9A%84%E6%9C%BA%E5%99%A8%E4%B8%8A%E5%8A%A8%E6%80%81%E7%94%9F%E6%88%90%E6%9E%84%E6%9E%B6%E6%96%87%E4%BB%B6%E4%BA%86~%0A%0A%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%90%8E%E7%AB%AF%E7%9A%84%E4%B8%80%E5%A5%97%E7%8E%A9%E6%B3%95%EF%BC%8CJS%E9%83%BD%E8%83%BD%E7%8E%A9%E5%84%BF%E4%BA%86%E3%80%82%0A%0A%3E%E4%B8%8D%E8%BF%87%EF%BC%8C%E5%B0%B1%E4%B8%80%E4%B8%AAVUE%E6%9C%80%E6%9C%80%E6%9C%80%E7%AE%80%E5%8D%95%E7%9A%84%E9%A1%B9%E7%9B%AE%EF%BC%8C%E8%A6%81%E4%B8%8B%E8%BD%BD2W%E5%A4%9A%E4%B8%AA%E6%96%87%E4%BB%B6%2C%E4%B9%9F%E9%9A%BE%E6%80%AANODE%E5%88%9B%E5%A7%8B%E8%A6%81%E6%94%BE%E5%BC%83NODE%0A%0Awebpack%0A\-\-\-\-%0A%3Enpm%20install%20webpack%20\-g%0A%3Enpm%20install%20webpack\-cli%20\-g%0A%0A%E6%89%93%E5%8C%85%E5%B7%A5%E5%85%B7%20%2C%20%E6%8A%8A%E4%BD%A0%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84JS%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%89%93%E5%8C%85%E6%88%90%E4%B8%80%E4%B8%AAJS%E6%96%87%E4%BB%B6%EF%BC%8C%E5%86%8D%E5%8E%8B%E7%BC%A9%EF%BC%8C%E7%84%B6%E5%90%8E%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84%E6%96%87%E4%BB%B6%E3%80%82%0A%3E%E4%B8%8D%E8%BF%87%EF%BC%8C%E4%BD%A0%E5%BE%97%E5%BE%97%E6%8C%89%E7%85%A7%E5%AE%83%E7%9A%84%E6%96%B9%E5%BC%8F%E6%9D%A5%E5%86%99%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%BE%97%E6%9C%89%E4%B8%AAmodules%EF%BC%8C%E5%BE%97%E6%9C%89main%E5%85%A5%E5%8F%A3%0A%0A%E6%9C%80%E5%90%8E%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%AAwebpack.config.js%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%8Bmodules%E7%9B%AE%E5%BD%95%E5%8F%8A%E8%BE%93%E5%87%BA%E4%BD%8D%E7%BD%AE%E3%80%82%0A%0A%3E%E5%8F%A6%E5%A4%96%EF%BC%8C%E4%BB%96%E8%83%BD%E6%8A%8A%E4%BD%A0%E7%9A%84ES6%E6%89%93%E5%8C%85%E6%88%90ES5%E6%A0%BC%E5%BC%8F%0A%0A%0A%E5%85%88%E4%B8%8D%E7%9C%8B%E5%90%84%E7%A7%8D%E6%A1%86%E6%9E%B6%EF%BC%8C%E5%8D%95%E4%BB%8E%E8%BF%99%E5%87%A0%E4%B8%AA%E5%9F%BA%E7%A1%80%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%9Aes6%20ts%20node%20webpack%20%EF%BC%8CJS%E5%B7%B2%E7%BB%8F%E7%94%B1%E6%88%91%E6%9C%80%E5%88%9D%E7%9A%84%E8%AE%A4%E7%9F%A5%EF%BC%9A%E5%8F%AA%E8%83%BD%E5%86%99%E5%86%99%E5%89%8D%E7%AB%AF%EF%BC%8C%E7%8E%B0%E5%9C%A8%E7%9C%9F%E7%9A%84%E8%83%BD%E6%A0%B9%E6%9C%AC%E4%B8%8A%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%8CJS%E4%B9%9F%E5%8F%AF%E4%BB%A5%E5%86%99%E5%90%8E%E7%AB%AF%E4%BA%86%E3%80%82%E5%B0%A4%E5%85%B6%E5%86%8D%E9%85%8D%E5%90%88%E5%AE%83%E7%9A%84%E5%8C%85%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%EF%BC%8C%E5%A4%A7%E4%BD%93%E4%B8%8A%E5%B0%B1%E8%B7%9F%E5%90%8E%E7%AB%AF%E7%9A%84%E6%A8%A1%E5%BC%8F90%25%E9%83%BD%E7%9B%B8%E4%BC%BC%E4%BA%86%E3%80%82%0A%0A%0A%E7%9C%8B%E7%9C%8Bwebpack%E7%9A%84%E5%A4%A7%E4%BD%93%E8%AF%AD%E6%B3%95%0A%0A1.%20entry%3A%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%85%A5%E5%8F%A3%E6%96%87%E4%BB%B6%2C%E6%9C%89%E7%82%B9main%E5%87%BD%E6%95%B0%E7%9A%84%E6%84%8F%E6%80%9D%E3%80%82%E8%BF%99%E9%87%8C%E5%8F%AF%E4%BB%A5%E6%9C%89%E5%A4%9A%E4%B8%AA%EF%BC%8C%E4%B8%80%E8%88%AC%E6%98%AF%E4%B8%80%E4%B8%AA%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%98%AF%E5%8D%95%E9%9D%A2%E5%BA%94%E7%94%A8%E3%80%82webpack%E5%BC%80%E5%A7%8B%E6%89%93%E5%8C%85%E6%97%B6%EF%BC%8C%E5%BE%97%E6%89%BE%E5%88%B0%E6%9F%90%E4%BA%9B%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%85%A5%E5%8F%A3%EF%BC%8C%E7%94%B1%E6%AD%A4%E5%85%A5%E5%8F%A3%E5%86%8D%E4%BE%9D%E6%AC%A1%E7%BC%96%E8%AF%91%E6%89%80%E4%BE%9D%E8%B5%96%E7%9A%84%E4%B8%9C%E8%A5%BF%E3%80%82%0A2.%20output%3A%E6%89%93%E5%8C%85%E5%A5%BD%E7%9A%84%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BE%93%E5%87%BA%E7%9A%84%E4%BD%8D%E7%BD%AE%0A%60%60%60%0A%20%20%20%20path%3A%20path.resolve\(\_\_dirname%2C%20'..%2Fdist'\)%2C%0A%20%20%20%20publicPath%3A%20'%2Fassets%2F'%2C%0A%20%20%20%20filename%3A%20'static%2Fjs%2F%5Bname%5D.js'%2C%0A%20%20%20%20chunkFilename%3A%20'%5Bid%5D.bundle.js'%2C%0A%60%60%60%0A3.%20%20module%3A%E8%87%AA%E5%AE%9A%E4%B9%89%E8%A6%81%E5%8A%A0%E8%BD%BD%E5%93%AA%E4%BA%9B%E6%96%87%E4%BB%B6%EF%BC%8C%E4%B8%8D%E5%8A%A0%E8%BD%BD%E5%93%AA%E4%BA%9B%E6%96%87%E4%BB%B6%EF%BC%8C%E4%BD%BF%E7%94%A8%E4%BB%80%E4%B9%88%E5%8A%A0%E8%BD%BD%E5%99%A8loader%0A4.%20devServer%3A%E5%BC%80%E5%90%AF%E4%B8%80%E4%B8%AAhttp%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B~dev%3D%E5%BC%80%E5%8F%91%EF%BC%8Cserver%3A%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E7%BB%99%E5%89%8D%E7%AB%AF%E5%BC%80%E5%8F%91%E8%80%85%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AA%E5%90%8E%E7%AB%AFHTTP%E7%9B%91%E5%90%AC%E8%BF%9B%E7%A8%8B%E3%80%82%0A%3E%E5%AE%83%E5%86%85%E9%83%A8%E6%98%AF%E5%8F%A6%E4%B8%80%E5%8C%85%EF%BC%8Cwebpack\-dev\-server%EF%BC%8C%E5%BC%80%E5%90%AF%E5%90%8E%EF%BC%8Coutput%E5%8F%82%E6%95%B0%E5%9F%BA%E6%9C%AC%E5%A4%B1%E6%95%88%EF%BC%8C%E4%BD%86%E6%94%AF%E6%8C%81%E7%83%AD%E6%89%93%E5%8C%85%EF%BC%8C%E5%8D%B3%EF%BC%9A%E4%BD%A0JS%E6%9C%89%E4%BF%AE%E6%94%B9%EF%BC%8C%E7%AB%8B%E5%88%BB%E5%8F%AF%E4%BB%A5%E6%9F%A5%E7%9C%8B%E5%88%B0%E7%BB%93%E6%9E%9C%E3%80%82%E5%85%B6%E5%AE%9E%E4%B9%9F%E6%8C%BA%E5%A5%BD%E7%90%86%E8%A7%A3%EF%BC%8C%E5%AE%83%E6%98%AF%E4%B8%AA%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%EF%BC%8C%E8%83%BD%E6%97%B6%E6%97%B6%E7%9B%91%E5%90%AC%E5%88%B0%E6%89%80%E6%9C%89%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8Coutput%E7%9A%84%E6%89%93%E5%8C%85%E6%96%87%E4%BB%B6%EF%BC%8C%E8%A2%AB%E5%8A%A0%E8%BD%BD%E5%88%B0%E5%86%85%E5%AD%98%E4%B8%AD%EF%BC%8C%E5%90%8C%E6%97%B6%E7%9B%91%E5%90%AC%E6%96%87%E4%BB%B6%E5%8F%98%E5%8C%96%E3%80%82%0A%0A%60%60%60%0Aport%3A%0Aproxy%3A%0A%60%60%60%0A%0A%E9%85%8D%E7%BD%AE%E5%A5%BDwebpack.config.js%E5%90%8E%EF%BC%8C%E5%BC%80%E5%A7%8B%E6%89%93%E5%8C%85%EF%BC%9A%0A%3Ewebpack%0A%0A%E4%BC%9A%E5%9C%A8output%E6%8C%87%E5%AE%9A%E7%9A%84%E4%BD%8D%E7%BD%AE%E7%94%9F%E6%88%90%E6%89%93%E5%8C%85%E5%A5%BD%E7%9A%84%E6%96%87%E4%BB%B6%E3%80%82%0A%0A%0A%3E%E4%B9%8B%E5%89%8D%E6%88%91%E8%B7%9F%E5%89%8D%E7%AB%AF%E8%81%94%E8%B0%83%EF%BC%8C%E4%BB%96%E4%BB%AC%E5%B1%85%E7%84%B6%E8%BF%98%E8%A6%81%E6%89%93%E5%8C%85%E7%BC%96%E8%AF%91...%20%E6%88%91%E5%B0%B1%E4%B8%8D%E5%A4%AA%E7%90%86%E8%A7%A3%EF%BC%8C%E4%B8%80%E4%B8%AA%E7%A0%B4%E5%89%8D%E7%AB%AF%E8%BF%98%E7%94%A8%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%EF%BC%9Fjs%E6%96%87%E4%BB%B6%E7%BC%96%E8%AF%91%E4%B8%AA%E6%AF%9B%E7%BA%BF%EF%BC%8CJS%E5%BE%97%E5%8A%A0%E8%BD%BD%E5%88%B0%E6%B5%8F%E8%A7%88%E5%99%A8%E6%89%8D%E8%83%BD%E8%BF%90%E8%A1%8C%EF%BC%8C%E8%BF%99%E4%B8%8D%E9%80%97%E6%88%91%E5%91%A2%EF%BC%9F%E7%8E%B0%E5%9C%A8%E7%9C%8B%E5%AE%8C%EF%BC%8C%E4%B9%9F%E8%83%BD%E7%90%86%E8%A7%A3%E4%BA%86%E3%80%82%0A%0A%0A%E6%95%B4%E4%BD%93%E7%9C%8B%E7%8E%B0%E5%9C%A8%E7%9A%84%E5%89%8D%E7%AB%AF%E7%9A%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E7%94%A8node.js%E6%97%A5%E5%B8%B8%E5%BC%80%E5%8F%91%EF%BC%8C%E9%85%8D%E5%90%88NPM%E5%8C%85%E7%AE%A1%E7%90%86%EF%BC%8C%E6%9C%80%E5%90%8E%E6%8A%8A%E8%BF%99%E4%BA%9BJS%5CCSS%5CIMG%EF%BC%8C%E7%BB%9F%E4%B8%80%E7%94%A8webpack%E6%89%93%E5%8C%85%E6%88%90%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E6%89%A7%E8%A1%8C%E7%9A%84%E6%96%87%E4%BB%B6%EF%BC%8C%E6%9C%80%E5%90%8E%E5%BC%80%E5%90%AFweb\-service%E7%9B%91%E5%90%AC%E3%80%82%0A%0A%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%90%8E%E7%AB%AF%E7%9A%84%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E9%83%BD%E5%8F%AF%E4%BB%A5%E4%B8%B2%E8%B5%B7%E6%9D%A5%E4%BA%86%E3%80%82%0A%0A%E6%84%9F%E8%A7%89%E8%BF%98%E6%98%AF%E6%9C%89%E7%82%B9%E7%BB%95%EF%BC%8Cnode.js%E8%83%BD%E5%86%99%E7%82%B9%E5%90%8E%E7%AB%AF%EF%BC%8C%E5%A6%82%EF%BC%9Awebservice%EF%BC%8C%E5%90%8C%E6%97%B6%E6%94%AF%E6%8C%81%E4%B8%80%E4%BA%9B%E5%90%8E%E7%AB%AF%E8%AF%AD%E8%A8%80%E7%9A%84%E8%AF%AD%E6%B3%95%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%BC%BA%E7%B1%BB%E5%9E%8B%E3%80%81import%2Fexport%E7%AD%89%EF%BC%8C%E6%9C%80%E5%90%8E%E6%89%93%E5%8C%85%E3%80%82%E4%BD%86%E9%97%AE%E9%A2%98%E6%98%AF%E6%89%93%E5%8C%85%E5%AE%8C%E4%BA%86%E8%BF%98%E6%98%AF.js%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%B0%B1%E6%9C%89%E7%82%B9%E8%AE%BD%E5%88%BA%E3%80%82%E4%B8%8D%E8%BF%87%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%98%AF%E5%9B%A0%E4%B8%BA%E5%AE%83%E5%BE%97%E5%86%85%E5%B5%8C%E5%88%B0%E6%B5%8F%E8%A7%88%E5%99%A8%EF%BC%8C%E8%B5%84%E6%BA%90%E4%B9%9F%E6%AF%94%E8%BE%83%E7%A2%8E%E7%89%87%E5%8C%96%EF%BC%8C%E5%8A%A0%E4%B8%8A%E8%BF%99%E4%B8%AA%E6%B5%81%E7%A8%8B%E7%A1%AE%E5%AE%9E%E6%98%AF%E6%96%B9%E4%BE%BF%E4%BA%9B%E3%80%82%0A%0A%0A%0A%0A%E8%BF%99%E9%87%8C%E6%9C%89%E7%AF%87%E6%96%87%E7%AB%A0%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%BB%86%E7%BB%86%E5%93%81%E5%93%81%3A%0A%3Ehttps%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F32148338%0A%0A%E6%84%9F%E8%A7%89%E5%89%8D%E7%AB%AF%E7%9C%9F%E7%9A%84%E6%98%AF%E7%9E%8EJB%E6%8A%98%E8%85%BE%EF%BC%8C%E7%9C%8B%E4%BC%BC%E6%98%AF%E6%83%B3%E5%90%8E%E7%AB%AF%E4%BD%93%E7%B3%BB%E6%8C%A3%E8%84%B1%E5%87%BA%E6%9D%A5%EF%BC%8C%E6%83%B3%E7%85%A7%E6%90%AC%E5%90%8E%E7%AB%AF%E6%A8%A1%E5%BC%8F%EF%BC%8C%E6%83%B3%E8%A6%81%E6%A8%A1%E5%9D%97%E5%8C%96%E3%80%81OOP%EF%BC%8C%E5%8F%88%E6%83%B3%E7%AE%80%E5%8D%95%E5%8C%96%EF%BC%8C%E6%9C%80%E5%90%8E%E6%90%9E%E7%9A%84%E7%9C%9F%E6%98%AF%E5%A4%8D%E6%9D%82%E5%88%B0%E4%BD%A0%E6%83%B3%E6%94%BE%E5%BC%83%E3%80%82%E6%88%91%E4%BE%9D%E7%84%B6%E8%A7%89%E5%BE%97%EF%BC%9AJS%2BJQUERY%E6%9B%B4%E5%A5%BD%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%A4%8D%E6%9D%82%E8%BF%87%E5%BA%A6%E5%90%8E%EF%BC%8C%E5%B0%B1%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98%E4%BD%A0%E6%97%A0%E6%B3%95%E8%A7%A3%E5%86%B3%EF%BC%9A%E5%A6%82%E4%BD%95%E6%8E%92%E6%9F%A5%E9%94%99%E8%AF%AF%EF%BC%9F%E6%89%80%E6%9C%89%E7%9A%84%E4%BB%A3%E7%A0%81%E9%83%BD%E6%98%AF%E7%BB%8F%E8%BF%87%E6%A1%86%E6%9E%B6%E6%89%93%E5%8C%85%E7%BB%84%E5%90%88%E5%90%8E%E7%94%9F%E6%88%90%EF%BC%8C%E5%B0%A4%E5%85%B6JS%E8%A2%AB%E6%89%93%E5%8C%85%E6%88%90%E4%B8%80%E4%B8%AA%E5%A4%A7%E5%A4%A7%E7%9A%84JS%E6%96%87%E4%BB%B6%E3%80%82%0A%0A%E5%89%8D%E7%AB%AF%E4%B8%89%E5%A4%A7%E6%A1%86%E6%9E%B6%0A\-\-\-\-\-%0A%0AAngular%3A2012%E5%B9%B4%EF%BC%8Cgoogle%E7%9A%84%E5%87%A0%E4%B8%AAJAVA%E5%90%8E%E7%AB%AF%E6%90%9E%E5%BE%97%E4%B8%80%E4%B8%AA%E6%A1%86%E6%9E%B6%EF%BC%8C%E5%9F%BA%E4%BA%8EMVC%EF%BC%8C%E8%82%AF%E5%AE%9A%E6%98%AF%E5%90%8E%E7%AB%AF%E4%BD%BF%E7%94%A8%E9%9D%9E%E5%B8%B8%E9%A1%BA%EF%BC%8C%E5%AF%B9%E5%89%8D%E7%AB%AF%E4%B8%8D%E5%A4%AA%E5%8F%8B%E5%A5%BD%EF%BC%8C%E5%8F%A6%E5%A4%96%E5%AE%83%E4%B9%9F%E6%9C%89%E7%82%B9%E9%87%8D%EF%BC%8C%E7%89%88%E6%9C%AC%E5%85%BC%E5%AE%B9%E4%B9%9F%E4%B8%8D%E5%A4%AA%E5%A5%BD%E3%80%82%E4%BD%86%E5%AE%83%E7%9A%84%E5%8A%9F%E5%8A%B3%E6%98%AF%20%E6%AF%94%E8%BE%83%E6%97%A9%E7%9A%84%E6%95%B4%E5%90%88%E4%BA%86MVC%E6%A8%A1%E5%BC%8F\(%E4%B8%AA%E4%BA%BA%E6%84%9F%E8%A7%89backbone%E6%9B%B4%E6%97%A9\)%E3%80%82%0Areact%3Afacebook%E5%87%BA%E5%93%81%EF%BC%8C2013%EF%BC%8C%E9%A6%96%E6%AC%A1%E6%8F%90%E5%87%BA%E8%99%9A%E6%8B%9FDOM%E3%80%82%0Avue%3A2014%E5%B9%B4%EF%BC%8C%E5%9B%BD%E4%BA%BA%E8%AE%BE%E8%AE%A1%EF%BC%8C%E6%9C%89mvvm%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%8C%E4%B9%9F%E6%9C%89%E8%99%9A%E6%8B%9Fdom%E6%A8%A1%E5%BC%8F%EF%BC%8C%E5%90%8C%E6%97%B6%E5%8F%88%E9%9D%9E%E5%B8%B8%E5%B0%8F%EF%BC%8C%E5%8F%88%E6%9C%89%E7%BB%84%E4%BB%B6%E5%8C%96%E4%B8%94%E4%B8%93%E6%B3%A8%E4%BA%8Eview%E5%B1%82%E3%80%82%0A%0A%E6%80%BB%E7%9A%84%E6%9D%A5%E8%AF%B4%EF%BC%9A%E5%B9%B4%E4%BB%A3%E8%B6%8A%E6%96%B0%EF%BC%8C%E8%9E%8D%E5%90%88%E7%9A%84%E6%80%9D%E6%83%B3%E4%B9%9F%E5%B0%B1%E8%B6%8A%E5%A5%BD%EF%BC%8C%E7%94%A8%E7%9A%84%E4%BA%BA%E4%B9%9F%E6%9C%80%E5%A4%9A%E3%80%82%E4%BD%86%E6%9B%B4%E6%96%B0%E7%9C%9F%E6%98%AF%E5%A4%AA%E5%BF%AB%E5%A4%AA%E4%B9%B1%E4%BA%86%EF%BC%8C%E6%97%A9%E6%9C%9F%E7%9A%84JUQERY%2BJQUERY\-UI%EF%BC%8C%E5%86%8D%E5%88%B0%EF%BC%8C%E5%89%8D%E5%87%A0%E5%B9%B4%E6%84%9F%E8%A7%89react%E4%B8%80%E7%BB%9F%E5%A4%A9%E4%B8%8B%E7%9A%84%E6%84%8F%E6%80%9D%EF%BC%8C%E8%BF%992%E5%B9%B4%E9%A3%8E%E5%90%91%E5%B0%B1%E5%8F%98%E4%BA%86%EF%BC%8C%E5%AE%8C%E5%85%A8%E8%BD%AC%E8%BA%ABVUE%E4%BA%86%E3%80%82%0A%0Amvvm%0A\-\-\-\-\-%0Amodel%20view%20view\-model%EF%BC%9A%E8%B7%9Fmvc%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E5%B0%B1%E6%98%AF%E4%B8%AA%E6%A1%86%E6%9E%B6%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E3%80%82%E5%AE%83%E7%9A%84%E8%AF%9E%E7%94%9F%EF%BC%8C%E7%AE%97%E6%98%AF%E7%BB%88%E4%BA%8E%E7%BB%99%E5%89%8D%E7%AB%AF%E7%BB%9F%E4%B8%80%E4%BA%86%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%8C%E5%8F%AF%E6%8C%81%E7%BB%AD%E5%8F%91%E5%B1%95%E7%9A%84%E6%A8%A1%E5%BC%8F%E3%80%82%0A%3E%E6%84%9F%E8%A7%89%E5%89%8D%E7%AB%AF%E5%8F%91%E5%B1%95%E5%B0%B1%E6%98%AF%E5%BD%93%E6%97%B6%E5%90%8E%E7%AB%AF%E7%9A%84%E5%8F%91%E5%B1%95%E8%BF%87%E7%A8%8B%EF%BC%8C%E4%BD%86%E6%84%9F%E8%A7%89%E7%9C%9F%E7%9A%84%E6%98%AF%E4%B8%80%E9%98%B5%E4%B9%B1%E5%B0%84%EF%BC%8C%E6%9C%80%E7%BB%88%E6%9C%89%E5%87%A0%E6%94%AF%E7%AE%AD%E5%B0%84%E4%B8%AD%E4%BA%86%EF%BC%8C%E5%A4%A7%E5%AE%B6%E5%86%8D%E5%9B%9E%E5%88%B0%E8%BF%99%E4%B8%AA%E6%AD%A3%E7%A1%AE%E7%9A%84%E6%96%B9%E5%90%91%EF%BC%8C%E4%B9%8B%E5%90%8E%E7%BB%A7%E7%BB%AD%E4%B9%B1%E5%B0%84....%E8%BF%98%E6%98%AF%E5%9F%BA%E7%A1%80%E4%B8%8D%E8%A1%8C%E5%95%8A~%0A%0A%E8%99%9A%E6%8B%9Fdom%0A\-\-\-\-%0A%E5%89%8D%E7%BD%AE%E8%AF%B4%E6%98%8E%3A%E5%89%8D%E7%AB%AF%E7%9A%84%E6%97%A5%E5%B8%B8%E5%BC%80%E5%8F%91%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%B0%B1%E6%98%AF%E4%BF%AE%E6%94%B9dom%E6%A0%91%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E6%A0%B9%E6%8D%AE%E4%BF%AE%E6%94%B9%E5%86%85%E5%AE%B9%E5%86%8D%E9%87%8D%E6%96%B0%E5%81%9A%E6%B8%B2%E6%9F%93%E3%80%82%E5%A5%BD%E5%83%8F%E7%8E%B0%E5%9C%A8%E6%9C%89%E5%B7%A5%E5%85%B7%E8%83%BD%E7%9B%B4%E6%8E%A5%E6%8A%8APS%E7%94%9F%E6%88%90HTML%2BCSS%2FDIV.%0A%0Avdom%3A%E7%94%A8js%E8%99%9A%E6%8B%9F%E5%87%BA%E4%B8%80%E5%B1%82dom%0A%E5%8E%9F%E7%90%86%EF%BC%9A%E5%85%88%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAJS%E5%AF%B9%E8%B1%A1%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E6%8A%8A%E6%9F%90%E4%B8%AA\(%E4%B9%9F%E5%8F%AF%E8%83%BD%E5%8C%85%E5%90%AB%E5%A4%9A%E4%B8%AA%E5%AD%90DOM%E8%8A%82%E7%82%B9\)%E7%9C%9F%E5%AE%9E%E7%9A%84dom%E8%8A%82%E7%82%B9%E7%BB%91%E5%AE%9A%E5%88%B0%E8%BF%99%E4%B8%AAjs%E5%AF%B9%E8%B1%A1%E4%B8%8A%EF%BC%8C%E5%90%8C%E6%97%B6%E5%86%8D%E7%BB%99%E8%BF%99%E4%B8%AAJS%E5%AF%B9%E8%B1%A1%E7%BB%91%E5%AE%9A%E4%B8%8A%E8%87%AA%E5%B7%B1%E7%9A%84%E6%97%A5%E5%B8%B8%E6%93%8D%E4%BD%9C%E5%87%BD%E6%95%B0%EF%BC%8C%E8%BF%99%E6%A0%B7%EF%BC%8C%E6%89%80%E6%9C%89%E7%9A%84%E6%93%8D%E4%BD%9C%E9%83%BD%E6%98%AF%E6%93%8D%E4%BD%9C%E8%BF%99%E4%B8%AAJS%E5%AF%B9%E8%B1%A1%EF%BC%8C%E8%80%8C%E4%B8%8D%E7%9B%B4%E6%8E%A5%E6%93%8D%E4%BD%9Cdom%EF%BC%8C%E6%9C%80%E5%90%8E%E8%99%9A%E6%8B%9Fdom%E5%8F%AF%E8%83%BD%E4%BC%9A%E4%BF%AE%E6%94%B9dom%E4%B9%9F%E5%8F%AF%E8%83%BD%E4%B8%8D%E6%94%B9%E3%80%82%0A%0A%3E%E7%9B%B8%E6%AF%94%E4%B9%8B%E4%B8%8B%EF%BC%8Cjquery%E5%B0%B1%E6%98%AF%E7%9B%B4%E6%8E%A5%E6%93%8D%E4%BD%9CDOM%0A%0A%E7%90%86%E8%AE%BA%E4%B8%8A%EF%BC%9A%E6%AF%94%E4%B8%80%E7%9B%B4%E4%BF%AE%E6%94%B9dom%E6%A0%91%E9%80%A0%E6%88%90%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B9%B3%E5%87%A1%E9%87%8D%E6%96%B0%E6%B8%B2%E6%9F%93%EF%BC%8C%E6%95%88%E7%8E%87%E6%9B%B4%E9%AB%98%E4%BA%9B%E3%80%82%0A%0A%E6%89%80%E4%BB%A5%EF%BC%8C%E6%88%91%E4%BB%AC%E7%9C%8B%E4%B8%8A%E9%9D%A2%E7%9A%84%E5%8E%9F%E7%90%86%EF%BC%8C%E6%A0%B8%E5%BF%83%E7%82%B9%E5%B0%B1%E6%98%AF%E8%BF%99%E4%B8%AAJS%E5%AF%B9%E8%B1%A1%EF%BC%8C%E5%85%B6%E5%AE%9E%E8%BF%99%E4%B8%AA%E5%AF%B9%E8%B1%A1%E5%B0%B1%E6%98%AFVUE...react%0A%0A%0A%0A%E4%BC%98%E7%82%B9%0A\-\-\-%0A1.%20%E6%80%A7%E8%83%BD%E5%8F%AF%E8%83%BD%E4%BC%9A%E5%8A%A0%E5%BC%BA%EF%BC%8C%E9%81%BF%E5%85%8D%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%80%E7%9B%B4%E9%87%8D%E5%A4%8D%E6%B8%B2%E6%9F%93%0A2.%20%E7%A8%8B%E5%BA%8F%E5%91%98%E5%8F%AF%E8%83%BD%E6%9B%B4%E4%B8%93%E6%B3%A8%E4%BA%8E%E5%9C%A8JS%E4%B8%8A%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AFHTML%E6%A0%87%E7%AD%BE%E4%B8%8A%0A3.%20%E6%9C%89%E7%BB%84%E4%BB%B6%E6%80%9D%E6%83%B3%EF%BC%8C%E4%BB%A3%E7%A0%81%E5%88%A9%E7%94%A8%E7%8E%87%E6%9B%B4%E9%AB%98%0A%0A%E7%BC%BA%E7%82%B9%0A\-\-\-\-%0A1.%20%E5%AD%A6%E4%B9%A0%E6%88%90%E6%9C%AC%E3%80%81%E4%BD%BF%E7%94%A8%E6%88%90%E6%9C%AC%0A2.%20%E5%A4%8D%E6%9D%82%E5%8C%96%EF%BC%8C%E9%80%A0%E6%88%90%E6%9B%B4%E5%A4%A7%E7%9A%84%E7%BB%B4%E6%8A%A4%E6%88%90%E6%9C%AC%0A%0A%E8%99%9A%E6%8B%9Fdom%E7%9A%84%E8%AF%9E%E7%94%9F%E4%B9%9F%E5%B0%B1%E6%98%AFmvvm%E4%B8%AD%E7%9A%84vm%EF%BC%8C%E5%85%B6%E7%BB%93%E6%9E%9C%E5%B0%B1%E6%98%AFvue%20react%E5%BC%95%E9%A2%86%E7%9A%84%E8%BF%99%E5%9C%BA%E5%8F%98%E9%9D%A9%EF%BC%8C%E6%89%80%E6%9C%89%E7%9A%84%E5%89%8D%E7%AB%AF%E4%BB%A3%E7%A0%81%E5%B0%B1%E9%83%BD%E5%BE%97%E5%BC%80%E5%A7%8B%E7%8E%A9vdom%E4%BA%86%0A%0A%E5%9F%BA%E6%9C%AC%E4%B8%8A%E7%90%86%E8%A7%A3%E4%BA%86vm%EF%BC%8C%E8%83%BD%E5%A4%9F%E5%A5%97%E7%94%A8%E5%88%B0mvvm%E6%A8%A1%E5%BC%8F%E4%B8%AD%EF%BC%8C%E5%A4%A7%E4%BD%93%E4%B8%8A%E4%B9%9F%E5%B0%B1%E7%90%86%E8%A7%A3%E4%BA%86%E7%8E%B0%E5%9C%A8%E7%9A%84%E5%89%8D%E7%AB%AF%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E7%9A%84%E6%A0%B8%E5%BF%83%E4%BA%86%EF%BC%8C%E5%89%A9%E4%B8%8B%E7%9A%84%E5%B0%B1%E6%98%AF%E5%9B%B4%E7%BB%95vm%E7%9A%84%E5%90%84%E7%A7%8D%E6%93%8D%E4%BD%9C%E4%BA%86%E3%80%82%0A%0A%E5%8D%95%E9%A1%B5%E5%BA%94%E7%94%A8%0A\-\-\-\-%0A%0A%0A%0AVUE%0A\-\-\-\-%0A%E5%9B%A0%E4%B8%BA%E5%AE%83%E7%8E%B0%E5%9C%A8%E6%9C%80%E7%81%AB%EF%BC%8C%E5%B0%B1%E6%8B%BF%E5%87%BA%E5%AE%83%E6%9D%A5%E5%AD%A6%E4%B9%A0%E4%B8%8B%E3%80%82%0A%E5%AE%83%E5%85%B6%E5%AE%9E%E5%B0%B1%E6%98%AF%E7%94%A8JS%E5%86%99%E7%9A%84%E4%B8%80%E4%B8%AA%E5%BA%93%EF%BC%8C%E7%84%B6%E5%90%8E%E5%AE%9E%E4%BE%8B%E5%8C%96%E6%88%90%E4%B8%80%E4%B8%AAJS\-OBJ%EF%BC%8C%E5%B0%B1%E6%98%AFV\-DOM%E7%B1%BB%E5%9E%8B.%0A%3E%E5%90%8C%E7%B1%BB%E5%9E%8B%E5%AF%B9%E6%AF%94LINUX%E7%B3%BB%E7%BB%9F%EF%BC%9A%E7%94%A8%E6%88%B7%E6%80%81%E4%B8%8E%E5%86%85%E6%A0%B8%E6%80%81%EF%BC%8C%E5%86%85%E6%A0%B8%E6%80%81%E5%B0%B1%E6%98%AFVUE%0A%0A%E5%8E%9F%E6%9C%AC%E5%8F%AF%E8%83%BD%E6%9D%BE%E6%95%A3%E7%9A%84JS%E4%BB%A3%E7%A0%81%EF%BC%8C%E8%A2%AB%E7%BB%9F%E4%B8%80%E6%95%B4%E7%90%86%E5%88%B0VUE%E4%B8%AD%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%89%8D%E7%AB%AF%E7%A8%8B%E5%BA%8F%E5%91%98%E6%8A%8A%E5%90%84%E7%A7%8D%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%E6%B3%A8%E5%86%8C%E5%88%B0%E8%BF%99%E4%B8%AAJS\-OBJ%E4%B8%AD%EF%BC%8C%E7%94%A8%E6%88%B7%E7%9A%84%E6%93%8D%E4%BD%9C%E7%AC%AC%E4%B8%80%E6%97%B6%E9%97%B4%E4%B9%9F%E6%98%AF%E6%93%8D%E4%BD%9C%E8%BF%99%E4%B8%AAJS\-OBJ%EF%BC%8C%E7%AD%89%E4%BA%8E%E4%B8%AD%E9%97%B4%E5%8A%A0%E4%BA%86%E6%95%B4%E6%95%B4%E4%B8%80%E5%B1%82%E8%80%8C%E4%BB%A5%E3%80%82%0A%0A%E8%BF%99%E7%A7%8DJS\-OBJ%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%B8%80%E4%B8%AA%E9%A1%B5%E9%9D%A2%E5%B0%B1%E6%9C%89N%E4%B8%AA%EF%BC%8C%E6%89%80%E4%BB%A5%EF%BC%8C%E5%AE%83%E4%B9%9F%E6%9C%89%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%9A%84%E6%A6%82%E5%BF%B5%EF%BC%9A%0AbeforeCreate%20created%20beforeMount%20mounted%20beforeUpdate%20udpated%20destory%0A%0A%3E%E6%84%9F%E8%A7%89%E5%8F%88%E6%98%AF%E5%AE%89%E5%8D%93%20IOS%20%E9%82%A3%E4%B8%80%E5%A5%97.....%E9%97%AE%E9%A2%98%E6%98%AF%EF%BC%9A%E5%AE%89%E5%8D%93%20IOS%20%E9%82%A3%E4%B8%80%E5%A5%97%E6%98%AF%E7%9C%9F%E7%9A%84%E7%BB%99%E4%B8%80%E4%B8%AA%E5%AE%8C%E6%95%B4%E7%9A%84APP%E5%9C%A8%E4%B8%8A%E9%9D%A2%E8%BF%90%E8%A1%8C%E7%9A%84%EF%BC%8C%E4%BD%A0%E7%9A%84APP%E4%BB%A3%E5%AE%89%E8%A3%85%E6%97%B6%EF%BC%8C%E5%BE%97%E7%BB%8F%E8%BF%87%E7%BC%96%E8%AF%91%E6%88%90%E5%8F%AF%E6%89%A7%E8%A1%8C%E6%96%87%E4%BB%B6%EF%BC%8C%E6%94%BE%E5%9C%A8%E7%A1%AC%E7%9B%98%E4%B8%8A%EF%BC%8C%E7%84%B6%E5%90%8EOS%E7%9B%B4%E6%8E%A5%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E4%BA%86...%20%E8%80%8CJS%E8%BF%99%E5%9E%83%E5%9C%BE%EF%BC%8C%E5%B9%B6%E4%B8%8D%E6%98%AF%E7%9B%B4%E6%8E%A5%E8%BF%90%E8%A1%8C%E5%9C%A8OS%EF%BC%8C%E6%98%AF%E8%BF%90%E8%A1%8C%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E9%87%8C%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E8%BF%98%E4%B8%8D%E8%83%BD%E7%9B%B4%E6%8E%A5%E6%89%A7%E8%A1%8C%EF%BC%8C%E8%BF%98%E5%BE%97%E5%86%8D%E7%BB%99V8%E5%BC%95%E6%93%8E%EF%BC%8C%E6%9C%80%E5%90%8EVUE%E5%8F%88%E5%BC%BA%E8%A1%8C%E6%A8%A1%E6%8B%9F%E5%87%BA%E6%9D%A5%E8%BF%99%E4%B9%88%E4%B8%80%E5%B1%82....%0A%0A%E8%BF%99%E4%B8%AA%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%EF%BC%8C%E5%85%B6%E5%AE%9E%E4%B9%9F%E8%83%BD%E6%9B%B4%E5%A5%BD%E7%9A%84%E8%A7%A3%E9%87%8A%E4%BB%80%E4%B9%88%E6%98%AF%EF%BC%9AV\-DOM%0A%0A%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E8%BF%99%E4%B8%80%E5%A0%86%E5%87%BD%E6%95%B0%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%8F%AF%E4%BB%A5%E7%BB%9F%E4%B8%80%E5%AE%9A%E4%B9%89%E4%B8%BA%EF%BC%9A%E9%92%A9%E5%AD%90%EF%BC%8C%E5%A6%82%EF%BC%9A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAAPP%EF%BC%8C%E5%9C%A8%E6%9C%80%E7%BB%88%E6%8C%82%E8%BD%BD%E6%88%90%E5%8A%9F%E4%B9%8B%E5%89%8D%EF%BC%8C%E8%A6%81%E7%94%A8AJAX%E8%8E%B7%E5%8F%96%E5%88%B0%E5%90%8E%E7%AB%AF%E6%95%B0%E6%8D%AE%E6%89%8D%E8%A1%8C%EF%BC%8C%E9%82%A3%E7%94%A8mound%E5%87%BD%E6%95%B0%E3%80%82%0A%0A\*\*vue%E4%B8%83%E5%A4%A7vm%E5%B1%9E%E6%80%A7\*\*%0A%0Adata%3Aview%20%E5%B1%82%E9%9C%80%E8%A6%81%E7%9A%84%E6%95%B0%E6%8D%AE%E5%9D%87%E4%BB%8E%E8%BF%99%E9%87%8C%E6%8B%BF%0Amethods%3A%E5%89%8D%E7%AB%AF%E7%A8%8B%E5%BA%8F%E7%94%A8%E6%9D%A5%E6%8E%A7%E5%88%B6DOM%E5%8F%98%E5%8C%96%E7%9A%84%E4%B8%80%E4%BA%9B%E8%87%AA%E5%AE%9A%E4%B9%89%0Ael%3A%E7%BB%91%E5%AE%9A%E5%88%B0%E7%9C%9F%E5%AE%9E%E7%9A%84%E6%9F%90%E4%B8%AADOM%E8%8A%82%E7%82%B9ID%0Atemplate%3A%E6%A8%A1%E6%9D%BF%0Acompute%3A%E8%B7%9Fmethords%E6%9C%89%E7%82%B9%E5%83%8F%EF%BC%8C%E4%B9%9F%E6%98%AF%E4%B8%80%E4%BA%9B%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0%EF%BC%8C%E4%BD%86%E5%AE%83%E6%98%AF%E5%B1%9E%E6%80%A7%EF%BC%8C%E5%B8%B8%E9%A9%BB%E5%86%85%E5%AD%98\(%E5%89%8D%E6%8F%90%E4%B8%8D%E5%8F%91%E7%94%9F%E6%94%B9%E5%8F%98\)%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E7%BC%93%E5%AD%98%E7%9A%84%E6%84%8F%E6%80%9D%E3%80%82%0Awatch%3A%0Arender%3A%0A%0A%E5%8F%A6%E5%A4%96%E8%BF%98%E6%9C%89%E4%B8%80%E4%BA%9B%E5%85%B6%E5%AE%83%E7%9A%84%E5%B1%9E%E6%80%A7%EF%BC%9A%0Aslot%3A%E6%8F%92%E6%A7%BD%2C%E7%B1%BB%E4%BC%BC%E9%A1%B5%E9%9D%A2%E5%B8%83%E5%B1%80%EF%BC%8C%E5%AE%9A%E5%A5%BD%E5%B8%83%E5%B1%80%E5%90%8E%EF%BC%8C%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E6%A7%BD%E4%B8%80%E4%B8%AA%E6%A7%BD%EF%BC%8C%E7%84%B6%E5%90%8E%E4%BD%A0%E5%86%8D%E5%8A%A8%E6%80%81%E7%9A%84%E6%9B%BF%E6%8D%A2.%0Aprops%3A%E6%8E%A5%E6%94%B6%E5%A4%96%E5%B1%82bind%E7%9A%84%E6%95%B0%E6%8D%AE%0A%0A%0A%E6%8E%A5%E4%B8%8A%E9%9D%A2%E7%9A%84%E6%80%BB%E7%BB%93%EF%BC%8C%E7%90%86%E8%A7%A3%E4%BA%86VM%E7%9A%84%E5%8E%9F%E7%90%86%EF%BC%8C%E6%8E%A5%E7%9D%80%E5%B0%B1%E6%98%AF%E5%9C%A8VM%E4%B8%8A%E5%81%9A%E5%90%84%E7%A7%8D%E6%93%8D%E4%BD%9C%EF%BC%8C%E6%97%A5%E5%B8%B8%E7%9A%84%E5%BC%80%E5%8F%91%E5%9F%BA%E6%9C%AC%E5%B0%B1%E6%98%AF%E5%9C%A8%E8%BF%997%E4%B8%AA%E5%B1%9E%E6%80%A7%E4%B8%AD%E4%BA%86.%0A%3E%E5%85%B6%E5%AE%9E%E9%AB%98%E7%BA%A7VUE%E6%88%91%E7%8C%9C%EF%BC%8C%E5%B0%B1%E6%98%AFVUE%E5%86%85%E9%83%A8%E7%9A%84%E8%BF%99%E7%A7%8DV\-DOM%E9%97%B4%E7%9A%84%E6%93%8D%E4%BD%9C%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%98%AF%E5%90%8C%E7%BA%A7%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%98%AF%E7%88%B6%E5%AD%90%E7%BB%84%EF%BC%8C%E4%B9%9F%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%98%AF%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%0A%0A%0Avue%E7%A1%AE%E5%AE%9E%E5%A4%9F%E5%B0%8F%E5%B7%A7%EF%BC%8C%E5%B9%B6%E4%B8%94%E4%BF%9D%E6%8C%81%E4%B8%93%E6%B3%A8%EF%BC%8C%E5%85%B6%E4%BB%96%E7%9A%84%E4%B8%80%E6%A6%82%E4%B8%8D%E7%AE%A1%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%83%B3%E6%9C%89%E6%9B%B4%E5%A4%9A%E5%8A%9F%E8%83%BD%EF%BC%8C%E9%82%A3%E5%B0%B1%E5%BC%95%E5%85%A5%E5%85%B6%E4%BB%96%E5%8C%85%EF%BC%81%EF%BC%81%EF%BC%81%0A%3E%E6%84%9F%E8%A7%89vue%E8%B7%9F%E6%88%91%E5%BD%93%E5%B9%B4%E7%9A%84jquery%E6%8C%BA%E5%83%8F%EF%BC%8C%E4%BB%BB%E4%BD%95%E7%AE%80%E5%8D%95%E7%9A%84%E4%B8%9C%E8%A5%BF%E9%83%BD%E5%8F%AF%E8%83%BD%E4%BC%9A%E7%81%AB~%0A%0Avuex%0A\-\-\-%0A%E7%B1%BB%E5%BA%93%EF%BC%8C%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%0A%0A%E5%AE%83%E6%9B%B4%E5%83%8F%E6%98%AF%E4%B8%80%E4%B8%AA%E4%BB%93%E5%BA%93%EF%BC%8C%E5%85%A8%E5%B1%80%E4%BB%93%E5%BA%93%EF%BC%8C%E6%88%96%E8%80%85%E5%8F%AB%E5%85%A8%E5%B1%80%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93~%0A%0A%E6%AF%8F%E4%B8%AA%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%EF%BC%9A%E4%B8%80%E4%BD%86%E6%9F%90%E4%B8%80%E4%B8%AAmodel%E5%B1%82%E7%9A%84%E6%95%B0%E6%8D%AE%E5%8F%91%E7%94%9F%E5%8F%98%E5%8C%96%EF%BC%8C%E8%BF%9E%E5%B8%A6%E7%9D%80%E5%8F%AF%E8%83%BD%E6%89%80%E6%9C%89%E7%BB%84%E4%BB%B6%E9%83%BD%E5%BE%97%E8%B7%9F%E7%9D%80%E5%8F%98%EF%BC%8C%E9%82%A3%E4%B9%88%E7%BB%84%E4%BB%B6%E9%97%B4%E7%9A%84%E6%95%B0%E6%8D%AE%E9%80%9A%E4%BF%A1%E5%B0%B1%E6%98%AF%E4%B8%AA%E9%97%AE%E9%A2%98%E3%80%82%E8%BF%99%E4%B8%AA%E5%BA%93%E5%B0%B1%E6%98%AF%E8%A7%A3%E5%86%B3%E8%BF%99%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%8C%E5%B0%86%E5%90%84%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%9A%84%E5%8F%98%E5%8C%96%E5%AE%9A%E4%B9%89%E6%88%90%E7%8A%B6%E6%80%81%EF%BC%8C%E7%84%B6%E5%90%8E%E5%AD%98%E5%88%B0%E4%BB%93%E5%BA%93%E4%B8%AD%EF%BC%8C%E5%B9%B6%E8%AE%BE%E7%BD%AE%E5%BD%93%E5%8F%91%E7%94%9F%E6%94%B9%E5%8F%98%E5%90%8E%E7%9A%84%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%E3%80%82%0A%0A1.%20state%3A%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%E4%BB%93%E5%BA%93%E9%87%8C%E7%9A%84%E5%85%B7%E4%BD%93%E6%95%B0%E6%8D%AE%0A2.%20mutations%3A%E6%9C%89%E4%B8%80%E4%BA%9B%E6%96%B9%E6%B3%95%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%B8%8A%E9%9D%A2state%E9%87%8C%E7%9A%84%E6%95%B0%E6%8D%AE%0A3.%20action%3A%E5%81%9A%E5%BC%82%E5%B8%B8%E7%9A%84%E5%A4%84%E7%90%86%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E9%92%A9%E5%AD%90%E7%9A%84%E6%84%8F%E6%80%9D%0A4.%20getter%3A%E8%AF%BB%E5%8F%96state%E9%87%8C%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%98%AF%E4%B8%AA%E5%87%BD%E6%95%B0%EF%BC%8C%E4%B9%9F%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E9%92%A9%E5%AD%90%E7%9A%84%E6%84%8F%E6%80%9D%0A5.%20module%3A%E5%A6%82%E6%9E%9C%E8%BF%99%E7%A7%8D%E5%85%A8%E5%B1%80%E7%8A%B6%E6%80%81%E6%95%B0%E6%8D%AE%E8%BF%87%E5%A4%9A%EF%BC%8C%E5%B0%B1%E6%8A%8A%E4%B8%80%E4%BA%9B%E7%89%B9%E5%AE%9A%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A%E5%88%B0%E5%9B%BA%E5%AE%9A%E7%9A%84%E6%A8%A1%E5%9D%97%E4%B8%8A%0A%0A%E5%A4%96%E9%83%A8%E7%B1%BB%EF%BC%8C%E6%83%B3%E8%A6%81%E4%BF%AE%E6%94%B9state%E9%87%8C%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%BF%85%E9%A1%BB%E5%BE%97%E8%B0%83%E7%94%A8%E5%AE%83%E7%9A%84mustations%E9%87%8C%E7%9A%84%E6%96%B9%E6%B3%95%E4%BF%AE%E6%94%B9%E3%80%82%0A%3E%24store.commit\(%22mutations%E9%87%8C%E7%9A%84%E5%87%BD%E6%95%B0%E5%90%8D%22\)%0A%0Amutations%E9%87%8C%E7%9A%84%E5%87%BD%E6%95%B0%E5%B0%B1%E6%98%AF%E7%9C%9F%E6%AD%A3%E4%BF%AE%E6%94%B9state%E9%87%8C%E7%9A%84%E6%95%B0%E6%8D%AE%E3%80%82%0Aaction%E7%9A%84%E4%BD%BF%E7%94%A8%0A%3E%24store.dispach%0A%0A%E5%92%8B%E4%B8%AA%E5%BD%A2%E5%AE%B9%E5%91%A2%EF%BC%8C%E8%BF%99%E4%B8%9C%E8%A5%BF%E7%9A%84%E5%88%9D%E8%A1%B7%E6%88%91%E8%83%BD%E7%90%86%E8%A7%A3...%E5%8C%85%E6%8B%AC%20%E5%90%84%E7%A7%8D%E9%92%A9%E5%AD%90%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E9%94%81%E7%AD%89%E7%AD%89...%20%E5%88%9D%E8%A1%B7%E6%98%AF%E5%A5%BD%E7%9A%84%EF%BC%8C%E4%BD%86%20%E5%B0%8F%E9%A1%B9%E7%9B%AE%EF%BC%8C%E4%BD%A0%E5%B0%B1%E5%AE%9A%E4%B9%89%E4%B8%AA%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E4%B8%8D%E5%B0%B1%E5%BE%97%E4%BA%86...%0A%0A%0AVuexPersistence%20%0A\-\-\-\-\-\-%0A%E9%85%8D%E5%90%88vuex%E4%BD%BF%E7%94%A8%E3%80%82%0A%0Avuex%E5%AD%98%E5%82%A8%E4%BA%86%E5%85%A8%E5%B1%80%E6%95%B0%E6%8D%AE%E5%90%8E%EF%BC%8C%E8%BF%98%E6%98%AF%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD~%E5%88%B7%E6%96%B0%E5%90%8E%EF%BC%8C%E6%95%B0%E6%8D%AE%E4%B8%A2%E5%A4%B1%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%83%BD%E5%AD%98%E5%82%A8%EF%BC%8C%E9%82%A3%E5%B0%B1%E5%AE%8C%E7%BE%8E%E4%BA%86%EF%BC%8C%E4%BA%8E%E6%98%AF%E6%9C%89%E4%BA%86VuexPersistence%EF%BC%8C%E5%B8%AE%E5%8A%A9vuwx%E6%8C%81%E4%B9%85%E5%8C%96%E6%95%B0%E6%8D%AE%E7%9A%84%E3%80%82%0A%0A%0A%E8%BF%98%E5%BE%97%E5%85%88%E8%AE%B2%E4%B8%8B%EF%BC%9AlocalStorage%0A\*\*localStorage\*\*%0AHTML5%E4%B8%AD%EF%BC%8C%E6%96%B0%E5%8A%A0%E5%85%A5%E4%BA%86%E4%B8%80%E4%B8%AAlocalStorage%E7%89%B9%E6%80%A7%EF%BC%8C%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93%E3%80%82%E8%A7%A3%E5%86%B3%E4%BA%86cookie%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4%E4%B8%8D%E8%B6%B3%E7%9A%84%E9%97%AE%E9%A2%98\(cookie%E4%B8%AD%E6%AF%8F%E6%9D%A1cookie%E7%9A%84%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4%E4%B8%BA4k\)%EF%BC%8ClocalStorage%E4%B8%AD%E4%B8%80%E8%88%AC%E6%B5%8F%E8%A7%88%E5%99%A8%E6%94%AF%E6%8C%81%E7%9A%84%E6%98%AF5M%E5%A4%A7%E5%B0%8F%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%9C%A8%E4%B8%8D%E5%90%8C%E7%9A%84%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%ADlocalStorage%E4%BC%9A%E6%9C%89%E6%89%80%E4%B8%8D%E5%90%8C%E3%80%82%0A%0A%E7%9B%B8%E5%90%8C%E5%9F%9F%E5%90%8D%2F%E7%AB%AF%E5%8F%A3%EF%BC%8C%E5%A4%9ATAB%E5%8F%AF%E4%BB%A5%E5%85%B1%E4%BA%AB%E3%80%82%0A%0A%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%E6%8C%BA%E5%A5%BD%EF%BC%8C%E7%AD%89%E4%BA%8EWEB%E5%89%8D%E7%AB%AF%E6%9C%89%E6%9C%AC%E5%9C%B0%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%86%EF%BC%8C%E7%BC%93%E5%AD%98%E7%82%B9%E5%90%8E%E7%AB%AFAPI%E6%95%B0%E6%8D%AE%E3%80%81%E5%AD%98%E7%82%B9%E8%87%AA%E5%B7%B1%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%8F%AF%E6%96%B9%E4%BE%BF%E5%A4%9A%E4%BA%86%E3%80%82%0A%0A%E8%BF%98%E6%9C%89%E4%B8%AAsessionStorage%EF%BC%8C%E8%B7%9F%E5%AE%83%E6%9C%89%E7%82%B9%E5%83%8F%EF%BC%8C%E4%BD%86%E6%98%AF%E9%A1%B5%E9%9D%A2%E5%85%B3%E9%97%AD%E5%90%8E%E5%8D%B3%E6%B6%88%E5%A4%B1%EF%BC%8C%E4%B9%9F%E6%98%AF%E6%AD%A3%E5%B8%B8%EF%BC%8C%E4%B8%BA%E4%BA%86%E5%AE%89%E5%85%A8%E5%98%9B~%0A%0A%0AVuexPersistence%E5%B0%B1%E6%98%AF%E5%9F%BA%E4%BA%8ElocalStorage%EF%BC%8C%E6%84%9F%E8%A7%89%E4%B9%9F%E6%B2%A1%E5%95%A5%E5%88%9B%E6%96%B0%EF%BC%8C%E4%BC%B0%E8%AE%A1%E5%B0%B1%E6%98%AF%E5%B0%81%E8%A3%85%E4%B8%80%E5%B1%82%E6%88%96%E5%81%9A%E7%82%B9%E5%85%BC%E5%AE%B9%E4%B9%8B%E7%B1%BB%E7%9A%84%E5%90%A7%E3%80%82%0A%0Avue\-router%0A\-\-\-%0A%E7%B1%BB%E5%BA%93%EF%BC%8C%E8%B7%AF%E7%94%B1%E5%99%A8%0A%0A%E8%BF%99%E4%B8%9C%E8%A5%BF%E4%B8%BB%E8%A6%81%E6%98%AF%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%E7%9A%84%E8%B7%B3%E8%BD%AC%0A%0A%E5%90%8E%E7%AB%AF%E5%AF%B9%E8%B7%AF%E7%94%B1%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9A%E5%81%9A%E5%88%86%E5%8F%91%2B%E7%9C%9F%E7%9A%84%E8%B7%B3%E8%BD%AC%EF%BC%8C%E8%BF%98%E6%98%AF%E6%A8%A1%E5%BC%8F%E4%B8%8D%E4%B8%80%E6%A0%B7%EF%BC%8C%E5%89%8D%E7%AB%AF%EF%BC%9A%E6%98%AF%E7%94%B1%E7%94%A8%E6%88%B7%E9%A9%B1%E5%8A%A8%EF%BC%8CN%E4%B8%AA%E7%BB%84%E4%BB%B6%E5%93%8D%E5%BA%94%E6%A8%A1%E5%BC%8F%EF%BC%8C%E5%B0%B1%E6%98%AF%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%E7%9A%84%E8%B7%B3%E8%BD%AC%EF%BC%8C%E8%80%8C%E5%90%8E%E7%AB%AF%E6%9C%89%E7%82%B9%E5%83%8F%E6%98%AF'%E8%A2%AB%E5%8A%A8%E5%BC%8F'%E6%8E%A5%E6%94%B6%E5%89%8D%E7%AB%AF%E8%AF%B7%E6%B1%82%EF%BC%8C%E7%84%B6%E5%90%8E%E8%B7%AF%E7%94%B1%E4%B8%BB%E8%A6%81%E6%98%AF%E5%81%9A%E5%88%86%E5%8F%91%E5%8A%9F%E8%83%BD%0A%0A%E7%BB%84%E4%BB%B6%E7%9A%84%E5%AF%B9%E6%AF%94%EF%BC%9A%E5%90%8E%E7%AB%AF%E7%9A%84%E7%BB%84%E4%BB%B6%E7%9B%B4%E6%8E%A5%E5%B0%B1%E6%8C%82%E5%9C%A8%E5%AE%B9%E5%99%A8%E6%88%96%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E4%B8%8A%E4%BA%86%EF%BC%8C%E5%A4%8D%E6%9D%82%E4%B8%80%E7%82%B9%E7%9A%84%E5%B0%B1%E8%B5%B0RPC%E4%BA%86%E3%80%82%0A%0A%0Avue\-cli%0A\-\-\-\-\-\-%0A%E4%B8%80%E4%B8%AA%E6%8C%87%E4%BB%A4%E8%A1%8C%E6%8E%A7%E5%88%B6%E7%9A%84%E6%8F%92%E4%BB%B6%EF%BC%8C%E5%B8%AE%E5%8A%A9%E5%BC%80%E5%8F%91%E8%80%85%E5%BF%AB%E9%80%9F%E6%90%AD%E5%BB%BA%E3%80%81%E6%89%93%E5%8C%85VUE%E3%80%82%0A%3Enpm%20install%20vue\-cli%0A%0A%E5%AE%89%E8%A3%85%E5%A5%BD%E5%90%8E%EF%BC%8C%E6%88%91%E4%BB%AC%E6%89%BE%E4%B8%AA%E7%9B%AE%E5%BD%95%EF%BC%8C%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AAVUE%E7%9A%84%E9%A1%B9%E7%9B%AE%0A%3Evue%20init%20webpack%20xxxxxx%0A%0A%E8%BF%99%E4%B8%AA%E6%97%B6%E5%80%99%EF%BC%8C%E7%9B%AE%E5%BD%95%E4%B8%8B%E4%BC%9A%E6%9C%89%E4%B8%80%E5%A0%86vue\-cli%E8%84%9A%E6%9C%AC%E5%88%9B%E5%BB%BA%E7%9A%84%E6%96%B0%E7%9B%AE%E5%BD%95%E5%92%8C%E6%96%B0%E6%96%87%E4%BB%B6%0A%60%60%60%0A%E2%94%9C%E2%94%80%E2%94%80%20README.md%0A%E2%94%9C%E2%94%80%E2%94%80%20build%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20build.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20check\-versions.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20logo.png%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20utils.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20vue\-loader.conf.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20webpack.base.conf.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20webpack.dev.conf.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20webpack.prod.conf.js%0A%E2%94%9C%E2%94%80%E2%94%80%20config%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20dev.env.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20index.js%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20prod.env.js%0A%E2%94%9C%E2%94%80%E2%94%80%20index.html%0A%E2%94%9C%E2%94%80%E2%94%80%20package.json%0A%E2%94%9C%E2%94%80%E2%94%80%20src%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20App.vue%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20assets%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20logo.png%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%9C%E2%94%80%E2%94%80%20components%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20HelloWorld.vue%0A%E2%94%82%C2%A0%C2%A0%20%E2%94%94%E2%94%80%E2%94%80%20main.js%0A%E2%94%94%E2%94%80%E2%94%80%20static%0A%60%60%60%0A%3E%E5%88%9D%E5%A7%8B%E5%8C%96%E5%8F%AA%E6%98%AF%E5%B8%AE%E5%8A%A9%E5%BB%BA%E7%AB%8B%E5%A5%BD%E7%BB%93%E6%9E%84%EF%BC%8C%E5%85%B6%E9%87%8D%E7%82%B9%E6%98%AF%EF%BC%9Apackage.json%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%98%AF%E4%BE%9D%E8%B5%96%E5%8C%85%0A%0A%0A%E4%BB%8E%E8%BF%99%E4%B8%AA%E7%BB%93%E6%9E%84%E7%9C%8B%EF%BC%8C%E4%B9%8B%E5%89%8D%E6%89%80%E6%9C%89%E7%9A%84VUE%E5%AD%A6%E4%B9%A0DEMO%E3%80%81%E8%A7%86%E9%A2%91%E7%9A%84%E5%86%99%E6%B3%95%EF%BC%8C%E5%9D%87%E6%98%AF%E7%94%A8%E4%BA%8E%E6%BC%94%E7%A4%BA%EF%BC%8C%E8%80%8C%E7%9C%9F%E5%AE%9E%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%BC%80%E5%8F%91%EF%BC%8C%E4%B8%8D%E5%8F%AF%E8%83%BD%E9%82%A3%E4%B9%88%E9%9B%B6%E6%95%A3%E3%80%82%E8%BF%98%E6%9C%89%E5%AE%9E%E9%99%85%E9%A1%B9%E7%9B%AE%E4%B8%AD%E5%8F%AF%E8%83%BD%E4%BC%9A%E7%94%A8%E5%88%B0N%E5%A4%9A%E4%B8%AA%E7%B1%BB%E5%BA%93%EF%BC%8C%E8%AE%A9%E4%BD%A0%E9%9A%8F%E4%BE%BF%E6%94%BE%E4%B9%9F%E6%9C%89%E7%82%B9LOW%EF%BC%8C%E6%89%80%E4%BB%A5%EF%BC%8C%E5%AE%83%E5%B0%B1%E6%9C%89%E4%B8%AAcli%20%E8%BF%99%E4%B8%AA%E9%AC%BC%E4%B8%9C%E8%A5%BF%EF%BC%8C%E7%B1%BB%E4%BC%BC%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84%E7%9B%AE%E5%BD%95%E7%94%9F%E6%88%90%E5%99%A8%EF%BC%8C%E6%9D%A5%E5%B8%AE%E5%8A%A9%E4%BD%BF%E7%94%A8%E8%80%85%EF%BC%8C%E5%BF%AB%E9%80%9F%E6%8C%89%E7%85%A7%E4%B8%80%E5%AE%9A%E7%BB%93%E6%9E%84%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE.%0A%0A%0A%E6%8E%A5%E7%9D%80%E5%BC%80%E5%A7%8B%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85%E5%90%84%E7%A7%8D%E4%BE%9D%E8%B5%96%E5%8C%85%EF%BC%9A%0A%3Enpm%20install%20%0A%0A%E5%93%8E%EF%BC%8C%E8%BF%99%E4%B8%9C%E8%A5%BF%E4%B8%8D%E6%8C%82%20VPN%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%98%AF%E4%BC%9A%E6%8C%82....%0A%0A%E5%90%AF%E5%8A%A8%0A%3E%20npm%20run%20dev%0A%0A%E9%97%AE%E9%A2%98%EF%BC%9A%E5%AE%83%E6%B2%A1%E6%9C%89webpack.config.js%20%EF%BC%8C%E9%82%A3%E5%AE%83%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%93%E5%8C%85%E8%83%BD%E8%BF%90%E8%A1%8C%E7%9A%84%E5%91%A2%EF%BC%9F%E5%A6%82%EF%BC%9A%0A1.%20%E6%B2%A1%E6%9C%89entry%EF%BC%8C%E9%82%A3%E5%AE%83%E5%A6%82%E4%BD%95%E6%89%BE%E5%88%B0src%2Fmain.js%20%E7%9A%84%EF%BC%9F%0A2.%20output%20%E8%99%BD%E7%84%B6%E5%9C%A8vue.config.js%20%E4%B8%AD%E6%9C%89%E5%AE%9A%E4%B9%89%EF%BC%8C%E5%8F%AF%E4%B8%BA%E5%95%A5%E4%B8%8D%E8%BE%93%E5%87%BA%E5%91%A2%EF%BC%9F%0A3.%20index.html%E4%B8%8D%E5%9C%A8%E6%A0%B9%E7%9B%AE%E5%BD%95%EF%BC%8C%E5%AE%83%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%BE%E5%88%B0%E7%9A%84%E5%91%A2%EF%BC%9F%0A%0A%E8%A7%A3%E5%86%B3%EF%BC%9A%0A1.%20node\_modules%2F%40vue%2Fcli\-service%2Flib%2Fconfig%2Fbase.js%E4%B8%AD%E6%89%BE%E8%A7%81%E5%88%B0%E4%BA%86...%20entry%3A%2Fsrc%2Fmain.js%0A2.%20webpack%E5%A6%82%E6%9E%9C%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E5%BC%80%E5%90%AF%E4%BA%86devServer%20%EF%BC%8C%E5%B0%B1%E4%B8%8D%E5%86%8D%E8%BE%93%E5%87%BA%E4%BA%86%0A3.%20%E6%B2%A1%E6%89%BE%E5%88%B0...%0A%0A\*\*%E5%90%AF%E5%8A%A8%E5%8E%9F%E7%90%86\*\*%0A%3E%E4%BB%8Epackage.json%E4%B8%AD%E6%89%BEscript\-%3Evue\-cli\-service%20serve%0A%0Avue\-cli\-service%20%E5%B1%85%E7%84%B6%E5%9C%A8%20%20node\_modules%2F.bin%20%E4%B8%8B%EF%BC%8C%E6%98%AF%E4%B8%AA%E8%BD%AF%E8%BF%9E%E6%8E%A5%EF%BC%8C%E4%BC%B0%E8%AE%A1%E6%98%AFnpm%20%E7%9A%84%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F%0A%E6%89%93%E5%BC%80%E5%90%8E%EF%BC%8C%E5%8F%91%E7%8E%B0%E5%8C%85%E5%90%AB%E4%BA%86%20..%2Flib%2FService%20%EF%BC%8C%E5%92%8B%E4%B9%9F%E6%89%BE%E4%B8%8D%E8%A7%81...%E6%9C%80%E5%90%8E%EF%BC%9A%0A%3Enode\_modules%2F%40vue%2Fcli\-service%2Flib%2FService.js%20%20%E6%89%BE%E8%A7%81%E4%BA%86....%0A%0A%40vue%20%E8%BF%99%E6%98%AF%E4%BB%80%E4%B9%88%E9%AC%BC%EF%BC%9F%0A%0A%0A%0A%0A%0Avue%20%E6%95%B4%E6%95%B4%E5%8C%85%E4%BA%86%E4%B8%80%E5%B1%82webpack....%0A%0A%0A%0A%0AVUE%E5%88%9B%E5%BB%BA%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%B5%81%E7%A8%8B%0A\-\-\-\-%0A1.%20%E5%AE%89%E8%A3%85NODE.JS%0A2.%20%E5%AE%89%E8%A3%85NPM%0A3.%20%E5%AE%89%E8%A3%85%E5%8C%85%EF%BC%9Avue\-cli%20webpack%0A4.%20%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B8%80%E4%B8%AA%E6%96%B0%E9%A1%B9%E7%9B%AE%3Avue%20init%20webpack%20XXXX%0A5.%20%E5%AE%89%E8%A3%85%E8%BF%99%E4%B8%AA%E6%96%B0%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BE%9D%E8%B5%96%E5%8C%85%20npm%20install%0A6.%20%E5%BC%80%E5%A7%8B%E6%AD%A3%E5%B8%B8%E7%BC%96%E5%86%99%E4%BB%A3%E7%A0%81%2Csrc%20%E7%9B%AE%E5%BD%95%20%0A7.%20%E9%A1%B9%E7%9B%AE%E7%BC%96%E5%86%99%E5%AE%8C%E6%88%90%EF%BC%8C%E5%90%AF%E5%8A%A8%2F%E6%B5%8B%E8%AF%95%20npm%20run%20server%0A8.%20%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B\-%3Epackage.json\-%3Escript\-%3Ebuild.js%0A9.%20build\-%3E%E6%A3%80%E6%9F%A5%E5%BD%93%E5%89%8D%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F\-%3E%E6%A0%B9%E6%8D%AE%E6%8C%87%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0%EF%BC%8C%E5%86%B3%E5%AE%9A%E9%80%89%E6%8B%A9%E5%93%AA%E4%B8%AA%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E7%9A%84buile%E8%84%9A%E6%9C%AC\(%E5%B0%B1%E6%98%AF%E9%85%8D%E7%BD%AE%E4%B8%8D%E4%B8%80%E6%A0%B7\)\-%3Ewebpack%E6%89%93%E5%8C%85JS%E6%96%87%E4%BB%B6\-%3E%E5%86%8D%E6%89%BE%E5%88%B0%E9%A1%B9%E7%9B%AE%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6\-%3E%E5%90%AF%E5%8A%A8webservice~%0A%0A%0Areact%0A\-\-\-\-%0A%E5%9B%A0%E4%B8%BA%E9%A1%B9%E7%9B%AE%E5%B0%B1%E6%98%AFVUE%EF%BC%8C%E7%B2%BE%E5%8A%9B%E5%85%A8%E8%8A%B1%E5%9C%A8%E4%BA%86VUE%E4%B8%8A%E9%9D%A2%EF%BC%8C%E4%BD%86%E6%88%91%E8%BF%98%E6%98%AF%E7%AE%80%E5%8D%95%E6%9F%A5%E4%BA%86%E6%9F%A5%E8%BF%99%E4%B8%A4%E4%B8%AA%E7%9A%84%E5%AF%B9%E6%AF%94%EF%BC%8C%E5%A4%A7%E8%87%B4%EF%BC%9A%0A1.%20react%E6%AF%94%E8%BE%83%E5%BC%80%E6%94%BE%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E5%8E%9F%E7%94%9FJS%E7%9A%84%E6%84%9F%E8%A7%89%EF%BC%8C%E5%B9%B6%E4%B8%8D%E4%BC%9A%E5%83%8FVUE%E6%8F%92%E6%89%8B%E9%82%A3%E4%B9%88%E5%A4%9A%E3%80%82%0A2.%20vue%E6%9C%89%E7%82%B9%E6%A1%86%E6%AD%BB%E4%BA%86%EF%BC%8C%E5%B0%B1%E5%BE%80V\-DOM%20OBJ%E4%B8%8A%E6%8C%82%E5%90%84%E7%A7%8D%E5%87%BD%E6%95%B0%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%20%E9%97%AD%E5%8C%85%E7%9A%84%E6%84%9F%E8%A7%89%E3%80%82%0A%0A%E5%8C%BA%E5%88%AB%EF%BC%9A%0A%3E%E5%89%8D%E7%BD%AE%EF%BC%9A%E6%B2%A1%E6%9C%89%E5%93%AA%E4%B8%AA%E5%A5%BD%E5%93%AA%E4%B8%AA%E5%9D%8F%EF%BC%8C%E4%B8%80%E5%88%87%E7%9C%8B%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E3%80%82%0A1.%20vue%E5%9B%A0%E4%B8%BA%E5%B0%81%E8%A3%85%E7%9A%84%E5%A5%BD%EF%BC%8C%E5%AD%A6%E4%B9%A0%E6%88%90%E6%9C%AC%E4%BD%8E%EF%BC%8C%E4%B8%8A%E6%89%8B%E5%AE%B9%E6%98%93%EF%BC%8C%E5%A5%BD%E6%8B%9B%E4%BA%BA%EF%BC%8C%E4%B9%9F%E4%B8%8D%E5%AE%B9%E6%98%93%E5%A4%9A%E5%BC%BA%E7%9A%84%E6%8A%80%E6%9C%AF%EF%BC%8C%E9%80%82%E5%90%88%E4%B8%AD%E5%B0%8F%E9%A1%B9%E7%9B%AE%E3%80%82%0A2.%20react%E6%AF%94%E8%BE%83%E5%BC%80%E6%94%BE%EF%BC%8C%E9%9C%80%E8%A6%81%E5%90%84%E7%A7%8D%E9%80%A0%E8%BD%AE%E5%AD%90%E3%80%82%E9%80%82%E5%90%88%E5%A4%A7%E5%8E%82%EF%BC%8C%E6%9C%89%E7%89%9BB%E7%9A%84%E6%8A%80%E6%9C%AF%E3%80%82%E5%AD%A6%E4%B9%A0%E6%88%90%E6%9C%AC%E6%9C%89%E7%82%B9%E9%AB%98%E3%80%82%0A%0A%E5%8F%A6%E5%A4%96%EF%BC%8Creact%E5%A5%BD%E5%83%8F%E6%8F%92%E4%BB%B6%E6%AF%94%E8%BE%83%E5%A4%9A%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9JAVA%20SPRING%E7%9A%84%E6%84%8F%E6%80%9D%EF%BC%8C%E5%AF%B9TS%E5%85%BC%E5%AE%B9%E5%A5%BD%EF%BC%8C%E5%AE%83%E8%BF%98%E6%9C%89react%20native%E5%8A%A0%E6%8C%81%EF%BC%8C%E5%9C%A8%E7%A7%BB%E5%8A%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B9%9F%E6%9C%89%E6%89%80%E5%BB%BA%E6%A0%91%E3%80%82%0A%0Aflutter%0A\-\-\-\-\-%0A%E4%B9%8B%E5%89%8D%E4%B9%9F%E5%8F%AA%E6%98%AF%E5%90%AC%E8%AF%B4%EF%BC%8C%E7%8E%B0%E5%9C%A8%E4%BE%9D%E7%84%B6%E4%B9%9F%E5%8F%AA%E6%98%AF%E5%90%AC%E8%AF%B4%EF%BC%8C%E4%BD%86%E7%9C%8B%E4%BA%86%E7%9C%8B%E6%96%87%E6%A1%A3%EF%BC%8C%E6%84%9F%E8%A7%89%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%E6%89%8D%E6%98%AF%E6%AF%94%E8%BE%83%E7%89%9BB%E7%9A%84%E5%91%A2%E3%80%82%0A%0Avue%20%E6%88%91%E6%98%AF%E6%B2%A1%E6%89%BE%E5%88%B0%E8%83%BD%E7%BB%9F%E4%B8%80%E4%B8%89%E7%AB%AF%E7%9A%84%E6%8F%92%E4%BB%B6%E3%80%82%0Areact%20%E5%8F%AF%E4%BB%A5%EF%BC%8C%E4%BD%86%E4%BB%96%E5%BF%85%E9%A1%BB%E4%BE%9D%E8%B5%96V8%0Afluter%20google%E5%87%BA%E5%93%81%EF%BC%8C%E5%8D%95%E7%8B%AC%E6%9C%89%E4%B8%AA%E5%BC%95%E6%93%8E%EF%BC%8C%E8%83%BD%E6%94%AF%E6%8C%813%E7%AB%AF%0A%0Aaxios%0A\-\-\-\-%0A%E7%B1%BB%E5%BA%93%EF%BC%8C%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86%0A%0Ahttp%20XMLHttpRequests%20ajax%0A%0A%E5%B0%B1%E6%98%AF%E6%8A%8A%E4%B8%8A%E9%9D%A2%E7%9A%84%EF%BC%8C%E6%9C%89%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E7%9B%B8%E5%85%B3%E7%9A%84%E6%93%8D%E4%BD%9C%EF%BC%8C%E7%BB%9F%E4%B8%80%E5%B0%81%E8%A3%85%E4%BA%86%E4%B8%80%E5%B1%82%EF%BC%8C%E6%8A%8A%E5%8E%9F%E6%9C%AC%E8%A6%81%E7%A8%8B%E5%BA%8F%E5%91%98%E5%86%99%E7%9A%84%E4%B8%80%E5%A0%86%E4%BB%A3%E7%A0%81%E7%BB%9F%E4%B8%80%E6%8A%BD%E7%A6%BB%E5%B0%81%E8%A3%85%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%9C%89%E4%B8%80%E5%AE%9A%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%A0%E5%B0%84%E5%8A%9F%E8%83%BD%EF%BC%88%E5%A6%82%EF%BC%9AORM%EF%BC%89%0A%0A%0A%0A%0A%E5%9F%BA%E6%9C%AC%E4%B8%8Avue%2Bvuex%2Brouter%2Baxios%E5%B0%B1%E7%AD%89%E4%BA%8E%E5%90%8E%E7%AB%AF%E7%9A%84%E4%B8%80%E4%B8%AA%E6%AF%94%E8%BE%83%E5%AE%8C%E6%95%B4%E7%9A%84%E6%A1%86%E6%9E%B6%E4%BA%86%E3%80%82%0A%E5%8F%8D%E6%AD%A3%E5%81%9A%E4%B8%BA%E5%90%8E%E7%AB%AF%E7%9A%84%E6%88%91%E6%AF%94%E8%BE%83%E5%96%9C%E6%AC%A2%E8%BF%99%E7%A7%8D%E6%A8%A1%E5%BC%8F%EF%BC%8C%E5%A4%A7%E8%80%8C%E5%85%A8%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E6%B3%A8%E5%AE%9A%E8%BF%87%E9%87%8D%EF%BC%8C%E4%B8%94%E5%AD%A6%E4%B9%A0%E6%88%90%E6%9C%AC%E5%A4%AA%E9%AB%98%E3%80%82%E5%8F%8D%E8%80%8C%E7%94%B1%E7%A8%8B%E5%BA%8F%E5%91%98%E6%89%8B%E5%8A%A8%E7%BB%84%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%A1%86%E6%9E%B6%E6%9B%B4%E8%88%92%E6%9C%8D%E3%80%82%0A%0A%3E%E4%BB%8EVUE%E7%9A%84%E6%95%B4%E4%BD%93%E6%A8%A1%E5%BC%8F%E6%9D%A5%E7%9C%8B%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%8A%8A%E5%89%8D%E7%AB%AF%E4%BB%A5%E5%89%8D%E5%90%84%E7%A7%8D%E6%9D%BE%E6%95%A3%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E8%BF%9B%E8%A1%8C%20%E6%8A%BD%E8%B1%A1%E3%80%81%E6%8A%BD%E7%A6%BB%E3%80%81%E5%B0%81%E8%A3%85%E6%88%90%E7%BB%9F%E4%B8%80%E7%9A%84%E7%B1%BB%E5%BA%93%EF%BC%8C%E5%86%8D%E7%BB%99%E7%A8%8B%E5%BA%8F%E5%91%98%E4%BD%BF%E7%94%A8%E3%80%82TMD%EF%BC%8C%E6%80%8E%E4%B9%88%E7%9C%8B%E9%83%BD%E6%98%AF%E5%9C%A8%E8%B7%9F%E5%90%8E%E7%AB%AF%E5%AD%A6...%0A%0A%E5%89%8D%E7%AB%AFui%E6%A1%86%E6%9E%B6%0A\-\-\-\-\-%0Aant%20design%3A%E9%98%BF%E9%87%8C%EF%BC%8Creact%20%0Aelementui%20iview%20ice%20%3Avue%20%0Aamazeui%3A%0Alayui%0Adocsify%0A%0Aelementui%0A\-\-\-\-%0A%3Enpm%20I%20element\-ui%20\-S%0Acnpm%20install%20sass\-loader%20node\-sass%0A%0A%0A%0A%E8%87%B3%E6%AD%A4%EF%BC%9A%E5%A4%A7%E5%89%8D%E7%AB%AF%E7%9A%84%E6%95%B4%E4%BD%93%E6%B5%81%E7%A8%8B%E5%B0%B1%E9%83%BD%E7%A9%BF%E8%B5%B7%E6%9D%A5%E4%BA%86%E3%80%82%E9%9C%80%E8%A6%81%E6%8E%8C%E6%8F%A1%E7%9A%84%E7%9F%A5%E8%AF%86%E7%82%B9%E4%B9%9F%E5%B0%B1%E5%A6%82%E4%B8%8A%E6%89%80%E8%BF%B0%E5%91%80%E3%80%82%0A%0A%E5%A4%9A%E7%AB%AF%0A\-\-\-\-\-%0A%E6%97%A9%E6%9C%9F%E7%9A%84%E5%BC%80%E5%8F%91%E5%9F%BA%E6%9C%AC%E5%B0%B1%E4%B8%80%E7%AB%AF%EF%BC%9APC%EF%BC%8C%E8%80%8C%E7%8E%B0%E5%9C%A8%E8%A6%81%E8%80%83%E8%99%91PC%20H5%20andrio%20IOS%EF%BC%8C%E5%8F%A6%E5%A4%96%EF%BC%9APC%20H5%20%E5%9C%A8%E6%88%91%E9%82%A3%E4%BC%9A%E5%84%BF%E6%9C%89%E4%B8%AA%E5%BE%88%E5%A4%B4%E7%96%BC%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%9A%E6%B5%8F%E8%A7%88%E5%99%A8%E5%85%BC%E5%AE%B9%E3%80%82%0A%0A%E6%89%80%E4%BB%A5%EF%BC%8C%E4%BB%8E%E8%BF%99%E4%B8%AA%E8%A7%92%E5%BA%A6%E7%9C%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E7%9A%84%E5%8F%91%E5%B1%95%E4%B8%8E%E8%BF%9B%E5%8C%96%EF%BC%8C%E5%A4%9A%E5%A4%9A%E5%B0%91%E5%B0%91%E4%B9%9F%E6%98%AF%E8%A2%AB%E5%B8%82%E5%9C%BA%E9%9C%80%E6%B1%82%E6%89%80%E9%80%BC%EF%BC%8C%E5%AE%83%E5%A6%82%E6%9E%9C%E5%86%8D%E4%B8%8D%E8%BF%9B%E5%8C%96%EF%BC%8C%E9%82%A3%E4%B9%88%E8%BF%99N%E7%AB%AF%E5%8F%8A%E6%B5%8F%E8%A7%88%E5%99%A8%E5%85%BC%E5%AE%B9%E7%9C%9F%E7%9A%84%E4%BC%9A%E5%A4%A7%E5%A4%A7%E5%BD%B1%E5%93%8D%E6%95%88%E7%8E%87%E3%80%82%0A%0A%E7%BB%93%E5%90%88%E7%9C%8BVUE%E5%A5%BD%E5%83%8F%E5%9C%A8%E7%A7%BB%E5%8A%A8%E7%AB%AF%E6%B2%A1%E6%9C%89react%20native%E3%80%81fluter%E6%9B%B4%E5%BC%BA%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E5%9C%A8%E5%81%9A%E6%9E%84%E6%9E%B6%E7%9A%84%E6%97%B6%E5%80%99%E5%B0%B1%E5%BE%97%E5%A5%BD%E5%A5%BD%E6%83%B3%E6%83%B3%E4%BA%86%0A%3Eps%3A%E4%BB%8E%E8%BF%99%E9%87%8C%E4%B9%9F%E8%83%BD%E7%90%86%E8%A7%A3%E5%87%BA%EF%BC%8C%E7%A1%AE%E5%AE%9EVUE%E5%8F%AF%E8%83%BD%E5%AF%B9%E5%A4%A7%E9%A1%B9%E7%9B%AE%E4%B8%8D%E9%80%82%E5%90%88%0A%0A%E5%A6%82%E6%9E%9C%E6%89%80%E6%9C%89%E5%85%AC%E5%8F%B8%E7%9A%84%E6%83%B3%E6%B3%95%E9%83%BD%E6%98%AF%EF%BC%9A%E5%85%BC%E5%AE%B9%E5%A4%9A%E7%AB%AF%EF%BC%8C%E9%82%A3%E4%B9%88%E5%8F%AF%E6%82%B2%E7%9A%84%E5%B0%B1%E6%98%AF%E7%8E%B0%E5%9C%A8%E7%9A%84android%20ios%20%E7%9A%84%E5%BC%80%E5%8F%91%E7%A8%8B%E5%BA%8F%E5%91%98%E4%BA%86%EF%BC%8C%E5%8F%AA%E4%BC%9A%E5%8D%95%E4%B8%80%E8%BD%BD%E4%BD%93%E5%BC%80%E5%8F%91%EF%BC%8C%E8%A2%ABJS%E9%99%8D%E7%BB%B4%E6%89%93%E5%87%BB%EF%BC%8C%E5%BF%85%E7%84%B6%E6%98%AF%E5%A4%A7%E6%89%B9%E5%A4%B1%E4%B8%9A%E6%BD%AE%E3%80%82\(%E6%84%9F%E8%A7%89%E8%BF%99%E9%87%8CJS%E5%89%8D%E7%AB%AF%E5%8F%88%E5%BE%97%E8%BF%B7%E4%B9%8B%E8%87%AA%E4%BF%A1%E4%B8%80%E4%BC%9A%E5%84%BF...\)%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-\-\-%0A%E7%8C%9B%E4%B8%80%E7%9C%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E8%BF%99%E5%87%A0%E5%B9%B4%EF%BC%9A%E4%B8%80%E5%A0%86%E6%96%B0%E9%B2%9C%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8Cbackbone%20react%20vue%20angular%20node.js%20npm%20%E7%AD%89%E7%AD%89%EF%BC%8C%20%E4%BD%86%E6%98%AF%E4%B8%80%E5%9C%B0%E9%B8%A1%E6%AF%9B...%E5%85%B6%E6%A0%B8%E5%BF%83%E7%9A%84%E5%8F%91%E5%B1%95%E5%8F%B2%E8%BF%98%E6%98%AF%E5%9C%A8%E5%90%91%E5%90%8E%E7%AB%AF%E5%AD%A6%E4%B9%A0%EF%BC%8C%E8%80%8C%E4%B9%8B%E6%89%80%E4%BB%A5%E4%B8%80%E5%9C%B0%E9%B8%A1%E6%AF%9B%EF%BC%8C%E6%A0%B9%E6%9C%AC%E5%8E%9F%E5%9B%A0%EF%BC%9A%E8%BF%98%E6%98%AF%E5%9F%BA%E7%A1%80%E5%A4%AA%E5%B7%AE%EF%BC%8C%E9%80%A0%E6%88%90%E6%83%B3%E6%B3%95%E9%A3%98%E9%80%B8%E8%80%8C%E4%B8%8D%E4%B8%94%E5%AE%9E%E9%99%85%EF%BC%8C%E6%A1%86%E6%9E%B6%E7%99%BE%E5%87%BA%EF%BC%8C%E4%BD%86%E6%B2%A1%E6%9C%89%E8%83%BD%E5%8A%9B%E5%8E%BB%E8%AE%A4%E8%AF%86%E6%80%9D%E8%80%83%E5%BA%95%E5%B1%82%E7%9A%84%E5%8E%9F%E7%90%86%E3%80%82%0A%0A%E4%BD%86%E6%98%AF%EF%BC%8C%E5%89%8D%E7%AB%AF%E7%A1%AE%E5%AE%9E%E6%98%AF%E5%9C%A8%E8%BF%9B%E6%AD%A5%2C%E5%A4%A7%E5%89%8D%E7%AB%AF%E7%9A%84%E6%A6%82%E5%BF%B5%E6%8A%9B%E5%87%BA%E6%9D%A5%EF%BC%8C%E5%89%8D%E7%AB%AF%E7%A1%AE%E5%AE%9E%E6%AF%94%E4%BB%A5%E5%89%8D%E8%A6%81%E5%AD%A6%E4%B9%A0%E5%BE%88%E5%A4%9A%E7%9F%A5%E8%AF%86%E3%80%82%E5%B0%A4%E5%85%B6%E4%B8%8D%E8%83%BD%E5%85%89%E5%85%89%E7%9B%AF%E7%9D%80%E5%89%8D%E7%AB%AF%E9%82%A3%E7%82%B9%E4%B8%9C%E8%A5%BF%EF%BC%8C%E8%BF%98%E5%BE%97%E4%BC%9A%E7%82%B9nodejs%EF%BC%8C%E6%87%82%E7%82%B9%E5%90%8E%E7%AB%AF%E7%9F%A5%E8%AF%86%E3%80%82%E4%B9%9F%E8%AE%A9%E5%90%8E%E7%AB%AF%E4%B8%8D%E5%86%8D%E9%82%A3%E4%B9%88%E5%AD%A4%E7%8B%AC%EF%BC%88%E6%8B%BF%E4%B8%80%E6%A0%B7%E7%9A%84%E8%96%AA%E6%B0%B4%EF%BC%8C%E9%9C%80%E8%A6%81%E5%AD%A6%E7%9A%84%E7%9F%A5%E8%AF%86%E5%8D%B4%E6%AF%94%E5%89%8D%E7%AB%AF%E5%A4%9A%E4%B8%80%E5%A0%86\!\!\!%EF%BC%89%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8C%E4%B9%9F%E8%83%BD%E9%A9%B1%E6%95%A3%E4%B8%80%E4%BA%9B%E5%89%8D%E7%AB%AF%E6%B7%B7%E5%AD%90%E3%80%82%0A%0A%E4%BB%8E%E5%90%8E%E7%AB%AF%E7%9C%8B%E8%BF%99%E7%A7%8D%E8%B6%8B%E5%8A%BF%EF%BC%8C%E5%96%9C%E5%89%A7%E5%8F%82%E5%8D%8A%EF%BC%9A%0A1.%20%E5%A5%BD%E7%9A%84%E6%98%AF%EF%BC%8C%E5%9B%A0%E4%B8%BA%E7%8E%B0%E5%9C%A8%E7%9A%84%E5%89%8D%E7%AB%AF%EF%BC%9A%E5%B7%B2%E7%BB%8F%E6%8A%8A%E5%90%8E%E7%AB%AF%E5%A4%A7%E9%83%A8%E5%88%86%E7%9A%84%E5%B7%A5%E4%BD%9C%E9%83%BD%E6%8B%BF%E8%BF%87%E5%8E%BB%E4%BA%86%EF%BC%8C%E5%A6%82%EF%BC%9Aaction%E8%B7%AF%E7%94%B1%E3%80%81%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E3%80%81%E6%9D%83%E9%99%90%E9%AA%8C%E8%AF%81%E7%AD%89%E3%80%82%0A2.%20%E4%B8%8D%E5%A5%BD%E7%9A%84%E6%98%AF%EF%BC%9A%E5%90%8E%E7%AB%AF%E7%9A%84%E7%94%9F%E5%AD%98%E7%A9%BA%E9%97%B4%E8%A2%AB%E5%8E%8B%E7%BC%A9%E3%80%82%0A%0A%E8%BF%99%E5%B0%B1%E6%8C%BA%E7%BA%A0%E7%BB%93%EF%BC%8C%E8%B7%9F%E5%89%8D%E7%AB%AF%E5%AF%B9%E6%8E%A5%E5%8F%A3%EF%BC%8C%E6%80%BB%E8%A7%89%E5%BE%97%E4%BB%96%E4%BB%AC%E7%AC%A8%EF%BC%8C%E8%A6%81%E5%90%8E%E7%AB%AF%E5%81%9A%E5%BE%88%E5%A4%9A%EF%BC%8C%E4%BD%86%E7%9C%9F%E7%9A%84%E8%A2%AB%E6%8B%BF%E8%BF%87%E5%8E%BB%E5%90%8E%EF%BC%8C%E5%8F%91%E7%8E%B0%EF%BC%9A%E8%87%AA%E5%B7%B1%E4%B8%8D%E5%86%8D%E6%98%AF%E6%9C%80%E9%87%8D%E8%A6%81%E7%9A%84%E9%82%A3%E4%B8%AA%E5%90%8E%E7%AB%AF%E4%BA%86....%0A%0A%E5%86%B2%E7%AA%81%0A\-\-\-\-%0A%E6%8C%89%E7%85%A7%E6%80%BB%E7%BB%93%E5%90%8E%E7%9A%84%E7%9C%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E5%9C%A8%E6%95%B4%E4%BD%93%E5%90%8E%E7%AB%AF%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E4%BB%A5node%E8%AF%9E%E7%94%9F%E6%9D%A5%E7%9C%8B%EF%BC%8C%E5%89%8D%E7%AB%AF%E6%9C%AA%E6%9D%A5%E4%BC%9A%E4%B8%8D%E4%BC%9A%E6%97%A0%E8%81%8A%E7%9A%84%E5%8F%88%E6%90%9E%E5%87%BA%E5%90%8E%E7%AB%AF%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%9F%E5%A6%82%EF%BC%9A%E9%98%9F%E5%88%97%E7%B3%BB%E7%BB%9F%E3%80%81%E6%97%A5%E5%BF%97%E7%B3%BB%E7%BB%9F%E7%AD%89%E3%80%82%E6%88%91%E8%A7%89%E5%BE%97%E4%BB%96%E4%BB%AC%E5%B9%B2%E7%9A%84%E5%87%BA%E6%9D%A5%E3%80%82%E9%82%A3%E4%B9%88%EF%BC%8C%E8%A1%8C%E4%B8%9A%E5%8F%88%E5%BE%97%E5%87%BA%E7%8E%B0%E4%B8%80%E6%AC%A1%E6%B7%B7%E4%B9%B1%E6%9C%9F%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%EF%BC%9A%E5%88%86%E5%88%86%E5%90%88%E5%90%88%EF%BC%8C%E5%90%88%E5%90%88%E5%88%86%E5%88%86%E7%9A%84%E6%84%8F%E6%80%9D%E5%91%A2%E3%80%82%E8%BF%99%E4%B9%9F%E6%98%AF%E6%9C%89%E4%BA%9B%E5%89%8D%E7%AB%AF%E8%BF%B7%E4%B9%8B%E8%87%AA%E4%BF%A1%E7%9A%84%E5%8E%9F%E5%9B%A0%E3%80%82%0A%0A%E5%90%88%E6%A0%BC%E7%9A%84%E5%89%8D%E7%AB%AF%0A\-\-\-\-%0A%E5%9F%BA%E7%A1%80%EF%BC%9Anode.js%20npm%20webpack%20vue%20react%20react\-native%20ios%20android%20fluter%20%E6%95%B0%E6%8D%AE%E5%BA%93%20%0A%E6%A6%82%E5%BF%B5%EF%BC%9Av\-dom%20%E6%8F%92%E4%BB%B6%20callback%20es6%E5%9F%BA%E7%A1%80%20%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%20%E7%BC%93%E5%AD%98%0A%E8%AE%BE%E8%AE%A1%EF%BC%9A%E8%B7%AF%E7%94%B1%E5%99%A8%20%E6%9D%83%E9%99%90%E9%AA%8C%E8%AF%81%20%0A%E8%AF%AD%E8%A8%80%EF%BC%9Ajs%20ts%20java%20oc%20sql\-query%20dart%20html%20xml%20yaml%20css%2Fdiv%0A%0A%E8%A1%8C%E4%B8%9A%E5%88%86%E6%9E%90%0A\-\-\-\-\-%0A%E4%BB%8E%E6%B8%B8%E6%88%8F%E8%A1%8C%E4%B8%9A%E7%9A%84%E5%89%8D%E7%AB%AF%E7%9C%8B%EF%BC%8C%E5%90%8E%E7%AB%AF%E6%AD%A3%E5%9C%A8%E8%A2%AB%E5%BC%B1%E5%8C%96%EF%BC%8C%E5%9F%BA%E6%9C%AC%E5%8F%AF%E8%83%BD%E4%B8%80%E4%B8%AA%E6%B8%B8%E6%88%8F%E5%85%AC%E5%8F%B8%E4%B8%80%E5%A0%86%E5%89%8D%E7%AB%AF%EF%BC%8C%E5%90%8E%E7%AB%AF%E4%B9%9F%E5%B0%B1%E5%87%A0%E4%B8%AA%EF%BC%8C%E8%80%8C%E4%B8%94%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%87%A0%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%85%B1%E4%BA%AB%E8%BF%99%E5%87%A0%E4%B8%AA%E5%90%8E%E7%AB%AF%E3%80%82%E4%BC%B0%E8%AE%A1%E5%90%8E%E9%9D%A2%E5%8F%AF%E8%83%BD%E4%BA%92%E8%81%94%E7%BD%91%E5%85%AC%E5%8F%B8%E4%B9%9F%E4%BC%9A%E8%BF%99%E6%A0%B7%E3%80%82%E4%B9%9F%E5%B0%B1%E6%98%AF%EF%BC%9A%E4%BB%A5%E5%90%8E%E5%89%8D%E7%AB%AF%E8%A6%81%E5%81%9A%E5%85%A8%E6%A0%88%E5%B7%A5%E7%A8%8B%E5%B8%88%EF%BC%8C%E6%9C%89%E9%82%A3%E4%B9%88%E7%82%B9%E9%A3%8E%E6%B0%B4%E8%BD%AE%E6%B5%81%E8%BD%AC%E7%9A%84%E6%84%9F%E8%A7%89%E3%80%82%0A%0A%E9%82%A3%E5%90%8E%E7%AB%AF%E6%85%A2%E6%85%A2%E5%8F%AF%E8%83%BD%E6%85%A2%E6%85%A2%E5%8F%98%E6%88%90%EF%BC%9A%0A1.%20api%E6%8E%A5%E5%8F%A3%EF%BC%8C%E6%9B%B4%E5%85%B3%E6%B3%A8%E4%B8%9A%E5%8A%A1%E7%9A%84%E5%85%B7%E4%BD%93%E9%80%BB%E8%BE%91%E3%80%82%0A2.%20%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%98%E5%82%A8%EF%BC%8C%0A3.%20%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%AE%89%E5%85%A8%E5%92%8C%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%0A4.%20%E6%A1%86%E6%9E%B6%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%0A5.%20%E5%A4%A7%E6%95%B0%E6%8D%AE%0A%0A%E4%BD%86%E6%98%AF%EF%BC%8C%E5%89%8D%E7%AB%AF%E7%9A%84%E5%8F%91%E5%B1%95%E5%8F%B2%E5%AE%8C%E5%85%A8%E6%98%AF%E7%85%A7%E6%8A%84%E5%90%8E%E7%AB%AF%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E5%90%8E%E7%AB%AF%E7%9A%84%E8%BF%99%E4%B8%AA'%E8%BF%87%E7%A8%8B'%EF%BC%8C%E5%85%B6%E5%AE%9E%EF%BC%8C%E5%89%8D%E7%AB%AF%E5%BA%94%E8%AF%A5%E4%B9%9F%E6%98%AF%E8%BF%99%E6%A0%B7%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%BD%93%E5%89%8D%E7%AB%AF%E8%BF%9B%E5%8C%96%E5%88%B0%E4%B8%80%E5%AE%9A%E7%A8%8B%E5%BA%A6%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%AE%8C%E6%95%B4%E6%9C%9F%E3%80%81%E5%B9%B3%E8%A1%A1%E6%9C%9F%E5%90%8E%EF%BC%8C%E5%90%84%E7%A7%8D%E6%A1%86%E6%9E%B6%E5%8F%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E3%80%81%E6%80%9D%E6%83%B3%E8%A2%AB%E7%BB%9F%E4%B8%80%E5%90%8E%EF%BC%8C%E9%82%A3%E4%B9%88%E5%89%8D%E7%AB%AF%E4%B9%9F%E4%BC%9A%E5%A4%A7%E6%89%B9%E9%87%8F%E7%9A%84%E4%B8%8B%E5%B2%97%EF%BC%8C%E5%A6%82%EF%BC%9Aunity%20ue%20%E5%AF%B9%E6%AF%94%3D%3E%20java\-spring%0A%0A%0A%E4%BA%A4%E6%B5%81%0A\-\-\-\-\-\-%0A%E8%B7%9F%E5%90%8C%E4%BA%8B%E4%BA%A4%E6%B5%81%EF%BC%8C%E4%BB%96%E6%8F%90%E5%88%B0%E4%B8%80%E7%82%B9%E7%8E%B0%E5%9C%A8%E7%9A%84WEB%E5%89%8D%E7%AB%AF%E6%AD%A3%E6%85%A2%E6%85%A2%E8%BD%AC%E6%8D%A2%E6%88%90%E5%89%8D10%E5%B9%B4%E7%9A%84CS%E6%A8%A1%E5%BC%8F%EF%BC%8C%E6%88%91%E8%A7%89%E5%BE%97%E8%BF%99%E4%B8%AA%E6%80%BB%E7%BB%93%E6%8C%BA%E6%9C%89%E9%81%93%E7%90%86%E3%80%82%0AVUE%2BNODE.JS%20%E8%BF%98%E7%9C%9F%E6%98%AF%E5%B9%B2%E4%BA%86%E4%BB%A5%E5%89%8DCS%E6%A8%A1%E5%BC%8F%E7%9A%84client%E7%AB%AF%EF%BC%8C%E5%BE%88%E9%87%8D%E3%80%82%E8%BF%87%E5%8E%BB%E7%9A%8410%E5%B9%B4%E6%98%AF%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%98%AF%E9%87%8D%E5%90%8E%E8%BD%BB%E5%89%8D%EF%BC%8C%E7%8E%B0%E5%9C%A8%E5%8F%88%E5%8F%8D%E8%BF%87%E6%9D%A5%E6%98%AF%E9%87%8D%E5%89%8D%E8%BD%BB%E5%90%8E%E3%80%82%E8%AE%BD%E5%88%BA%E7%9A%84%E6%98%AF%EF%BC%9A%E7%8E%B0%E5%9C%A8%E8%BD%AC%E5%8F%98%E6%88%90%E5%89%8D10%E5%B9%B4%E7%9A%84%E6%A8%A1%E5%BC%8F%EF%BC%8C%E6%9C%AA%E6%9D%A510%E5%B9%B4%E6%B2%A1%E5%87%86%E5%8F%AF%E8%83%BD%E5%8F%88%E6%8D%A2%E6%88%90%E7%8E%B0%E5%9C%A8%E7%9A%84%E6%A8%A1%E5%BC%8F%E3%80%82%E5%8F%98%E6%9D%A5%E5%8F%98%E5%8E%BB%EF%BC%8C%E5%85%B6%E6%A0%B8%E5%BF%83%E5%B0%B1%E9%82%A3%E4%B9%88%E7%82%B9%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%83%8F%E6%88%91%E8%AE%B0%E8%BF%99%E7%AF%87%E7%AC%94%E8%AE%B0%EF%BC%8C%E6%BB%A1%E6%89%93%E6%BB%A1%E7%AE%97%E5%B0%B1%E4%B8%80%E5%A4%A9%E7%9A%84%E6%97%B6%E9%97%B4%EF%BC%8C%E4%B9%8B%E6%89%80%E4%BB%A5%E5%BF%AB%EF%BC%8C%E6%98%AF%E5%BA%95%E5%B1%82%E7%9A%84%E5%8E%9F%E7%90%86%E9%83%BD%E6%87%82....%0A%0A%E6%88%91%E6%83%B3%E8%B5%B7%E6%88%91%E9%A2%86%E5%AF%BC%E7%9A%84%E4%B8%80%E5%8F%A5%E8%AF%9D%EF%BC%9AUNIX%20LINUX%20%E8%BF%99%E9%83%BD%E8%80%81%E5%8F%A4%E8%91%A3%E7%BA%A7%E5%88%AB%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E4%B8%BA%E5%95%A5%E7%8E%B0%E5%9C%A8%E8%BF%98%E5%9C%A8%E7%94%A8%EF%BC%9F%E8%80%8C%E4%B8%94%E5%85%A8%E4%B8%96%E7%95%8C%E9%83%BD%E5%9C%A8%E7%94%A8%EF%BC%9F%0A%0A%E5%90%90%E6%A7%BD%0A\-\-\-\-%0A%E5%A4%A7%E5%89%8D%E7%AB%AF%E7%9A%84%E5%85%B4%E8%B5%B7%EF%BC%8C%E8%AE%A9%E4%B8%80%E4%BA%9B%E5%89%8D%E7%AB%AF%E5%BC%80%E5%A7%8B%E4%BA%86%E8%BF%B7%E4%B9%8B%E8%87%AA%E4%BF%A1%E3%80%82%0A1.%20%E4%BC%9A%E4%B8%80%E4%B8%AAJS%2FTS%2BNODE.JS%20%E5%B0%B1%E5%AE%8C%E5%85%A8%E5%A4%9F%E8%A1%8C%E8%B5%B0%E5%A4%A9%E4%B8%8B%E4%BA%86%E3%80%82%0A2.%20VUE%E6%98%AF%E9%9D%9E%E5%B8%B8%E7%89%9B%E9%80%BC%E7%9A%84%E4%B8%80%E4%B8%AA%E4%B8%9C%E8%A5%BF%EF%BC%8C%E6%98%AF%E7%A5%9E~%E5%AE%8C%E5%85%A8%E6%8A%8AVUE%E7%BB%99%E7%A5%9E%E5%8C%96%E4%BA%86...%0A%0A%E5%AF%B9%E4%BA%8E%E8%BF%99%E7%A7%8D%E6%83%B3%E6%B3%95%EF%BC%8C%E6%88%91%E6%98%AF%E7%9C%9F%E6%97%A0%E8%AF%AD%E5%95%8A%E3%80%82%E5%86%8D%E7%89%9B%E9%80%BC%E7%9A%84JS%E4%BB%96%E4%B9%9F%E5%8F%AA%E6%98%AF%E4%B8%80%E4%B8%AA%E5%89%8D%E7%AB%AF%EF%BC%8C%E8%80%8C%E4%B8%80%E4%B8%AA%E5%85%AC%E5%8F%B8%E7%9A%84%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84%EF%BC%8C%E6%98%AF%E5%A4%9A%E8%AF%AD%E8%A8%80%E6%95%B4%E5%90%88\(%E9%85%8D%E5%90%88%E7%9D%80%E5%AE%8C%E6%88%90%E6%89%80%E6%9C%89%E4%BB%BB%E5%8A%A1\)%0A%0A%E6%AF%94%E5%A6%82%EF%BC%9A%0APHP%EF%BC%9A%E9%80%82%E5%90%88%E5%86%99WEB%E7%AB%AF%E6%8E%A5%E5%8F%A3%EF%BC%8C%E9%80%82%E5%90%88%E4%B8%AD%E5%B0%8F%E5%9E%8B%E9%A1%B9%E7%9B%AE%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%8F%98%E5%8C%96%E5%BF%AB%20%E8%BF%AD%E4%BB%A3%E5%BF%AB%20%E5%BC%80%E5%A7%8B%E5%91%A8%E6%9C%9F%E7%9F%AD%0APY%20%E9%80%82%E5%90%88%E4%B8%AD%E5%B0%8F%E5%9E%8B%E9%A1%B9%E7%9B%AE%E7%9A%84BI%E5%88%86%E6%9E%90%0AJAVA\-SPIRNG%EF%BC%9A%E9%80%82%E5%90%88%E4%B8%AD%E5%A4%A7%E5%9E%8B%20%E9%A1%B9%E7%9B%AE%0AJAVA\-SPRING\-CLOUE%3A%E9%80%82%E5%90%88%E5%A4%A7%E5%9E%8B%E9%A1%B9%E7%9B%AE%0AGO%3A%E5%9F%BA%E4%BA%8EJAVA%E8%B7%9FPHP%E4%B8%AD%E9%97%B4%E7%9A%84%E8%AF%AD%E8%A8%80%EF%BC%8C%E5%B0%8F%E8%80%8C%E7%BE%8E%EF%BC%8C%E4%BD%86%E8%BF%98%E5%8F%AF%E4%BB%A5%E6%80%A7%E8%83%BD%E9%AB%98%EF%BC%8C%E5%B9%B6%E5%86%99%E5%90%8E%E7%AB%AF%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E7%9A%84%E6%A8%A1%E5%BC%8F%E7%9A%84%E9%A1%B9%E7%9B%AE%0AC%23%20.NET%20%3A%E5%9C%A8WIN%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%EF%BC%8C%E8%BF%98%E6%9C%89%E6%B8%B8%E6%88%8F%E8%A1%8C%E4%B8%9A%EF%BC%8C%E8%BF%98%E6%98%AF%E9%9D%9E%E5%B8%B8%20%E5%A5%BD%E7%9A%84%E3%80%82%0AC%2FC%2B%2B%3A%E8%B6%85%E5%BA%95%E5%B1%82%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E9%9D%9E%E5%B8%B8%E8%AE%B2%E7%A9%B6%E6%80%A7%E8%83%BD%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E8%BF%99%E4%B8%AA%E9%9D%9E%E5%B8%B8%20%E9%80%82%E5%90%88%E3%80%82%0A%0A%E8%BF%98%E6%9C%89%E5%90%84%E7%A7%8D%E5%BC%80%E6%BA%90%E7%9A%84%E8%BD%AF%E4%BB%B6%2F%E4%B8%AD%E9%97%B4%E4%BB%B6%EF%BC%9A%0A1.%20%E9%98%9F%E5%88%97%E7%B3%BB%E7%BB%9F%0A2.%20%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%99%A8%0A3.%20%E8%BF%9E%E6%8E%A5%E7%AE%A1%E7%90%86%E5%99%A8%0A4.%20%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%E5%99%A8%0A5.%20%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E%0A6.%20%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%86%E4%BD%93%E7%B3%BB%0A7.%20gitlab%0A8.%20%E5%8F%91%E5%B8%83%2F%E5%B8%83%E7%BD%AE%0A......%0A%0A%E9%80%80%E4%B8%80%E6%AD%A5%E8%AE%B2%EF%BC%8C%E9%83%BD%E4%B8%8D%E8%AF%B4%E8%BF%99%E4%BA%9B%E5%A4%8D%E6%9D%82%E7%9A%84%EF%BC%8C%E5%B0%B1%E9%9A%8F%E9%9A%8F%E4%BE%BF%E4%BE%BF%E8%AF%B4%E4%B8%AA%E7%AE%97%E6%B3%95%EF%BC%9A%E8%AF%BB%E6%97%B6%E5%86%99%20%E4%BA%8C%E5%8F%89%E6%A0%91%20hashMap%20%E5%85%AB%E7%9A%87%E5%90%8E%20%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%20bitmap%20geo%20%E9%80%92%E5%BD%92%20TCP%E6%8B%A5%E5%A1%9E%E7%AD%96%E7%95%A5%EF%BC%8C%E7%9C%9F%E6%98%AF%E5%88%86%E5%88%86%E9%92%9F%E7%A7%92%E4%BA%86%E4%BD%A0%E5%95%8A....%0A%0A%0A%E5%93%8E%EF%BC%8C%E5%8F%8D%E6%AD%A3%E6%80%BB%E7%BB%93%E4%B8%8B%E6%9D%A5%E5%90%A7%EF%BC%8C%E5%BD%92%E6%A0%B9%E7%BB%93%E5%BA%95%EF%BC%9A%E8%BF%98%E6%98%AF%E5%89%8D%E7%AB%AF%E7%9A%84%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87%E5%A4%AA%E5%B7%AE%E3%80%82%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A%0A
