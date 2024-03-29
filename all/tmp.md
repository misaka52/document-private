京喜业务系统（商城融合平台）

对接商城商家签约、承保、理赔、数据管理等功能

从职责上划分为前端、前台、后台

前端：负责交互ui展示，前后端分离，前端与前台进行交互。目前包括京东商家使用的商家pc后台；京东用户使用的h5平台和pc端平台；供内部员工使用的后台管理系统

前台：与前端进行交互。包括登录态校验，登录态获取，权限认证，防刷处理，参数组装等

后台：负责对外提供服务，满足签约、承保、理赔等功能



组件：

- 数据库：mysql-存储主要数据
- mongodb：主要存储日志数据
- 缓存：redis-保存缓存数据
- 文件存储：京东云对象存储
- 消息队列：rocketmq，jmq-接收商场订单完成数据、订单出库数据、理赔服务单数据
- 配置中心：使用apollo统一配置中心
- 定时调度：使用xxl-job进行分布式调度
- 应用监控：k8s+健康检查监控

 

前台

- 商家后台-shop前台
  - 签约管理
  - 理赔数据管理
  - 类目管理、协议查看、品类查看、费率查看、轮播图片
- h5端前台
  - 查询订单详情
  - 保障商品详情
  - 上传文件
  - 预计理赔金额查询
  - 理赔详情
- 商家后台管理
  - 批量商家签约
  - 签约商家管理
  - 保险产品管理
  - 承保信息查询
  - 服务单查询
  - 理赔审核管理
  - 申诉审核管理
  - 类目规格设置

后台

- 保障服务
  - 订单基础信息查询
  - 保单基础信息查询
  - 定价查询
  - 赔付率获取
  - 保费获取
  - 类目管理
  - 广告管理
  - 品类管理
  - 操作日志管理
  - 费率管理
  - 定价服务（查询保费、赔付率、预计理赔金额）
- 理赔服务
  - 运费险
  - 优品管运费险
  - 价保险
  - 正品保障
  - 先用后付
- 订单过滤服务
  - 出库消息

  - 订单完成消息

    - 过滤逻辑：接mq消息，按照全量商家id+ 商家签约信息+类目维度+spu维度过滤，过滤成功后落库订单表，若遇到重复键异常则结束

  - 接mq消息规划及展望

    - 问题：按照现在的方式，需要每种产品都接一套mq消息，而mq消息量又非常大，资源消耗过大。

    - 计划方案：

      - 表结构设计：设计待承保订单消息表（订单号+消息类型+险种标志位+json数据，订单号+消息类型作为索引）、承保订单消息表（订单号+消息类型+险种标志位，分表，用来保存全量承保订单消息）

      - > 险种标志位：按照位数组的设计思路，每个险种分配一位，标志位用long类型保存，方便计算（直接进行位运算）。为险种分配指定位，当指定位为1时表示存在某个险种。后续采用惰性删除或定时清理的方式，将标志位为0的数据从待承保订单消息表中移除

      - 接到mq消息时，1、结果全量商家本地缓存过滤；2、再通过险种及消息订阅关系表过滤；3、商家险种签约信息；4、类目信息；5、获取承保险种，填充险种标志位；6、保存到承保订单消息表，若遇重复键异常则结束（幂等），再保存到待承保订单消息表

      - 通过定时任务扫描待承保订单消息表：扫描一批数据，对订单号及消息类型添加分布式锁，然后将每条数据作为一个任务，放入线程池中处理，定时调度结束。处理订单消息时，进程风控校验、折扣计算、实付金额获取、赔付率计算、保费计算，最后落库，建立T+1推送任务

- 签约服务
  - 签约（签约条件控制、签约work、签约信息查询）
  - 批量签约
  - 解约、解约work、到期停约
  - 续约、自动续约、手动续约、续约提醒

靓点

- 使用mq，异步解耦，削峰，承接京东商城全量出库订单、完成订单、服务单。线程池配置
- 缓存设计
  - 采用先更新数据库，在删除缓存的方式保证数据库与缓存的一致性
  - 缓存失效时如何防止缓存穿透
    - 设置随机过期时间，key过期时间设计为固定时间+随机时间，避免某一时刻大量缓存同时失效
    - 增加同步锁（单机锁或分布式锁）
    - 缓存标识+缓存。缓存时间为缓存标识的两倍有效期，缓存标识用来更新缓存，防止缓存过期
  - 防止缓存击穿：对不存在的做特殊值缓存，避免击穿数据库
  - 缓存预热：暂无使用。可能在项目启动时提前保存好热点数据
- 使用本地缓存+redis多级缓存过滤消息
  - 本地缓存：chm，key-商家id，value-CopyOnWriteArrayList\<自定义类（有效期区间，时间戳）>，更新有效期时直接替换新对象
    - 项目初始化缓存预热，加载所有的商家有效期信息
    - 更新：通过rocketMQ广播模式，当商家签约信息变更时，通过mq更新
  - redis多级缓存
    - 商家id及对应的签约有效期
    - 商家+险种的特殊类目
    - 险种对应的承保类目
- 通过本地消息表实现 消费幂等性及统一消费
  - 之前每个险种都接一种mq，导致同一mq被多个险种订阅，
- 状态机设计。为解决复杂的状态流转
- 签约流程配置化
- 大健康批处理流程设计及开发



待优化点

- 保证数据库与缓存的一致性
  - 原：事务提交后，发送mq，mq消费端删除缓存
  - 优化：通过监听binlog日志来完成数据库更新后删除缓存的操作，解耦
- 数据库读写分离
- 分布式锁优化
  - redis分布式锁高可用设计，Redlock
  - zookeeper分布式锁引入
- 历史数据归档或清理
- 常用数据归档