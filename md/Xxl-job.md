## Xxl-job

### 个人理解-清单

- 注册
  - acccessToken，访问令牌。用于客户端启动时调用服务端注册地址。客户端应与服务端配置的令牌一致。客户端启动时注册ip，关闭时移除注册ip
- XxlJobLogger
  - xxl-job专属日志，可在后管中查看，但不在后台日志中
- 避免任务重复执行
  - 调度密度高和耗时高的，小概率发生重复执行。可结合“单机路由策略（如：第一台、一致性哈希）”+“阻塞策略（如：单机串行、丢弃后调度）”来避免重复执行



### 任务jobinfo

路由策略，分片广播，调用所有注册机器

### 任务执行

```java
	/**
     *  
     * @param jobId 任务 id
     * @param triggerType 触发类型。MANUAL(手动执行)、CRON(corn表达式定时执行)、RETRY(重试)、PARENT(父任务带动子任务执行)、API(接口执行)
     * @param failRetryCount 失败重试次数
     * @param executorShardingParam 用于分片广播，目前无实际用处。值：${分片地址下标}/${分片地址总数}
     * @param executorParam 执行参数
     * @param addressList 机器执行地址
     */
JobTriggerPoolHelper.trigger(int jobId, TriggerTypeEnum triggerType, int failRetryCount, String executorShardingParam, String executorParam, String addressList)
```

- 执行任务，将任务添加到 指定机器任务运行队列。

- 一个线程一直扫描任务运行队列，执行任务。并将调用结果添加到回调队列中

- 一个线程一直扫描回调队列，将执行结果回调给服务端。若有子任务，则执行子任务。并将结果保存到日志中

  ### 任务调度

- 根据路由规则，循环时间，指定下次调度的时间。但是接受调度的客户端仅仅把调度请求放入队列中，循环扫描队列执行任务，无中断等待时间，多客户端堆积时触发重复调度问题



### 执行器 jobgroup







