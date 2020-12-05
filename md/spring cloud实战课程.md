### 2-1 广告系统概览

模块划分

1. 广告投放系统：用于投放广告，给广告主/媒体方使用，最前端系统
2. 广告检索系统：核心系统，接收入参检索出合适的广告
3. 曝光检测系统：广告系统的检测系统和第三方公司的检测系统。广告的播放时间决定于广告主支付的广告费，一般广告主会聘请第三方公司对广告的曝光情况做监控，最后用第三方系统和广告系统得出的数据作对比，得出准确的费用
4. 扣费系统：扣费算账。
5. 报表系统：做报表，数据收集



### 3 骨架开发

#### Zuul

微服务中往往包括很多的子系统或微服务，各个微服务一般不直接开放给调用端，都是通过网关路由到各个微服务

Zuul提供了服务网关的功能，负载均衡、反向代理、动态路由、请求转发等。大部分都是通过过滤器实现的

![img](http://wx2.sinaimg.cn/mw690/c5ab9976ly1fytq9dcy57j21280jqgmx.jpg)

- pre filters: 请求路由之前被调用
- routing filters: 在请求路由时被调用
- error: 处理请求时发生错误被调用
- post filters: 在rout和error过滤器之后被调用



### 11 广告检索服务

#### 11-2

- AdSlot 广告位类

### 12 Kafka

- 数据零丢失
- 分区有序
- 主和副本，副本不会消费数据
- broker服务器，接收生产者发送的消息并保存。供消费者消费

#### 12-2 kafka安装使用

进入kafka目录

```sh
# 启动zookeeper，通过jps查看，存在QuorumPeerMain程序表示启动成功
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties  
# 启动kafka服务
bin/kafka-server-start.sh config/server.properties
# 创建topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test-001
# 查看topic列表
bin/kafka-topics.sh --list --zookeeper localhost:2181
# 启动producer，指定topic
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-001
# 启动consumer，指定topic，从最开始处消费消息
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-001 --from-beginning
# 查看topic详细信息，副本，消费起始位置等
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test-001
```

#### 12-3 使用原生API发送消息

> 发送消息中，存在key-value，只消费value

**发送方式**

1. 只发送消息，不管发送结果。
2. 同步发送消息。发送函数返回Future结果，通过get获取发送结果，发送出错则抛出异常
3. 异步发送消息

**分区**

> 所有具有相同key的消息会分配到同一个分区。key为null时分配到默认的分区分配器

#### 12-7 使用原生api消费消息

消费方式

1. 自动提交位移。默认每隔5秒提交一次位移。ennable.auto.commit: ture， 开启自动提交位移
2. 手动提交当前位移。当发起提交时，消费者阻塞。当发送失败时，同步提交会发送重试，若失败一定次数则抛出异常
3. 手动异步提交当前位移。不会发生阻塞，但发送失败不会重试。异步提交重试的话，若之前发送失败的消费成功，则位移会回到之前消息的位置，导致位移回滚，消息重复消息。对于A，offset 2000，B offset 3000。若A失败了，重试再成功，offset发生回滚到2000，期间的消息会被重新消费
4. 手动异步提交当前位移带回调
5. 混合同步与异步提交位移。异步提交如果失败，则通过同步提交保证消息成功提交，比较常用。



|                            | kafka                                                        | rocketmq                                                     |      |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| Client SDK                 | Java, Scala, etc                                             | Java,C++,Go                                                  |      |
| Protocol and Specification | Push model, support TCP                                      | Push model, support TCP<br />JMS<br />OpenMessaging          |      |
| Ordered Message            | 确保分区有序（通过设置一个分区<br />达到全局有序）           | 可确保严格的顺序，并且优雅的扩展                             |      |
| 定时消息                   | 不支持                                                       | 支持                                                         |      |
| 批量消息                   | 支持，使用异步生产者                                         | 支持，使用同步模式以避免消息丢失                             |      |
| 广播消息                   | 不支持                                                       | 支持                                                         |      |
| 消息过滤                   | 支持，使用kafka流过滤                                        | 支持基于SQL92的属性筛选器表达式                              |      |
| 消息重试                   | 不支持，但是可以实现                                         | 支持。默认至多重试3次，若发送失败，<br />则轮转到下一个broker，总耗时<br />不超过10s |      |
| 消息存储                   | 高性能文件存储。内存、磁盘、<br />数据库，支持大量堆积       | 高性能、低延迟文件存储<br />磁盘，支持大量堆积               |      |
| 消息回溯                   | 支持偏移量指示。通过指定分区offset实现消息回溯               | 支持时间戳和偏移量两种                                       |      |
| 消息有限级                 | 不支持                                                       | 不支持                                                       |      |
| 高可用和故障切换           | 支持，需要ZooKeeper服务器                                    | 支持，主从模式，无需其他组件                                 |      |
| 消息跟踪                   | 不支持                                                       | 支持                                                         |      |
| 配置                       | 使用键-值形式配置                                            | 开箱即用，配置少                                             |      |
| 管理界面                   | 使用terminal命令                                             | 存在多功能的web页面。但不是项目<br />自带，需单独部署服务器支持 |      |
| 资料文档                   | 中等，有作者自己写的书                                       | 较少，官网文档简介                                           |      |
| 开发语言                   | Scala                                                        | Java                                                         |      |
| 事务消息                   | 支持                                                         | 支持                                                         |      |
| 集群方式                   | 主-从模式，每台服务器即是主也是从<br /> 集群依赖于ZooKeeper，ZooKeeper支持<br />热扩展，所有的broker、producer、consumer都可以动态加入移除，无需关闭服务 | 常用多对主-从模式，开源版本需手动将slave切换至master         |      |
| 吞吐量TPS                  | 极大，按批次发送和消费消息                                   | 大，消费端可配置批次的大小。但发送端不是批量发送             |      |
| 消息分发                   | 根据key来分发，若key为null，则将消息<br />轮询发送到不同分区中；若key不为null，<br />根据key的hashCode分发到不同分区中（可配置） | 轮询发送                                                     |      |

#### rocketmq

1. 定时消息。消息发送者可发送延迟消息（定时消息），工18级（1s-2h）。延迟消息发送到SCHEDULE_TOPIC_XXXX的消息队列，消息存在commitLog中，延迟消息不会被消费，再通过定时任务扫描将延迟消息队列中取出转为真正的消息发送。当消息延迟等级不为0时，都会投递到定时消息队列，等待定时任务扫描并转化重发，比如消费失败的消息。