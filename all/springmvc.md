[TOC]

# 基本介绍

## 基础概念

### BS和CS开发架构

- BS架构，浏览器-服务器架构
- CS架构，客户端-服务器架构

> 目前大部分Java开发都是基于BS架构的，应用系统分为标准三层架构：表现层、业务层、持久层

JaveEE制定了一套规范，进行BS架构的处理，这套规范就是Servlet

### 应用系统三层架构

- 表现层
  - web层，负责接收客户端请求，返回相应结果。通常客户端使用http协议请求web层
  - 表现层包含展示层和控制层：控制层负责接收请求
  - 表现层接到请求后会调用业务层
- 业务层
  - service层，负责处理业务逻辑。业务层可能依赖持久层，获取数据
- 持久层
  - dao层，负责数据持久化

### MVC设计模式

- model（模型）：包括业务模型和数据模型，数据模型用于封装数据，业务模型用于处理业务
- view（视图）：展示页面。通常指jsp或html
- controller（控制器）：处理业务逻辑

## SpringMVC介绍

SpringMVC是一种基于MVC设计模式的请求驱动类型的轻量级web框架，属于springframework的产品。是目前比较主流的MVC框架，已超越struct2

通过一套注解，能简单的编写请求控制器，支持Restful的请求

### 六大组件说明

#### 前端控制器 DispatcherServlet

用户请求到达前端控制器，它就相当于mvc模式中的C，dispatcherServlet是整个流程控制的中心，

由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

#### 处理器Handler

Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。

由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。

#### 视图View

#### mvc流程处理

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用处理器映射器HandlerMapping
3. 处理器映射器根据请求url找到具体的处理器，生成处理器执行链HandlerExecutionChain（包括处理器对象和处理器拦截器）一并返回给DispatcherServlet
4. DispatcherServlet根据Handler获取处理器适配器HandlerAdapter执行HandlerAdapter处理一系列的操作。如参数封装，数据格式装换，数据验证等操作
5. 执行处理器Handler（Controller，也叫页面控制器）
6. Handler执行完成返回ModelAndView
7. HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
9. ViewReslover解析后返回具体View
10. DispatcherServlet对View进行渲染视图（即将模型数据model填充到视图中）
11. DispatcherServlet响应用户

# 项目搭建

## 入门工程

添加mvc依赖、jstl依赖和servlet-api依赖

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ysc</groupId>
    <version>1.0.SNAPSHOT</version>
    <artifactId>springmvc</artifactId>
    <packaging>war</packaging>

    <dependencies>
        <!-- spring MVC依赖包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.0.7.RELEASE</version>
        </dependency>

        <!-- jstl -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!-- servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>
    <build>
        <plugins>
            <!-- 配置Maven的JDK编译级别 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <path>/</path>
                    <port>8080</port>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

### 配置文件

添加web.xml文件至目录 webapp/WEB-INF/

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns="http://java.sun.com/xml/ns/javaee"
   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
   version="2.5">
   <!-- 学习前置条件 -->
   <!-- 问题1：web.xml中servelet、filter、listener、context-param加载顺序 -->
   <!-- 问题2：load-on-startup标签的作用，影响了servlet对象创建的时机 -->
   <!-- 问题3：url-pattern标签的配置方式有四种：/dispatcherServlet、 /servlet/* 、* 、/ ,以上四种配置，优先级是如何的 -->
   <!-- 问题4：url-pattern标签的配置为/*报错，原因是它拦截了JSP请求，但是又不能处理JSP请求。为什么配置/就不拦截JSP请求，而配置/*，就会拦截JSP请求 -->
   <!-- 问题5：配置了springmvc去读取spring配置文件之后，就产生了spring父子容器的问题 -->

   <!-- 配置前端控制器 -->
   <servlet>
      <servlet-name>springmvc</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <!-- 设置spring配置文件路径 -->
      <!-- 如果不设置初始化参数，那么DispatcherServlet会读取默认路径下的配置文件 -->
      <!-- 默认配置文件路径：/WEB-INF/springmvc-servlet.xml -->
      <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:springmvc.xml</param-value>
      </init-param>
      <!-- 指定初始化时机，设置为2，表示Tomcat启动时，DispatcherServlet会跟随着初始化 -->
      <!-- 如果不指定初始化时机，DispatcherServlet就会在第一次被请求的时候，才会初始化，而且只会被初始化一次 -->
      <load-on-startup>2</load-on-startup>
   </servlet>
   <servlet-mapping>
      <servlet-name>springmvc</servlet-name>
      <!-- URL-PATTERN的设置 -->
      <!-- 不要配置为/*,否则报错 -->
      <!-- 通俗解释：/*，会拦截整个项目中的资源访问，包含JSP和静态资源的访问，对于静态资源的访问springMVC提供了默认的Handler处理器 -->
      <!-- 但是对于JSP来讲，springmvc没有提供默认的处理器，我们也没有手动编写对于的处理器，此时按照springmvc的处理流程分析得知，它短路了 -->
      <url-pattern>/</url-pattern>
   </servlet-mapping>
</web-app>
```

添加springmvc.xml文件，DispatcherServlet启动会加在该文件，上面已配置。地址resources/

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:mvc="http://www.springframework.org/schema/mvc"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 配置处理器Bean的读取 -->
   <!-- 扫描controller注解,多个包中间使用半角逗号分隔 -->
   <context:component-scan base-package="com.ysc.controller"/>
    
    <!-- 配置三大组件之处理器适配器和处理器映射器 -->
    <!-- 内置了RequestMappingHandlerMapping和RequestMappingHandlerAdapter等组件注册-->
    <mvc:annotation-driven />   
    
    <!-- 配置三大组件之视图解析器 -->
    <!-- InternalResourceViewResolver:默认支持JSP视图解析-->
    <!-- 完整路径：   /WEB-INF/jsp/xxx.jsp -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/jsp/" />
      <property name="suffix" value=".jsp" />
   </bean>
</beans>
```

### 配置说明

Mvc:annotation-drivern

```
- ContentNegotiationManagerFactoryBean
- RequestMappingHandlerMapping(重要)
- ConfigurableWebBindingInitializer
- RequestMappingHandlerAdapter(重要)
- CompositeUriComponentsContributorFactoryBean
- ConversionServiceExposingInterceptor
- MappedInterceptor
- ExceptionHandlerExceptionResolver
- ResponseStatusExceptionResolver
- DefaultHandlerExceptionResolver
- BeanNameUrlHandlerMapping
- HttpRequestHandlerAdapter
- SimpleControllerHandlerAdapter
- HandlerMappingIntrospector
```

### 编码部分

#### Controller

处理器开发方式有很多

- 实现HttpRequestHandler接口
- 实现Controller接口
- 使用注解@Controller（推荐）

> @Controller注解的类中的每个@RequestMapping注解的方法，最终都会转换为HandlerMethod类，这个类就是springmvc真正意义上的处理器

