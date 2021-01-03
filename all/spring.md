[TOC]



## 介绍

### Spring

Spring是一个分层的Java SE/EE full-stack 轻量级开源框架。官网地址：https://spring.io

> 一般常说的Spring指的是Spring framework框架

### Spring优势

特点

1. 方便解耦，简化开发。提供IOC容器，无需关心对象的生命周期
2. 对AOP编程的支持
3. 声明式事务的支持
4. 集成各种框架。比如Junit、Struts、Hibernate、Hessian等

优点

1. 低侵入式设计，代码污染低
2. 独立于各种应用服务器，基于Spring应用框架的应用，可真正实现一次编译到处运行的承诺
3. Spring的DI机制降低了业务对象的复杂性，提交了组件间的解耦
4. Spring的AOP支持运行一些通用任务如安全、事务、日志等集中式管理
5. Spring的ORM和DAO提供了与第三方框架的良好整合，简化了底层的数据库访问

> ORM：对象关系映射。通过实例对象的方法，完成关系型数据库操作

### Spring核心概念

- Spring容器：指的就是IOC容器，底层实现是一个BeanFactory
- IOC：Inverse Of Control，控制反转，将对象的控制全交给Spring容器
- DI：Dependency Inject，依赖注入，Spring通过通过依赖注入的方式将对象注入到bean组件中。属于控制反转的一种方法
- AOP：Aspect Oriented Programing 面向切面编程，在不修改目标对象源代码的基础上，增加Ioc容器中bean的功能

## 核心基础

### 基于XML的使用

#### Ioc配置

##### bean标签

配置被Spring容器管理的bean对象，默认调用类中的无参构造器生产实例

标签属性

- id：对象唯一标识
- class：类的全限定名。用于反射构造对象，默认调用无参构造函数
- init-method：初始化方法
- Destroy-method：类销毁方法
- scope：指定对象的作用范围
  - singleton：默认。单例对象
  - prototype：多例。每次获取bean的时候都会新建对象
  - request：将Spring容器创建的bean放入request中
  - session：将Spring容器创建的bean放入session域中

##### bean初始化的三种方式

1. 使用默认无参构造器（重点）：直接配置<bean\>
2. 静态工厂。配置一个静态工厂，提供创建对象的静态方法
3. 实例工厂。配置实例工厂

#### DI配置

依赖：指Bean实例中的属性。分为简单类型（8种）、POJO类型、数组类型

##### 依赖注入的方式

**1. 构造函数注入**

通过调用类中的构造函数注入属性

```xml
<bean id="userService" class="com.xxx.UserServiceImpl">
  <constructor-arg name="id" value="1"></constructor-arg>
  <constructor-arg name="name" value="zhangsan"></constructor-arg>
</bean>
```

constructor-arg标签属性

- index：指定参数在构造函数中的索引位置
- name：指定参数在构造函数中的名称
- value：赋值基本类型和String类型
- ref：赋值bean类型

**2. Set方法注入**

- 手动装配方式（xml）
  - 配置bean标签的property子标签
  - 配置bean中指定的setter方法
- 自动装配方式
  - @Autowired
    - 按照类型查找实例。注入的类型Spring只能有一个实例。由Autowired由AutowiredAnnotationBeanPostProcessor类实现
    - 默认情况下要求bean必须存在，不可为null。若允许为null可设置属性required属性为false
    - 可结合@Qualifier来实现根据实例名称查询
    - spring自带注解
  - @Resource
    - 默认根据实例名称获取实例。如果没有指定名称，则先根据名称查找，不存在则再根据类型查找；若指定了名称，则只能根据名称查找
    - JDK自带
  - @Inject：与@Autowired类似
    - 默认根据类型查找，可结合@Named来实现根据名称查找bean
    - JDK自带

**3. 依赖注入不同类型的属性**

简单类型（value）：8种基本类型和String

引用类型（ref）：引用类型，用来注入引用。引用bean id

基本类型（数组）：可注入数组或List集合、set集合、map集合、properties集合

