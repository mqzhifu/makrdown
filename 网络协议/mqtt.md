# mqtt

# mqtt 协议

|协议号|KEY        |说明                          |
|------|-----------|------------------------------|
|0     |Reserved   |保留                          |
|1     |CONNECT    |请求连接                      |
|2     |CONNACK    |请求应答                      |
|3     |PUBLISH    |发布消息                      |
|4     |PUBACK     |\(qos=1\)发布应答             |
|5     |PUBREC     |\(qos=2\)发布已接收，保证传递1|
|6     |PUBREL     |\(qos=2\)发布释放，保证传递2  |
|7     |PUBCOMP    |\(qos=2\)发布完成，保证传递3  |
|8     |SUBSCRIBE  |订阅请求                      |
|9     |SUBACK     |订阅应答                      |
|10    |UNSUBSCRIBE|取消订阅                      |
|11    |UNSUBACK   |取消订阅应答                  |
|12    |PINGREQ    |ping请求                      |
|13    |PINGRESP   |ping响应                      |
|14    |DISCONNECT |断开连接                      |
|15    |Reserved   |保留                          |

以上是，MQTT的所有协议号，代码级别可控制的是1\-14，0和15是保留

## 协议分析

一个MQTT数据包由：固定头（Fixed header）、可变头（Variable header）、消息体（payload）三部分构成

```
+----------------------------+
|      固 定 头 部 (必 需 )    |
+----------------------------+
|     可 变 头 部 (非 必 需)   |
+----------------------------+
|        载 荷 (非 必 需 )     |
+----------------------------+
```

> 固定头（Fixed header），存在于所有MQTT数据包中，表示数据包类型及数据包的分组类标识。

> 可变头（Variable header），存在于部分MQTT数据包中，数据包类型决定了可变头是否存在及其具体内容。

> 消息体（Payload），存在于部分MQTT数据包中，表示客户端收到的具体内容。

## 固定头

长度2\-5字节\(变长\)

Byte 1

报文类型\+标识

> bits 0:retain
> 
> 
> bits 1\-2:qos level
> 
> 
> bits 3:dup flag
> 
> 
> bits 4\-7：报文类型，messageType，也就是消息类型，0\-15

Byte 2

数据长度（变长头\+消息体）,remaining length

> 最多4个字节，每个字节7位是正常数，保留一位标识是否连接后面字节。

ps:单字节最大可表示的数：2^7，4个字节是2^28，最多可表示的长度为256Mb左右。也就是说消息大小，最大值肯定不会超过256M

```

+---------------------------------------------------------+
|   bit   |  7  |  6  | 5   |  4  |  3  |  2  |  1  |  0  |
+---------------------------------------------------------+
|  byte1  |      Packet type      |         Flags         |
+---------------------------------------------------------+
| byte2...|              Remaining Length                 |
+---------------------------------------------------------+

```

结合看，前2个字节必须得有，固定头至少得2个字节，至多是5个字节

## 变长头

这个比较复杂，不同的协议号，变长头都不太一样,这里就不做一一复述了，挑几个重点吧。

有些协议号是需要有packet id 、payload ，有些是不需要的。

payload

消息体，这个就不做复述了，下表：

|Control Packet|Payload |
|--------------|--------|
|CONNECT       |Required|
|CONNACK       |None    |
|PUBLISH       |Optional|
|PUBACK        |None    |
|PUBREC        |None    |
|PUBREL        |None    |
|PUBCOMP       |None    |
|SUBSCRIBE     |Required|
|SUBACK        |Required|
|UNSUBSCRIBE   |Required|
|UNSUBACK      |None    |
|PINGREQ       |None    |
|PINGRESP      |None    |
|DISCONNECT    |None    |

packet id

2字节，最大10进制数 ：65536

|Control Packet|Packet Identifier field|
|--------------|-----------------------|
|CONNECT       |NO                     |
|CONNACK       |NO                     |
|PUBLISH       |YES \(If QoS \> 0\)    |
|PUBACK        |YES                    |
|PUBREC        |YES                    |
|PUBREL        |YES                    |
|PUBCOMP       |YES                    |
|SUBSCRIBE     |YES                    |
|SUBACK        |YES                    |
|UNSUBSCRIBE   |YES                    |
|UNSUBACK      |YES                    |
|PINGREQ       |NO                     |
|PINGRESP      |NO                     |
|DISCONNECT    |NO                     |

这个ID，个人感觉意义不太大，因为：每个clientId，每次启动的时候，会都会被初始化为1，那就是A跟B的pid 是完全可能出现重复的情况

## 备忘路

https://www.cnblogs.com/dongkuo/p/11360419.html

> consumer 如果是正常非cleanSession,不管两端设置的QOS为几
> 
> 
> consumer 离线后，product发送消息，将会默认进入到messageQueue中
> 
> 
> 但是BROKER可以配置mq缓存的qos为几
