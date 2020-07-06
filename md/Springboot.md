### 参考

学习视频：https://www.bilibili.com/video/BV1Et411Y7tQ?p=1





### 1.项目配置文件application.properties

#### 多环境
application-{profile}.properties，指定各环境配置文件，默认使用application.properties。通过spring.profile.active属性指定激活环境配置
#### 配置文件加载顺序
application.properties配置文件加载顺序。按照下面从上往下顺序加载，前者优先级更高
- {项目路径}:/config/
- {项目路径}:/
- {classpath, resources资源文件地址}:/config
- {classpath, resources资源文件地址}:/
#### 外部配置文件加载顺序
application.properties外部配置加载顺序，前者优先级更高。（下面仅列举常用的几项）
**1.命令行参数**
java -jar server.port=8082

**2.配置文件：jar包外部，优先**

**3.@Configuration注解类上的@PropertySource**

**4.通过Application.setDefaultProperties指定的默认属性**

#### 配置文件自动加载

@SpringBootApplication注解中包含@EnableAutoConfiguration，

@EnableAutoConfiguration引入AutoConfigurationImportSelector.class

AutoConfigurationImportSelector.class会载入META-INF/spring.factories文件的属性，作为配置属性加载到项目中

```java
@ConditionalOnMissingBean  // 但bean不存在时才生效。不写任何参数默认类型。可指定bean名称等
@EnableConfigurationProperties // @ConfigurationProperties注解bean是不会加载成到spring容器里的。配合@EnableConfigurationProperties或@Component才能注到容器中。jar包中用@EnableConfigurationProperties注入
```

**aotuCofiguration，基本每个配置都有一定的条件，可通过添加配置debug=true，运行项目查看生效的配置类**

```java
============================
CONDITIONS EVALUATION REPORT
============================
  Positive matches: //生效配置
-----------------
  Negative matches: //不生效配置
-----------------
```



### 2.日志

Slf4j，日志的抽象层。具体实现有logback、log4j、log4j2。但是Slf4j是比部分日志较晚实现的，所以部分实现中间增加了适配层，即sfl4j的实现是调用老的日志实现。

![concrete-bindings](/Users/ysc/Documents/learning/md/imgs/concrete-bindings.png)

**日志转换成Slf4j**：将项目中其他非Slf4j的日志包排除，增加部分适配的包（适配包中包含排除日志包的所有类，并且实现是调用Slf4j）。

![legacy](/Users/ysc/Documents/learning/md/imgs/legacy.png)

| logging system | Custumization                   |
| -------------- | ------------------------------- |
| Logback        | Logback-spring.xml, logbook.xml |
| Log4j2         | Log4j2-spring.xml, log4j2.xml   |
|                |                                 |

一般使用log4j2。hogback-spring.xml相比logbook.xml能额外识别<springProfile>标签

```xml
<!-- 能读取log4j2.yml格式日志文件 -->
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>

<!-- springboot默认log包 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

