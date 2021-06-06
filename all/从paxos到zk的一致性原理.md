一致性模型

- 最终一致性
  - DNS，在服务器添加的一个域名后，域名会回传到上层根域名服务器，最后再由根域名服务器发送给其他子服务器
- 强一致性
  - 多数派。每次写都必须成功写入N/2个节点才算成功，每次读都从N/2个节点中读出数据
    - 问题：在并发场景中，因执行的顺序错误导致得到错误的结果。比如两个操作inc5，set0，部分节点先增后更新，部分节点先更新后增。

## 强一致性算法

### Paxos算法

**角色**

client：系统外部角色，请求发起者，不参与投票

proposer：接收client请求，向集群提出议题，类似议员

accpetor：提议投票和接收者，最后在提议经过多数人通过的情况下，才会被最终接受。类似国会

learner：提议接受者，备份，类似记录员

**阶段**

1. Phase 1a: Prepare。client提出申请，proposer接收提出一个提案，编号为N，此新编号必须大于proposer之前提出最大提案编号，请求accpetor接收
2. Phase 1b: Promise。每个accept接收提案，如果N大于之前接收的任何提案则通过，否则拒绝
3. Phase 2a: Accept。如果proposer通过的提案到达多数派，则会发出accept请求，此请求包括提案编号N，提案内容（前面步骤只发出提案编号）
4. Phase 2b: Accepted。如果此acceptor在此期间没有收到任何编号大于N的天，则接收此提案内容，否则忽略

**问题**

1. 部分acceptor节点异常，无法决策：通过多数派通过则通过的方式，保证分区容错性
2. proposer在Accept之前异常。解决：client重新发起请求，新的proposer接收此请求，重新拟定新提案，编号为N+1，内容不变，发起promise请求
3. 活锁。因acceptor只能同时讨论一个提案，当不断出现提案还未讨论完成，proposer又提出新提案，导致一直无法通过一个提案（可能两个proposer同时提出提案，但发现自己的提案不断被抢先，就立即重新提起，循环往复）。解决：控制等待时间，若acceptor正在讨论天，则等待一定时间再发起新提案，保证各自的提案都能顺利通过
4. 效率低，经历了两轮RPC，提案选举和提案+内容选举

### Multi Paxos算法

新增proposer leader，后续仅由它来提出提案，只经历议论RPC，直接将提案和内容发送给acceptor

**proposer leader选举**

proposer请求选举为leader，发送给acceptor，当收到多数派投票时即可成功选举，成功第N任leader

**client请求**

client请求proposer leader，leader提出提案I，发送给acceptor请求通过

### Raft算法

**角色**

leader：主写

follower：同步leader数据

candidator：选举人，当集群中不存在leader时，每个flower都可以成功候选人竞选leader

**集群写请求**：首先发送给leader写请求，leader同步给其他follower日志信息，当leader收到大部分follower ack后，leader成功写入，返回给客户端成功。并同步给其他follower日志提交

步骤演示：https://thesecretlivesofdata.com/raft/

raft官方演示：https://raft.github.io/

#### 问题

1. leader宕机选举问题。leader定时给follower发送心跳信息，follower启动定时监控，若一定时间没有收到leader的心跳信息，则认为集群中leader，则自己成功候选人准备竞选leader，N+1任，并向其他follower发送竞选请求，follower投票给第一个请求的候选人，当候选人获取多数票后则成为leader。若本次不存在得票超过半数的候选人，则候选人进入一个随机的等待时间（每个候选人的等待时间不一致），之后重新竞选拉票，直到选出新leader为止
2. 网络分区问题。若集群中有5个节点，其中发生了网络分区，leader和一个follower在一个分区，另外3个follower在另一分区。随后follower分区经过计时超时后发生无leader，成功竞选得到一个新leader，此时一个集群一分为二，得到两个leader。当集群收到写请求时，原follower集群可以成功写入，当原leader集群无法得到多数成功写入导致写入失败。当后续网络分区问题解决后，集群合二为一，以新leader为集群最终leader（因为新leader的任期更高）
3. 集群返回三个结果：成功、失败、未知。当集群节点个数少于原集群一般时，新的写请求无法成功执行，此时可能返回未知或超时，但这种情况集群在节点恢复后是可能成功写入的，只是超时了

### ZAB算法

zookeeper底层一致性算法。和raft算法类似，不过任期号raft使用term，zab使用epoch。对于心跳信息，raft中leader定时向follower发送心跳，zab中follower定时咨询leader是否存活





