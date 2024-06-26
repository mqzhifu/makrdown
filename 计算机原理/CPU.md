
# 厂商

|公司          |国家          |指令集       |应用市场                            |芯片                 |备注                                                                          |
|--------------|--------------|-------------|------------------------------------|---------------------|------------------------------------------------------------------------------|
|intel         |美国          |X86 X64/amd64|PC/服务器                           |设计/生产            |龙头企业，霸占PC/服务器 市场，只可惜大公司病，在手机端的CPU市场马失前蹄       |
|amd           |美国          |X86 X64/amd64|PC/服务器                           |设计/生产            |没啥存在感，捡点 intel 的市场，如：价格便宜的CPU。反而这两年在显卡市场有点作为|
|arm           |英国/日本/美国|arm/arm64    |PC/嵌入式：手机 pad 车机 路由器     |设计/少量生产        |它在嵌入式领域的崛起，赶上了 手机 IOT 智能家居行业，彻底火出圈                |
|高通\-骁龙    |美国          |ARM V9       |手机/无线设备                       |设计/台积电代工      |主要是设计CPU，由台积电代工。但在 GPU 无线领域 的设计还是很厉害               |
|联发科\-天玑  |中国          |ARM V9       |手机                                |设计/台积电代工      |                                                                              |
|华为\-海思麒麟|中国          |ARM V8       |手机 pad PC 笔记本 电视 智能家居 Iot|设计/台积电代工      |                                                                              |
|苹果          |中国          |ARM V8       |手机 pad PC 笔记本                  |设计/台积电/三星 代工|                                                                              |



# 参数：


| 参数名 | 解释 | 备注 |
| :--- | :--- | :--- |
| 厂商 | AMD INTEL | 参数上看AMD性价比更高，但 如果不是非常熟悉AMD,建议intel。 |
| 核数 | 大核心数+小核心数 | 不要看一共多少核，要看大核心的数量 |
| 线程数 | 超线程技术 | 单核CPU上模拟多线程，能提高一点CPU性能 |
| 主频 | CPU内部每秒定时器的频率 | 参数越高性能越高 |
| 外频 | CPU内部的北桥与外部通达信的频率 | 决定了内存总结的频率 |
| 功耗 | 使用电量的功耗 | 决定了主板 |
| 缓存 | 几级缓存，每级缓存大小 | 确实能提高CPU速度，最好是3级，容量大一些 |
| 核显 | CPU自带显卡，可做图形渲染 | 只要是有显卡，这东西就是鸡肋 |
| 超频 | 发挥CPU的超过100%的性能 | 榨取CPU性能。鸡肋，小白基本就别碰了 |
| 第几代构架 | 不同时代的构架，性能差距也不小 | 目前最新是14代，不同时代主板型号也不一样 |


# 作用

主要作用：读取 内存 中的（指令+数据），解码指令，执行运算，将（数据）结果输出：内存、CPU内部使用、硬盘、显示器
其它作用：读取 外设 中的指令+数据，解码指令，执行运算，将结果输出：内存、内部使用、显示器

>简单说就是控制整个计算机软/硬件系统
# 计算流程

计数器 -> 内存地址寄存器 -> 内存缓冲区寄存器 -> 指令寄存器  ->
控制单元 -> 累加寄存器 -> alu运算器（逻辑运算、数学运算）


CPU与内存的连接  CPU和内存与主板的连接
主板会提供：CPU插槽 、CPU与内存连接的线
CPU核心的外部/底部会有插针（现在是触点），针角落到主板的插槽上~每部分针角都其特殊的意义。 其中一部分是总线，分成3类：地址总线、数据总结、控制总结。    

8根针：是数据总线
8根针：是数据总线
2根针：是控制总线

# 频率


主频、外频、倍频、睿频
>这里最重要其实是主频，一般说的CPU频率大多也是指主频


### 主频

