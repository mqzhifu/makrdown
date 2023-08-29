LVS：Linux Virtual Server，采用IP负载均衡技术和基于内容请求分发技术

LVS 有几种分发模式：NAT、DR、TUN

NAT 模式，大概流程：

1 客户端发起请求

2 服务端(Director Server)IPTABLE 接收到请求

3 IPTABLES-->PREROUTING判断客户的请求目标IP地址是本地，转INPUT链

4 IPVS比对数据包请求的服务是否为集群服务，若是，修改数据包的目标IP地址为后端服务器IP

然后将数据包发至POSTROUTING链。 此时报文的源IP为客户ID，目标IP改为后端服务器IP

5 POSTROUTING链通过选路，将数据包发送给-后端服务器IP

6 Real Server比对发现目标为自己的IP，开始构建响应报文发回给Director Server。

此时报文的源IP为本机IP，目标IP为客户IP

7 Director Server在响应客户端前，此时会将源IP地址修改为自己的VIP地址

然后响应给客户端。 此时报文的源IP为VIP，目标IP为CIP

优点：支持端口转发

缺点：请求/响应，都要经常 负载机

DR 模式：

同上，但换MAC，不换IP，最后，返回数据的时候，不走负载机，直接返回客户端

优点：返回数据不用再经常负载机，性能好

缺点：不支持端口映射，网段要一致

TUN，是在IP头部再增加一层，IP

LVS 由2部分程序组成：ipvs 和 ipvsadm。

1.ipvs(ip virtual server)：一段代码工作在内核空间，工作在INPUT链上

叫ipvs，是真正生效实现调度的代码。

2. ipvsadm：另外一段是工作在用户空间，叫ipvsadm，负责为ipvs内核框架编写规则，定义谁是集群服务，而谁是后端真实的服务器(Real Server)

十种调度算法，分静/动

静态

RR：roundrobin，轮询；

WRR：Weighted RR，加权轮询，性能高的服务器，加权，处理的请求多一点

SH：Source Hashing，实现session sticy，源IP地址hash；将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现会话绑定；

DH：Destination Hashing；目标地址哈希，将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡

动态

lc：least connections，根据连接数，Overhead=activeconns*256+inactiveconns

WLC：Weighted LC，权重最小连接，Overhead=(activeconns*256+inactiveconns)/weight数

lblc：根据请求IP，寻找距离最近的IP-SERVER，且该服务器没有超载，否则会根据且连接数分发

lblcr：同上，不同的是本地会维护一个表，一个IP到一组IP的距离。一般是做CDN-缓存

SED：Shortest Expection Delay，Overhead=(activeconns+1)*256/weight

NQ：Never Queue

总结：感觉就是利用IPTABLE 做了一层代理（IPVS），且在内核态不需要用户介入

keepalived

最初，是为LVS设计，专门用来监控集群系统中各个服务节点的状态。它根据TCP/IP参考模型的第三、第四层、第五层交换机制检测每个服务节点的状态，如果某个服务器节点出现异常，或者工作出现故障，Keepalived将检测到，并将出现的故障的服务器节点从集群系统中剔除，这些工作全部是自动完成的，不需要人工干涉。

后来Keepalived又加入了VRRP的功能，VRRP（Vritrual Router Redundancy Protocol,虚拟路 由冗余协议)出现的目的是解决静态路由出现的单点故障问题，通过VRRP可以实现网络不间断稳定运行，因此Keepalvied 一方面具有服务器状态检测和故障隔离功能。

防单点故障，如上的LVS 负载机，如果挂了，就全挂了

VRRP协议：将多个物理路由器，合并成一个虚拟路由器。虚拟路由器会产生虚拟 IP，并对外服务。同一时间只能有一个路由器处理网络的所有工作，其它的路由器只接收主路由器发送的PING包

1、两台机器安装KEEPALIVED，一台配置中写名MASTER，另一台标记BAK。

2、选择一个虚拟IP，绑定到一个网卡接口，设置虚拟路由ID号（两台一样），配置上两台机器的实际IP。

3、设置权重值，主要高于BAK，主要两台机器是竞争上岗。

4、当请求过来的时候，会先到虚拟IP，再到MASTER。同时，MASTER间隔向BAK发送心中包，BAK 只负责接收，一但接收不到。BAK就会修改自己为MASTER，虚拟IP就会分发到BAK机器。