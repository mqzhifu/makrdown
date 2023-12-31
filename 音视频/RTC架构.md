# RTC架构

## 概览

提到架构，肯定先想到的是后端服务器，RTC 是不是一定需要服务端？

不一定需要服务端，拿 webRtc 来看，如果：

1. 双方可以NAT穿越成功
2. 通信参与者只有4人以下
3. 参与者物理距离不太远且网络稳定
4. 都用浏览器
5. 不需要转码合流、数据存储、录制等

直连 P2P 模式就够用了，就算用到中继器，运维给简单配置一下也即可。

什么场景一定需要服务端：

1. P2P 不可直连，中继器的部署与维护还是需要个后端/运维的
2. 录制、转码、合流
3. P2P 连接双方 物理距离太远\(跨国、跨运营商\)，传输不稳定
4. 直播连麦
5. 各端接入的设备/OS不同，得兼容

架构其实并不一定是只选一种模式，它更多是多种模式的混合使用，先说一下RTC自有的3种架构

## MCU \(MultiPoint Control Unit\)

这是一种传统的中心化架构，每个C端仅与中心的MCU 服务器连接，MCU服务器负责所有的视频编码、转码、解码、混流等复杂逻辑

优点：

1. 可以转码合流
2. 可以做数据分析
3. 可以推流到CDN/旁路服务器
4. 客户端只要一个连接即可
5. 客户端不需要占用太多带宽
6. 客户端CPU负载略小
7. 可以承载很多人一起视频通信

缺点：

1. 延迟高一些

> 因为要拆解包、混流、封包等

1. 服务器成本高很多

> 必须配置高，需要大量的计算

1. 带宽成本高一些
2. 人力开发/维护成本高一些

> 协议拆包/封包 音视频文件 拆包/封包 音视频数据流拆包 合流转码 协议转换

1. 自由度不高，因为单连接必须得S端合流后推一路总流给C端，客户端想针对某一路的流做特殊处理不能实现
2. 单一性，因为计算量本就高：
    1. 容易挂。一但是整个机器挂了，所有服务全挂
    2. 线路越多，合流计算量越大，服务器可承载的人数就越少

> 这个阶段大致停留在电话上网的阶段\(猫\)。www 兴起，网页浏览器，也算是 大量 PC机 正式接入互联网的阶段

总结：

这应该是最早的 RTC 架构了，感觉其核心点就是：尽量减少 C端的 压力 ，所有压力 转交 S端，我猜可能也是因为早期：C端的 带宽小、计算量差，这种模式非常的适合

## Mesh

每个端都与其它端互连，也算是 webRtc 默认\(标准\)的模式，用ICE\(stun\+turn\)模式承载

假设：P2P直连，有3个人参与通信，一共是3个连接，每个客户端就得有2个连接。因为连接是需要双向维护，一共得维护6个socketFD。视频流是双向的，连接也就得是全双工的。每个客户端共计得有4个通道\(自己的流推给B C 两个通道，还得有两个通道接收B C的视频流\)

> 即使是turn模式做中继也一样,中继不做任何逻辑处理，只要有连接就单独做一个流

优点：

1. 快\(人少时\)
2. P2P模式下：服务器、宽带、人力、成本低
3. 项目的复杂度低

缺点：

1. 人数不能太多，不然客户端就爆了
2. 带宽要求，如果流过大，就完蛋
3. 因为没有中转服务器\(中继器不做任何逻辑处理\)，不能转码合流
4. 大部分计算集中在客户端，尤其多人连接推送时，对CPU 可能压力较大
5. 并不是所有的客户端网络都能支持 NAT 穿越
6. turn 模式下，借用中继器转发，服务器 带宽成本略高
7. P2P 这种模式，具说得向相关部门报备，不然算是间接违法

这个阶段的革新的网络技术：

1. ADSL，并衍生了：OICQ/MSN、BT/eMule/迅雷
2. GIPS ，在 PC 上，并没有对声音有较好的处理，GIPS 开发了 3A 算法、NetEQ、高质量语音编码器。

> 具说到2018年，QQ的代码中还有GIPS协议

1. h.264 正式修订，视频流得以在网络上能较好的传输

> skype，它写了很多算法。不过，他更倾向于1V1，普通人基础的聊天

## SFU\(Selective Forwarding Unit\)

跟MCU好象没什么区别，但是思路不同，仍然有中心节点服务器，但是中心节点只负责转发，不做太重的处理

优点

1. 服务器的压力会低很多

> 没有拆包/封包、合流转码

