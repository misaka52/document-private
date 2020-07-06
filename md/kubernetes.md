### 简介

Kubernetes是容器集群管理系统，可实现容器集群的自动化部署、自动扩缩容、维护等功能

特点

- 可移植：支持公有云、私有云、混合云、多重云
- 可扩展：模块化、插件化、可挂载、可组合
- 自动化：自动部署、自动重启、自动复制、自动伸缩/扩展

 容器优点

- 快速创建/部署应用：与VM虚拟机相比，容器镜像的创建更加容易
- 持续开发、 集成和部署：提供可靠且频繁的创建、部署，并使用快速和简单的回滚（由于镜像不可变性）
- 开发和运行相分离：在build或release阶段创建容器镜像，是的应用和基础设施解耦
- Loosely coupled，分布式，弹性，微服务化：应用程序分为更小的、独立的部件，可以动态部署和管理
- 资源独立、隔离
- 资源利用：更高效

### 架构

提供了容器集群部署和管理系统，目的旨在消除编排物理/虚拟计算，网络和存储基础设施的负担。kubernetes也提供稳定、兼容的基础（平台），用于构建定制化的workflows和更高级的自动化任务

#### Brog简介

大规模集群管理系统，负责对谷歌内部很多应用的调度和管理

![borg](https://feisky.gitbooks.io/kubernetes/architecture/images/borg.png)

组成：

- BorgMaster为整个集群的大脑，负责维护整个集群的状态，并将数据持久化到paxos中
- scheduler负责任务的调度，根据应用的特点将其调度到具体的机器上去
- Borglet负责真正运行任务（在容器中）
- brogcfg是Brog的命令行工具，用于更Brog系统交互

#### kubernetes架构

![architecture](https://feisky.gitbooks.io/kubernetes/architecture/images/architecture.png)

核心组件

- etcd保存整个集群的状态
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现的机制
- controller manager负责管理集群的状态，比如故障检测、自动扩展和滚动更新等
- scheduler负责资源的调度，按照预定的调度策略将pod调度到相应的机器上
- kubelet负责维护容器的生命周期，同时负责Volume（CVI）和网路（CNI）的管理
- Container runtime负责镜像管理以及pod的真正运行（CRI）
- kube-proxy负责Service提供cluster内部的服务发现和负载均衡

add-one：

- kube-dns负责为整个集群提供DNS服务
- Ingress Controller为服务提供外网入口
- Heapster提供资源监控
- Dashboard提供GUI
- Federation提供跨可用区的集群
- Fluentd-elasticsearch提供集群日志采集、存储和查询

#### ![14791969222306](https://feisky.gitbooks.io/kubernetes/architecture/images/14791969222306.png)

![14791969311297](https://feisky.gitbooks.io/kubernetes/architecture/images/14791969311297.png)

#### 分层架构

类似linux的分层架构，如下图：

![14937095836427](https://feisky.gitbooks.io/kubernetes/architecture/images/14937095836427.jpg)

- 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层：系统调度（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层：kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可分为两个范畴
  - Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等
  - Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

### 设计理念

API设计原则

1. 所有API应是声明式的。更容易使用，扩展性强
2. API对象彼此互补是可组合的。即高内聚、低耦合
3. 高层API以操作意图为基础设计。从实际业务场景出发
4. 底层API根据高层API的控制需要设计
5. 尽量避免简单封装，不要有在外部API无法显式知道的内部隐藏的机制
6. API操作复杂度与对象数量成正比，操作复杂度不能超过O(N)
7. API对象不能依赖于网络连接状态，分布式系统经常网络连接断开
8. 尽量避免让操作机制以来与全局状态，分布式系统很难维护全局状态

控制机制设计原则

1. 控制逻辑应该只依赖于当前状态
2. 假设任何错误的可能，并做容错处理
3. 避免复杂状态机，不要依赖无法监控的内部状态
4. 假设任何操作都可能被任何操作对象拒绝，设置被错误解析
5. 每个模块都可以在出错恢复
6. 每个模块都可以在必要时自动降级

#### Master组件

提供集群的管理控制中心。可以在集群的任何节点上运行，但为简单起见，通常在VM/机器上启动所有Master组件，且不会再此VM/容器上运行用户容器

#### kube-apiserver

用于暴露Kubernets API

#### ETCD

是Kubernetes提供的默认存储系统，保存所有集群数据，使用时需要为etcd数据提供备份计划

#### Kube-controller-manager

运行管理控制器，是集群中处理常规任务的后台进程。每个控制器是单一的进程，都别编译成单个二进制文件

控制器包括：

- 节点控制器
- 副本控制器：负责维护系统中每个副本的pod
- 端点控制器：天成Endpoints对象（即链接Service & Pods）
- Service Account和Token控制器：为Namespace创建默认账户访问API Token

#### cloud-controller-manager

负责与底层云提供商的平台交互

具体功能：

- 节点控制器
- 路由控制器
- Service控制器
- 卷控制器

#### kube-scheduler

监视新创建没有分配到Node的Pod。为pod选择一个Node

#### 插件addons

是实现集群pod和Services功能的

### Pod

#### Pod安全策略

能够控制pod的行为，以及访问什么的能力



