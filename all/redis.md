# 通识

## 一、基本知识

### 1. 缓存

#### 缓存类型

分为本地缓存（可以采用LRUMap实现）、分布式缓存、多级缓存（结合本地缓存和分布式缓存）

#### memcache

一种分布式高速缓存系统，采用多线程异步IO的方式，充分利用cpu。数据全部保存在内存中

采用延迟失效策略，当被访问时检查是否失效。

当容量爆满时，采用lru策略对数据进行剔除

**缺点**

- 数据只能保存30天（可配置），不能持久化
- key最大不超过250个字节（可配置），value最大不超过1M（可配置）
- 仅支持key-value数据类型保存，不支持持久化和主从同步

#### Redis优点

- 采用单线程，避免上下文切换带来的cpu消耗

  > 为什么选择单线程？
  >
  > 1. 可维护性更高，方便开发、调试和测试
  > 2. 单线程也能并发处理请求，使用I/O多路复用机制同时监听多个文件描述符的可读和可写状态，并发处理多个客户端的连接。
  > 3. Redis中绝大部分操作的性能瓶颈都不在cpu，反而在线程上线文切换上会产生一些消耗。性能瓶颈主要在网络I/O操作上

- 基于内存操作

- 简单数据结构的设计

- C语言实现，更快

- 提供主从同步，集群，哨兵功能，保证高可用性

**后续引入的多线程操作**

对于一些超大键值对的删除，处理时间可能较长，可采用多线程异步处理方式

### 2. Redis内存模型

#### 2.1 内存统计

> info memory # 查看redis内存配置

**used_memory**

由redis内存分配器分配的数据内存和缓存内存的总量，包括使用的虚拟内存。used_memory_human展示具体单位，更人性化的显示

**used_memory_rss**

由系统内存分配的redis进程内存和redis内存中无法再被jemalloc分配的内存碎片

> Used_memory和used_memory_rss的区别：
>
> 前置是从redis角度得到的量，后者从操作系统得到的量。一方面可能因为后者redis进程和内存碎片的存在，导致前者比后者小；另一方面因为前者虚拟内存的存在，导致前者比后者大

**mem_fragmentration_ration**

内存碎片比率，该值为 `used_memory_rss/used_memory`

该值一般大于1，且越大表示内存碎片越多

若该值小于1，说明使用了虚拟内存。而虚拟内存的媒介是磁盘，比内存速度慢得多。若出现这种情况应该及时排查，如果内存不足应及时增加redis内存，增加redis节点，优化等

一般来说`mem_fragmentration_ration`为1.03时比较健康的状态。redis刚启动时可能比较大，因为还没想redis中存入数据，redis进程本身占用一定的内存

**mem_allocator**

redis使用的内存分配器，在编译时指定。可以是libc、jemalloc或tcmalloc。默认`jemalloc`

> 5.0.8、6.0.01默认是`libc`

#### 2.2 内存划分

**数据**

作为数据库，数据时最重要的部分。该部分占用的内存会统计在used_memory中

**进程**：该部分不是有内存分配器分配，不在used_memory中

**缓冲内存**

包括客户端缓冲区、复制积压缓冲区、AOP缓冲区等

客户端缓冲区存储客户端连接的输入输出缓冲；复制积压缓冲区存储部分复制功能；AOP缓冲在进行aop重写时，保存最近的命令

**内存碎片**

操作系统已分配但redis无法利用的内存

#### 2.3 redis数据存储细节

redis是key-value型数据库，每个键值对都会有一个dictEntry

> distEntry
>
> - key: 键。只有字符串类型
> - value：值。有字符串、列表、哈希、集合、有序集合五种类型
> - dictEntry* next：下一节点
>
> key和value都是redisObject类型
>
> RedisObject
>
> - type：类型，4位
> - encoding:4 :编码，4位
> - ptr：指向实现数据结构的指针。8字节
> - int refcount：引用数量。4字节
> - unsigned lru:24 ：记录键值最后一次被访问的时间，24位
>
> 一个redisObject大小为=4bit+4bit+24bit+4byte+8byte=16byte

内存模型

>  info mermory // 查看内存模型
>
>   type ${key}// 查看key的value类型
>
> object encoding ${key} // 查看value的编码

