## rocketmq

### 1 概念

阿里双十一电商级消息中间件

- topic，消息主体，逻辑概念，在发布/订阅模式下，生产者和消费者消息传递的类别或标识。一个消息对应多个queue。每个broker可以存储多个topic的消息，每个topic的消息也可以分片存储在不同的broker
- tag，作为消息的第二级类型（topic是第一级）。可用于指定同一个topic下不同消息分组订阅
- nameServer，topic路由注册中心，支持broker的动态注册和发现。保存了broker，producer和consumer的地址信息，用来进行消息的路由转发。无状态节点，节点间无任何信息同步
- broker，mq中最核心的组件，主要负责消息的存储、投递和查询以及服务高可用性。broker和topic是多对多关系，因为一个topic对应多个queue，这个queue可能会分布在不同的broker上。broker不负责消息的推送。borker分为master和slave，一对多关系，borkerId为0表示master，非0为slave，master可以建立多个。每个broker与name server集群中的所有节点建立长连接，定时注册topic信息到所有name serve上。
- producer/ producer group消息生产者，producer负责把数据发送到指定某个queue中。与name server集群的一个节点建立长连接，定期获取topic路由信息，并向提供topic服务的master建立长连接，且定时发送心跳。无状态
- consumer/consumer group消息消费者，一个消费者可以消费多个消息队列，但一个消息队列同一时间只能被一个消费者消费。职责：拉取消息；消息进度的管理；集群下，从那个queue上拉去消息。与name server集群的一个节点建立长连接，定期获取路由信息，并向提供topic服务的master、slave建立长连接，定时发送心跳信息。可从maser和slave订阅消息，消费者在想master拉取消息时，master服务器会根据数据偏移量和最大偏移量（判断是否是老消息，产生I/O），以及从服务器是否可读建议下一次从master还是slave拉取

部署的集群方式有四种
-   单Master
- 多Master，配置简单。单台机器宕机时，该机器上未被消费的消息在恢复之前不可订阅
- 多Master多Slave（异步刷盘），每个Master配置一个Slave，消息采用异步复制方式，消息丢失的非常少，实时性不会受影响。
- 多Master多Slave（同步刷盘），每个Master配置一个Slave，消息采用同步双写方式，主从都写成功才返回成功。

  

####  回溯消费
指重新消费已经消费成功的消息。broker在想consumer投递消息成功后，还要保存消息一段时间，以支持回溯消费。例如consumer系统故障，重启后需要重新消费前一段时间的消息
#### 消息堆积
消息中间件的主要功能是异步解耦，还有挡住数据洪峰。消息堆积分两种

- 消息堆积在内存buffer，一旦超过内存buffer，采用一些丢弃策略来丢弃消息。性能下降不大
- 消息堆积在持久化存储系统中，性能影响大，会产生大量都IO
#### 消息分类
- 顺序消息

  - 全局顺序消息，对于某个topic下的所有消息都要保证顺序。严格按照FIFO的顺序进行发布和消费。性能要求不高
  - 分区顺序消息，对于一个指定topic，根据sharding key进行区块分区，同一个分区严格保证顺序。性能高

- 广播消息，发送的消息所有的订阅者都能收到。consumer.setMessageModel(MessageModel.BROADCASTING)

- 定时消息（延迟队列），指消息发送到broker后，不会被立即消费，等待特定时间投递给真正的topic。共18个等级1s-2h。message.setDelayTimeLevel(3)
  
- > 共18个等级：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h 
  
  - level = 0，消息为非延迟消息
  - 1<=level < maxLevel （当前版本18），延迟特定时间
  - level >maxLevel ， level=maxLevel
  
- 批次消息

#### 消息重试

消息消费失败后，提供重试机制。

设置maxReconsumeTimes，表示最大消费重试次数。当mq消费返回RECONSUME_LATER表示消费失败，准备重试。默认-1，重试16次

消息可以设置延迟消费等级msg.setDelayTimeLevel(1)，但对于一个消息来说，只有第一次设置是有些有效的。（比如设置等级3，后续几次延迟为10s 30s 1m ...）

#### 消息过滤
可根据tag进行消息过滤，也可通过sql过滤指定消息，对消息properties里的内容做限制

语法：https://rocketmq.apache.org/docs/filter-by-sql92-example/
```java
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");	
```
#### 消费方式
- 集群消费，consumer group下的实例平分消息
- 广播消息，consumer group下的实例都接收全量消息

#### 流量控制

**生产者流控，broker处理能力到达瓶颈**