```xml
<!--list-->
<bean id="userService" class="com.xxx.UserServiceImpl">
  <property name="arrs">
    <list>
      <value>1</value>
      <value>2</value>
    </list>
  </property>
</bean>

<!--set-->
<property name="set">
  <set>
    <value>1</value>
    <value>2</value>
  </set>
</property>
<!--map-->
<property name="map">
  <map>
    <entry key="zhangsan" value=98></entry>
    <entry key="lisi" value=87></entry>
  </map>
</property>
<!--Properties-->
<property name="properties">
  <props>
    <prop key="zhangsan" value=98></prop>
    <prop key="lisi" value=87></prop>
  </props>
</property>
```

### 基于注解和XML混合使用

> 部分注解在Spring容器中就实现了，Springboot扩展实现了其他注解

#### IoC注解使用方法

- Step1：spring配置文件中，配置context:component-scan标签

> beans标签增加属性 
>
> ```xml
> xmlns:context="http://www.springframework.org/schema/context"
> http://www.springframework.org/schema/context
> http://www.springframework.org/schema/context/spring-context.xsd
> ```
>
> 然后组增加bean扫描配置：\<context:component-scan
> 		base-package="com.kkb.spring.aop">/context:component-scan\>

- step2：在类上添加注解@Component，或者衍生注解@Controller、@Service、@Repository

#### 常用注解

##### Component

将该类作为一个Spring管理的bean，相当于在xml中配置一个bean。

value属性：指定bean的id，未指定value时以小驼峰写法名作为bean id

##### Controller&Service&Repository

均为Component衍生注解，属性也与它一致

#### DI注解

##### @Autowired

##### @Qualifier

##### @Resource

##### @Inject

##### @Value

给基本类型和String类型复制，可通过占位符获取属性文件中的值

```java
// 基本类型，冒号后的为默认值
@Value("${name:test}")
private String name;
// 集合类型，自动转换
@Value("#{${map:{'zhangsan':'10','lisi':'20'}}}")
private Map<String, String> map;
```

##### @Scope

限定类作用返回，同xml中 scope

##### 生命周期相关注解

- @PostConstruct：实例构造完成后触发
- @PreDestroy：实例销毁之前触发

#### 关于注解和XML的对比

- 注解优势：配置简单，维护方便
- xml优势：不用修改源码，修改后无需重新编译和部署

### 基于纯注解方式使用

#### @Configuration

相当于xml中的配置文件。同@Component注解

#### @Bean

相当于bean标签，主要用来配置非自定义的bean。如DruidDataSource、SqlSessionFactory

#### @ComponentScan

相当于xml中 context:component-scan标签

组件扫描器，扫描Spring管理的bean。不设置时默认扫描本类的基本目录

属性

- basePackages：指定需要扫描的包

#### @PropertySource

相当于xml中的context:property-placeholder标签

属性value[]:指定properties文件路径，若在类路径下，需要写上classpath

```java
@Configuration
// .yml文件不生效？@PropertySource注解无法识别yml文件
@PropertySource("classpath:datasource.properties")
@ToString
public class MysqlConfiguration {
    @Value("${mysql.username}")
    private String userName;
    @Value("${mysql.password}")
    private String password;
}
```

#### @Import

相当xml中的import标签，引入其他配置文件

```java
@Configuration
@Import({MysqlConfiguration.class})
@ToString
public class JdbcSource {
    private MysqlConfiguration mysqlConfiguration;

    public JdbcSource(MysqlConfiguration mysqlConfiguration) {
        this.mysqlConfiguration = mysqlConfiguration;
    }
}
```

#### 创建上下文容器

**java应用**

```java
ApplicationContext context = new AnnotationConfigApplicationContext(JdbcSource.class);
        JdbcSource bean = context.getBean(JdbcSource.class);
        bean.print();
```

**web应用（AnnotationConfigWebApplicationContext，后续解析）**

## 核心高级篇

### Aop介绍

#### 概念

aop，Aspect Oriented Programming，面向切面编程。通过预编译和运行期动态代理实现功能统一维护的技术

AOP是一种编程范式，隶属于软工范畴。Spring将aop思想引入

AOP是OOP（面向对象编程）的延续，是函数式变成的一种衍生型

#### 为什么使用AOP

作用：aop采用横向抽取机制，补充了传统纵向继承体系（OOP）无法解决的重复代码问题（性能监控、安全检查、事务管理、缓存、日志记录），将业务逻辑和系统逻辑解耦

#### aop相关术语

Joinpoint（连接点）：指的是被拦截到的点。在spring中指的是被拦截的方法，因为spring中只支持方法类型的连接点

