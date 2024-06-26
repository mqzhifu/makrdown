# 概览

http 是无状态协议，但日常开发是需要 http 有状态的，至少有一些基础状态的，如：uid name 等等，不然业务没法写了。
>没必要纠结http为什么没状态，既是优点也是缺点

cookie session jwt 就是让 http 有状态的一些技术机制

# cookie

每次浏览器与服务器交互的时候，都会带上这个值。这个值，由服务器进行设置，存储于客户端的浏览器

# SESSION

## 流程图


## 原理
cookie 与 seesion+cookie 对比，其实也差不多，区别是：cookie 受 http 协议限制，不可能存太多数据，而实际存储过程中，又不能明文全明文，得有些加密的手段，优点就是：完全存于客户端，服务器压力较小。

session+cookie:挺简单的，核心就是 session_id，登陆成功后，即生成一个 session_id，并发回给客户端，客户端保存起来，之后，每次请求带上即可。（免重复登陆）

## 优点

1. 浏览器已实现了 cookie 机制，无需要重复开发，且稳定
2. 后端各语言已实现了 session 机制，，无需要重复开发，且稳定
3. C 端/S 端，只需要调用几个 API 即可实现
4. S 端可控制 COOKIE，删除 创建 修改 查询 失效时间

## 缺点：

1. C 端依赖浏览器，如果此时是移动端实现此功能，即不行
2. C 端依赖浏览器，浏览器做限制，CS 端者无法直接控制，如：跨域
3. S 端依赖语言(php java)的 API，想要扩展些功能 需要自行开发，如：分布式 SESSION 存储

## 从功能上看，分析问题：

cookie 的实现是浏览器，浏览器为了保证安全，使用的是同源策略，连带出的问题？

1. 请求其它服务/项目时，如果域名不同，会被浏览器拒绝，这里分 2 种情况
1. 主域名都均一样，只是子域名不同，这种简单，你就设定 COOKIE 的时候加:\* 即可
1. 主域名不同，接收方把 web service 设置下可跨域即可，但可能有认证总是，下面说
1. 请求其它服务/项目时，就算解决跨域，还有登陆认证问题和会话数据读取， 这个挺复杂，有两 2 种实体：认证与数据存储
1. 认证，核心 sessionId,然而，受浏览器同源限制，一但域名变更 ，sessionId 会消失 ，同时后端会再创建一个新的 sessionId 返回给前端，造成登陆失效，需要重复登陆

> 这里可以曲线救国，A 从自己的 cookie 取出 sid，请求 B 的时候 ，再加一个 key ，把自己的 SID 写进去。感觉这种土办法 有点扯，每个项目要维护两种类型的 sessionID。不过好像没太好的办法(主域名都一样的话，可以 用共享 session),或者干脆放弃 cookie+session 模式，改 jwt

2. session 在后端是语言自己实现的，默认情况是在该代码的机器上创建一个文件，用于存储 session info，间接说明：一个用户的登陆状态就是保存在某台机器上，如果 子系统的代码 在其它机器上，就算 sid 能提出来，也依然没用。

> 主域名都一样的话，可以 用共享 session 解决

> 负载均衡也会出现以上问题，不过可以用固定 IP、HASH 而非轮轮询

## 使用场景：

### 适合

1. 都是一个主域名
2. 自实现 session 共享存储
3. 非完整 SSO 机制(域名不同)

### 不适合

1. 主域名不同
2. 完整 SSO 机制 (域名不同)

## 结论

小项目，非多服务、多域名、分布式构架，session+cookie 的模式还是挺适合的，因为代码量少简单方便，且稳定

关于 sessionId 的存储有 2 种试，硬盘持久化与内存持久化

1. 内存持久化，就是这个 sessionId 存于浏览器内存中，一但浏览器关闭，该 session 即失效，也就是常见的：一但浏览器关闭后再打开，需要重新登陆