1. 略快一些

> 没有拆包/封包，直接将数据转发。但没有P2P快，比 MCU 快

1. 人力成本略小一点

> 没有 MCU 那么复杂的构架，也能省些人力

1. 可承载的人更多

缺点：

1. 无法合流转码
2. 无法转换协议
3. 无法推流到CDN
4. 如果人数多时，各路的流是直接转发，可能出现不同步的情况
5. 不太好录制

#### SFU Simulcast

同上面差不多，多了一个功能：一次传输多种码率的视频流，如：1080 720 360

优点

1. 解决了 SFU 不能转码的功能

缺点

1. 客户端要更多发两路流

其实，就是在C端多加点去处，减少S端计算。如：C端如果是电脑，计算能力还是够的，他多推一路 分辨率低的流，那么，如果对端是手机，直接拉流分辨率低的，交互更流畅些

#### SFU SVC

同上面差不多，多了一个功能：动态码率传流，即：网速好 推拉流 360 ，中等就是720，较好就是1080

将原视频流，分成了3个级别：核心层\(360p\)、中间层\(720p\)、顶层\(1080p\)

优点

1. 解决了 SFU 不能转码的功能
2. 针对不同带宽处理不同的流
    缺点
3. 客户端得做一些动态计算
4. SFU 也要加一些简单代码，做策略，如：监测客户端的网速
5. 实施起来要比 SFU / sFU Simulcast 更复杂一点

这个阶段产生了：YY，直播行业、订阅/发布 推拉流模式

## 架构模式的发展趋势\(历史 \)

MCU \(1996~2004\) \-\> MESH \(2001~2007\) \-\> SFU \(2003~2012\) \-\> 云SFU阶段\(2010~2017\) \-\> 全球实时云通信阶段\(2016至今\)

其中 P2P 阶段 比较重要的转折因素:

1. 带宽提升

> 不仅仅是线路的变大，还有：由传统的 电话线\+猫 转化成了 ADSL\)

1. PC机器的硬件提升

> 不仅仅是提升了，但是价格返回便宜了

1. 编码H.264的产生
2. 网络传输协议的升级

> stun turn VoIp

1. 音视频流的质量控制，如：3A 解决了回声等问题，NETEQ解决了网络抖动造成的卡顿、丢包

RTC的黄金期就是从 P2P这个阶段开始，比较典型的就是：skype 公司的产生

SFU 阶段其实是比较宽泛的，其还可以分解成：

1. 单SFU
2. 云SFU
3. 全球SFU

并且，随着进入黄金期，各种民用业务也可以使用后，又连带其它技术的进步：

1. 声音的优化
2. 视频的优化，如：美颜、虚拟背影

即使这样，依然还是有些问题，如：物理距离\(传输慢\)、移动互联网的崛起（网络不确定性多）

最终，再加上云计算，完全弱化客户端要求，直接用云服务器把网络问题解决

最后，再把这两年兴起的 AI 加进去，RTC 就真是有那么点 如日中天的感觉了。。。。。

从研究微服务，发现是N年前 由 netflix 发明。到现在查 RTC 最终发现的：skype软件\+adobe的研发的rtmp协议，还有电话时代的发展等等吧，我们只能说是站在巨人的肩膀上。并且，所有的底层技术都是欧美的......

## 3种模式对比

![RTC-构架.png](image/RTC-构架.png)

1. mesh: P2P 这种模式限制太大，如：NAT 不可穿越、无法转码合法、公司无法拿到数据流做分析等等吧。turn模式做中继器，服务器、宽带成本依然很高。

> 属于多连接，即有增加一个C端就多一路连接

1. MCU : 这种模式兼容性/扩展性最强、最成熟稳定，但：自研投入也是最高的，另外具说现在就有MCU类似的硬件，不过很贵。感觉没一定技术实力轻易也不太会

> 属于单连接，不管多少人加入，C端只要一个连接就到S端就够了，S端更像是一个:大的音视频处理器

1. SFU : 这种算是新的策略，它介于mesh mcu 之间，缺点也是介于两者之间，是个比较好的选择

> 推流就一个连接，收流还是多个连接

假如5个人参与会议，对于C端来说，如下表格：

|构架||连接数|流的通道数|S端压力                          |C端压力|延迟|适合人数\(单房间\)                 |C端自由度|
|----||------|----------|---------------------------------|-------|----|-----------------------------------|---------|
|mesh||4     |8         |轻\(都是P2P\)，略重\(都是中继器\)|重     |小  |超过4人即不稳定，尤其手机端计算较差|较好     |
|MCU ||1     |2         |很重                             |无感知 |大  |50人以下，超过50人S端抗不住        |不好     |
|SFU ||5     |5         |中等                             |中等   |中等|20人左右，超过20人C端抗不住        |中等     |