## 1、分布式架构

### 从ACID到CAP/BASE

#### CAP

一致性-C：保证分布式节点的数据一致性。分为强一致性和最终一致性

可用性-A：服务一直可用，对于客户端请求都能在一定时间内返回处理结果

分区容错性-P：系统存在多个副本，保证高可用性，但其中一个副本怪了其他副本仍然可用

放弃P：数据都放在单节点上，保证强一致性。

放弃A：当遇到网络分区或故障时，收到影响的服务要等待一定时间，即服务不可用

放弃C：放弃强一致性， 允许数据短暂的不一致性。常用

#### BASE理论

basically avaiable(基本可用)：允许损失部分可用性。响应时间上的损失：当出现故障时允许查询结果的响应时间增加的1-2s；功能上的损失，当大促期间抢购商品时，部分用户直接降级或熔断

Soft state(软状态)：也成为弱状态，允许系统的数据存在中间状态，允许不同节点间数据在同步过程中存在延迟

eventually consistent(最终一致性)：保证数据最终能达到一致的状态

## 2、一致性协议

### 2.1 2PC和3PC

#### 2.1.1 2PC

**事务询问**：协调者询问并锁定参与者相关资源。参与者锁定资源会一致性等待协调者的命令，提交或回滚，无超时限制

**等待响应**：协调者等待所有参与者返回ok。若存在一个参与者返回失败或超时，则向所有参与者发送回滚消息；否则发送事务提交消息

**事务提交**：参与者收到提交消息后，提交事务，并释放响应资源。返回ack

**完成事务**：协调者收到所有参与者ack之后，完成事务

2PC优点：原理简单实现方便

2PC缺点

1. 同步阻塞：参与者在一阶段询问之后锁定资源，必须等待协调者的二阶段消息才能提交或回滚，释放资源
2. 单点问题：协调者若出现问题，2PC阶段无法成功完成。特别是在一阶段参与者锁定资源之后，若协调者宕机则会导致参与者资源一直被锁定
3. 数据不一致：当协调者二阶段发送提交或回滚消息时，可能导致部分消息发送成功后协调者宕机，导致各参与者数据不一致
4. 太过保守：协调者进行事务提交询问时，只能等待参与者反馈或超时机制得到结果，没有合适的容错机制

#### 2.1.2 3PC

**CanCommit**：协调者询问参与者能否执行事务

**PreCommit**：参与者事务预提交，锁定事务资源。参与者存在超时机制，默认超时提交事务

**DoCommit**：参与者事务提交或回滚

## 3、Paxos的工程实践

### 3.1 Chubby

Google  Chubby是一个有名的分布式锁服务，GFS和Big Table等系统都是用它来解决分布式写锁、元数据存储和Master选举等相关分布式锁服务问题，Chubby底层以Paxos算法为基础

#### 3.1.1 概述

Chubby是一个由大量小型计算机构成的松耦合分布式系统，且提供粗粒度的分布式锁服务，直接调用chubby提供的接口即可实现分布式锁，无需关注底层的多个计算机组成。并且能够发布chubby服务器文件变更消息来完成事件通知

#### 3.1.2 应用场景

应用于集群的Master选举

#### 3.1.3 设计目标

quorum机制：过半机制，集群中只要有超过一般的机器正常运行，整个集群就可以对外提供服务

提供事件通知机制：chubby客户端需要实时的感知master服务器的变化，若通过客户端的轮询，当客户端不断增多时，对服务器的性能和带宽压力都会逐渐增大。chubby在master机器变化发送消息，通过订阅的形式感知变化

#### 3.1.4 Chubby技术架构

**Master选举**

通过投票过半的方式选出master，在租期期间，不会再由其他服务器称为master。master通过不断续租的方式来延长master租期。当master出现故障，才会选出新master

**目录与文件**

chubby的数据存储可以看成一个文件系统，通过文件+目录组成树结构。如/ls/foo/wombat/pouch

/ls时所有chubby节点共有的前缀，代表锁服务，时Lock Service缩写

/foo表示chubby集群的名字

其他部分/wombat/pouch表示业务含义的节点名字

chubby任意数据节点都可当成读写锁来使用

**事件通知机制**

chubby的客户端通过向服务端注册，当触发某些事件时，服务端会向客户端发送通知，常见事件如下