#### 查看引用数量

 object refcound {key} // 查看key的引用数量

> int、string对象不共享

#### 查看lru时间

object idletime {key}

###  3. redis的对象类型和内存编码

#### 3.1 字符串

**SDS 简单动态字符串**

3.2版本之前

- int len：总长度
- int free：剩余空闲长度
- char buf[]：字节数组

3.2版本之后，如sdshdr8

- uint8_t len: 总长度。1字节存储，存储的字符串长度小于256
- unit8_t alloc：已使用长度。1字节存储
- unsigned char flags，用3bit表名类型，后5bit未使用。
- buf[]：字节数组

##### 编码

1. int，保存数字类型
2. embstr，固定字符串，3.2版本之前长度39，3.2版本之后长度44
3. raw，保存字符串，用sds结构保存

> embstr和raw对比
>
> embstr和raw都是使用redisObject和sds保存数据，不过embstr只分配一次内存，redisObject和sds是连续的，raw需要分配两次
>
> embstr的好处是分配和删除都少一次内存操作，缺点是当修改时需要对两种结构进行重新分配内存，所以embstr是只读的，修改时会自动转化的raw类型

> embstr长度计算
>
> 3.2版本之前，redisObject大小固定16字节，len为4字节，free为4字节，字节数据指针为1字节，jemalloc内存分配器刚好分配64字节，所以16+9+39=64，embstr长度为39字节
>
> 3.2版本之后，使用sdshdr8结构，len为1字节，alloc为1字节，flags为1字节，字节数据指针1字节，所以16+4+44=64，embstr长度为44字节

#### 3.2 链表

3.0版本之前，linkedList、zipList  

当以下两个条件全部满足时使用ziplist，否则使用linkedList

- 元素个数小于等于512
- 保存的元素长度都不超过64

在3.2版本之后，只有quicklist，quickList由linkedList和zipList组成，每个节点都是一个压缩列表节点，且存在前后指针

> rpush {key} {value1} {value2} ...   创建并保存list

#### 3.3 哈希表

当以下两个条件同时满足时使用ziplist，否则使用hashtable。转成hashtable不可再转回ziplist

- 元素个数小于等于512时
- 所有键值对长度都不超过64

> hset {hkey} {key1} {value1}

#### 3.4 集合

当以下两个条件同时满足时使用intset，否则使用hashtable

- 所有元素都是整数类型
- 不超过521个元素

> sadd {key} {member1} {member2}...    设置set
>
> smembers {key}  遍历set

#### 3.5 有序集合

当满足以下两个条件时，使用ziplist，否则使用skiplist

- 元素数量小于128
- 所有元素长度都不小于64字节

## 二、架构原理

### 1. redis持久化

#### 1.1 RDB

rdb是redis默认的持久化方式，通过快照的方式保存数据库中所有的键值对。当满足一定的条件时，redis进行rdb备份，将内存中的数据备份到rdb文件中

**触发rdb快照的时机**

1. 符合指定配置的快照规则
2. 执行save或bgsave时
3. 执行fushall
4. 执行主从复制操作

**配置快照规则**

save time updateNum 表示指定时间内超过指定个数键值被修改

save "" # 不适用rdb备份

```shell
# redis.conf 默认配置规则
save 900 1 # 900s内超过1个键被修改则进行rdb持久化
Save 300 10 # 300s内超过10个键被修改则进行rdb持久化
save 60 10000 # 60s内超过10000个键被更改则进行rdb持久化
```

##### rdb快照实现原理

**快照过程**

1. redis调用fork函数创建当前一份进程的副本（子进程）
2. 父进程接收并处理客户端请求，子进程将内存中的数据备份到临时文件
3. 子进程备份完毕后，用临时文件替换老的`dump.rdb`文件，一次rdb持久化完成

**注意事项**

1. rdb持久化过程中不会修改老的rdb文件，备份完成后直接用新文件替换老文件。所以rdb文件生成后是稳定不变的
2. rdb是经过压缩过的二进制文件，占用的空间小于内存中占用的空间，更利于传输

**优点**：rdb可最大化redis的性能。仅仅在fork时需要占用父进程，之后便不用父进程关心，子进程独自处理

