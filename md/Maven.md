# Maven

## 简介

Apache Maven是一个软件管理和综合工具，基于项目对象模型（POM）的概念

**安装配置**

下载好压缩包解压即可。确定安装了jdk（并配好JAVA_HOME环境变量），再将maven的bin配到环境变量中。都配置好，用mvn -v指令测试是否成功安装



## Maven仓库

**Maven中央仓库**  [http://repo1.maven.org/maven2/](http://repo1.maven.org/maven/) 

**本地仓库**

**远程仓库** 自定义远程仓库

私服和镜像可理解为远程仓库

>依赖包寻找顺序：本地仓库->远程仓库(repository， setting.xml配置 > pom.xml > parent pom.xml) >镜像(mirror)->中央仓库(central)
>
>maven的配置文件settings.xml，存在多个。
>
>MAVEN_HOME/conf/settings.xml，是maven的全局配置文件；
>
>HOME_DIR/.m2/settings.xml，家目录maven仓库下的配置文件，是用户级配置文件
>
>优先级： 用户级 > 全局

> 一般国内网络从Maven中央仓库下载包比较慢，推荐配置阿里镜像仓库，需在maven settings.xml中配置
>
> <mirror>
>		  <id>alimaven</id>
> 		  <name>aliyun maven</name>
>		  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
> 		  <mirrorOf>central</mirrorOf>        
>		</mirror>



### settings.xml

**localRepository**  本地仓库路径配置，默认${user.home}/.m2/repository

**interactiveMode**  maven是否需要输入，默认true

**offline** maven是否在离线状态下工作

**Servers**

```xml
<!--配置仓库服务端的一些设置。一些设置如安全证书不应该和pom.xml一起分发。比如仓库验证的账号密码 -->
  <servers>
    <!--服务器元素包含配置服务器时需要的信息 -->
    <server>
      <!--这是server的id（注意不是用户登陆的id），该id与distributionManagement中repository元素的id相匹配。 -->
      <id>server001</id>
      <!--鉴权用户名。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。 -->
      <username>my_login</username>
      <!--鉴权密码 。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。密码加密功能已被添加到2.1.0 +。详情请访问密码加密页面 -->
      <password>my_password</password>
      <!--鉴权时使用的私钥位置。和前两个元素类似，私钥位置和私钥密码指定了一个私钥的路径（默认是${user.home}/.ssh/id_dsa）以及如果需要的话，一个密语。将来passphrase和password元素可能会被提取到外部，但目前它们必须在settings.xml文件以纯文本的形式声明。 -->
      <privateKey>${usr.home}/.ssh/id_dsa</privateKey>
      <!--鉴权时使用的私钥密码。 -->
      <passphrase>some_passphrase</passphrase>
      <!--文件被创建时的权限。如果在部署的时候会创建一个仓库文件或者目录，这时候就可以使用权限（permission）。这两个元素合法的值是一个三位数字，其对应了unix文件系统的权限，如664，或者775。 -->
      <filePermissions>664</filePermissions>
      <!--目录被创建时的权限。 -->
      <directoryPermissions>775</directoryPermissions>
    </server>
  </servers>
```

**Mirrors**

```xml
<!-- 为仓库列表配置的镜像地址 -->
<mirrors>
     <!-- 该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
      <id>planetmirror.com</id>
      <!-- 镜像名称 -->
      <name>PlanetMirror Australia</name>
      <!-- 该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 -->
      <url>http://downloads.planetmirror.com/pub/maven2</url>
      <!-- 该镜像的服务器的id。例如，如果我们要设置了一个Maven中央仓库（http://repo.maven.apache.org/maven2/）的镜像，就需要将该元素设置成central。这必须和中央仓库的id central完全一致。 -->
      <mirrorOf>central</mirrorOf>
</mirrors>
```

**proxies** 代理配置

```xml
<proxies>
    <!--代理元素包含配置代理时需要的信息 -->
    <proxy>
      <!--代理的唯一定义符，用来区分不同的代理元素。 -->
      <id>myproxy</id>
      <!--该代理是否是激活的那个。true则激活代理。当我们声明了一组代理，而某个时候只需要激活一个代理的时候，该元素就可以派上用处。 -->
      <active>true</active>
      <!--代理的协议。 协议://主机名:端口，分隔成离散的元素以方便配置。 -->
      <protocol>http</protocol>
      <!--代理的主机名。协议://主机名:端口，分隔成离散的元素以方便配置。 -->
      <host>proxy.somewhere.com</host>
      <!--代理的端口。协议://主机名:端口，分隔成离散的元素以方便配置。 -->
      <port>8080</port>
      <!--代理的用户名，用户名和密码表示代理服务器认证的登录名和密码。 -->
      <username>proxyuser</username>
      <!--代理的密码，用户名和密码表示代理服务器认证的登录名和密码。 -->
      <password>somepassword</password>
      <!--不该被代理的主机名列表。该列表的分隔符由代理服务器指定；例子中使用了竖线分隔符，使用逗号分隔也很常见。 -->
      <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
    </proxy>
  </proxies>
```

**Repositories**

```xml
<repositories>
  <!--包含需要连接到远程仓库的信息 -->
  <repository>
    <!--远程仓库唯一标识 -->
    <id>codehausSnapshots</id>
    <!--远程仓库名称 -->
    <name>Codehaus Snapshots</name>
    <!--如何处理远程仓库里发布版本的下载 -->
    <releases>
      <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
      <enabled>false</enabled>
      <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
       <!-- 经实验，release包不管选择什么都是never，一旦下载下来就不会更新。snapshot可配置四种 -->
      <updatePolicy>always</updatePolicy>
      <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
      <checksumPolicy>warn</checksumPolicy>
    </releases>
    <!--如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
    <snapshots>
      <enabled />
      <updatePolicy />
      <checksumPolicy />
    </snapshots>
    <!--远程仓库URL，按protocol://hostname/path形式 -->
    <url>http://snapshots.maven.codehaus.org/maven2</url>
    <!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
    <layout>default</layout>
  </repository>
</repositories>
```

**pluginRepositories** 发现插件的远程仓库列表。和repository类似，只是repository是管理jar包的仓库，pluginRepository是管理插件的仓库

**ActiveProfiles** 选择激活的profile





## Maven构建生命周期

官网详情：http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html

**validate** 验证，校验项目是否正确并且所有的必要信息可以完成项目的构建过程

**compile** 编译

**test** 运行测试文件（可跳过）

**package** 打包项目

**verify** 对集成测试的结果进行检查，以确保项目的质量

**install** 将打包好的包放到本地仓库

**deploy** 将打包好的包发送到远程仓库

> 上述构建生命周期指令从上往下顺序执行，即运行指令会先把前面的指令先运行一遍。
>
> 比如：mvn package，会先 validate compile test完成在package



## Maven插件

**clean** 清除构建项目生成的文件（java工程删除target目录）

**site** 针对项目生成文档站点，类似于项目介绍文档之类的

不常见

**prerequisites ** 描述了项目构建的前提条件

​	**maven** 构建项目需要的Maven最低版本

**issueManagement** 项目问题管理系统（Bugzilla，Jira，Scarab）

**ciManagement** 项目持续集成信息

**release** 自动化部署

> 插件是在pom.xml的plugins元素定义的



**上传jar包至私服-客户端上传**

首先在maven的settings.xml配置账号信息

```xml
<servers> 
  <server>
     <id>snapshots</id>
     <username>username</username>
     <password>password</password>  <!--jd私服，填写API key即可-->
  </server>
</servers>
```

在项目pom中配置

```xml
<distributionManagement>
    <snapshotRepository>
        <id>snapshots</id>
        <name>JD maven2 repository-snapshots</name>
        <url>http://artifactory.jd.com/libs-snapshots-local</url>
    </snapshotRepository>
</distributionManagement>
```

命令上传语法

```shell
mvn deploy:deploy-file -DgroupId=testArty -DartifactId=testArty-artyjava -Dversion=1.0-SNAPSHOT -Dpackaging=jar -Dfile=D:\testArty-artyjava-1.0-SNAPSHOT.jar -Durl=http://artifactory.jd.com/libs-snapshots-local/ -DrepositoryId=snapshots
  
/**注意：DrepositoryId 必须与setting中server模块的ID一致，否则不能上传
```



上传的代码包只是编译后的class文件，不是源代码，可以额外打包源代码上传

**maven指令打包时添加源码**

mvn clean source:jar deploy -pl groupId:artifactId

**项目中增加插件以保证打包时加入源码**

```xml
<plugin>
    <artifactId>maven-source-plugin</artifactId>
    <version>3.0.1</version>
    <configuration>
        <attach>true</attach>
    </configuration>
    <executions>
        <execution>
            <phase>compile</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```





## 项目pom文件

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

​		**filtering** 是否使用参数值代替参数名。参数值取自properties元素或文件里的配置

​		**incldes/excludes** 包含/排除 的模式列表

**testResources** 单元测试存放的资源

**filters** 当filtering开关打开时，使用到的过滤器文件列表

**pluginManagement** 子项目可以引用的默认插件信息。该配置知道被引用时才会被解析或绑定到生命周期

**plugins** 使用到的插件列表

**profile/profiles** 构建配置，可通maven指定profile。dev、test、preprod、prod

​	**activation** 自动触发profile的条件逻辑。

​	**activeByDefault** profile默认被激活的标志

**modules** 模块/子模块，用于多模块项目。列出每个模块的路径

**repositoriy/repositories** 发现依赖和扩展的远程仓库

​	**releases** 正式发行版本包配置

​		**enabled** true/false，表示该仓库是否开启下载该类型（正式、快照）包

​		**updatePolicy** 包更新的频率。选项：always（一直）、daily（每日，默认）、interval：X（单位为分钟）、never

​		**checksumPolicy** 当验证失败时怎么做，ingore、fail、warn

​	**snapshots** 快照版本包配置

**pluginRepositories** 发现插件的远程插件列表，用于构建和报表

**dependences/dependency** 依赖

​	**type** 包类型，默认jar

​	**classifer** 依赖的分类器。可用于标识同一个包不同编译器类型编译，例如java7和java8编译器

​	**scope** 依赖范围，在项目发布过程中，决定哪些构件被包括起来

> scope依赖范围取值
>
> compile：默认，用于编译
>
> provided： 类似于编译，但支持jdk或容器提供，类似于classpath
>
> runtime：执行时使用
>
> test：测试任务时使用
>
> system：系统本地jar包，需要提供外在相应的元素，通过systemPath获得
>

​	**optional** 当自身被依赖时，标注依赖是否被传递。用于连续时依赖

​	**systemPath** 与system对应

​	**exclusions** 排除依赖构件	

​	**optional** 可选依赖，阻断依赖的传递性。

**reporting** 生成报表插的规范，用于 mvn site

**dependencyManagement** 所有子项目的默认依赖信息。这部分的依赖信息不会立即解析，而是当子项目中声明一个依赖没有声明除groupId和artifactId之外的信息，子项目就会到着来匹配其它信息，如版本号

​	**dependencies** 同上

**distributionManagement** 项目发布配置，在使用 mvn deploy 指定发布的配置

​	**repository** 项目产生的构件到远程仓库的配置信息

​	**snapshotRepository** 快照配置信息

​	**relocation** 如果构件有了新的groupId和artifactId，此处列出构件的重定位信息

**properties** 配置变量，用pom中



## 快照&正式版

**快照** 项目开发时每天都在更新，而开发中的项目别人又需要使用，所以制定一个快照版本，指不稳定版本，snapshot。快照版本maven每次构建都会去下载

**正式版** 与快照相对的是正式版，一个版本一个，稳定，release。正式版maven构建时只要仓库存在就不会去远程仓库下载



## 依赖管理

传递依赖：B依赖A，C依赖B，C就依赖A。可以利用exclusion排除传递依赖，或optional设置可选传递依赖

就近原则，两个依赖版本相同深度下，第一个声明的依赖会被使用。不同深度下依赖深度较小的



## 指令

- 查询mavenjar包依赖：mvn dependency:tree -Dverbose -Dincludes=commons-collections:



#### 参考

1. https://www.runoob.com/maven
2. https://maven.apache.org/
3. https://www.cnblogs.com/jingmoxukong/p/6050172.html