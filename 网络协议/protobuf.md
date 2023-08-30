# protobuf

## 概览

主要功能：压缩数据。数据在网络传输过程中，对数进行格式定义，压缩其内容，提高传输性能。可跨语言、跨平台。

> 类似json、xml格式，也可以简单理解为将：将数据序列化

## 大体流程

![protobuf.png](image/protobuf.png)

1. 使用方/发起方，需要先定义\(协调\)好.proto描述文件
2. 使用方，需要先下载一个protobuf编译器软件
3. 使用软件，编译 .proto 描述文件
4. 编译之前，需要设定好要生成哪种语言的可能执行代码
5. 将编译器生成好的代码嵌入到自己的类库中
6. 最后，项目使用时执行序列化/反序列化

## 编译原理

它的核心其实是: .proto描述文件。它的文件内容定义了通信的数据结构与数据类型。

而分析看，它的内容格式：与任何语言无关，往更底层说更像是一个文本格式\(自定义格式\)，编译器就是解析这些广本格式，生成不同语言的可执行代码。

先看一下最终二进制数据流的格式：

![protobuf-field.png](image/protobuf-field.png)

![protobuf-tag.png](image/protobuf-tag.png)

每个字段占一部分，而每个部分又被分成了3个子部分\(TLV\(tag\-length\-value\) \)，其实TAG子部分又被划分成了：两个子子部分：field\_number\+wite\_type

#### wire\_type

|type|meaning          |User for                                         |
|----|-----------------|-------------------------------------------------|
|0   |varint           |int32 int64 unit32 uint64 sint32 sint64 bool enum|
|1   |64\-bit          |fixed64 sfixed64 dobule                          |
|2   |length\-delimited|string bytes \(embedded messages\) repeated      |
|3   |放弃             |                                                 |
|4   |放弃             |                                                 |
|5   |32\-bit          |fixed32 sfixed64 float                           |

varint:其实也算是定长，所以不需要length，其内部结构就变成了：TV格式

64\-bit和32\-bit:其实也算是定长，所以不需要length

#### varint

varint，直译：变长的整数数字。也算是一种压缩算法，对整形进行进一步的压缩。

如：int32 ，4个字节，此时有一个数字：1 ，其最终的二进制：

> 00000000 00000000 00000000 00000001

我们发现，前3个字节均为0，且没什么用。开启压缩模式：删除前3个字节，只留最后一个字节

我们换个数字，1024

> 00000000 00000000 00000010 00000000

开始变换：

从最右侧\(低字节\)开始，截取7位，0000000，因为前面还有数字，首个二进制位补0，第一字节就是：

> 10000000

接着再截取出7位 0000010，因为它的前面没有数字了，首个二进制位：补0，就是：

> 00000010

合并两个字节

> 10000000 00000010

核心原理：将int类型的数字的二进制值， 从二右侧\(低字节\)开始，每次拿到7个二进制位，开始计算，添加msb位\(标识位/占位符\)：

1：前面还有数字\(1\)

0：没有

这样，一个字节有7个二进制位表示数字，最高位一个占位符，循环

分析：数字越小占的字节越少，如127以内，一个字节就够了。如果数字非常大的时候，原本4个字节表示一个int变成了5个字节，因为：每个字节即少了一个二进制位

现在考虑两种情况：\-1 和 2147483647

> 10000000 00000000 00000000 00000000
> 
> 
> 11111111 11111111 11111111 11111111

当为负数或者数字非常大的时候，还是以7位划分成一组，每个字节首位再加上占用符，就变成了：32位 \+ 4位 ，貌似，都是5个字节了...反倒是浪费空间了...

> 日常能直接把数字用到上限：2147483647，很少，基本用不到，那么varint这种压缩更好\(处理正数\)，传输性能也就略高些。
> 
> 
> 但是负数肯定不行，任何一个负数，最最怕小的\-1，也有问题

处理负数，使用的是：ZigZag算法，这里不做详细讲解了......简单说下：

引入：sint32 和 sint64 类型，来表示负数。先采用 Zigzag 编码，将有符号的数转成无符号的数，在采用 Varint 编码，从而减少编码后字节数

varint 缺点：虽然传输快了，但两端的压缩/反压缩会吃些性能。

> 个人觉得，这种计算量，按照现代的计算机配件配置，微乎其微，反倒是网络传输的开销才更影响性能、稳定性。
> 
> 
> 但游戏开发除外，大量的序列化/反序列化，控制不好内存，容易引起垃圾回收或内存不够

#### TLV