电脉冲信号震荡周期，每秒多少次电脉冲，也叫时钟
目前的单位是 hz ，速度是:GB
>如：3G hz/秒   ，每秒有30亿次时钟跳动。 一次周期的时间 = 30亿 / 1秒 = 3000000000 / 10000000000 (1秒)    = 0.3纳秒 

CPU制定一个周期时间，主板、内存等与CPU看齐，这样一个主机就是一个整体，以这个周期为单位，进行IO操作
>换个角度也可以看成是主频是机器的时间，只不过人类的时间是：时分秒

一个时钟内：可能执行多条指令，也可能是一条指令，也可能是多个时钟才能执行一条指令。
>具体得看当前CPU的构架、外部组件(内存、硬盘、主板)频率

所以，主频是一个重要的参考信息，但不能绝对。完全有可能，几年前的CPU主频非常高，但是可能还不如现在的低主频的CPU，毕竟现在的CPU构架更新，技术更高，工艺也更好。还有就是也得看内存硬盘等等


### 倍频

主频与外频之比的倍数，是个参考概念。早期的CPU并没有倍频这个概念，那时主频和系统总线的速度是一样的。随着技术的发展，CPU速度越来越快，内存、硬盘等配件逐渐跟不上CPU的速度了，而倍频的出现解决了这个问题，它可使内存等部件仍然工作在相对较低的系统总线频率下，而CPU的主频可以通过倍频来无限提升(理论上)。


### 睿频

一项加速技术，不针对所有的CPU。英特尔在最新酷睿i系列CPU中加入的新技术，以往CPU的主频是出厂之前被设定好的，不可以随意改变。而有睿频的CPU可以自动调节频率


### 周期


指令周期：读指令、解指令、执行指令、结果回写

机器周期：对指令周期的每一步进行具体执行，如：读/写数据

时钟周期：计算机最小的单位，比如：获取内存地址，开始调取内存数据

指令周期 > 机器周期 -> 时钟周期
# 缓存

CPU是一个整体，其内部又包含若干组件。但它得靠内存提供数据。但如果CPU从外部的内存取数据，和CPU本身处理数据，相差是100倍。CPU太快了。这个时候，如果能把一些常用的内存数据预告读取到缓存。速度是能提升的。

整个CPU会有一个L3缓存
每个单核内会有：L1 + L2 两个缓存

从距离CPU的远近来看：
L1最近
L2中等
L3是公共缓存，距离每个核心是最远的，速度肯定也是3个里面最慢的。但是容量是最大的

##### 大小:
>这里以13490F为例子：

L1：6 * 32KB( 代码) + 6 * 48KB(数据)
L2：6 * 1.35mb + 2MB
L3：24MB

因为每核都有L1 L2  ，如果某一核改变了缓存内容，另外的核也得更改。那么就得有一致性协议了，好像是每个厂商都有各自的协议，主流有一个：mesi 协议

##### 速度：

>这里以13490F + 32G 3200频率 为例子

![[test_cpu_memory.png]]

L1 > L2 > L3 > 内存

L1的IO速度是内存的100倍
L2的IO速度是内存的40倍
L3的IO速度是内存的27倍

延迟差不多也是几十倍

##### IO顺序:

L1 > L2 > L3 > 内存

##### 读取数据
即使一次只读一个字节，但是缓存一次会读64个bytes（多读取数据），也叫：cache line
>这里也得看CPU厂商，有的cache line 是128 bytes

# 内部物理结构

##### 表面
顶部：铝白白色外壳（保护芯片/散热）
中间：绿色主板（这上面有N多线，连接主板和芯片）+ 芯片
底部：针脚/导线

最中心的位置就是芯片，最核心的东西：它的背部也有个壳子（保护芯片/散热）。大概也就是指甲盖（姆指）大小 。

###### 芯片的组成

计算核心 + L3缓存 + graphics  + memory控制器  + PCI-E控制器 + GNA + display  + PCIE4 + RNG + imaging 

