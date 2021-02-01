[TOC]



## application配置文件

- key-value格式：yml文件中value前需要添加一个空格 key: xxx

## 插件

### 1 Actuator监控器

添加监控功能配置信息

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

springboot默认使用logback，spring-boot-starter-web 包中包含logback包。性能上logback要由于slf4j

1. 应用配置文件配置

```yml
# logback配置
logging:
  # 指定格式及位置
  pattern:
    # %t 调用线程名；@caller{1} 日志打印位置，1表示深度1
    # %contextName：上下文名称；@level：日志级别
    # %msg：日志信息；%n：换行符
    console: LOG-%t %d{yyyy-MM-dd HH:mm:ss.SSS}||%caller{1}||%contextName||%level||%msg%n
  level:
    root: info
    # 指定包日志级别
    com.ysc.springboot.runner.log: warn
```

2. 添加配置文件 src/main/resources/loback.xml （可选）

### 2.8 拦截器

#### 2.8.1 配置类方法

1、 新增拦截器，实现org.springframework.web.servlet.HandlerInterceptor

2、新增继承 org.springframework.web.servlet.config.annotation.WebMvcConfigurtaionSupport类，配置拦截器

```java
private final LogInterceptor logInterceptor;

@Override
protected void addInterceptors(InterceptorRegistry registry) {
  registry.addInterceptor(logInterceptor)
    .addPathPatterns("/test/**")
    .excludePathPatterns("/test/hello/**");
}
```

### 2.9 使用Servlet

在Servlet2.5版本，只能使用配置类方式；在Servlet3.0版本，新增注解方式

#### 2.9.1 配置类方式

1、继承javax.servlet.http.HttpServlet类，实现需要的接口

2、定义bean注册类，将servlet添加到注册类中

```java
@Bean
public ServletRegistrationBean<TestServlet> getServletBean() {
  TestServlet servlet = new TestServlet();
  return new ServletRegistrationBean<>(servlet, "/servlet2");
}
```

#### 2.9.2 注解方式

1、 继承javax.servlet.http.HttpServlet类，实现需要的接口；在类上添加注解@WebServlet("/servlet")

```java
@WebServlet("/servlet")
public class TestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();
        out.println("hello sprintboot servlet-annotation");
    }
}
```

2、配置类上添加注解开启指定包servlet扫描

```java
@ServletComponentScan("com.ysc.springboot.servlet")
```

#### 2.10 使用filter

过滤器，Servlet2.5版本只能使用配置类方式；Servlet3.0版本，新增注解方式

**过滤器和拦截器**

容器包裹（前者包裹后者）：tomcat > filter > servlet > interceptor > controller

> 过滤器过滤在拦截器prehandler之前处理

1. 拦截器基于java的反射机制实现；过滤器基于函数回调实现
2. 拦截器不依赖servlet容器（不能拦截servlet）；过滤器依赖
3. 拦截器可以获取IOC容器中的bean，过滤器不行
4. 拦截器只对请求拦截，而过滤器可以拦截进入过滤器的所有的请求，比如静态资源的请求
5. 拦截器是spring的，过滤器是在servlet容器的

### 2.11 模板引擎

官方推荐使用的模板引擎

**配置方法**

1、引入配置包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2、配置application

```yml
spring:
	thymeleaf:
    # 开发阶段关闭缓存
    cache: false
    # 文件前缀，默认classpath:/templates/
    prefix: classpath:/templates/
    # 文件后缀，默认.html
    suffix: .html
```

3、生成controller，添加注解@Controller。返回文件名，自动定位到文件

```java
@Controller
@RequestMapping("index")
public class IndexController {
    @GetMapping("")
    public String index() {
        return "index";
    }
}
```

### 2.12 Redis集成

### 2.13 Dubbo集成

### 2.14 自定义starter

- spring官方定义的starter包命名：spring-boot-starter-{name}
- 非官方定义的starter命名：{name}-spring-boot-starter

#### 1. 注解解释

**@EnableConfigurationProperties**

获取配置类，该方式指定的配置类，无需被spring ioc容器管理

**@ConditionalOnClass**

当指定类存在时生效

**@ConditionalOnBean**

当指定bean存在时生效

**@ConditionalOnMissingBean**

当bean确实时生效

**@ConditionalOnProperty**

当属性满足条件后生效

- value：指定属性名
- matchIfMissing：当属性缺失时匹配结果，默认falise
- havingValue：指定匹配值，当属性为该值时匹配结果为true。默认""，当不指定该属性时，匹配结果一定为true

**@Import**

直接将指定类导入到IOC容器中，可以导入一般类（非ioc容器管理类）和三方jar包类

**@Inherited**

指使用该注解的类的子类，能自动继承该注解。不如B继承了A，A上存在@Inherited的注解，B能直接拥有

#### 2. 代码样例

引入包

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
  <version>2.1.5.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <version>2.1.5.RELEASE</version>
  <optional>true</optional>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure-processor</artifactId>
  <version>2.1.5.RELEASE</version>
  <optional>true</optional>
</dependency>
```

```java
package com.ysc.exercise;

import com.ysc.exercise.service.WrapConfiguration;
import com.ysc.exercise.service.WrapService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

// 自动注入类
@Configuration
@EnableConfigurationProperties({WrapConfiguration.class})
@ConditionalOnClass(WrapService.class)
public class WrapAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(value = "spring.wrap.enable", matchIfMissing = true, havingValue = "true")
    public WrapService registerWrap(WrapConfiguration wrapConfiguration) {
        return new WrapService(wrapConfiguration.getPrefix(), wrapConfiguration.getSuffix());
    }

    /**
     * 默认注入
     * @return
     */
    @Bean
    @ConditionalOnMissingBean
    public WrapService registerWrap2() {
        return new WrapService("", "");
    }

}
```

META-INF/spring.factories

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.ysc.exercise.WrapAutoConfiguration

# 多配置样例
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.ysc.exercise.WrapAutoConfiguration,\
com.ysc.exercise.WrapAutoConfiguration2
```

#### 3. 源码 研究

**@EnableXXX**

该注解表示开启某一项功能，为了简化代码的导入，使用了该注解，就能自动导入某些类，一般是组合注解。被导入的类分为三种：配置类、选择器、注册器

- 配置类：@Import中指定的类一般以Configuration结尾，并且携带@Configuration注解， 
- 选择器：@Import中指定的类一般以Selector结尾，且实现了ImportSelector接口，如@EnableAutoConfiguration
- 注册器：@Import中指定的类一般以Register结尾，且该类实现了接口，用于导入注册器。该类可以在代码运行时动态注册指定类的实例

starter加载

1. 启动类注解@SpringBootApplication中携带注解@EnableAutoConfiguration，具体选择器AutoConfigurationImportSelector
2. AutoConfigurationImportSelector.getAutoConfigurationEntry -> getCandidateConfigurations() -> loadFactoryNames() -> loadSpringFactories()