- 文件内容变更：BigTable集群使用Chubby来确定集群中的那台BigTable机器时master，其他客户端通过监视这个chubby文件的变化来确定master
- 节点删除：通过该特性监听判断客该临时节点对应的客户端会话是否有效

- 子节点新增、删除
- master服务器转移

**Chubby缓存**

chubby在客户端对文件内容和元数据信息进行缓存

当文件数据或元数据信息修改，chubby首先将其对应的缓存失效，然后再更新数据，保证缓存的一致性

**KeepAlive请求**

为客户端进行会话续租。Master在接收到客户端的keepalive请求时，首先阻塞请求，等到会话租期即将过期时才为其续租，并相应这个keepalive请求

通过keepalive响应来传递chubby事件通知和缓存过期通知给客户端

### 3.2 Hypertable

hypertable是一个使用C++开发的开源、高性能、可伸缩的数据库，其以google的Bigtable相关论文为基础知道，采用与HBase相识的分布式模型，构建一个针对海量数据的高并发数据库

#### 3.2.1 概述

仅支持基本的增、删、改、查功能，事务和关联查询尚不支持

hypertable优势如下

- 支持大量数据并发请求的处理
- 支持对海量数据的管理
- 扩展性良好，在保证可用性的前提下，进行添加机器的水平扩展
- 可用性极高，具有非常高的容错性，任何节点的失效不会造成系统瘫痪或影响数据数据完整性

#### 3.2.2 算法实现

Hyperspace是整个hypertable的核心

**Active Server**

hyperspae通常以一个服务集群的形式部署，一般由5-11个服务器组成。其中选举一个服务器作为Active Server（类似于master），其余为Standby Server

在hypertable启动初始化阶段，Master模块会连接上Hyperspace集群中的任意一台服务器，若服务器为active状态则初始化连接完成；若为standby状态，那么该standby服务器将active服务器发送给master模块，master重新发起连接。只有Active Hyperspace才能真正对外提供服务

**事务请求处理**

BDB集群，实现分布式数据一致性。Hyperspace对外提供服务室，任何对于元数据的操作，Master模块都会将对应的事务请求发送给Hyperspace服务器。

在收到该事务请求后，Hyperspace服务器会向BDB集群中的master服务器发起事务操作。

BDB服务器在接收该事务请求，会在集群内部发起一轮事务请求投票流程，当集群内超过半数的服务器成功应用该事务操作，就返回Hyperspace服务器成功

例：对于写请求，仅当BDB集群中超过半数的服务器执行成功，该事务才操作成功

**Active Hyperspace选举**

当Actibe Hyperspace机器宕机后则进行选举，事务日志更新时间越新的机器被选举的概率越高，保证数据的一致性

完成选举之后，Hyperspace服务器对应的BDB数据库的数据都需要和Master BDB保持一致

## 4、Zookeeper与Paxos

### 4.1 初始Zookeeper

#### 4.1.1 ZooKeeper介绍

采用ZAB一致性协议，可实现数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。ZooKeeper可以保存一下分布式一致性特性

- 顺序一致性：同一个客户端发送的事务请求，最终会按照严格的顺序应用到zk中
- 原子性：所有的事务要么应用到全部机器上，要么忽略该事务
- 单一视图：无论客户端连接zk的哪个服务器，其看到的数据都是一致的
- 可靠性：对数据的变更具有持久性
- 实时性：zk仅仅保证在一定的时间内，客户端最终一定能从服务端读取到刚刚写入的数据

**ZooKeeper的设计目标**

致力于提供一个高性能、高可用，且具有严格的顺序访问控制能力的分布式协调服务。具有四个设计目标

1. 简单的数据模型：通过共享的树形结构实现数据存储，类似于文件系统
2. 可以构建集群：一般由3-5台集群组成可用的zk集群。只要集群中超过一般的机器存活则认为集群可对外提供服务。自动重连：当zk的客户端与集群中任意一台创建TCP连接，而该服务器因一场断开，客户端会自动重连另一台服务器
3. 顺序访问：对于客户端的操作，zk通过分配全局唯一的编码，来控制所有事务的顺序
4. 高性能：zk的全量数据保存在内存中。以3台服务器作为集群测试，支持12W的QPS读请求

#### 4.1.2 ZooKeeper由来

雅虎创建，ZooKeeper引申义“动物园管理员”

#### 4.1.3 ZooKeeper基本概念

**集群角色**

Leader：单台，提供读和写服务

Follower：多台，提供读服务，参与leader选举，参与“过半写成功”策略

