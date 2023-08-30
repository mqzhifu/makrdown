# swoole

TCP、UDP、UnixSocket Stream/Dgram、

SSL/TLS

同步/异常

master进程

work进程组

task进程组

manager进程

reactor线程

启动：会生成2\+N\+M个进程，N为WORKER，M为TASK，2是MASTER\+,MANAGER

master：创建manager进程，监控 woker task，再创建reactor线程、

woker：真正处理请求、发送请求、定时器任务、业务逻辑

task：woker可以向task投递异步任务，然后回调

master：reactor线程、master线程、心跳检测线程、UDP收包线程

reactor：epoll实例，acceptC端连接、接收C端数据，主要功能如：receive、sendto、close、buffer、dispatch

master线程：创建WOKER、TASK进程，主要功能如：accept、信号处理、定时器任务、业务逻辑

当一个连接进来，reactor线程收到请求，以管道的方式发送到某一个WorKER

work分发模式：dispatch\_mode

轮循模式 ：

固定模式：根据连接的文件描述符分配worker。这样可以保证同一个连接发来的数据只会被同一个worker处理

抢占模式：主进程会根据Worker的忙闲状态选择投递，只会投递给处于闲置状态的Worker

IP分配：根据客户端IP进行取模hash，分配给一个固定的worker进程。可以保证同一个来源IP的连接数据总会被分配到同一个worker进程 。ip2long\(ClientIP\) % worker\_num

UID分配：需要用户代码中调用 $serv\-\> bind\(\) 将一个连接绑定1个uid。然后swoole根据UID的值分配到不同的worker进程。算法为 UID % worker\_num，如果需要使用字符串作为UID，可以使用crc32\(UID\_STRING\)

stream模式，空闲的worker会accept连接，并接受reactor的新请求

一个Worker进程的生命周期如图所示：

当一个Worker进程被成功创建后，会调用onWorkerStart回调，随后进入事件循环等待数据。当通过回调函数接收到数据后，开始处理数据。如果处理数据过程中出现严重错误导致进程退出，或者Worker进程处理的总请求数达到指定上限，则Worker进程调用onWorkerStop回调并结束进程。

Worker进程通过Unix Sock管道将数据发送给Task Worker
