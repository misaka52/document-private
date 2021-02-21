## 一、云计算&容器技术

### 1 基本组件

#### 1.1 Docker

容器技术，基于线程级别的隔离，操作系统内部进程与进程之间的隔离

常用于

1. 服务应用开发、测试、生产，可持续发布
2. pass，saas-云计算领域

云计算领域

1. iaas 基础设施服务--物理机，kvm。把IT基础设置当成服务通过网络提供给外部使用，如CPU、内存、磁盘等
2. paas 平台及服务。将服务器平台当成服务，以saas的方式提供给用户
3. saas 软件及服务，通过网络提供软件服务。saas供应商将应用软件部署在自己的服务器上，根据实际工作需求，如使用时间、使用资源来向用户计费

当docker容器非常多，难以管理时，可以使用kubernetes管理，利用编排技术

#### 1.2 kvm

kernel vm（linux内核，虚拟化技术），kvm是集成在linux内核的虚拟化技术。可以使用linux基于物理硬件进行虚拟化的隔离

1. 构建安全级别更高的私有云，必须使用kvm
2. 公有云，虚拟化技术
3. 混合云，虚拟化技术

当使用虚拟化技术部署了很多服务器，使用了很多的虚拟机怎么管理？可以通过openStack管理

#### 1.3 openStack

openStack用来构建云计算的基础设施层（网络、磁盘、cpu、内存）

#### 1.4 VMWare

虚拟化的pc端管理平台

### 2 容器的发展

#### 2.1 LXC

指的是Linux Containers，其通过Cgroups以及Linux namespace实现，是第一套linux容器管理实现方案

#### 2.2 Docker

Docker起初也是用LXC，后续利用自己的libcontainer库将LXC替换下来，其内部还是通过CGroups + Linux namespace实现

## 二、云原生

一种软件架构开发思想，为了让开发的软件、应用程序等运行在容器上

定义：容器化、微服务、可以动态调度

云原生特点

- 容器化：所有服务都部署在容器上
- 微服务：使用云原生，最好采用微服务架构
- devops：开发+运维结合体，一种敏捷开发思想，可实现项目持续交付，部署
- Ci/cd：可持续集成、可持续部署

云架构扩容思维

1. 云计算三个层次模式：iaas、paas、saas
2. caas：容器即服务，所有软件运行在容器中
3. faas：函数即服务，微服务架构
4. service mash 服务网络架构（服务限流、服务降级、服务监控、链路追踪...）
5. Serverless：少服务器或无服务器概念，开发者无需关系服务器资源

## 三、容器编排技术

### 1 应用部署变迁

当前应用部署一般分为三种

1. 物理机模式：应用放在物理机上，管理物理机
2. 虚拟化模式：应用放在虚拟机上，管理虚拟机。如openstack
3. 云端模式：应用放在容器上，管理容器。如kubernetes

三大主流容器平台：Swarm、Mesos和kubernetes，分别具有不同的容器调度系统

### 2 Swarm

Swarm是一个容器编排工具，docker公司自己开发的，但docker公司自己不用，而使用k8s

特点是直接调度Docker容器，并且提供与Docker一样的api

### 3 Mesos

mesos针对不同的运行框架采用相对独立的调度系统，其框架提供了Docker的原生支持

mesos并不负责调度而负责委派，由其他框架调度

### 4 kubernetes

采用pod和label的概念将容器组成一个个具有依赖关系的逻辑单元

kubernetes功能

- 自动化容器部署和复制
- 随时扩展和收缩容器规模
- 将容器组织成组，提供容器间负载均衡
- 提供容器弹性，可以简单替换失效容器

### 5 Docker-compose

Docker-compose是一个管理工具，可以批量创建容器，管理容器，可实现比较简单的容器管理

## 四、kubernetes相关概念

### 1 brog系统

google公司开发的一个服务管理软件，kubernetes的前身，k8s也是google开发的

![685359-20160410160946281-544441973](/Users/ysc/IdeaProjects/learning/document-private/image/685359-20160410160946281-544441973.png)

### 2 kubernetes