GNA:
Intel GNA(Intel Gaussian & Neural Accelerato): 英特尔高斯和神经加速器。这是一个AI 加速IP，简单的说在AI 计算中会有一些常见的算法，如果用 CPU 进行会占用大量的 CPU资源，于是 Intel 将这部分分离出来专门定制了一个IP，如果有这方面的需求那么直接将数据丢给这个IP进行处理能够节省CPU资源，特别体现在省电上。

这个东西大部分主板 BIOS 默认是关闭的，且得单独下驱动。用的人反映其实没什么太大的效果。

也有叫：协处理器的。为CPU辅助提供一些算法。

它官方宣传功能：
1. 降低CPU运算减少电池开销 
2. 优化照片(降噪) + 实时音频转录

RNG：我没找到相关信息，但查到这个   生成随机数芯片



##### 计算核心

控制器、运算器，存储单元

控制器：主要是从内存中读取指令和数据，并根据自己的指令集~解码成CPU自己的可执行的指令
1. instruction Register：保存当前正在执行的指令。当执行一条指令时，先把他从内存取到缓冲寄存器中，然后在传送到指令寄存器。
2. Instruction Decoder：解析从内存拿到的指令，转换成CPU自己的指令
3. Operation Controller：用来产生各种操作控制信号
4. 时序发生器：也叫时钟，按照一个非常规律的时候跳动。也可以理解为：一次指令的从读取到执行完毕后的周期。
5. 程序计数器：
6. 指令queue：缓存指令队列
7. 地址总线、数据总线、控制总线



运算器
1.  alu 算术逻辑单元：逻辑运算器 (与或非异或)+  算法运算器(加减乘除)  
2. 累加器
3. 数据缓冲寄存器
4. 状态寄存器
5. 通用寄存器
6. 晶体管

存储单元
1. 各种寄存器
2.  L1/L2缓存

>它们之间是相互访问的:  控制器  => 寄存器 <= 运算器 ,   控制器 => 运算器


大体的一次执行过程：
fetch -> decode -> execute -> write back

1. 先把内存中的第一行代码对应的内存地址读取到    instruction point Register(也叫程序计数器) 中，然后，自动加1(具体取决于指令内容占几个字节)，跳到下一条需要执行的指令的地址上
2. Decoder  指令：把 指令寄存器中的值（内存地址）读取出来，向总线发出请示，拿到这行代码的地址的对应的数据，开始 decode  <- 指令集  ，确定是哪种操作
3. 开始运算
4. 结果，写回内存/寄存器 或 什么也不做

decode：有多个 decode  模块，并行解析从内存来的指令
execute：有多个（超标量执行） alu 并行计算CPU指令

ISA instruction set : 软件与硬件的中介。定义了：内存位数、数据的大小、指令类型

看一段代码，详细的执行过程 ：
```汇编
push rbp
mov rbp , rsp
......
call 32 <add+0x1d>
```


压入 rbp 地址
设置 rsp  地址
之后是一系列的计算
最后，返回栈顶

>rbp:register base pointer   栈底指针 寄存器（32位也有叫EBP）
>rsp:register stack pointer   栈顶指针 寄存器（32位也有叫ESP）

从这个流程分析：rip不调的向下运动(CPU自己控制)，来更新内存地址，也就是 CPU具体要执行的内存中的指令内容及执行顺序

从内存中，划分出一小段连续的空间，存储执行指令，CPU从这段区域读取指令，并通过 bp sp 来回游走执行代码

再从内存中，划分出一段较大的连续的空间，存储一些较大的指令或数据，继续提供给CPU调取与执行

>貌似，如果更改这3个寄存器的值，就间接等于 切换进程.....

##### 寄存器

本质就是临时的存储器，存放CPU经常使用的一些数据。

通用寄存器

