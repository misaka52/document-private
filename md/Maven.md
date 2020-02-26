# Maven

## 1 简介

Apache Maven是一个软件管理和综合工具，基于项目对象模型（POM）的概念

**安装配置**

下载好压缩包解压即可。确定安装了jdk（并配好JAVA_HOME环境变量），再将maven的bin配到环境变量中。都配置好，用mvn -v指令测试是否成功安装

**Maven中央仓库**  [http://repo1.maven.org/maven2/](http://repo1.maven.org/maven/) 

寻找依赖包顺序 本地仓库->私服(profile)->远程仓库(repository, 项目pom中配置)和镜像(mirror)->中央仓库(central)

> 本地仓库：本地配置的maven仓库
>
> 远程仓库：远程Maven仓库。私服为特殊的远程仓库，只能供局域网内的Maven使用
>
> 镜像：特殊的远程仓库
>
> Maven中央仓库：Maven默认的仓库
>
> 一般国内网络从Maven中央仓库下载包比较慢，推荐配置阿里镜像仓库，需在maven settings.xml中配置
>
> <mirror>
> 		  <id>alimaven</id>
> 		  <name>aliyun maven</name>
> 		  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
> 		  <mirrorOf>central</mirrorOf>        
> 		</mirror>



> maven的配置文件settings.xml，存在多个。
>
> MAVEN_HOME/conf/settings.xml，是maven的全局配置文件；
>
> HOME_DIR/.m2/settings.xml，家目录maven仓库下的配置文件，是用户级配置文件
>
> 优先级：用户级 > 全局

### settings.xml

**proxies**

代理配置

**localRepository**  本地仓库路径配置



#### Maven构建的生命周期

官网详情：http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html

**validate** 验证

**compile** 编译

**test** 运行测试文件（可跳过）

**package** 打包项目

**verify** 对集成测试的结果进行检查，以确保项目的质量

**install** 将打包好的包放到本地仓库

**deploy** 将打包好的包发送到远程仓库

> 上述构建生命周期指令从上往下顺序执行，即运行指令会先把前面的指令先运行一遍。
>
> 比如：mvn package，会先 validate compile test完成在package



#### Maven插件

**clean** 清除构建项目生成的文件，常见情况是删除target目录

**site** 针对项目生成文档站点，类似于项目介绍文档之类的



#### Maven pom

**modelVersion** 模型版本，一般使用默认的4.0.0

**groupId** 工程组标识，必需

**artifactId** 工程标识，必需，与groupId确定唯一一个项目

**version** 工程版本号，必需

**relativePath** 父项目的pom.xml文件相对路径，默认../pom.xml，可以手动指定。Maven构建时先在当前项目寻找父项目的pom，其次再去文件系统的这个位置（relativePath位置）找pom，再依次是本地仓库、远程仓库

**packaging** 项目产生的构建类型，例如jar、war、ear、pom。

**build** 构建项目需要的信息

​	**sourceDirectory** 设置项目源码目录，当构建是系统会编译目录里的源码，路径是相对于pom.xml的相对路径

​	**resource/resources **项目相关所有资源路径列表

​		**targetPath** 资源的目标路径，相对于target/classes目录，

​		**directory** 描述存放资源的目录，相对于pom路径





不常见

**prerequisites ** 描述了项目构建的前提条件

​	**maven** 构建项目需要的Maven最低版本

**issueManagement** 项目问题管理系统（Bugzilla，Jira，Scarab）

**ciManagement** 项目持续集成信息

... (省略其他简单、易懂的标签)