2. 硬盘持久化，就是真的存于 cookie 中，浏览器就算是关闭了，再启动的时候，依然是已登陆状态，浏览器内部也有加密措施

> 如果用户禁了 cookie，也可以使用 url 传输

以上两种应用于不同场景，也没好坏之分，适用场景不同罢了，不过大部分应用还是存 cookie 中，方便日常使用，不然产品不得骂街

# token 机制


## 令牌，是访问一个系统的凭证

1. 用户不一定非得使用浏览器，不一定是 HTTP 协议，比如，在 TCP 层面，发送 用户名密码，没有什么附加的东西了（比如：cookie）

2. server 接收到，验证没有问题后，生成一个加密串：base64 ( UID + KEY + 失效时间 )，不保存任何数据，只返回一个串给前端，最多，把串存于 REDIS 中

3. 前端拿到串后，所有的请求前面加上这个 TOKEN 就可以 了


# jwt 

jwt:json web token,算是 tokon 的一种格式吧，包含 3 个部分，中间逗号隔开

1. header 一些基础描述信息
2. payload 内容体，如：uid uname 等，但因为是可以 decode 所以不放敏感信息
3. sign 签名，防止内容体被修改

## 详细格式：

> header: {type:JWT,alg:HS256}
> payload:{uid:1111,uname:'wang','expire':11111,atime:1111,ext1:""，appId:1}
> sign:aaaaaaaaaaaaaa

最后格式：aaaaa.bbbbbbbb.ccccccc

## 加密算法：

RS256 (采用 SHA-256 的 RSA 签名) 是一种非对称算法, 它使用公共/私钥对: 标识提供方采用私钥生成签名, JWT 的使用方获取公钥以验证签名。由于公钥 (与私钥相比) 不需要保护, 因此大多数标识提供方使其易于使用方获取和使用 (通常通过一个元数据 URL)。