Pointcut（切入点）：指的是我们要对那些连接点拦截的定义

Advice（通知/增强）：值拦截到连接点之后的动作

Introduction（引介）：是一种特殊的通知在不修改源码的情况下，可以在运行期为类添加一些方法和field

Target（目标对象）：代理的目标对象

Weaving（织入）：指把增强到目标对象来创建对象新的代理对象的过滤

Proxy（代理）：一个类被代理增强织入后，就会生成一个代理类

Aspect（切面）：切入点和通知的集合

Advisor（通知器，顾问）：类似于Advice

#### AOP实现AspectJ（了解）

- AspectJ是java实现的一个aop框架，对java代码进行aop编译（一般是编译器进行），让代码具有AspectJ的aop功能
- AspectJ应用到java代码过程，这个过程称为织入。可理解为AspectJ应用到目标函数（类）的过程
- 动态织入，在运行时动态地将要增加的代码织入到目标类中，往往通过动态代理技术完成。如jdk动态代理和cglib代理
- AspectJ采用的是静态代理的方式，主要在编译器织入。先编译AspectJ再编译目标类

#### AOP实现Spring AOP

基于动态代理实现，动态代理又基于反射实现。分为jdk动态代理和cglib动态代理

jdk动态代理：目标对象必须实现接口

cglib动态代理：目标对象无需实现接口。通过继承目标对象生成子对象来实现

### 基于AspectJ的AOP使用

Spring已经把AspectJ整合到自身框架中，通过动态织入方式

#### 添加依赖

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>5.0.7.RELEASE</version>
</dependency>
<dependency>
  <groupId>aopalliance</groupId>
  <artifactId>aopalliance</artifactId>
  <version>1.0</version>
</dependency>
```

#### 使用xml实现

- 生成通知类（增强普通类）
- 配置bean，将通知类交由spring ioc容器管理
- 配置aop切面，指定方法

切入点表达式：execution([修饰符] 返回值类型 包名.类名.方法名.(参数))

- 包名：多包名可使用*代替；多级包名可使用多个\*代理；中间包名可用..省略
- 类名：可以使用*代替；可以使用通配符\*DaoImpl
- 方法名：同类名
- 参数：若多个参数可用..代替

通知类型

- 前置通知：before
- 后置通知：afterReturning
- 最终通知：after
- 环绕通知：around
- 异常抛出通知：afterThrowing

#### 使用注解实现

编写切面类，结合类注解@Aspect，及方法注解@Before，@After等。类需要配置spring ioc容器管理类

开启aop自动代理：xml中添加 \<aop:aspectj-aotoproxy /\>

> 注解方式开启aop自动代理，在spring管理bean添加 @EnableAspectJAutoProxy注解

##### 注解解释

```java
//@Before
try {
  // 连接点方法
  //@AfterReturning
  catch (Exception e) {
    // @AfterThrowing
  } finally {
    // @After
  }
}

// aroud 自定义写法
```

##### 定义切入点

可定义一个通用的切入点，其他同志可引入该切入点

```java
@Component
@Aspect
@Slf4j
public class LogAspect {
    @Before(value = "execution(* com..TestController.hello())")
    public void before() {
        log.info("before-----");
    }

    @After(value = "execution(* com.*.*.TestController.hello(..))")
    public void after() {
        log.info("after-----");
    }

    // 定义切入点
    @Pointcut(value = "execution(* com.*.*.TestController.hello2(..))")
    public void point() {
    }