**缺点**：当redis退出会丢失rdb备份到退出时间的内存数据

#### 1.2 AOF

`append only file`，通过保存redis的写命令来实现持久化

```conf
# 是否开启aof持久化，默认no不开启。
appendonly no
# 若开启aof持久化，设置持久化频率。always-产生一条写命令则进行一次备份；everysec-默认，每秒备份一次；no-不备份
appendfsync always
appendfsync everysec
appendfsync no
```

**aof优化**

aop会对某个键的多个操作进行优化，覆盖或批量添加

#### 1.3 混合持久化方式

redis4.0之后新增混合持久化方式，结合rdb和aof的优点，在备份的时候，先把当前的数据以rdb的方式写入文件，再将后续实时产生的写命令用aof的格式存入文件，即保证了备份的速率又能降低数据丢失的风险

redis5.0之后默认开启混合持久化方式

> 127.0.0.1:6379> config get aof-use-rdb-preamble # 返回yes表示开启

### 2. Redis主从复制

redis数据保存在内存中，机器故障会造成数据丢失。主从复制可避免单点故障，设置一主多从，主写从读。主从复制不会阻塞master。一个redis可以即是主又是从，即从节点也可能当成主节点同步给其他从节点数据

#### 2.1 实现原理

1. 主从复制分为全量同步和增量同步
2. 只要第一次连接上主机的为全量同步
3. 中间断线重连的可能触发全量同步或增量同步（master判断ruuid是否一致）
4. 除此之外所有情况都是增量同步

> psync命令实现
>
> 老版命令sync只有全量同步

**全量同步**：分为三个节点

- 同步快照阶段：master创建并发送快照给slave，slave接收并解析。同时master将此阶段产生的写命令保存在写缓冲区
- 同步写缓冲区阶段：master将写缓冲区的数据发送给slave
- 同步增量数据阶段：master向slave实时同步增量数据

> 通常情况下，增量数据同步阶段，master每执行一个写命令都会发送给slave，salve接收并执行

**增量同步**：只有增量数据同步阶段

#### 2.2 redis配置

修改redis.conf文件

```conf
#指定主服务器地址
replicaof ip port
```

**缺点**：主从同步无法自动切换主节点

### 3. Redis哨兵机制

sentinel（哨兵）进程监控主服务器的状态，若主服务器发生故障，自动将从服务器切换成主服务器，完成自动切换，保证系统高可用

#### 3.1 故障判定原理

1. 每个哨兵定时(默认每秒一次)向主服务器发送`ping`命令，若上次有效的ping命令距离当前时间超过配置时间(down-after-milliseconds，默认30s)，则哨兵将该主服务器标记为主观下线`SDOWN`。
2. 标记主观下线的哨兵会询问其他哨兵，主服务器是否为主观下线。当确定主观下线的哨兵数量超过配置的个数，则将该主服务器标记为客观下线`ODOWN`，进行自动故障转移

#### 3.2 自动故障转移

1. 选举客观下线后的主服务器其中的一台从服务器，升级为主服务器，并让失效的主服务器改为从服务器复制新的主服务器
2. 当客户端连接主服务器时，若失败会返回给客户端新主服务器的地址
3. 服务器切换之后，主从服务器和哨兵的配置文件redis.conf都改变。主从服务器改变复制对象，哨兵改变监控目标

#### 3.3 Redis配置

修改sentinel.conf配置

```conf
# 监控主服务器。quorum表示满足客观下线的最小的sentinel个数
sentinel monitor <master-name> <ip> <port> <quorum>

# 开启哨兵
redis-server sentinel.conf --sentinel
```

### 4. Redis集群

#### 4.1 Redis主从复制

缺点：不能自动切换主从服务器

#### 4.2 Replication+Sentinel 高可用

主从复制+哨兵机制实现高可用，可自动切换主从服务器

缺点

1. 主从切换时会丢失数据
2. redis只能单点写，不能水平扩展

#### 4.3 Proxy+Replication+Sentinel （了解）

proxy可选择两种：Codis（豌豆荚）和Twemproxy（推特）

缺点

1. 部署结构异常复杂，难以运维
2. 可扩张性差，扩容需要人工干预