Observer：多台，提供读服务，仅用于同步、备份、读服务，不参与leader选举和“过半写策略”

**会话**

客户端与服务器之间通过建立TCP长连接进行通信，zk服务器对外端口默认2181

会话（Session）的sessionTimeout用来设置客户端会话的超时时间，在规定的sessionTimeout时间内，若客户端断开连接则自动重连另一服务端

**数据节点（Znode）**

数据树结构保存，每条数据保存在数据节点上，数据节点分为两种

- 持久节点：一旦被创建则永久保存，除非显示移除
- 临时节点：数据创建后，直到会话结束数据自动移除

**版本**

每个znode都会储存数据，且维护的数据的版本。分别记录了version（当前znode的版本）、cversion（当前Znode子节点的版本）和aversion（当前ZNode的ACL版本）

**Watcher**

时间监听器，允许用户注册一些watch，并在时间触发的时间，zk服务器会将时间通知到注册的客户端上

**ACL**

权限控制，类似于UNIX文件的控制权限，分为5种

- CREATE：创建子节点权限
- READ：获取节点和子节点的权限
- WRITE：更新节点的权限
- DELETE：删除子节点权限
- ADMIN：设置节点ACL的权限

#### 4.1.4 为什么选择ZooKeeper

开源、免费、应用广泛（Hadoop、HBase、Kafka）

### 4.2 ZooKeeper的ZAB协议

zk底层采用zab协议实现一致性

#### 4.2.1 ZAB协议

所有事务请求必须交由全局唯一的服务器协调处理，即leader服务器。leader服务器接收到请求后将事务转换为Proposal（提议），并分发给集群中follower，在一定时间内leader收到超过半数的反馈，则leader再次向所有follower发送commit消息，表示本次事务提交

#### 4.2.2 协议介绍

**消息广播**

leader服务器会为每个事务消息生成对应proposal进行广播，并为每个follower服务各自生成一个单独的队列来接收广播的proposal消息，根据FIFO策略进行发送

每个follower服务器在接收到这个事务proposal之后，都会将其以事务地址的形式写入到本地磁盘指哪个，并在写入成功后返回ack

但leader服务器接收到过半ack响应后，广播一个commit消息通知follower进行事务提交，同时leader一会完成事务的提交

**崩溃恢复**

一旦leader服务器崩溃或leader服务器失去了与过半follower的联系，就会进入崩溃恢复模式。然后重新选举leader

**基本特性**

选举机器中最大编码（ZXID最大）的事务Proposal作为leader，保证新选举出来的leader一定具有所有已经提交的提案

**数据同步**

ZXID：64位的数字。低32位可看做一个单调递增的计数器，针对的客户端的每个事务请求加一。高32位表示leader周期的epoch，每选举产生一个新leader服务器加一，并把低32位置0

#### 4.2.3 深入ZAB协议

##### 算法描述？？

术语介绍

整个ZAB协议主要包括消息广播和崩溃恢复两个过程，进一步可细分为三个节点

**阶段一：发现**

主要是leader选举过程，用于在多个分布式进程中选举处主进程，准Leader L和Follower F工作流程如下

**阶段二：同步**

在选举完成后，对外提供服务前，进行follower与新leader的数据同步。能够有效保证leader在新的周期提出事务proposal之前，所有进程都完成之前所有事务proposal的提交

**阶段三：广播**

完成同步阶段后，ZAB协议就可以正式开始接受客户端新的事务请求，并进行消息广播流程

##### 运行分析

在ZAB协议中，每个进程的状态

- LOCKING：leader选举阶段
- FOLLOWING：Follower服务器和Leader保持同步状态
- LEADING：leader服务器作为主进程领导状态

初始状态下，进程都处于LOCKING状态，此时进程组中不存在leader。随后尝试选举一个新leader，选举完成后对于LEADING状态进程称为leader，FOLLOWERING状态的进程为follower

当leader为获得半数以上follower心跳时，则状态切换为LOCKING，进行新一轮leader选举

#### 4.2.4 ZAB与Paxos算法的联系与区别

联系

- 两个都存在一个leader进程的角色，由其负责协调多个follower进程运行
- leader进程都会等待超过半数follower作出正确反馈后，才会将一个提案进行提交
- 在ZAB协议中，每个proposal包含了一个epoch表示leader周期。在paxos算法中同样存在该标识，用ballot表示

ZAB主要功能用于构建一个高可用的分布式数据主备系统，例如zk。而Paxos算法则用于构建一个分布式的一致性状态机系统