介绍一下 字符串类型

```
message Test2 {
  optional string b = 2;
}
```

设置:b = testing

编译后的最终结果：

```
12 07 74 65 73 74 69 6e 67
```

> 一堆16进制的数

1. tag=12, \-\> 0001 0010 ,前5位0001 0 ，为2，代表第2个字段\(field number\), 后三位 010 为 wire type = 2
2. length=07,内容总长度为7个字节
3. value=74 65 73 74 69 6e 67 ,testing的asic码

#### demo

上面讲的还是太抽象，举个实际点的例子：

JSON

> {"id":42}，9 bytes

xml

> \<id\>42\</id\>，11 bytes

protobuf

> 0x08 0x2A，2 bytes
> 
> 
> 0x08 = field 1, type ：Variant
> 
> 
> 0x2A = 42 \(raw\) or 21 \(zigzag\)

#### 总结

优点：

1. 传输性能较好，因为压缩了
2. 加密。因为压缩成二进制文件，没有 .proto 描述文件，不可还原，间接实现了加密功能
3. 跨语言，各组、各部分之间通信时，只需要有一份 .proto 描述文件，各自是什么语言就都不重要了

缺点：

1. 增加了复杂度，原本通信双方就正常定义个格式即可，现在还需要：
    1. 多出一个概念： .proto 描述文件
    2. 得下载proto buf 编译器\(熟悉编译器的使用、及参数\)
    3. 得编译 .proto 描述文件
    4. 不同语言可能还得再下载一个插件，如：PHP、golang就得单独下载
    5. 编译器有版本之分，如果描述文件是v=2，你用protobuf3的编译器，可能会出错。
2. 调试麻烦，原本都是字符符明文传输，直接能看结果，而现在都是一堆16进制码，人眼看不出问题，反序列化时就算出错，也不好调试
3. 虽然压缩后提高了传输性能，但是增加了两端的计算量：序列化/反序列化，如果使用不好，造成垃圾回收，挺可怕的。
4. 容易走入扩展的误区，因为，描述文件的产生，大家把更多的功能也都想集成进来，如统一数据结构：空结构体、msg结构体\(code\+message\+errMsg\)、枚举常量，然后就开始DIY魔改原protobuf的编译器，造成公司自有一套protbuf编译器，一但protobuf编译器升级，自有的这套基本上不兼容了。

使用场景：长连接、游戏

> 反正我目前只在游戏领域使用过，因为游戏对实时性要求较高，但讽刺的是：如果访问量大且操作非常频繁，对两端的计算量~稳定性也有要求，protobuf反而满足不了，得自己写一套。

具说一些大型公司，微服务的场景也使用.proto文件：用于多语言统一通信数据定义。也就是面向.proto做通信编程，但我没大批量的使用过，仅在小项目使用。

## 使用

先下载 protobuf 编译软件\(根据自己的OS自行选择\)

> wget https://github.com/protocolbuffers/protobuf/releases/download/v3.15.8/protoc\-3.15.8\-osx\-x86\_64.zip

> GOLANG 和protobuf 都是google公司出品，按说一个包应该就够了，然而，还得再下载个插件。其：将.proto文件用上面的工作转换成对应的文件后，还得再用插件转换成GO语言的，可能的解释的就是：

1. probuf存在的时候压根没有GO语言
2. 两个东西不是一个组开发的
3. 模块分割清楚，protobuf就是帮助做最基础的解析，具体使用者是什么语言，再单独安装

> 这里以GOLANG为使用语言，其它语言自行查阅文档吧

安装 golang 的 protobuf 编译器插件：

> go install github.com/golang/protobuf/protoc\-gen\-go
> 
> 
> go install github.com/golang/protobuf/protoc\-gen\-go@latest

这两个包不确定是干什么的：

> google.golang.org/protobuf/reflect/protoreflect@v1.26.0
> 
> 
> go install google.golang.org/grpc/cmd/protoc\-gen\-go\-grpc@latest

这里注意，正常安装 GRPC PROTOBUF 的包是问题，但是ETCD与GRPC冲突，不能使用最新的GRPC，所以，如果想一起使用，就得降低 protoc\-gen\-go 版本

> go install github.com/golang/protobuf/protoc\-gen\-go@v1.2.0

好像连带着 库都给下了，先不管了，先找一下包下到哪里了

> go env

看下gopath

可执行文件被搞到这里了

> ~/go/bin \-\> /var/root/go/bin
> 
> 
> 类包在这里
> 
> 
> /var/root/go/pkg/mod/google.golang.org