|64-bit|32-bit|16-bit|8-bit (low)|
|---|---|---|---|
|RAX|EAX|AX|AL|
|RBX|EBX|BX|BL|
|RCX|ECX|CX|CL|
|RDX|EDX|DX|DL|
|RSI|ESI|SI|SIL|
|RDI|EDI|DI|DIL|
|RBP|EBP|BP|BPL|
|RSP|ESP|SP|SPL|
|R8|R8D|R8W|R8B|
|R9|R9D|R9W|R9B|
|R10|R10D|R10W|R10B|
|R11|R11D|R11W|R11B|
|R12|R12D|R12W|R12B|
|R13|R13D|R13W|R13B|
|R14|R14D|R14W|R14B|
|R15|R15D|R15W|R15B|

- rax：通常用于存储函数调用返回值
- rsp：栈顶指针，指向栈的顶部
- rdi：第一个入参
- rsi：第二个入参
- rdx：第三个入参
- rcx：第四个入参
- r8：第五个入参
- r9：第六个入参
- rbx：数据存储，遵循Callee Save原则
- rbp：数据存储，遵循Callee Save原则
- r12~r15：数据存储，遵循Callee Save原则
- r10~r11：数据存储，遵循Caller Save原则


特殊寄存器

指针寄存器(instruction pointer, RIP.)  
状态寄存器(status register, RFLAGS)

段寄存器 (Segment registers)：


|名称|全称|功能|
|---|---|---|
|cs |代码段寄存器(Code Segment)|代码段基址|
|ds |数据段寄存器(Data Segment)|默认数据段基址|
|ss |堆栈段寄存器(Stack Segment)|堆栈段基址|
|es |扩展段寄存器(Extra Segment)|扩展段基址|
|fs |扩展段寄存器(e 后面是 f，按顺序排列，没有语义)|扩展段基址|
|gs |扩展段寄存器(f 后面是 g，按顺序排列，没有语义)|扩展段基址|

#### CPU\-指令集架构

x86：intel 最先研发，32位总线，后期 amd 跟进。

x64：以X86基础之上研发，64位总线。intel 先搞出来一般，但兼容不，随后，amd研发成功。intel跟进。所以x64 有时候也叫 amd64

arm64：：CPU架构，指令集，对比 x86/x64 它更精简，指令少了很多。对硬件要求略低，但对硬件的兼容一般。所以它的领域不在PC，而是嵌入式。拥抱 linux。

x86/x64 对硬件兼容也好，同时对硬件要求\(功耗\)略高，用于：PC/服务器/笔记本电脑


# 总线

总线：所有的组件，是统一配合工作，需要传输数据，需要有线缆进行传输。
>cpu <=> 内存 <=> 硬盘 <=> 外设

##### 工作任务分类

数据总线
CPU <==> 外设/内存        (双向传输数据的)
常件的接口就是：PCI-E  PCI

地址产品线
CPU  ==> 外设/内存  （只能是CPU单向，向外部传输）

##### 控制总线
CPU <==> 外设/内存        (双向传输数据控制指令)



传输数据分类：并行、串行
并行：多根总结，并行的传输数据，内存就是，PCI 接口就是

串行：就是一根总线，适合离CPU远的的外设，进行数据传输。如：SATA PCI-E USB 等

##### 总线数量
单总线结构：所有设备都连接一条线上，进行传输数据，缺点就是：单时操作，某个设备有IO，其它设备就得等。
双总线结构：所有外设与CPU有一条总线，CPU与内存单独有一条总线。缺点就是：其它设备不能直接访问内存，得通过CPU中转
三条总线：在双总线的基础上，增加一条IO总线，外设也可以直接访问内存

##### 总线的工作方式：
CPU 要访问内存某个地址的数据，先要在地址总线上发出地址，接着在控制总线上告诉是:IO，然后再通过数据总线接收/输出数据


# PCI/PCIe