#### 4.4 Redis cluster

实现细节

- 所有redis节点彼此互联（PING-PONG机制），内部使用二进制协议优化传输速度和带宽
- 节点的fail状态是超过半数投票才确认的
- 客户端可以与任意一个redis机器直连，不需要任何的proxy
- redis-cluster把16384个槽分配给各个节点，一个槽只能被一个节点监测

**redis-cluster投票：容错**

- **节点失效判断**：集群所有节点参与投票，当超过半数的master节点与该master节点连接失败，则判定该节点挂掉
- **集群失效判断**
  - 若集群中任意的节点挂掉且没有slave。槽映射不全，则将该集群下线
  - 若集群超过半数的master挂掉，无论有没有slave，都将集群下线

优点

- 无需哨兵监控，若主服务器挂了，集群自动将其从服务器切换为主服务器
- 可以水平扩展
- 可以自动转移多余slave节点。当主服务器的slave节点没有时，高可用性无法保证，集群自动会将其他有多余从节点的机器转移过来，实现高可用

缺点

- 批量操作坑？
- 资源隔离性较差，可能产生相互影响？

##### Redis指令实现

```shell
# redis.conf配置文件
# 表示开启集群模式
cluster-enabled yes
# 指定集群节点文件地址
cluster-config-file node-8001.conf
# 指定节点宕机决策超时时间
cluster-node-timeout 15000

# redis实现步骤
# 1. 生成多个节点配置文件，官方推荐至少需要3个主节点。（每个节点建议增加一个从节点，生成6个节点）
# 2. 启动所有节点
# 3. 创建集群：redis-cli --cluster create <ip>:<port> [<ip>:<port> ...] --cluster-replicas 1 -a ww
# 	--cluster-replicas 表示每个主服务器从节点数
# 	-a 密码
# 4. Can I set the above configuration? (type 'yes' to accept): yes
# 5. 创建成功，登录其中一台查看情况，使用cluster nodes查看
127.0.0.1:8001> cluster nodes
0f0dbf57dfca7f9c3a0c3b35a05eb282933db632 127.0.0.1:8002@18002 master - 0 1610895591000 2 connected 5461-10922
bccdb23322b3d525338ae0019a0254b529d4ba69 127.0.0.1:8005@18005 slave 0f0dbf57dfca7f9c3a0c3b35a05eb282933db632 0 1610895590817 2 connected
05c04ff540c6e35f65758c90d4a8e6af8382a819 127.0.0.1:8001@18001 myself,master - 0 1610895590000 1 connected 0-5460
46373639b3e99a922abd9e93b95196d0c1299621 127.0.0.1:8003@18003 master - 0 1610895591827 3 connected 10923-16383
5d2b0f2b883041080bd8572d81cfea838d28ffab 127.0.0.1:8006@18006 slave 46373639b3e99a922abd9e93b95196d0c1299621 0 1610895592838 3 connected
c120a0d875877169d7fd025bd2438a460c52e39c 127.0.0.1:8004@18004 slave 05c04ff540c6e35f65758c90d4a8e6af8382a819 0 1610895591000 1 connected

# cluster info查看集群信息
```

```shell
# redis服务器
# 将槽分配给指定机器
clsuter addslots <槽下标> [<槽下标>...]
```

## 三、高级拓展

### 1. Redis的特殊数据结构

#### 1.1 Bitmap

通过一个bit位来记录一个元素的状态，可用来统计一年内的用户访问量，占用内存小

```shell
setbit key offset value #offset-目标索引位置，可以根据取余计算；value-0或1，1表示在线
setbit k1 2 1
getbit k1 2 #获取bit位
bitcount k1 #获取键值的在线位个数
bitop and destkey k1 k2 #将k1和k2做并集，将结果赋给destkey，还支持OR NOT XOR
```

#### 1.2 HyperLogLog（2.8）

用来统计唯一元素的数量（近似值）。基于bitmap基数计数；基于概率基数计数

每个HyperLogLog键只有12K大小，就可以计算接近2^64个不同元素的基数

存在三个指令

