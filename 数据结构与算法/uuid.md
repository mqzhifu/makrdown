
uuid：Universally Unique Identifier 通用唯一标识，是为分布式系统生成不重复的唯一值，极端情况也会有重复的情况。

优点：不需要请求3方，在单台机器上即可生成

总共36个字符（数字\+16进制字母\[a\-f\]\+中划线）

其中：中划线是分隔符，没实际意义，但占了4个字节，实际有效的位数是32位

具体生成32个字符的源：当前日期\+时间、时钟序列、计数器、硬件标识（MAC）、命名空间、随机/伪随机等

格式=xxxxxxxx\-xxxx\-Mxxx\-Nxxx\-xxxxxxxxxxxx

格式长度=8\-4\-4\-4\-12

格式中的字母M和N：

M：代表版本，包括：1、2、3、4、5

1, date\-time & MAC address

2, date\-time & group/user id

3, MD5 hash & namespace

4, pseudo\-random number

5, SHA\-1 hash & namespace

N：代表最高有效位表示，只有 8,9,a,b

前16个字符：时间\(12位\)\+版本号\(4位\)

后16个字符：Clock Sequence\(4位\) \+ 节点标识\(12位\)

考虑到前端，获取mac地址、IP地址等数据比较麻烦~ SO，我们使用的uuid版本为4

也就是uuid4

重复概率：0.00000000006

（ps:具体计算重复概率的公式自己查吧，不上传了）

样例：

78243022\-b787\-4df4\-9f12\-c1c054810a32

e2613f46\-2af9\-4a50\-bc04\-b79ef7e5ddd5

5b4fd027\-09b8\-4ae9\-8332\-e19e39842cee

337c849c\-b178\-4bd4\-b7ce\-51b746dd7175

d6758fe1\-d8c1\-4ac8\-83de\-33fde7e59f3f

LUA生成UUID

local uuid = io.open\("/proc/sys/kernel/random/uuid", "r"\):read\(\)

//下面忽略

uuid只是一个协议，具体的实现，大部分使用的是微软的算法GUID

guid：Globals Unique Identifiers，全局唯一标识

8\-4\-4\-16
