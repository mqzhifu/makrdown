content delivery network：内容分发网络


# 遇到的问题

## 跨运营商

运营商：国内 国外
国内：联通、移动、电信、铁通、教育、长城宽带、其它小运营商
国外：美国、新加坡、香港、日本、韩国

##  BGP机房

北网通 南电信，各运营商之间没法能得 ，得用 BGP 协议，速度要增加 80 MS

但是 BPG 机房非常贵

CDN 是 BGP  的  10/1 的价格

## 地理距离

中国北部黑龙江省的双鸭山市和最西部的新疆的喀什市：4458KM
中国北部漠河-南到南沙群岛南端的曾母暗沙，约5500公里

距离远最直接的影响：实时速度慢（延迟高)、不稳定（丢包、重传）

## 大并发/D-DOS

量只要够大，单个机房不管部署多少台机器都有点难。尤其中小公司，根本就不可能多机房，也不太可能大批量采购服务器做集群。

# CDN大概流程

用户访问域名->CNAME->CDN域名->DNS服务器(bind+view)->智能路由/负载均衡->边缘节点


# CDN两大结构

中心 + 连续节点

## 中心

负责全局：
1. DNS 解析 
2. 智能路由/负载均衡
3. 缓存内容管理
4. 监控/预警/报警
5. 客户管理系统
6. 组网：各个机房、各个运营商、各个城市，组建高效的一个大‘内部局域网’


## 边缘节点 

1. 性能指标统计，用于回传中心，做负载均衡
2. 收/发 缓存内容
3. 健康指令统计，用于回传中心，做最近/报警
# CDN缓存的内容

音频：听歌网站
长视频：也叫点播，像 优酷/土豆、搜狐视频、爱奇艺等
短视频：快手、抖音
直播：淘宝、快手、抖音、
安装包：exe、 apk、pkg
字体：宋、仿宋、艺术体
图片：jpg png gif jpg
代码文件：CSS、JS

总结就是：一切数据较大的、且不经常变更的，都可以缓存到CND中

二级缓存：城市
一级缓存：（边缘节点），县级


从大的方向看，CDN解决了：分布式/集群 + 负载均衡 + 缓存的存储管理 = 稳定、高效

# CDN 不能做什么

它的核心是缓存，只有静态、不轻易改变、对实时性要求不高的才能使用，但，如：
- 游戏(实时)
- 动态内容(不同用户不同显示)


# 总结

对于小公司，流量不大的前提下，没啥用，根本不用关心。
而对于中/中小公司更关注一些，因为自己搭建这套体系完全不现实，花钱租一套更实际。
大公司估计是早期会使用，但是后期，大概率都会自建CDN系统，除了开销小，更好的结合自己的业务。

总之，这个东西呢，有点类似辅助功能，程序员更关心的早代码逻辑的编写，动态的处理。对于大数据的静态文件外包给3方处理，更高效/省钱