OK，指令行工具完成后，开始下载官方的类包

> go get github.com/golang/protobuf/proto

上面这步可以省略了

> mv /Users/wangdongyan/Downloads/protoc\-3.15.8\-osx\-x86\_64 ~/go/bin

执行protoc 会提示找不到 protoc\-gen\-go

> export PATH "$PATH:/var/root/go/bin"

或

> export PATH=$PATH:/var/root/go/bin

## 测试demo

> touch RequestLogin.proto
> 
> 
> ./protoc \-\-go\_out=. RequestLogin.proto
> 
> 
> ls一下，RequestLogin.pb.go 文件就在当前目录生成了

截取一段代码分析吧

```
syntax = "proto3";

package myproto;
option go_package ="./;myproto";

message RequestClientPong{
    required    int32               add_time    = 1;
    optional    string              cc          = 2;
    repeated    Player              player_list = 3;
    optional    Player              player      = 4;
    required    map<int32,CfgActions> rtt_times = 5;
}

message Player{
    int64       id            = 1;
    bytes       role_id       = 2;
    string      nickname      = 3;
    fixed32       status        = 4;
    float       add_time      = 5;
    PhoneType   phone_type    = 6 [default=HOME];
    sex         bool        = 7;
}

enum PhoneType {
    //allow_alias = true;
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
}

```

1. syntax: 版本号，现在大部分都是3，旧的是2，关于两个版本区别下面讲
2. package: 该文件定义成一个包，包名，像对全名空间不感冒的PHP 可以忽略它
3. option go\_package: 属于哪个目录/输出路径，现在的版本：不写就报错

下面是核心，定义一个通信协议内容格式：

```
message 协议名{
    描述类型 字段类型 字段名 字段顺序
}
```

#### 描述类型:

1. required:必须有值，不能为空
2. optional:可有值可为空
3. repeated:用于定义数组，或引用其它自定义的结构类型
4. enum:枚举，可以嵌套的写，但，大多数都是单独拿到外层写

> proto3已经删除了:required和optional，默认都是：optional。

#### 字段类型

|关键字          |包含类型                                |说明    |注意                                |
|----------------|----------------------------------------|--------|------------------------------------|
|int             |int32 int64 uint32 uint64 sint32 sint64 |数字    |sint32 sint64，对负数优化较好       |
|fixed           |fixed32 fixed64 sfixed32 sfixed64       |定长数字|速度较好                            |
|bytes           |                                        |字节    |                                    |
|bool            |                                        |布尔    |                                    |
|string          |                                        |字符串  |                                    |
|float           |                                        |单精度  |                                    |
|double          |                                        |双精度  |                                    |
|map\<type,type\>|int fixed bytes bool string float double|字典    |它其实是复杂类型：包含上面所有的类型|
|any             |以上全部                                |泛型    |当不确定什么数据类型时，使用此值    |

> varint的优点是动态压缩，数字是动态的长度，好处是传输快，但是两端要消耗计算量，要没特别要求，用fixed感觉更好些。
> 
> 
> any这个东西个人觉得还是小心使用，不仅仅是性能问题，它在每个语言的处理上也略有差异。

#### 字段名

正常起个名字而以，不重复，有一定说明的意义即可

#### 字段顺序

对字段标识一下，顺序号，主要是序列化与反序列化时，filed\_number使用

## JS

先来试试

> protoc \-\-js\_out=. api.proto

TMD，真简单，直接成功...就是有点缺点：生成了一堆类文件，而不是一个PB文件

具说，生成的文件NODEJS 直接可以用，无奈我是浏览器版的JS，接着找文档

查了查，好像挺麻烦，大致得下载几个依赖包：browserify、google\-protobuf，但都NPM的教程，所以还得下个NPM，又一查，直接安个NODEJS编译器，自带NPM

于是NODE官方，直接下载MAC包，下一步下一步完成。

node安装位置

> /usr/local/bin

npm下载包的位置

> /usr/local/lib/node\_modules

```
npm install -g require
npm install -g browserify
npm install -g google-protobuf
```

> source /etc/profile

> cd /data/www/golang/src/frame\_sync/myproto

重新编译下

> protoc \-\-js\_out=import\_style=commonjs,binary:. api.proto

这回好了，不是一堆JS文件了，只有一个PB.JS 文件了，NICE\!\!\!

> browserify exports.js \> api\_web\_pb.js
> 
> 
> mv api\_web\_pb.js ../www/

> protoc \-\-go\_out=. api.proto
> 
> 
> mv api.pb.go ../
