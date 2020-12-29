## application配置文件

- key-value格式：yml中value前需要一个空格 key: xxx

## 插件

### 1 Actuator监控器

实现监控功能

```properties
# actuator监控端口号
management.server.port=9999
#监控根路径
management.server.servlet.context-path=/xxx
#监控基本路径
management.endpoints.web.base-path=/base

# 监控访问地址 ip:9999/xxx/base/health
```

#### 添加info信息

```yml
info:
	company:
		name: abc
		url: http://www.baidu.com
		addr: Beijing
    auth:
    	name: lisi
    	dep: develepment
```

获取info信息地址：ip:9999/xxx/base/info

#### 开放其他监控终端

默认只开放了health和info两个监控终端

> management.endpoints.web.exposure.include='*'  表示开放所有监控终端。['env', 'beans'] 开放指定监控终端

终端介绍

- /mappings：查看所有url与处理器的映射关系，详细的处理器方法机器映射规则
- /beans：查看当前应用所有bean对象信息
- /env：查看全部环境信息
- /env/{name}: 获取特定名称的环境信息
- /shutdown: post方法关闭应用程序。要求endpoints.shutdown.enabled设置为true
- /trace：提供基本的http请求跟踪信息（时间戳、http头等）
- /configprops: 模式配置属性如何注入bean

> management.endpoints.web.exposure.exclude=env,beans 单独管理某些终端