- pfadd key value # 将值添加到key中，重复元素返回0，非重复元素返回1
- pfcount key # 返回key的value值总个数，忽略重复元素。返回的只是一个近似值
- pfmerge destkey key [key ...] # 将key合并赋值给destkey

#### 1.3 Geospatial（3.2）

用来保存地理位置，并用做距离计算和半径计算等。底层依赖sorted set

```shell
getadd key 经度 纬度 名称
getadd cities 116 38 "beijing" 121 31 "shanghai" # 设置key
zrange 0 -1 #查看key
geodist cities beijing shanghai km # 查看两地距离，单位km
geopost cities beijing shanghai #查看经纬度
georadius cities 120 30 500 km # 查看指定经纬度500km范围内的元素
GEORADIUSBYMEMBER cities shanghai 200 km #距离指定元素小于200km的元素
```

### 2. Redis消息模式（了解，现在常用stream）

消息通过lpus推送到消息队列，消费者通过rpop消费

注意

- 如果接收方不知道队列中是否有消息，会一直发送rpop建立连接，浪费资源
- 可以使用brpop，阻塞获取消息，一定时间内未获取到返回null

### 3. Redis Stream

以更抽象的方式建模的日志结构，可以用作基于内存的mq，速度较快。可用作通信、大数据分析、异步数据备份等

有消息、生产者、消费者、消费者组成。同一消费者组消费相同的消息流，不同消费组都消费全量消息

```shell
# 发布消息
127.0.0.1:6379> xadd mystream * message apple
"1611072807094-0"
127.0.0.1:6379> xadd mystream * message orange
"1611072824618-0"
# 读取消息
127.0.0.1:6379> xrange mystream - +
1) 1) "1611072807094-0"
   2) 1) "message"
      2) "apple"
2) 1) "1611072824618-0"
   2) 1) "message"
      2) "orange"
# 阻塞读取消息
127.0.0.1:6379> xread block 0 streams mystream $
# 发送新消息。阻塞读取进程消费该条消息
127.0.0.1:6379> xadd mystream * message orange2
"1611072883427-0"
# 创建消费者组
127.0.0.1:6379> xgroup create mystream mygroup1 0
OK
127.0.0.1:6379> xgroup create mystream mygroup2 0
OK
# 通过消费者订阅消息，消费两条消息
127.0.0.1:6379> xreadgroup group mygroup1 zrange count 2 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1611072807094-0"
         2) 1) "message"
            2) "apple"
      2) 1) "1611072824618-0"
         2) 1) "message"
            2) "orange"
# 继续订阅新消息
127.0.0.1:6379> xreadgroup group mygroup1 zrange count 2 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1611072883427-0"
         2) 1) "message"
            2) "orange2"
# 更换消费者组订阅消息，从头开始消费
127.0.0.1:6379> xreadgroup group mygroup2 zrange count 2 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1611072807094-0"
         2) 1) "message"
            2) "apple"
      2) 1) "1611072824618-0"
         2) 1) "message"
            2) "orange"
```

### 4. Redis pipline

pipline（管道技术）是客户端提供的一种批处理技术

可以批量执行一组指令，一次性返回结果

样例

引入包

```xml
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
</dependency>
```

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

public class PiplineTest {
    public static void main(String[] args) {
        multi();
        pipline();
    }

    /**
     * 管道方式执行，一次执行批量命令
     */
    static void pipline() {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        Pipeline pipeline = jedis.pipelined();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; ++i) {
            pipeline.set("key" + i, "value" + i);
            pipeline.del("key" + i);
        }
        pipeline.sync();
        System.out.println(System.currentTimeMillis() - start);
    }

    static void multi() {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; ++i) {
            jedis.set("key" + i, "value" + i);
            jedis.del("key" + i);
        }
        System.out.println(System.currentTimeMillis() - start);
    }
}
```

### 5. Redis事务

multi 开启一个事务

exec 执行事务

watch 乐观锁

discard 丢失事务

redis通过multi开启一个事务，所有指令正常入队列，当遇到错误指令时报错（编译错误），事务中断，此时事务未提交，未执行任何一条指令；当遇到语法错误（运行时错误），仅错误指令执行失败，其他指令正常执行（不管是错误指令前面或后面的指令）

watch乐观锁，需配合事务使用，当遇到exec后，对锁的监控结束。在事务提交时，若watch监控的键被改变，事务失效，全部指令均不提交

### 6. Redisson分布式锁

引入包

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson</artifactId>
  <version>2.7.0</version>
</dependency>
```