## 3种模式 总结

1. 正常中小公司是基本不会用这3种构架，大概率不需要任何架构，因为各方面的成本都很高，直接买3方服务最给力了.....
2. 如果真的自研，MFU是一劳主逸的方案，也是最复杂的，成本也是最高的，从开发者的角度看学的东西更多，从公司的角度看不太划算。可能SFU更适合一些，也更主流

> 具说声网的就是 SFU 模式

1. 没有绝对完美的构架方案，甚至可能是每种方案都结合着使用，不可死记硬背

## 云 RTC 架构

上面3种都是 RTC 的一些主构架体系，而实际如果大的构架，且还在云上的话，还有其它组件配合组成的。如：

1. 网关
2. 路由
3. 边缘节点
4. MCU\+MESH\+SFU模式共同构架
5. 大数据
6. 大监控，各种事件/数据上报，且能够做出预警
7. 容器编排与部署
8. 旁路服务
9. 录制服务
10. 媒体服务
11. 信令服务

传统RTC：就是音视频流通信，双方安装一个 app\(native\)，打开后即可音视频通话，1V1/NVN 模式，并没有直播、CDN、推拉流这些功能

智能手机的加入，把原本 PC模式的 C端强计算能力，再次拉回 C端弱计算能力、弱网的时代

这个阶段产生的：声网、网易、ZOOM

## S端\-开源软件

|名称     |实现语言                        |MCU   |SFU |协议兼容                          |OS兼容                                                               |优点  |缺点                              |使用公司   |
|---------|--------------------------------|------|----|----------------------------------|---------------------------------------------------------------------|------|----------------------------------|-----------|
|mediasoup|数据层C\+\+ 应用层 NODE.JS      |不支持|支持|仅支持WebRTC                      |                                                                     |      |推出时间不长                      |           |
|licode   |媒体通信 C\+\+ 信令/房间 Node.js|支持  |支持|                                  |只支持 Ubuntu 14.04                                                  |      |推出时间较早，较重，性能略差一点点|Intel      |
|srs4     |C\+\+ or golang                 |      |    |                                  |                                                                     |      |                                  |阿里云 GRTN|
|Janus    |C语言                           |支持  |支持|仅支持WebRTC                      |支持 Linux/MacOS，不支持WIN                                          |      |架构太复杂，不适合初学者          |           |
|pion     |golang                          |      |    |仅支持 webrtc                     |                                                                     |      |                                  |           |
|Kurento  |c\+\+ or java                   |支持  |支持|                                  |老牌的RTC，它被 Twilio 收购了，核心也走了，貌似它们也在使用 mediasoup|      |                                  |           |
|Medooze  |C\+\+ NODEJS                    |      |    |与Mediasoup类似 但功能强，如：录制|                                                                     |      |                                  |           |
|Jitsi    |应用层 java 底层C\+\+           |不支持|支持|官方提供安卓/web/ios SDK          |                                                                     |很庞大|                                  |           |

https://github.com/ossrs/srs

## 动手\-简单实现

开源软件肯定是最简单的，但是纯黑盒模式~想自己动手搞搞，可以考虑用webRtc P2P模式

安装一个 coturn \+ 信令系统 ，客户端安卓应该是直接有webRtc的接口，浏览器端是直接兼容的

## RTC 与 元宇宙/AR的关系

元宇宙的最终形态可能就是在电子世界中，完全虚拟出一个真实世界的场景

> 这句话可能不对，是个人的理解

这个真实的场景肯定不是一个人的世界\(副本\)，肯定是人类一起共有的世界。也就是：这个世界的所有事物都会动、会变化，还得能交互。那么就得有连网，只要连网且有互动，就得有数据的传输，那么：RTC 就是元宇宙实现的必要的一门技术之一。

元宇宙的7个基本要素中包含一个社会沉浸感，其实现的核心之一就是：终端设备 AR/MR。

> 先不纠结7个要素是什么，另外：不光有AR，可能还需要其它终端设备，如：跑步机、枪、头盔等等，这里不纠结这些，只讨论 AR

那么，这里假设把 AR 比喻成元宇宙中，人类的眼睛，那么，眼睛所能看到的数据流，就是 RTC ~~~

## 待解决

OBS推流