kubernetes是dokcer容器用来编排和管理的工具，它是基于docker创建的一个容器调度服务，提供资源调度，服务容灾，服务注册，动态扩缩容等功能套件

发送请求：kubectl客户端指令，浏览器（可视化方式：rancher、dashboard）

Master节点：scheduler调度器，负责将请求分发到那个node节点

controller：控制器，负责维护node节点资源对象

apiServer：网关，所有请求都要经过网关

etcd：服务发现，注册，管理集群状态信息、调度信息

node节点：每个node节点都运行着一个kubectl进程，负责本机服务的pod创建维护

registry：镜像仓库

> k8s集群包括一个master节点，一群node节点

### 3 node节点

![u=2288438080,519456419&fm=11&gp=0](/Users/ysc/IdeaProjects/learning/document-private/image/u=2288438080,519456419&fm=11&gp=0.jpg)

node节点中存在很多pod

Pod：k8s管理的最小单位，可以运行一个或多个容器，一般只运行一个容器，便于管理。可理解为一种容器或机器，内部包含docker

docker：docker引擎，pod内部的容器都是由docker创建的，docker引擎的node节点基础服务

kubelet：node节点代理，代理master节点请求，在本地节点执行

kube-proxy：网络代理，主要用来生成网络规则和四层负载均衡工作，生成访问路由。

fluentd：收集日志

### 4 master节点

#### 4.1 ApiServer

集群统一入口，各组件协调者，以http api形式提供给调用者，接收请求后再提交给etcd存储

#### 4.2 controllers

管理（增删改查）k8s资源对象

其中包含

- replication controller：副本控制器，实现副本数量和预设数量一致。当出现pod宕机，自动创建新pod以位置原有数量
- service controller：管理维护service（虚拟ip），提供负载以及服务代理
- endpoints controller：管理维护endpoints，管理service和pod。关系：service->Endpoints->pods

>  service通过laber标签选择器，找到对应pod

- Persisent Volume controller: 持久化数据卷控制器。
- Daemon set controller：让每个node节点都运行相同的服务
- Deployment controller：无状态服务部署的控制器
- node controller：管理维护node，定期检查node的健康状态，清理无效的namespace

#### 4.3 Scheduler

k8s的调度器

创建pod的流程

1. kubectl发起创建pod的指令，被apiservice拦截，把创建的pod存储在etcd中，存储podQueue
2. scheduler发起调度，被apiservice拦截，获取etcd中的podQueue.NodeList。调度算法：预选调度；优选策略
3. 把合适的node、pod存储在etcd
4. node节点上有一个kubectl的进程，发起请求获取创建node、pod所需要的资源
5. 若kubelet发现pod是本机节点需要创建的，则创建pod

### 5 Pod

pod内部封装了docker容器，拥有自己的ip地址，拥有hostname，相当于一个物理机，是一个虚拟化的容器

pod是k8s管理的最小单位，由k8s创建

建议一个pod运行一个容器，一个容器部署一个服务。集群就是复制多个pod

Docker引擎创建容器

![image-20210217162226594](/Users/ysc/Library/Application Support/typora-user-images/image-20210217162226594.png)

#### 5.1 pod创建过程

1. 创建pod（kubectl xxx）
2. 在pod中创建pause的容器（默认自动创建）
3. pod中创建其他容器

pause容器是k8s默认创建的容器，目的是创建虚拟共享网卡，以及共享数据存储卷。在pod由于共享pause提供的网卡，是的这些服务相互访可以使用localhost直接访问

pause容器与pod一一对应，pause为每个业务容器提供以下功能

- pid命名空间：pod中不同的应用程序可以看到其他的应用程序的pid
- 网络命名空间：pod中多个容器能访问同一ip和端口访问
- IPC命名空间：Pod中的多个容器能够使用SystemV IPC或POSIX消息队列进行通信。
- UTS命名空间：Pod中的多个容器共享一个主机名；Volumes（共享存储卷）：
- Pod中的各个容器可以访问在Pod级别定义的Volumes。

## 五、核心组件原理

### 1 RC控制器

replication controller：维护用户定义的副本数，当有容器异常退出自动新建副本补齐，有多出的容器也会自动回收