## 5、使用ZooKeeper

### 5.1 部署与运行

zookeeper使用java语言编写

#### 5.1.2 集群与单机

**集群模式**

1. 准备java运行环境，安装jdk
2. 下载zookeeper
3. 配置文件zoo.cfg

参数配置

- server.{id}={host}:{port}[:{port}]：表示集群机器地址信息，id为server id。同时在数据目录下创建爱你一个myid文件，该文件只有一行内容，并且是一个数字，即server id
- 在zk的设计，集群中所有机器zoo.cfg文件内容应保持一致
- myid：范围为1-255

4. 创建myid：在dataDir配置目录下创建一个名为myid的文件，文件仅一个数字表示机器id
5. 配置其他机器zoo.cfg和myid文件
6. 启动服务器
7. 验证服务器：telnet 127.0.0.1 2181

**单机模式**

zk支持单机部署

**伪集群模式**

在同一套机器上启动多台zk，注意端口要不一致

```sh
# zk配置文件
# zoo1.conf
ticketTime=2000
dataDir=/usr/local/Cellar/zookeeper/data1/
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890

# zoo2.conf
ticketTime=2000
dataDir=/usr/local/Cellar/zookeeper/data2/
clientPort=2182
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890

# zk启动
zkServer start conf/zoo1.conf
..
# 查看 zk状态（follower）
➜  zookeeper zkServer status conf/zoo1.conf
ZooKeeper JMX enabled by default
Using config: conf/zoo1.conf
Client port found: 2181. Client address: localhost. Client SSL: false.
```

#### 5.1.3 运行服务

### 5.2 客户端脚本

### 5.3 Java客户端API使用

#### 5.3.1 创建会话

- watcher：监听器
- canBeReadOnly：是否支持只读模式。在zk集群中，当leader与半数以上机器失去了联系则集群不可用。但在某些场景下，允许此类故障的发生，且能读取到数据，这就是只读模式
- sessionId和sessionPasswd：表示会话ID和会话秘钥，能够确定唯一一个会话，可达到会话复用的效果。
- sessionTimeout：session超时时间，单位毫秒。临时界面在超时关闭后才会删除
- Watch：可使用多次，监听回调

> 监听注册通知仅生效一次，当通知后监听器失效，若需要反复监听需手动注册

```java
static ZooKeeper connect() throws IOException {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        ZooKeeper zooKeeper = new ZooKeeper("localhost:2181,localhost:2182,localhost:2183",
                10000, new MyWatcher(countDownLatch));
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("zk synchronized success");
        return zooKeeper;
    }
static class MyWatcher implements Watcher {
        private CountDownLatch countDownLatch;

        public MyWatcher(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void process(WatchedEvent watchedEvent) {
            System.out.println("receive " + watchedEvent);
            if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {

                if (Event.EventType.None == watchedEvent.getType() && null == watchedEvent.getPath()) {
                    System.out.println("synchronized success");
                    countDownLatch.countDown();
                } else if (watchedEvent.getType() == Event.EventType.NodeDataChanged) {
                    try {
                        System.out.println("watch callback");
                        System.out.println(new String(zooKeeper.getData(watchedEvent.getPath(), true, stat)));
                        System.out.println(stat.getCzxid() + "," + stat.getMzxid() + "," + stat.getVersion());
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
```

#### 5.3.2 创建节点

- path：节点路径
- data：节点数据
- acl：acl策略
- cb：异步回调函数，需实现StringCallback接口
- ctx：传入一个对象，传递上下文信息

同步创建方法，无回调函数，创建后返回结果，当节点存在时，返回NodeExistException

异步创建方法，实现回调函数，函数返回字段

- rc：result code
  - 0：接口调用成功
  - -4：客户端与服务端连接已断开
  - -110：节点已存在
  - -112：会话已过期
- path：创建节点时输入的路径
- ctx：上下文
- real path：znode实际的路径

```java
static void create(ZooKeeper zooKeeper) throws KeeperException, InterruptedException {
        // 同步创建节点
        String path1 = zooKeeper.create("/test/server/p1", "v1".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE
                , CreateMode.PERSISTENT_SEQUENTIAL);
        System.out.println("success create znode :" + path1);

        // 临时节点等待session关闭后才会删除
        String path2 = zooKeeper.create("/test/server/p1", "v2".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE
                , CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println("success create znode :" + path2);

        // 异步创建节点
        zooKeeper.create("/test/server/p2", "v3".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE
                , CreateMode.PERSISTENT_SEQUENTIAL, new MyStringCallback(), "hello zk");

    }
```