    // 复用切入点
    @Around(value = "LogAspect.point()")
    public Object around(ProceedingJoinPoint joinPoint) {
        try {
            log.info("around-before");
            Object[] args = joinPoint.getArgs();
            Object result = joinPoint.proceed(args);
            log.info("around-afterReturning");
            return result;
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            log.info("around-afterThrowing");
            return null;
        }
    }
}
```

## 组件支撑篇

### 整合Junit

juit程序不能识别spring，无法创建Spring容器。可通过@RunWith注解替换运行期

#### 具体实现

1. 添加依赖。添加spring-boot-starter-test包
2. 在运行类上添加@RunWith注解，指定spring的运行期 SpringJunit4ClassRunner.class
3. 添加测试启动类注解@SpringBootTest

### 事务支持

#### 数据库并发操作时可能产生的问题

- 脏读：一个事务读取到了另一个事务未提交的数据
- 重复读：一个事务中，对某一条数据读取两次的结果不一致
- 幻读：一个事务中，对某一查询条件两次查询得到的结果条数不一致

#### 事务隔离级别

- Read uncommitted（读未提交）：最低级别
- Read committed（读提交）：可避免脏读。
- Repeatable read（可重复读）：可避免重复读。mysql默认级别，且mysql该级别下通过间隙锁解决幻读问题
- Serialiable（可串行化）：任何读写操作都加锁。可避免一切其他异常情况

#### 事务传播行为

只两个一个事务方法A里调用另一个事务方法B，事务行为如何传播

- PROPAGATION_REQUIRED(默认)：A中有事务，使用A中的事务。若没有则B新建事务
- PROPAGATION_SUPPORTS: A中有事务则使用A中的事务，没有则不使用事务
- PROPAGATION_MANDATORY: A中有事务使用A中事务；没有则抛异常
- PROPAGATION_REQUIRED_NEW：A中有事务则将A中事务挂起新建一个事务
- PROPAGATION_NOT_SUPPORTED: A中有事务，将A中事务挂起
- PROPAGATION_NEVEW: A中事务则抛出异常
- PROPAGATION_NESTED: 嵌套事务，当A执行之后设置一个保存点。若B执行成功则通过，若B执行失败根据客户需求回滚到保存点或初始状态

#### Spring框架事务管理的分离

1. Spring的编程式事务管理，手动编写代码实现事务管理，不推荐使用
   1. 提供模板类TransactionTemplate，配置执行
2. Spring声明式事务（底层采用AOP的技术），通过配置实现@Transactional

#### 事务管理之XML方式

```xml
<!-- 配置数据源 -->
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"
			value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql:///kkb" />
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>
	
	<!-- 扫描AccountDao和AccountService -->
	<context:component-scan base-package="com.kkb.spring.tx"></context:component-scan>
	
	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>

	<!-- 配置事务通知类（Spring实现的事务增强类） -->
	<!--   TransactionInterceptor  -->
	<tx:advice id="txAdvice"
		transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="save*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="transfer*" propagation="REQUIRED" />
			<tx:method name="query*" read-only="true" />
		</tx:attributes>
	</tx:advice>
	
	<!-- 配置aop切面类或者通知器类 -->
	<aop:config>
		<!-- 这使用的是Spring AOP的实现 -->
		<!-- advice-ref：指定advice增强类 -->
		<aop:advisor advice-ref="txAdvice"
			pointcut="execution(* *..*.*ServiceImpl.*(..))" />
	</aop:config>
```

#### 事务管理之混合方式

1. 类或方法上添加注解@Transcational
   1. 类上添加注解@Transactional：表示该类下所有方法都被事务管理
   2. 类方法上添加注解@Transactional：表示该方法被事务管理

2. 开启事务注解

```xml
<!--配置平台事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!--注入数据源-->
  <property name="datasource" ref="datasource"></property>
</bean>
<!--开启事务注解-->
<tx:annotation-driven transaction-manager="transactionManager" />
```

#### 事务管理之基于AspectJ的纯注解方式

1. 类或方法上添加注解@Transcational

2. 开启事务注解：spring ioc容器管理bean上添加@EnableTransactionManagent

## 源码解析

### 1. 手写源码

背景：service、serviceImpl、dao、daoImpl、po

直接生成service的话，需要设置dao，dao又要设置数据源，数据源需要设置地址、账号、密码等属性，每次创建都需要执行这些操作，比较繁琐

spring通过ioc容器管理bean，需要时直接从容器中获取bean，无需关心它的生命周期。主要分为以下几步

1. 读取并解析bean配置文件beans.xml，每种bean生成一个beanDefinition，放入beanDefinition缓存中
2. 根据beanName获取bean，首先从bean缓存中获取，存在直接返回，不存在再从缓存中获取beanDefinition，创建bean
3. 创建bean第一步，生成bean对象。一般使用反射方式创建
4. 创建bean第二步，将依赖的属性注入到bean中
5. 创建bean第三步，调用bean的初始化方法完成初始化

### 2. 阅读源码

tips：

1. 依赖注入时,通过多种方式注入。通过反射注入域（但是实验都走了setter方法，包括基本类型、String、引用类型，待研究）；通过setter方法注入；数组、list、map通过其他方式填充