- commitLog文件被锁定超过osPageCacheBusyTimeOutMills，默认1000ms
- 若开启transientStorePoolEnable=true，且broker为异步刷盘的主机，且transientStorePool中资源不足，拒绝当前send请求
- broker每个10ms检查队列头部的请求的等待时间，超过waitTimeMillsInSendQueue，默认200s，拒绝当前send请求
- broker通过拒绝send请求方式实现流控

**消费者流控，不会尝试消息重投。消费者流控的结果是降低拉取频率**

- 消费者本地缓存的消息数超过pullThresholdForQueue时，默认1000
- 消费者本地缓存的消息大小超过pullThresholdSizeForQueue是，默认100MB
- 消费者本地缓存的消息跨度超过consumeConcurrentlyMaxSpan时，默认2000

#### 消息刷盘

![img](./imgs/rocketmq_1.png)

- 同步刷盘，只有在消息真正持久化至磁盘后RocketMq的broker端才会真正返回给producer端一个成功的ACK响应。保障了消息可靠性，但影响一定性能	
- 异步刷盘，能否充分利用OS的PageCache的优势，只要消息写入PageCache即可成功返回ACK至producer端。消息刷盘采用后台线程提交的方式进行，降低了读写延迟，提高了mq的性能和吞吐量

#### 通信机制

(1) broker启动后需要完成一次将自己注册到NameServer的操作；随后每隔30s定时向nameServer上报Topic路由信息

(2) 消息生产者Producer作为客户端发送消息时，根据Topic从本地缓存获取路由信息。如果没有向NameServer重新拉去，同事每隔30s向NameServer拉取一次路由信息

(3) produer根据(2)中路由信息选择一个队列进行消息发送；Broker作为消息的接收者接受消息并落盘存储

(4) consumer根据(2) 中获取路由信息，并在完成客户端的路由负载均衡后，选择其中一个或几个消息队列来拉取消息并进行消费

#### 消息发送

发送种类

- 同步消息，发送完等待返回发送结果，可靠性高
- 异步消息，异步发送，提供异步调用函数
- 单向发送消息，无返回结果

返回结果，只要不跑出异常，就表示发送成功，得到结果状态

- SEND_OK，消息发送成功。发送成功并不确保时可靠的，要想消息不丢失，还需要开启同步master服务器或同步刷盘
- FLUSH_DISK_TIMEOUT，服务器同步成功但服务器刷盘超时，消息已进入服务器消息队列，只要服务器不宕机，消息就不会丢失。同步刷盘才可能发生这种情况
- FLUSH_SLAVE_TIMEOUT，消息发送成功但同步到slave超时，消息已进入服务器消息队列，只要服务器不宕机，消息就不会丢失。如果服务器的角色是同步master，即SYNC_MASTER （默认异步同步服务器即ASYNC_MASTER），并且从broker未在同步刷盘时间内（默认5秒）内完成与主服务器 的同步，则返回改状态
- SLAVE_NOT_AVAILABLE，消息发送成功，但slave不可用。master同步到slave时出错

producer发送消息失败的处理

- 至多重试2次（同步为2次，异步为0次）
- 如果发送失败，则轮转到下一个broker。该方法的总耗时时间不超过sendMsgTimeout（默认10s）
- 如果本身向broker发送消息超时，不会重试

#### 事务消息

提供commit和rollback，消息只要在commit之后才能被消费者发现并消息（实则一阶段提交时修改了消息topic，消费者接收不到消息）



### 2 消费者

#### 2.1 消费过程幂等

rocketmq无法避免消息重复，需要在消费时进行幂等操作

#### 2.2 消费速度慢的处理

**提高消费并发度**

- 同一个consumerGroup下，增加consumer实例数量（超过订阅队列数的无效）
- 对同一个cousumer，增加线程数量，通过consumerThreadMin和consumerThreadMax控制

**批量消费方式**

批量消费消息，通过consumer的consumeMessageBatchMaxSize消息消费数量，默认1。与生产者是否发送批量消息无关

**跳过非重要消息（可计算消息堆积量）、优化消费流程**

#### 2.3 消费者类型

**pushConsumer**

broker主动向consumer推送消息，consumer注册一个listener，一有消息推过来就进行消费。缺点是当消费速度小于推送消息的速度，会造成消息在broker的积压。一般使用的都是改种类型消费者

**pullConsumer**

consumer主动向broker拉取消息，比如自己控制轮询去向拉取型消费者中获取消息





### 疑问

1. broker的角色只有SYNC_MASTER、ASYNC_MASTER和SLAVE，slave会直接接收消息吗，接收后怎么向master及其他slave同步

2. pull消费和push消费者区别？正常配置的消费时哪种类型？

   > 如2.3。一般使用pushConsumer

   



## 参考

1. https://github.com/apache/rocketmq/tree/master/docs/cn 官方中文文档