#### 5.3.3 删除节点

仅允许删除叶子节点

删除时必须携带版本号，以确保删除正确的数据。当版本号传入-1时表示不校验版本号

```java
// 同步删除方法，当删除不存在的节点或节点存在但版本号不正确，则抛出异常
static void delete(ZooKeeper zooKeeper) throws KeeperException, InterruptedException {
//        zooKeeper.delete("/test/server/p2", 3);
        zooKeeper.delete("/test/server/p2", -1);
    }
```

#### 5.3.4 读取数据

- watch：一旦节点发生变化触发回调，仅可使用一次。实现AsyncCallback.DataCallback

```java
static void getData2(ZooKeeper zooKeeper) throws KeeperException, InterruptedException {
        String path = "/test/server/p3";
        zooKeeper.create(path, "123".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        // 监听上一事件
        zooKeeper.getData(path, true, new MyDataCallback(), null);
        zooKeeper.setData(path, "233".getBytes(), -1);
        // 监听上一事件
        //zooKeeper.getData(path, true, new MyDataCallback(), null);
    }
```

#### 5.3.5 更新数据

#### 5.3.6 判断节点是否存在

- watch：实现Watcher，可一直使用

无论节点是否存在，都可以注册

#### 5.3.7 权限控制

### 5.4 开源客户端

#### 5.4.1 ZkClient

github上开源的ZooKeeper客户端

- createPresistent：当父节点不存在时，创建父节点
- deleteRecursive：循环遍历删除节点

#### 5.4.2 Curator

开源zk客户端框架，是目前使用最广泛的zk客户端之一

自动注册watcher

## 6 ZooKeeper的典型应用场景

### 6.1 典型应用场景的实现

ZooKeeper作为核心组件，如Hadoop、HBase和Kafka等

#### 6.1.1 数据发布/订阅

发布/订阅系统分为推模式和拉模式，通常客户端都采用定时轮询拉取的方式

zk采用推拉结合的方式，客户端向服务端注册自己需要关注的节点，第一次拉取全部信息，后续节点的数据一旦发生变更，服务端就会向客户端发送Watcher事件通知，以更新为最新数据

订阅配置的信息的通常特性

- 数据量比较小
- 数据内容在运行时会发生变化
- 集群中机器共享，配置一致

#### 6.1.2 负载均衡

**动态DNS服务**

当域名和机器的绑定关系少且变化不频繁时，可以采用配置本地host方式实现域名和ip的映射，且方便调试

相反则需要通过动态配置的方式实现映射关系

**域名配置**

可通过zk实现域名配置及动态变更

- 域名配置：新建路径/DDNS/app1/server.app1.company1.com，节点指为机器ip列表，用空格分隔
- 域名解析：应用自己解析，通常先拉取到域名节点信息保存到本地，再通过watcher监听机制更新
- 域名变更：通过zk的事件更新通知实现域名对应机器信息更新

**自动化的DNS服务**

zk可以实现域名变更通知，但变更域名与机器映射关系仍需要手动修改，触发更新事件。zk提供自动化的DNS服务，以实现自动更新，DNS系统架构组件如下

- Register负责集群域名的动态注册
- Dispatcher集群负责域名解析
- Scanner集群负责检测与维修服务状态（探测服务的可用性， 屏蔽异常节点）
- SDK提供各种语言的接入协议，提供服务注册及查询接口
- Monitor负责服务信息收集以及DDNS自身状态的监控
- Controller时一个后台管理的console，负责授权管理、流量控制、服务配置等

**域名解析**

服务消费者使用域名时，会向Dispatcher发送域名解析请求，Dispatcher收到后会从zk上的指定域名节点读取响应的IP:PORT列表，通过一定的策略选取其中一个返回给前端

**域名探测**

指健康检查。健康监测分为两种：服务端主动发起健康检查，需要在服务端和客户端之间建立TCP长连接；客户端主动发起健康状态汇报

Scanner采用的是客户端主动进行健康状态汇报，一旦超过5s没有收到状态汇报，则认定该ip+端口不可用，进行域名清理过程

#### 6.1.3 命名服务

java中的JNDI（Java Naming and Directory Interface）是一种典型的命名服务

在分布式环境中，应用需要一个全局唯一的名字

zk通过顺序节点创建唯一元素

#### 6.1.4 分布式协调通知