```java
private static Config config = new Config();
    private static Redisson redisson;
    static {
        String[] port = {"8001", "8002", "8003", "8004", "8005", "8006"};
        String[] urls = new String[port.length];
        for (int i = 0; i < urls.length; ++i) {
            urls[i] = "redis://127.0.0.1:" + port[i];
        }
        config.useClusterServers()
                .setScanInterval(2000)
                .addNodeAddress(urls);
        redisson = (Redisson) Redisson.create(config);
    }

    public static Redisson getRedisson() {
        return redisson;
    }


    public static void main(String[] args) {
        Redisson redisson = getRedisson();
        String key = "redisson";
        RLock lock = redisson.getLock(key);

      	// 锁定键1s
        lock.lock(1, TimeUnit.SECONDS);
        // 尝试锁定键2s，最大等待1s
      	lock.tryLock(1, 2, TimeUnit.SECONDS);
				// 释放锁
        lock.unlock();
    }
```

### 7. Redis性能调优

#### 7.1 设计优化

**估算内存使用量**

**优化内存占用**

1. 利用jemalloc进行优化：内存分为为2的指数倍，17变16实际占用内存就会减少一倍
2. 尽量使用内存占比小的结构，整型、embstr、ziplist
3. 共享对象的使用：默认0-9999，可配置REDIS_SHARED_INTEGERS
4. 缩短键值对的存储长度

#### 7.2 设置键值的过期时间

#### 7.3 限制redis内存的大小

maxmory表示，默认redis的内存大小为0，即不限制其他小，可能导致redis因内存不足而使用虚拟内存（依赖磁盘提供），操作卡顿

redis内存满时，使用内存淘汰策略，共8种。maxmemory-policy 属性配置

- Noevication：默认，不淘汰任何键值，当内存不足时新增指令报错
- Allkeys-lru: 对所有键，采用最近最近未使用使用策略淘汰
- Allkeys-randon: 对所有键随机淘汰
- Volatile-lru: **通用**。对设置了过期时间的键，采用最近最近未使用使用策略淘汰
- Volitile-random: 对设置了过期时间的键随机淘汰
- Volitile-ttl: 优先淘汰最早过期的键

4.0新增

- allkeys-lfu：对所有键，淘汰最近最少使用的键
- volitile-lfu

#### 7.4 使用lazy free

redis4.0 新增，惰性删除或延迟删除。在删除键的时候，采用异步的方式删除，减少主线程的阻塞

Lazyfree-lazy-evication： 默认no，表示redis超过最大内存后，是否使用惰性删除

Lazyfree-lazy-expire: 默认no，表示键过期，是否使用惰性删除

Lazyfree-lazy-del: 默认no，表示键删除时，是否使用惰性删除

Slave-lazy-flush: 当slave同步时，会加在rdb文件，此时会采用flushall，此时是否使用惰性删除

建议开启Lazyfree-lazy-evication，Lazyfree-lazy-expire, Lazyfree-lazy-del

#### 7.5 禁用耗时长的查询指令

keys pattern

#### 7.6 使用慢查询

Slowlog-log-slower-than: 表示指令超过多长时间，则标记为慢查询，加入到慢查询日志中。单位微秒

slowlog-max-len: 慢查询日志中的最大条数

slowlog get n获取慢查询日志

#### 7.7 避免大量键同时失效

使用固定时间+随机时间，使得过期时间尽量随机，不要在同一时间点同时过期

#### 7.8 检查数据持久化

rdb持久化

aof持久化

混合持久化（4.0+）

#### 7.9 使用pipeline管道技术批量操作

#### 7.10 客户端优化

尽量使用连接池

#### 7.11 使用分布式架构

- 主从复制 & 哨兵：读性能扩展
- 集群模式：读写性能扩展

#### 7.12 使用物理机而非虚拟机

#### 7.13 禁用THP特性

linux kenrel内核增加了THP特性，支持页最大内存2M