HS256(带有 SHA-256 的 HMAC 是一种对称算法, 双方之间仅共享一个 密钥。由于使用相同的密钥生成签名和验证签名, 因此必须注意确保密钥不被泄密。

## jwt 流程


感觉相比 session cookie，JWT 的交互更简单些，并且它并不一定依赖浏览器，不需要渲染登陆页面，也不依赖浏览器的 cookie，虽然它也加在 http 头中，或 url 中，但它更倾向的是自实现一套登陆验证。也就是说：它更单纯的倾向于用户验证这个功能

## jwt 状态


1. 无状态，只要生成一个令牌，所有项目/服务，均能用。所有项目可自行验证，无需要再请求 3 方服务验证

优点：真的实现了分布式，跨域名/服务/项目，自身即可验证

缺点：如果令牌有效期，这个用户被禁用了，其它人无法感知，这是个最最大的硬伤

2. 有状态，所有项目，拿到 JWT TOKEN 后，自身不能验证合法，得请求 3 方服务验证

间接看是多了一次请求(3 方验证服务)，往底层看，就是间接实现了一套 COOKIE+SESSION 机制...

> 好像有点蛋疼呢

优点：不依赖浏览器，天生具备分布式存储(redis)

缺点：多实现了一套 COOKIE+SESSION 机制，多工作量，不确定是否有 COOKIE+SESSION 机制稳定

# session 里存储什么？



1. 用户登陆状态。登陆时间、登陆地点
2. 用户基础信息。年龄 昵称 性别等
3. 用户/管理员权限列表
4. 用户/管理员配置信息，如：界面展现、管理员菜单等

好像除了第一条之外，都是静态数据，后端都可以实时计算，那么，分析出两种数据类型：

1. 登陆状态
2. 静态数据

先分析 2，这些数据为什么要放到 session 里？

因为：经常使用，每次计算麻烦，那如果这些数据放到 redis 里，是不是一样的？我觉得完全可以

那么实际上 session 里存储数据，唯一需要考虑的就是用户登陆状态了，或者换个角度看：

jwt 不是传统意义上的 session 存基础数据的，不要弄混了，这两东西不是互斥的，或者说，也可以一起使用

关键点就剩一条：用户登陆状态/验证，所以 JWT 实现这一条就能满足主要的需求

jwt 状态选择

---

我当然希望是无状态，各端自行验证即可，但问题是，它有两个硬伤：

1. 一但签发，不可更改|废弃，如：权限变更、用户信息变更

> 解决：JWT 中不要放任何数据，只放 UID，这个肯定是不变的，余下的都去 redis 里取

1. 一但签发，不可废弃，如：

1. 用户被禁止登陆，此时该用户还能访问所有服务的 API 接口

1. 踢掉多端登陆的用户

> 无法解决

2. 无法续签

> 这个感觉能接受，设置个 30 天，无所谓了

分析看：不可废弃这条没太好的办法，所以，它的应用场景：一次性的，签发时间过短，像：邮件注册的认证信息，或者能接受短时间内反复申请 jwt 的应用

所以，如果使用无状态就得接受：不可废弃 这条。结合实际场景分析下，分析下造成什么样的影响？

1. 如果正常一个项目大部分的 API 请求都无所谓，想拿数据就拿呗，关键性的操作，再加上诸如：支付密码，其实问题不大

2. 多端同时登陆，如：

3. 同类型登陆：PC/H5 FIREFOX 登陆，再用 chrome 登陆 或 app 分身/多开

4. 非同类型登陆：APP 端登陆，再用 H5 登陆 或 IOS 登陆 再 安卓登陆

处理逻辑，这里分两种情况：

1. 不允许重复登陆，只有一个 JWT，多端共用一个 JWT

优点：好管理

缺点：

1. 如果一个失效，全失效
2. 如果注册 3 方服务，如：腾讯 IM，使用 JWT 为 ID 注册，会出现踢人的情况
3. 允许重复登陆，会有多个 JWT，各端一个 JWT，也可能多于这个，有非常多的 JWT

缺点：不好管理

优点：能实现每端的失效时间各不同，并且，可以单独注册多个 3 方服务

结论：使用允许重复登陆，好像没太大问题呢，除了 JWT 过多不太好管理，有状态的可能更严谨一点，就是多了一次请求，并且能实现<废弃>功能

缺点：

1. 增加开发量，增加复杂度，增加维护成本
2. 可能没有 cookie+session 稳定
3. 增加了各端的请求量，加大了各端的开销

优点：

1. 完全不依赖浏览器
2. 天生具备分布式

结合实际场景分析看：

1 2 其实无所谓，3 感觉问题不在 JWT 上，反而是 提供 JWT 的服务的 健壮性问题，所以好像没太大问题呢

最终结论：我更倾向 有状态 JWT，同时优化一下多端同时生成 JWT 的限制，如下：

一共有几端是可确定的，肯定不超过 10 个(mac win ios android 等 )，每端只需要生成一个 JWT 就够了，用户在申请的时候

如：加上 souceType 标识，如果是同端多次申请，就注销掉之前的，这样同端也可以实现互相踢人。

不同端的情况，就正常多生成一个，这里有个疑问：验证的时候，要不要也得要求请求方加上 souceType，我想了想有点鸡肋。

# session token 对比


首先要声明：这两个东西不是完全互斥的，甚至可以互补，两个一起用，之所以对比，是觉得这两有些共用的功能。也没有绝对的谁好谁坏，一切看应用场景。

其次，我想了想，好像看似有很多区别，实际上都差不多

1. jwt 能实现 SSO（不同域名、不同项目）

2. session 能实现 部分 SSO（同主域名也可以）

# 总结

1. session 的意思是会话，是记录一次：双方通话的元数据，token 则是令牌，是访问一个系统的凭证

2. 感觉吧，token 就是不依赖于浏览器，独立实现了一套伪 SESSION 机制

实际更完整的方案，可以参考 JAVA 的 ：CAS （ Central Authentication Service ）
