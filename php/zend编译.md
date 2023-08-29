# zend编译

eaccelerator  PHP加速器

词法分析\-》语法分析\-》opcode\-》执行

PHP:解释型语言

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

\<?php

   echo "Hello World";

   $a = 1 \+ 1;

   echo $a;

?\>

PHP执行这段代码会经过如下4个步骤\(确切的来说，应该是PHP的语言引擎Zend\)

1. Scanning\(Lexing\) ,将PHP代码转换为语言片段\(Tokens\)

2. Parsing, 将Tokens转换成简单而有意义的表达式

3. Compilation, 将表达式编译成Opocdes

4. Execution, 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能。

题外话:现在有的Cache比如APC,可以使得PHP缓存住Opcodes，这样，每次有请求来临的时候，就不需要重复执行前面3步，从而能大幅的提高 PHP的执行速度。

Lexing:就是一个词法分析的依据表,处理结果如下:

Array\(

\[0\] =\> Array\(

\[0\] =\> 367

\[1\] =\> Array\(

\[0\] =\> 316

\[1\] =\> echo

\)

\[2\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[3\] =\> Array\(

\[0\] =\> 315

\[1\] =\> "Hello World"

\)

\[4\] =\> ;

\[5\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[6\] =\> =

\[7\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[8\] =\> Array\(

\[0\] =\> 305

\[1\] =\> 1

\)

\[9\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[10\] =\> \+

\[11\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[12\] =\> Array\(

\[0\] =\> 305

\[1\] =\> 1

\)

\[13\] =\> ;

\[14\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[15\] =\> Array\(

\[0\] =\> 316

\[1\] =\> echo

\)

\[16\] =\> Array\(

\[0\] =\> 370

\[1\] =\>

\)

\[17\] =\> ;

\)

分析这个返回结果我们可以发现，源码中的字符串，字符，空格，都会原样返回。每个源代码中的字符，都会出现在相应的顺序处。而，其他的比如标签，操作符，语句，都会被转换成一个包含俩部分的Array: Token ID \(也就是在Zend内部的改Token的对应码，比如,T\_ECHO,T\_STRING\)，和源码中的原来的内容。

接下来，就是Parsing阶段了，Parsing首先会丢弃Tokens Array中的多于的空格，然后将剩余的Tokens转换成一个一个的简单的表达式

1. echo a constant string

2. add two numbers together

3. store the result of the prior expression to a variable

4. echo a variable

然后就改Compilation阶段了，它会把Tokens编译成一个个op\_array, 每个op\_arrayd包含如下5个部分：

1. Opcode数字的标识，指明了每个op\_array的操作类型，比如add , echo

2. 结果 存放Opcode结果

3. 操作数1 给Opcode的操作数

4. 操作数2

5. 扩展值 1个整形用来区别被重载的操作符

比如，我们的PHP代码会被Parsing成:

\* ZEND\_ECHO ‘Hello World’

\* ZEND\_ADD ~0 1 1

\* ZEND\_ASSIGN \!0 ~0

\* ZEND\_ECHO \!0

呵呵，你可能会问了，我们的$a去那里了？

恩，这个要介绍操作数了，每个操作数都是由以下俩个部分组成：

a\) op\_type : 为IS\_CONST, IS\_TMP\_VAR, IS\_VAR, IS\_UNUSED, or IS\_CV

b\) u,一个联合体，根据op\_type的不同，分别用不同的类型保存了这个操作数的值\(const\)或者左值\(var\)

而对于var来说，每个var也不一样

IS\_TMP\_VAR, 顾名思义，这个是一个临时变量，保存一些op\_array的结果，以便接下来的op\_array使用，这种的操作数的u保存着一个指向变量表的一个句柄（整数），这种操作数一般用~开头，比如~0,表示变量表的0号未知的临时变量

IS\_VAR 这种就是我们一般意义上的变量了,他们以$开头表示

IS\_CV 表示ZE2.1/PHP5.1以后的编译器使用的一种cache机制，这种变量保存着被它引用的变量的地址，当一个变量第一次被引用的时候，就会被CV起来，以后对这个变量的引用就不需要再次去查找active符号表了，CV变量以！开头表示。

这么看来，我们的$a被优化成\!0了。

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

首先，Zend Engine\(ZE\)，调用词法分析器\(Lex生成的，源文件在 Zend/zend\_language\_sanner.l\), 将我们要执行的PHP源文件，去掉空格 ，注释，分割成一个一个的token。

然后，ZE会将得到的token forward给语法分析器\(yacc生成, 源文件在 Zend/zend\_language\_parser.y\)，生成一个一个的op code，opcode一般会以op array的形式存在，它是PHP执行的中间语言。

最后，ZE调用zend\_executor来执行op array，输出结果。

图1 处理流程

ZE是一个虚拟机，正是由于它的存在，所以才能使得我们写PHP脚本，完全不需要考虑所在的操作系统类型是什么。ZE是一个CISC（复杂指令处理器）， 它支持150条指令（具体指令在 Zend/zend\_vm\_opcodes.h），包括从最简单的ZEND\_ECHO\(echo\)到复杂的 ZEND\_INCLUDE\_OR\_EVAL\(include,require\)，所有我们编写的PHP都会最终被处理为这150条指令\(op code\)的序列，从而最终被执行。

那有什么办法可以看到我们的PHP脚本，最终被“翻译”成什么样的呢？ 也就是说，op code张的什么样子呢？ 呵呵，达到这个，我们需要重新编译PHP，修改它的compile\_file和zend\_execute函数。不过，在PECL中已经有这样的模块，可以 让我们直接使用了，那就是由 Derick Rethans开发的VLD \(Vulcan Logic Dissassembler\)模块。你只要下载这个模块，并把他载入PHP中，就可以通过简单的设置，来得到脚本翻译的结果了。具体关于这个模块的使用说 明－雅虎一下，你就知道^\_^。

接下来，让我们尝试用VLD来查看一段简单的PHP脚本的中间语言。

原始代码：

\<?php

$i = “This is a string“;

//I am comments

echo $i.‘ that has been echoed to screen‘;

?\>

采用VLD得到的op codes:

filename:/home/Desktop/vldOutOne.php

function name: \(null\)

number of ops: 7

line \#  op                 fetch       ext  operands

——————————————————————————————————————————\-

2 0 FETCH\_W local $0, ‘i‘

1 ASSIGN $0, ‘This\+is\+a\+string‘

4 2 FETCH\_R local $2, ‘i‘

3 CONCAT ~3, $2,‘\+that\+has\+been\+echoed\+to\+screen‘

4 ECHO ~3

6 5 RETURN 1

6 ZEND\_HANDLE\_EXCEPTION

我们可以看到，源文件中的注释，在op code中，已经没有了，所以不用担心注释太多会影响你的脚本执行时间（实际上，它是会影响ZE的词法处理阶段的用时而已）。

现在我们来一条一条的分析这段op codes，每一条op code 又叫做一条op\_line，都由如下7个部分，在zend\_compile.h中，我们可以看到如下定义：

struct \_zend\_op {

opcode\_handler\_t handler;

znode result;

znode op1;

znode op2;

ulong extended\_value;

uint lineno;

zend\_uchar opcode;

};

其中，opcode字段指明了这操作类型，handler指明了处理器，然后有俩个操作数，和一个操作结果。

   1. FETCH\_W, 是以写的方式获取一个变量，此处是获取变量名”i”的变量于$0（\*zval）。

   2. 将字符串”this\+is\+a\+string”赋值\(ASSIGN\)给$0

   3. 字符串连接

   4. 显示

可以看出，这个很类似于很多同学大学学习编译原理时候的三元式，不同的是，这些中间代码会被Zend VM（Zend虚拟机）直接执行。

真正负责执行的函数是，zend\_execute, 查看zend\_execute.h:

下一次，我将介绍PHP变量的灵魂 – zval, 你将会看到PHP是如何实现它的变量传递，类型戏法，等等。

[http://www.laruence.com/2008/08/11/147.html](http://www.laruence.com/2008/08/11/147.html)

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

opcache

以上的流程总结

![f49feccbcdb32d26e191082652e28b33.jpg](image/f49feccbcdb32d26e191082652e28b33.jpg)

使用opcache 流程

![a9f1815ecd4dcee057788bff450503b3.jpg](image/a9f1815ecd4dcee057788bff450503b3.jpg)

安装

5.3以上直接，安装包的扩展中即有。

编译完成后：

zend\_extension=opcache.so

几个关键配置值：

;OPcache共享内存存储大小,单位MB

opcache.memory\_consumption=128

;这个选项用于控制内存中最多可以缓存多少个PHP文件。这个选项必须得设置得足够大，大于你的项目中的所有PHP文件的总和。

设置值取值范围最小值是 200，最大值在 PHP 5.5.6 之前是 100000，PHP 5.5.6 及之后是 1000000。也就是说在200到1000000之间。

opcache.max\_accelerated\_files=4000

常用函数

opcache\_compile\_file\($php\_file\); \#预生成opcode缓存

opcache\_is\_script\_cached\($php\_file\) \#查看是否生成opcode缓存

opcache\_invalidate\($php\_file, true\) \#清除单个缓存

opcache\_reset\(\); \#清空缓存

opcache\_get\_status\(\); \#获取缓存的状态信息

opcache\_get\_configuration\(\); \#获取缓存的配置信息