当开启THP是，页内存从4K变成2M，大幅度增加内存消耗，甚至小指令也需要这么多内存

### 8. lua脚本

#### 8.1 基础

Lua helloworld.lua # 运行lua脚本

lua -i # 进入交互式编程页面

#### 8.2 数据类型

- nil ：只有nil值属于该类，用于表达式时为0

```lua
print(type("hello world"))  -->string
-- for
tab = {key1="v1", key2="v2","v3"}
for k, v in pairs(tab) do
	print(k .. "-" .. v);
end
--[[打印
1-v3
key1-v1
key2-v2
]]--
-- 
str = "hello"
print(#str) -- 5, #获取变量长度
```

**Redis.call()**  返回值就是redis执行的返回值，如果出错了，停止执行

**redis.pcall()** 返回值是redis执行的返回值，如果出错了，就错误信息继续执行

**Eval** redis中运行lua脚本

```shell
# redis中运行lua脚本
> eval "return redis.call('set', KEYS[1], KEYS[2])" 2 foo val
```

### 9. 布隆过滤器

布隆过滤器可快速判断键值是在不存在于当前容器中。

#### 9.1 简介

**添加值**：布隆过滤器可以当成一个bit数组，针对需要保存的key，先对key做hash取模，放入bit数组中，置该位为1。

**判断数据是否存在**：判断一个键是否存在时，先通过hash取模获取位置，查看是否为1，若不为1，则该键一定不存在与布隆过滤器中；若存在，由于hash取模之后可能重复，无法确定该键是否真正存在

**优点**：占用内存极少；快速判断数据是否存在

**缺点**：随着数据的增加，误判率会增加（产生hash冲突）；无法判断数据是否一定存在；只能新增无法删除数据

#### 9.2 实现

**bitmap**：位数组

> Redission中集成了布隆过滤器，底层基于bitmap实现

### 10. 客户端框架对比

#### 10.1 Jedis

redis基于java实现的客户端，提供了较全面的redis命令支持。指令基本与redis一一对应，使用简单

使用阻塞的IO，其调用方法都是同步的，不支持异步。Jedis客户端实例不是线程安全的，需要使用线程池。每次操作需要从线程池获取连接，当redis使用次数过大不建议使用jedis

redis客户端在springboot1.5.x默认Jedis实现，在springboot2.x默认lettuce实现

#### 10.2 lettuce

高级的Redis客户端，用于线程安全同步，异步和响应使用，支持集群、sentinel，管道和编译器。适用于分布式缓存

基于netty框架的事件驱动的通信层，其方法调用可以是异步的。lettuce的api是线程安全的，共享连接，连接时long-lived和线程安全的，自动重连

#### 10.3 Redisson

实现了分布式和可扩展的java数据结构，支持布隆过滤器，分布式锁，分布式集合等。适用于分布式开发

基于netty框架的事件驱动的通信层， 其方法调用可以是异步的，api线程安全，共享连接

## 四、面试题

1. redis有哪些结构？哪几种对象

2. 各结构使用场景

   String：缓存一般类型

   hash：产品属性单一

   List：粉丝列表、文章列表

   set：全局去重，微博的共同好友

   zset：排行榜，热搜

3. 单机性能瓶颈问题？

4. 多机之前如何进行通信？redis如何进行持久化？redis重启之后数据还在吗

5. 如何选择持久化方式

   4.0 以后选择混合持久化方式

6. 数据传输的时候网络断了或服务器断了，怎么办？

   持久化或哨兵

7. redis内存淘汰机制？

8. 哨兵机制原理？

   定时监视（三种定时）

   故障转移

   自动迁移

9. 哨兵进程的主要功能？是否保存数据？

10. Redis事务？“弱”事务？支持回滚吗？

11. lua如何解决Redis并发问题

12. 缓存穿透，缓存击穿，雪崩

13. 数据库与缓存一致性问题？常用的模式

14. mongodb与redis对比

    1. mongo也是内存非关系型数据库，会使用虚拟内存，所有的操作通过mmap的方式映射到内存中，避免了零碎的磁盘操作。然后mmap的内存刷新到磁盘中。当物理内存够用是，性能：redis>mongodb>mysql

