## nginx
nginx是一款轻量级web服务器、反向代理服务器，内存占用少，启动极快，高并发能力强

**正向代理：** 客户端利用其他服务器代理转到目标服务器，目标服务器不知道客户端通过什么访问的。例如vpn

**反向代理：**服务端的代理，当客户端访问服务端，服务单配置的转发到真正的服务器，这一过程对客户端透明。例如nginx

### Master-woker模式

![](https://pic4.zhimg.com/80/v2-b24eb2b29b48f59883232a58392ddae3_720w.jpg)

master进程：读取并验证配置文件nginx.conf，管理worker进程

work进程：每个worker进程都维护一个线程，处理连接和请求；进程数量由配置文件决定，一般和cpu个数相关

### nginx热部署

配置好nginx.conf，不需要停止nginx，不中断请求，直接使配置文件生效。

> nginx -t 检查nginx的配置是否异常
>
> nginx -s reload 重新加载配置文件
>
> nginx -s stop 停止nginx

方案一：修改配置文件nginx.conf，master进程将新的配置文件推送给work进程更新配置，work进程收到后更新内部线程信息

方案二：修改配置文件nginx.conf，重新生成新的work进程，新请求由新work进程处理，老的请求由老work进程处理完毕，干掉老进程。nginx通过这种方案是按热部署

### 如何高并发下的高效处理

采用linux的epoll模型，epoll模型基于事件驱动机制，它可以监控多个时间是否完毕，完毕放入队列，该过程是异步的，work只需要从队列中取事件。

### nginx挂了如何处理

Keepalived+nginx实现高可用。请求不直接打到nginx，而是通过keepalived（虚拟id）再转到nginx；keepalived监控nginx的生命状态，实现故障切换

### 负载均衡

proxy_pass 配置，负责将请求均匀分配的指定服务器列表中



## epoll

是linux内核为处理大批量文件描述符而作了改进的poll，是linux下多路复用select/poll增强版本，能显著提高程序在大量并发连接中只有少量活跃的情况下的cpu利用率。获取事件的时候，无需遍历整个被监听的描述符集，只要遍历队列中的描述符即可



### 参考

1. https://zhuanlan.zhihu.com/p/34943332