### 2 RS控制器

和rc差不多，并且支持集合式的selector

虽然RS可以单独使用，但建议和Deployment一起使用，来自动管理RS

Deployment支持滚动更新：先创建新pod，再删除老Pod

RC和RS的区别

- rc只支持单个标签选择器
- rs不仅支持单个标签选择器，还支持复合标签选择器

### 3 Deployment

RS可以控制副本的数量，但不支持滚动更新，只能手动更新

> 滚动更新：先创建新pod，再删除老pod

Deployment支持滚动更新，用于管理RS，RS管理Pod。更新模式是先创建一个新的RS，由RS创建新pod。回滚时重新创建老镜像

Deployment+RS提供的声明式的方法，用来代替RC，典型场景如下

1. 定义Deployment来创建pod和ReplicaSet
2. 滚动升级和回滚应用
3. 扩容和缩容
4. 暂停和继续Deployment

### 4 HPA

HPA在v1版本仅支持根据cpu的利用率进行自动扩容，在vlapha版本中，支持根据内存和用户自定义的mertic扩缩容

HPA仅适用于Deployment和ReplicaSet

### 5 StatefullSet

StatefullSet是用来部署有状态服务的（Deployment和RS是为了部署无状态服务的）

无状态

1. 没有实时的数据需要存储（即使有数据，也是一些静态数据）
2. 在服务集群扩容时，把其中一个服务拆离出去再添加进来，对服务集群无影响

有状态

1. 有实时的数据需要存储
2. 在服务集群扩容时，把其中一个服务拆离出去再添加进来，对服务集群有影响

statefullset应用场景包括

1. 稳定的持久化存储。即pod重新调度之后还能访问之前的持久化数据，基于PVC实现
2. 稳定的网络标识，即pod重新调度后hostname和podname不变，基于headless Service来实现
3. 有序部署，有序扩展，基于init container实现
4. 有序收缩，有序删除

pod可能随时宕机，当新建一个pod之后，新pod是否能找回原来的数据？

不能，这是全新的pod，新ip，新hostname

statefullset就是解决这种有状态服务部署的，当pod新建后，能保持原有的hostname和postname不变，访问原有的数据。不过需要有序部署和收缩，因为pod和pvc存储的数据时按照顺序一一对应的，比如pod1对应pvc1，pod2对应pvc2，若部署无序则对应关系会出错，找不到原有数据

### 6 DaemonSet

DaemonSet确保全部Node上运行一个Pod的副本。当有Node加入集群时，为其新增pod，当有Node删除时，回收这些pod。删除DaemonSet将回收其创建的所有Pod，典型应用

- 运行集群存储DaemonSet，如在每个Node上运行glustered, ceph
- 在每个Node上运行日志收集Daemon，如fluentd，logstash
- 在每个Node上运行监控Daemon。如Prometheus Node Exporter

### 7 Volume

数据集，共享容器使用的数据

### 8 Label

标签用于区分对象（如pod、service、node、RC等），以键值对形式存在于对象上。每个对象可以有多个标签

常用的label如下

```sh
版本标签："release":"stable","release":"canary"......
环境标签："environment":"dev","environment":"qa","environment":"production"
架构标签："tier":"frontend","tier":"backend","tier":"middleware"
分区标签："partition":"customerA","partition":"customerB"
质量管控标签："track":"daily","track":"weekly"
```

可以通过标签选择器来查询拥有指定标签的对象，实现了简单通用的查询功能

### 9 selector标签选择器

k8s的每个资源都可以有唯一的标签，标签的作用就是用来标识一个或一组资源对象，以便于通过标签选择器查找资源

标签以键值对形势存在于资源对象上

### 10 pod外网访问

pod像物理机一样虽然有自己的虚拟ip，但不能直接被外网访问，pod只是一个进程，没有物理实体，无法对外提供服务，通信，必须具有与外部通信的网卡

若pod想要与外部通信，需要借助Node节点。Node节点是物理机，在Node节点上开辟一个端口号，与pod（应该是一组pod，对应一个service）做好映射关系，外网通过访问node开辟的端口，请求转发到pod中，即实现了pod的对外访问

