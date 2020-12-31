[TOC]



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

接口介绍

- /mappings：查看所有url与处理器的映射关系，详细的处理器方法机器映射规则
- /beans：查看当前应用所有bean对象信息
- /env：查看全部环境信息
- /env/{name}: 获取特定名称的环境信息
- /shutdown: post方法关闭应用程序。要求endpoints.shutdown.enabled设置为true
- /trace：提供基本的http请求跟踪信息（时间戳、http头等）
- /configprops: 模式配置属性如何注入bean

> management.endpoints.web.exposure.exclude=env,beans 单独管理某些终端

## 第二章 重要用法

### 2.1 自定义异常页面

系统对应404，500 等固定异常默认映射到指定页面 src/resources/public/error

> 比如404映射页面 src/resources/public/error/404.html

### 2.2 单元测试

引入包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

测试类增加

```java
// 类注解
@SpringBootTest
@RunWith(SpringRunner.class)

// 方法注解
@Test
```

### 2.3 多环境选择

#### 2.3.1 多环境配置文件

application-{env}.yml/application-{env}.properties  env环境配置

默认配置文件application.yml，指定spring.profiles.active={env}

#### 2.3.2 @Profile

value表示环境，只在指定环境下才生效的类

#### 2.3.3 单配置文件实现多环境

通过---分割环境

```yml
spring:
	profiles:
		active: true
        
---
spring:
	profiles: dev
server:
	port: 8080
--- 
spring:
	profiles: prod
server:
	port: 80
```

### 2.4 自定义配置文件读取

通过@PropertySource读取，但该注解不支持yml文件。resources下需要添加classpath: 前缀

> @ConfigurationProperties 注解的类需要添加依赖set方法注入属性

### 2.5 Mybatis整合

1、maven包引入

```xml
<!--mybatis 与 spring boot 整合依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
<!--mysql 驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
<!-- druid 驱动 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.4</version>
</dependency>
```

2、配置文件

```yml
spring:
  datasource:
    username: root
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    password: 123456
mybatis:
  # 指定mapper文件地址
  mapper-locations: classpath:/mapper/*
```

tips

1. mapper类添加注解@Mapper
2. mapper.xml 
   1. 返回List对象，需指定resultMap

### 2.6 事务

1. 配置类增加注解@EnableTransactionManagement 开启事务
2. 类或方法上添加注解@Transactional，表示开启一个事务

### 2.7 使用logbak

springboot默认使用logback，spring-boot-starter-web 包中包含logback包

1. 应用配置文件配置

```yml
# logback配置哦
logging:
  # 指定格式及位置
  pattern:
    console: logs-%level||%msg%n
  level:
    root: info
    # 指定包日志级别
    com.ysc.springboot.runner.log: warn
```

2. 添加配置文件 src/main/resources/loback.xml

