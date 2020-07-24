## Log4j2

参考：https://blog.csdn.net/Q176782/article/details/78288734

### 根节点Configuration

有两个子节点Appender和Loggers

- status用来指定log4j本身的打印日志的级别.
- monitorinterval用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s.

### Appender

Appenders节点，常见的有三种子节点:**Console、RollingFile、File**.

- Console节点用来定义输出到控制台的Appender.
  　　　 name:指定Appender的名字.
  　　　 target:SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.
  　　　 PatternLayout:输出格式，不设置默认为:%m%n.

- File节点用来定义输出到指定位置的文件的Appender.

  ​			name:指定Appender的名字.
  　　　 fileName:指定输出日志的目的文件带全路径的文件名.
  　　　 PatternLayout:输出格式，不设置默认为:%m%n
  
- RollingFile节点用来定义超过指定大小自动删除旧的创建新的的Appender.
　　　  name:指定Appender的名字.
  　　　  fileName:指定输出日志的目的文件带全路径的文件名.
  　　　  PatternLayout:输出格式，不设置默认为:%m%n.
  　　　  filePattern:指定新建日志文件的名称格式.
  　　　  Policies:指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.
  　　      TimeBasedTriggeringPolicy:Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am.
  　　　  SizeBasedTriggeringPolicy:Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.
  　　　  DefaultRolloverStrategy:用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性),默认是7个文件。

### Logger节点

常见有两种：Root和Logger

- Root节点用来指定项目的根日志，若没有单独指定Logger，默认使用Root日志输出
  -  level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
  - AppenderRef：Root的子节点，用来指定该日志输出到哪个Appender.
- Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。
  -  AppenderRef：Logger的子节点，用来指定该日志输出到哪个Appender,如果没有指定，就会默认继承自Root.如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出。

### Pattern

- %c，列出logger名字空间的全称，若加上<层数>表示列出从最内层算起指定层数的名字空间

  - 假设logger 命名空间为a.b.c
  - %c = a.b.c
  - %c{2} = b.c
  - %20c ， 若名字空间长度小于20，左边用空格填充；%-20c，右边用空格填充
  - %.30，若名字空间超过30，截断。可配合使用 %20.30c

- %C，列出调用logger类的全名，包括包路径

  - %C{1}， 最内层一级类名

- %d，显示日志记录时间，<日期格式>使用ISO8601定义的日期格式

- %F，显示调用logger的源文件名。%F = MyClass.java

- %l， 输出日志时间的发生位置，包括类名、方法、日志打印行数

  - %logger{36}，仅打印类全名

- %L，显示调用Logger的代码行号

- %m，显示输出消息

- %M，显示调用logger的方法名

- %n，当前平台下换行符

- %p / %level，显示该条日志级别

- %r，显示程序从启动到记录该条日志过去的毫秒数

- %t，显示产生该日志的线程名

- %x，按NDC（线程堆栈）顺序输出日志

- %%，显示一个百分号

  