电脑的所有部件：CPU/内存/显卡/外设等，最终是要互相连接，互相协调工作的。但，不同的部件有不同的功能，接口也各不相同，即使同为CPU可能接口、总线都不太一样。主要是，各大厂商的对自己的产品规范都不同。所以，兼容问题很大。1981年左右，IBM出了一个相对统一的ISA总线接口规范。之后，IBM又搞 了3个协议，但好像出了些问题。

>ISA Industry Standard Architecture

1991 年下半年，Intel 公司，并联合IBM、Compaq、AST、HP、DEC 等100 多家公司成立了PCI 集团，并且Intel公司首先提出了PCI总线的概念，后由PCISIG (PCI Special Interest Group)整理后，于1993年推出了PC局部总线标准——PCI总线





# PCI
Peripheral Component Interconnect  ：外围设备组件互联


![[PCI总线结构图.jpg]]


南桥芯片：主要连接CPU 显卡 内存 南桥交互数据，比较重要且传输速度较高的设备
北桥芯片：与南桥相连，主要连接 普通外设，传输速度不是特别快，且不是最重要的设备。如：硬盘、USB、网卡、键盘、鼠标等

CPU如果要访问显卡或内存，需要通过北桥中转
CPU如果要操作外围设备，要先通过北桥，北桥面再访问南桥，最后到外围设备

另外，南北桥中间还有一个总线：PCI LOCAL BUS，这上面也可以挂外设，网卡~



# PCIE

Peripheral Component Interconnect Express ：外围设备组件互联

![[PCIE总线结构.png]]



PCIe采用的是树形拓扑结构，由：根组件(Root Complex)，交换设备(Switch)，终端设备(Endpoint)等类型的PCIe设备组成。

看图就一根大的总线：ROOT COMPLEX(具说内部极其复杂)。
算是多了一个SWITCH连接外设。对比PCI ，少了那根公共总线。
提出了 LANE 。
>LANE：  两条线路，双向传输，全双工

PCLe 总线：就是部件上传输数据的电路线
PCLe 通道：一条 LANE 就是一个通道，总线的宽度就是最大通道数，x1 x2 x4 
PCLe 插槽：PCLe接口， x1 x4 x16  (虽然有些是x16，但并不代表传输速率就是16通道，也可能缩水)






|  版本 | 行代码 | 传输速率 | x1 | x4 | x8 | x16 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1.0 | 8b/10b | 2.5[GT](https://baike.baidu.com/item/GT/0?fromModule=lemma_inlink)/s | 250[MB](https://baike.baidu.com/item/MB/0?fromModule=lemma_inlink)/s | 1[GB](https://baike.baidu.com/item/GB/0?fromModule=lemma_inlink)/s | 2GB/s | 4GB/s |
| 2.0 | 8b/10b | 5GT/s | 500MB/s | 2GB/s | 4GB/s | 8GB/s |
| 3.0 | 128b/130b | 8GT/s | 984.6MB/s | 3.938GB/s | 7.877GB/s | 15.754GB/s |
| 4.0 | 128b/130b | 16GT/s | 1.969GB/s | 7.877GB/s | 15.754GB/s | 31.508GB/s |
| 5.0 | 128b/130b | 32 or 25GT/s | 3.9 or 3.08GB/s | 15.8 or 12.3GB/s | 31.5 or 24.6GB/s | 63.0 or 49.2GB/s |



>CPU有自己的总线，同时主板也有自己的总线


# CPU与内存通信

因为北桥已经被嵌入到CPU中，其内部的总线控制 CPU基本上自己就能完成。剩下的就是：CPU的北桥 连接 外部总线了（内存总线、显卡总线、SSD总线、南桥总线）

>这里我不确定，北桥现在是一个整体的芯片，还是被打散成若干芯片，就是没有北桥这个东西了

内存总线连接CPU，一共有3根：两根32位（IO数据），一根21位（控制总线）

>这里准确的说应该是：直连CPU内部的内存控制总线


