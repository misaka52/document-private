问题

1. mysql RR隔离级别可以解决幻读中的插入数据问题，但仍然能读取到删除的数据。解决幻读是通过间隙锁来实现的吗？那么读取的时候是不是不能在间隙锁之间插入数据？ 

## 一、逻辑架构图

![](/Users/ysc/IdeaProjects/learning/document-private/image/src=http___s3.51cto.com_wyfs02_M01_75_48_wKiom1Y0c5ayz-IFAAMMUqTycYQ172.jpg&refer=http___s3.51cto.jpeg)

### 1 Connectors

连接器，指不同语言与sql的交互

### 2 Management Services & Utilities

系统管理和控制工具

### 3 Connection Pool 连接池

- 管理用户连接，等待处理连接请求
- 负责接收Mysql Server的各种请求，接收连接请求，转发所有连接到线程管理模块。为每个连接都分配一个线程处理
- 负责Mysql Server与客户端的通信。接收客户端的请求，传递Server端的结果信息等。线程管理模块负责线程的维护，创建和销毁等

### 4 SQL Interface：sql接口

接收客户端的命令，返回执行的结果。比如select from就是调用sql interface

### 5 Parser 解析器

sql命令传递到解析器会被验证和解析

主要功能

a. 将sql语句进行词法分析和语法分析，生成语法树，然后根据不同的操作的类型记性进行后续操作

b. 如果在分解过程中遇到错误，说明该sql就是不合法的

### 6 Optimizer 查询优化器

sql查询之前会进行查询优化。explain可以查看sql语句执行计划，进行分析

### 7 Cache和Buffer 查询缓存

将查询的结果添加到缓存中，后续查询若缓存中存在则直接返回。

若查询缓存有命中的缓存结果，直接从缓存中获取结果。缓存分为表缓存、记录缓存、key缓存、权限缓存等

### 8 Pluggable Storage Engines 存储引擎

存储引擎就是实现如何存储数据、查询数据、建立索引

存储引擎分类：

- Innodb：5.5版本之后默认引擎。功能强大，支持事务，行级锁。.frm 表定义文件；ibd 数据文件和索引文件
- Myisam：5.5版本之前默认引擎。不支持事务，表级锁。.frm 表定义文件；myd 数据文件；.myi 索引文件
- Memory：内存存储引擎，只保存数据结构到磁盘中。数据全量保存在内存中，一旦重启数据全部丢失
- CSV：以csv格式保存数据

> 查看存储引擎
>
> mysql> show engines;

## 二、环境说明

### 1 文件结构

#### 1.1 日志文件（顺序IO）

mysql通过日志记录了数据库操作日志和错误信息。常见有操作日志、错误日志、慢查询日志、事务redo日志、中继日志等

```mysql
mysql> show variables liek 'log_%';
```

**a.错误日志**

默认开启，5.5.7版本之后无法关闭。日志文件名 hostname.err

**b.二进制日志**

默认是关闭的。需要配置 log-bin=mysql-bin开启

binlog记录了数据中所有的DDL和DML语句，不记录select语句。**对于DDL语句，直接记录到binlog日志中；对于DML语句，必须在事务提交后才能记录到binlog日志**

binlog日志主要用来主从复制、数据备份、数据恢复

```mysql
mysql> show variables like '%log_bin%';
+---------------------------------+---------------------------------------+
| Variable_name                   | Value                                 |
+---------------------------------+---------------------------------------+
| log_bin                         | ON                                    |
| log_bin_basename                | /usr/local/mysql/data/mysql-bin       |
| log_bin_index                   | /usr/local/mysql/data/mysql-bin.index |
| log_bin_trust_function_creators | OFF                                   |
| log_bin_use_v1_row_events       | OFF                                   |
| sql_log_bin                     | ON                                    |
+---------------------------------+---------------------------------------+
6 rows in set (0.00 sec)
```

**c.通用查询日志**

默认关闭，记录所有的增删改查操作，不建议开启

```sh
mysql> show variables like '%general%';
+------------------+---------------------------------+
| Variable_name    | Value                           |
+------------------+---------------------------------+
| general_log      | OFF                             |
| general_log_file | /usr/local/mysql/data/bogon.log |
+------------------+---------------------------------+
2 rows in set (0.00 sec)
```

**d. 慢查询日志**

默认关闭，用来记录慢查询

```sh
# 开启慢查询日志
slow_query_log=ON
# 慢查询阙值，单位秒
long_query_time=10
# 日志文件路径
slow_query_log_file=file_name
```

#### 1.2 数据文件（随机IO）

```mysql
mysql> show variables like '%datadir%';
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| datadir       | /usr/local/mysql/data/ |
+---------------+------------------------+
1 row in set (0.00 sec)
```

**Innodb数据文件**

- .frm：存储表结构信息
- .ibd：使用独享表空间存储表数据和索引，一张表对应一个ibd文件
- ibdata文件：使用共享表空间存储表数据和索引，所有表或多个共享一个文件

**Myisam数据文件**

- .frm：存储表结构信息
- .myd文件：主要用来存储数据
- .myi文件：主要用来存储索引

## 三、MysqlServe层

### 1 sql执行流程

![image-20210130181712568](../image/image-20210130181712568.png)

### 2 连接器

连接数据库

```sh
mysql -h{ip} -P{port} -u{usename} -D{database} -p{password}
```

查询当前数据的连接状态

```mysql
mysql> show processlist;
+-----+------+-----------------+--------+---------+------+----------+------------------+
| Id  | User | Host            | db     | Command | Time | State    | Info             |
+-----+------+-----------------+--------+---------+------+----------+------------------+
| 206 | root | localhost:51955 | test   | Sleep   | 1764 |          | NULL             |
| 208 | root | localhost:52027 | test   | Sleep   | 1737 |          | NULL             |
| 209 | root | localhost:52034 | test   | Sleep   | 1735 |          | NULL             |
| 210 | root | localhost:52035 | test   | Sleep   | 1734 |          | NULL             |
| 211 | root | localhost:52051 | test   | Sleep   | 1729 |          | NULL             |
| 212 | root | localhost:52089 | test   | Sleep   | 1715 |          | NULL             |
| 213 | root | localhost:52173 | test   | Sleep   | 1683 |          | NULL             |
| 214 | root | localhost:52446 | test   | Sleep   | 1630 |          | NULL             |
| 215 | root | localhost:52532 | test   | Sleep   | 1599 |          | NULL             |
| 216 | root | localhost       | sakila | Query   |    0 | starting | show processlist |
| 217 | root | localhost:56706 | test   | Sleep   |    2 |          | NULL             |
+-----+------+-----------------+--------+---------+------+----------+------------------+
11 rows in set (0.00 sec)
```

创建连接后，若一定时间内没有操作（由参数wait_timeout控制，单位秒，默认8小时），连接自动断开

长连接值连接成功，客户端继续持有连接操作，则一直使用同一连接。短连接为使用完后就断开连接，下次重新建立，建立比较复杂，尽量使用长连接

长连接全部使用完成后，有时候mysql的内存上涨的特别快，因为mysql执行过程中临时使用的内存是保存在连接对象里的，只要连接断开才会释放。

如何解决清理长连接问题

1. 定期断开长连接。或者使用一段时间后，若执行过一个较大的操作，断开连接
2. mysql5.7版本，每个执行一个较大操作后，通过`mysql_reset_connection`来初始化连接资源，该过程不需要重连和重新鉴权，但是会将连接恢复到最初创建的状态

### 3 查询缓存

当执行查询语句时，会先从缓存中查询一次，若命中缓存直接返回；若未命中再执行下面操作查询，后续将查询结果保存在缓存中

缓存往往弊大于利：查询缓存失效的非常频繁，在更新数据之后便失效。反而再每次查询还赠加了查询缓存的操作。在mysql8已经移除了查询缓存

mysql可以设置指定语句开启查询缓存，将参数 `query_cache_type`设置为DEMAND，这样默认的sql不使用查询缓存，通过 select sql_cache 指定语句使用查询缓存。通过属性Qcache_hits (status)查看缓存命中次数

```sh
mysql> select sql_cache * from actor where actor_id  = 1;
+----------+------------+-----------+---------------------+
| actor_id | first_name | last_name | last_update         |
+----------+------------+-----------+---------------------+
|        1 | PENELOPE   | GUINESS   | 2006-02-15 04:34:33 |
+----------+------------+-----------+---------------------+
1 row in set, 1 warning (0.00 sec)

mysql> show status like '%Qcache%';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1030296 |
| Qcache_hits             | 0       |
| Qcache_inserts          | 1       |
| Qcache_lowmem_prunes    | 0       |
| Qcache_not_cached       | 1       |
| Qcache_queries_in_cache | 1       |
| Qcache_total_blocks     | 4       |
+-------------------------+---------+
8 rows in set (0.00 sec)

mysql> select sql_cache * from actor where actor_id  = 1;
+----------+------------+-----------+---------------------+
| actor_id | first_name | last_name | last_update         |
+----------+------------+-----------+---------------------+
|        1 | PENELOPE   | GUINESS   | 2006-02-15 04:34:33 |
+----------+------------+-----------+---------------------+
1 row in set, 1 warning (0.00 sec)

mysql> show status like '%Qcache%';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1030296 |
| Qcache_hits             | 1       |
| Qcache_inserts          | 1       |
| Qcache_lowmem_prunes    | 0       |
| Qcache_not_cached       | 1       |
| Qcache_queries_in_cache | 1       |
| Qcache_total_blocks     | 4       |
+-------------------------+---------+
8 rows in set (0.00 sec)
```

> 第一次查询缓存未命中，将结果缓存；第二次查询命中缓存。Qcache_hits表示查询命中次数

**清空查询缓存**

1. flush query cache // 清理查询缓存内存碎片
2. reset query cache // 清除查询缓存
3. flush tables // 关闭所有打开的表，同时清空查询缓存

### 4. 分析器

1. 词法分析，将sql分割多个字符串。比如关键字select from 等
2. 语法分析，根据词法分析的结果进行语法分析。
3. 若语法分析成功，生成解析树
4. 预处理器对解析树进行合法校验，如表是否存在，列是否存在等，校验用户是否有表的操作权限

### 5. 优化器

在sql执行前进行优化。通过explain命名查看sql执行过程

优化处理

1. 若存在多个索引，决定使用哪个索引
2. 若存在多个表的关联查询，决定各个表的连接顺序（小表优先）

### 6. 执行器

执行优化后的sql语句，执行步骤如下

1. 判断用户是否有权限执行，无权限结束
2. 如果有权限使用指定的存储引擎开始执行

## 四、InnoDB存储引擎

> 存储引擎是表结构的，一个数据库中可能存在多种存储引擎的表

innodb主要由内存池、磁盘文件、后台线程组成

### 1 InnoDB磁盘文件

InnoDB磁盘文件分为三大类：系统表空间、用户表空间、redolog日志文件及其归档文件

二进制文件是维护在Mysql Server层的

#### 1.1 系统表空间和用户表空间

![image-20210131121604210](/Users/ysc/Library/Application Support/typora-user-images/image-20210131121604210.png)

![image-20210131121713114](/Users/ysc/Library/Application Support/typora-user-images/image-20210131121713114.png)

**系统表空间存储那些数据？**

系统表空间是一个共享空间，被多个表共享。

系统表空间包含InnoDB数据字典（元数据及相关对象）、double write buffer、change buffer、undo logs的存储区域

系统表空间默认包含用户在系统表空间创建的数据和索引

**系统表空间配置解析**

系统表空间由一个或多个数据文件组成。默认一个初始大小10M，名为ibdata1的系统数据文件保存。用户可以用`innodb_data_file_path`对数据文件大小和数量进行配置

innodb_data_file_path为：innodb_data_file_path=datafile1[,datafile2]...

```mysql
mysql> show variables like 'innodb_data_file_path';
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
+-----------------------+------------------------+
# autoextend 可自动扩展
```

**使用用户表空间**

通过设置`innodb_file_per_table`来指定每个表创建一个独立的用户表空间，命名：表名.ibd。

通过这种方式，用户不会将所有数据保存在系统表空间中

```mysql
mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
```

#### 1.2 重做日志文件

**重做日志文件**

在InnoDB数据目录存在两个重做日志文件 `ib_logfile0`和`ib_logfile1`文件，记录存储引擎的事务日志

**重做日志的作用**

用于备份，数据恢复。为提升可高可行，可以设置多个镜像日志组，将重做日志保存在不同机器上

**重做日志组如何写入数据**

InnoDB存储引擎至少有一个重做日志组，每个重做日志组至少有两个重做日志文件。

> 在日志组中的每个重做日志文件大小一致
>
> InnoDB存储引擎以循环写入的方式写数据，先写重做日志文件1，满了之后再写重做日志文件2。经过一次循环之后，又写重做日志文件1

**设置重做日志文件大小**

可以通过设置`innodb_log_file_size`来设置重做日志文件大小，但其大小设置对性能影响较大

- 若文件设置过小，会导致依据checkpoint的检查需要频繁刷新脏页到磁盘中，产生大量IO
- 若文件设置过大，在数据丢失后恢复需要很长时间

```mysql
mysql> show variables like 'innodb_log_file%';
+---------------------------+----------+
| Variable_name             | Value    |
+---------------------------+----------+
| innodb_log_file_size      | 50331648 |
| innodb_log_files_in_group | 2        |
+---------------------------+----------+
```

#### 1.3 InnoDB存储结构

表存储结构分为五层：表空间、段、区、页、行

![image-20210131132902479](/Users/ysc/Library/Application Support/typora-user-images/image-20210131132902479.png)

##### 1 表空间

从功能上看，表空间分为共享表空间、独占表空间、通用表空间、临时表空间、Undo表空间

若设置属性`innodb_file_per_table`为1，表示每个表的数据都会保存在一个单独的表空间中

##### 2 段

表空间有多个段组成，常见段有数据段、索引段、回滚段

一个段空间是随着表大小自动扩展的，表有多大，段就有多大

##### 3 区

一个区有64个的页组成，一个区大小1M=64页（每页16K）。为保证区中页的连续性，区扩展时会一次申请4-5个区

##### 4 页

每页默认16K，是InnoDB管理磁盘的最小单位。通过属性`innodb_page_size`控制页大小

索引树上的一个节点就是一页，当节点数据满了再插入时，会发生页分裂

操作系统管理磁盘的最小单位是页，也是操作系统读写磁盘的最小单位。

```sh
# Unix系统获取页大小，默认4K
>getconf PAGE_SIZE
```

> InnoDB存储引擎默认每页16K，操作系统默认每页4K，所以每次InnoDB从磁盘中获取一页时，操作系统会分4次从磁盘文件中读取数据到内存，写入也是分为4次

##### 5 行

InnoDB数据以行为单位存储，一页中存在多行

### 2 InnoDB存储结构

存储结果图如下

![image-20210131134441959](/Users/ysc/Library/Application Support/typora-user-images/image-20210131134441959.png)

#### 2.1 Buffer Pool缓冲池

未解决磁盘与内存读写速度差问题，引入了缓冲。缓冲池中分为以下几种

##### 2.1.1 数据页和索引页

可能会用到的数据页和索引页，以page为最小单位将数据和索引加载在内存中

##### 2.1.2 更新缓冲（插入缓冲）

> insert buffer page（大概在5.7以后更新为change buffer）

在InnoDB存储引擎中，需要根据主键顺序插入。当存储辅助索引插入，辅助索引的顺序性无法保证，造成离散的访问索引页，导致插入性能下降

此时InnoDB存储引擎设计了channge buffer来进行插入优化，对于辅助索引的插入或更新，并非每一次都直接插入到索引页中，而是先保存在channge buffer再后续统一合并插入到索引页中。

##### 2.1.3 自适应哈希索引

InnoDB根据访问的频率和模式，建立热点页和哈希索引来提高查询效率。一般索引在B+树需要访问3，4次，而哈希索引仅一次

InnoDB会根据sql使用情况来自动判断是否需要建立hash索引，当满足一下条件会自动创建哈希索引

- 必须为等值查询。如where a = 123
- 以该模式访问了100次 或 页通过该模式访问了N次，其中N=页中记录 * 1/16

可以通过参数`innodb_adaptive_hash_index`来设置禁用或启用此特性。默认ON开启

##### 2.1.4 锁信息

InnoDB存储引擎支持行级锁，为保证数据的完整性和一致性，有时候需要用到锁

##### 2.1.5 数据字典信息

InnoDB的表信息缓存，可称为表定义缓存或数据字典

其中包含表结构、表名、表字段、表格式、视图、索引等

#### 2.2 内存数据落盘

![img](../image/20200325224909394.png)

##### 2.2.1 整体思路分析

InnoDB内存缓冲池刷新到page完成持久化，一个是脏页落盘；一个是预写redo log

当缓冲池中的数据比磁盘中新时，需要将缓冲池中数据持久化。刷新存在两种机制 WAL 和force log at commit

- WAL 要求数据变更写入磁盘前，必须将内存中的日志写入磁盘中
- force log at commit 要求一个事务提交时，所有日志必须刷新到磁盘中。若机器发生宕机可以利用日志恢复

在将缓冲中的日志刷新磁盘后，通过 fsync将缓冲数据刷新到磁盘中。

##### 2.2.2 脏页落盘

- 进行读取操作时，判断磁盘读取页是否存在缓冲池中，存在表示命中缓存直接读取；否则读取磁盘上的页
- 对数据库进行修改操作时，首先修改缓冲池中的页，再通过一定的频率刷新到磁盘中。通过checkPoint机制刷新到磁盘

##### 2.2.3 重做日志落盘

落盘策略，通过`innodb_flush_log_at_tx_commit`来控制

- 0：MYSQL每秒一次将log buffer中的数据写入到日志文件并同时fsync刷新到磁盘中。最多丢失1s的数据
- 1：每次事务提交时，将log buffer写入日志文件并同时fsync刷新到磁盘。默认，不会丢失数据
- 2：每次事务提交时，将log buffer写入日志文件，然后MYSQL以每秒一次将日志文件中数据同步到磁盘中。若机器发生故障，日志文件意外丢失则可能丢失1s的数据

> fsync是阻塞的，直到完成之后才能返回

#### 2.3 checkPoint检查点机制

当数据发生故障恢复时，需要依赖redo日志文件恢复，并不需要恢复全量日志，checkPoint记录了上次的刷盘的位置，直接执行checkPoint恢复未刷盘的数据。当缓冲池不够用或重做日志不可用且不能覆盖时，需要强制执行checkPoint进行数据刷盘

> 脏页：缓冲池中存在但磁盘中不存在的数据。遇到脏页需要淘汰时，必须强制执行checkPoint

checkPoint分类

**sharp checkpoint**

在数据库关闭时，将buffer pool中所有的数据刷新到磁盘中

**负重fuzzy checkpoint**

在数据库正常运行时，将不同脏页在不同实际刷新到磁盘，避免全量刷新因其性能问题。存在以下几种策略

1. Master Thread Checkpoint：在Master thread中以10s一次的频率将部分脏页从内存中刷新到磁盘中。该操作时也异步的，不阻塞数据库其他操作
2. FLUSH_LRU_LIST checkpoint：在单独的page cleaner线程中执行。通过LRU机制清理数据，当清理的页是脏页时需要执行落盘
3. ASYNC/SYNC Flush Checkpoint：在单独的page cleaner线程执行。当重做日志满了，将脏页刷新到磁盘。当未落盘的内容小于75%则无需落盘；若大于75%小于90%进行异步落盘；若大于90%进行同步落盘。在mysql5.6之后，无论同步还是异步落盘都不会阻塞数据库的操作
4. Dirty Page too much：由Master Thread线程1s一次执行，当脏页过多，超过变量`innodb_max_dirty_pages_pct`，则进行刷盘。默认75%

#### 2.4 Double Write

当将缓冲池的脏页刷新到磁盘时，并非直接刷新。而是先将数据刷新到Double write buffer中，double write buffer再同时同步到 doubleWrite空间和磁盘数据文件中。doubleWrite空间用作备份，后续恢复可以使用

> redo log日志不足以恢复吗？
>
> 一般情况时可以直接拿redo log日志恢复的，但是如果出现redo log复制页时只复制的一般，机器宕机，页只复制一半导致页不可用，数据丢失。且redo log中只记录了修改的数据，而非记录数据页的完整内容

#### 2.5 redo log buffer 重做日志缓冲

通过`innodb_flush_log_at_tx_commit`控制落盘策略，参考2.2.3

#### 3 落盘流程图

![image-20210131191418732](../image/InnoDB数据落盘流程.png)

## 五、事务

### 1 一条insert语句执行流程

![image-20210131191418732](/Users/ysc/Library/Application Support/typora-user-images/image-20210131191418732.png)

1. 索引加意向锁，指定是针对唯一索引加锁，若出现多条相同的唯一索引字段插入，其他sql只能等待，若执行的sql回滚，执行其他插入语句；若执行的sql成功，其他插入语句报错
2. undo log：用于事务回滚
3. 先写undo log的redo log ，再写undo log，再写变更记录的redo log。若事务执行过程中mysql崩溃了，恢复时需要通过redo log恢复undo log，以恢复事务

### 2. 事务介绍

#### 2.1 事务概述

数据库事务四大特性 ACID

- 原子性（automic）：整个事务是原子的不可拆分，要么全部提交，要么全部不提交
- 一致性（consistent）：事务的开始的结束后，数据库的完整性不会被破坏。数据库的完整性包括：实体完整性（如行的主键存在且唯一）、列完整性（如字段的类型、长度、大小符合要求）、外键约束、用户自定义完整性（转账之后总金额不变）
- 隔离性（isolation）：事务之间是相互隔离的，不同事务之间互不影响。存在四种隔离级别
- 持久性（durability）：事务一旦提交，对数据库造成的影响就是永久性的

#### 2.2 隔离级别

存在四种隔离级别，由变量`tx_isolation`控制

##### 2.2.1 未提交读（READ UNCOMMITTED）

该模式下事务可以读到其他事务未提交的数据。也称为脏读

##### 2.2.2 已提交读（READ COMMITTED）

该模式下可以解决脏读，但可能出现重复读

##### 2.2.3 可重复读（REPEATABLE READ）

该模式下可以解决脏读和重复读，但不能解决幻读

##### 2.2.4 串行化（SERIALIZABLE）

对所有操作都进行加锁，读加共享锁，写加排他锁。可解决脏读、重复读、幻读

隔离级别通过`transaction_isolation`变量控制

### 3 事务和MVCC底层原理

> 问题：当银行需要统计两个账户的总存款金额时，同时这两个账户又在进行转账交易，可能存在统计多了或少了的情况

#### 3.1 LBCC

基于锁的并发控制（LOCK BASE Concurrency Control），对于更新操作，增加记录锁，多个事务修改记录需要先获取锁，若锁已被占有需要等待。

对于上述问题，在统计的时候添加共享锁，等待全部数据统计完成再释放，可以保证统计的数据正常。但是加锁期间不能对数据进行修改，客户体验差

#### 3.2 MVCC

多版本并发控制（Multi Version Concurrency Control），为每条记录新增事务版本号和回滚版本号，使用多版本的方式来避免一些列完整性问题。

对于上述问题，统计相当于读取的是快照版本，用户之间可以随意进行交易，仍能保证的数据正常。

在每条记录新增隐藏字段创建版本号和删除版本号，当进行以下操作时。假设当前事务版本号为x

1. 读取：开启隐藏事务，读取创建版本号小于等于x，删除版本号大于x或为null的数据
2. 插入：新增数据时，设置创建版本号为x，删除版本号为null
3. 更新：先插入新数据，后删除。
4. 删除：设置删除版本号为x。

### 4 InnoDB的MVCC实现

#### 4.1 当前读和快照读

当前读：读取的数据为最新数据，读取时对数据添加锁，数据不能被修改。更新、删除操作使用

快照读：读取的快照版本，不加锁。查询使用

#### 4.2 一致性的非锁定读

mvcc在mysql中实现依赖的是undo log与read view

#### 4.3 回滚段undo log

回滚段分为插入回滚段和更新回滚段

- insert undo log：在插入操作中产生的，只对当前事务可见，事务提交后可以直接删除而不需要依赖purge清除
- update undo log：指在更新或插入时生成的，数据删除之后并非直接删除，而是先放至undo log链表中，等待purge线程清理

#### 4.4 行记录新增字段

参考：https://cloud.tencent.com/developer/article/1454636

InnoDB存储的MVCC实现为每条行记录新增了几个字段：DATA_TRX_ID、DATA_ROLL_PTR、DB_ROW_ID（当没有主键时会隐式生成该字段）

**DATA_TRX_ID**

记录更新该条记录的事务ID，大小为6字节

**DATA_ROLL_PTR**

表示指向该记录回滚段的指针，大小为7字节，InnoDB通过该字段找到之前版本的数据。在undo中以通过链条的形式连接

**DB_ROW_ID**

行表示（隐藏单调递增），大小为6字节。当表不存在主键时，InnoDB会为表生成这个隐式自动主键。另外，每条记录的头信息（record header）里有一个专门的删除表示（deleted_flag）来表示该记录是否已经被删除

**InnoDB表主键生成规则**

1. 自定义主键
2. 未自定主键时，系统自动选择一个非空唯一索引（若存在多个则选择第一个定义的索引，而不是第一个定义的索引的列）为主键
3. 上述条件均不满足，系统自动生成一个6字节大小的隐式id作为主键

#### 4.5 版本链

每条记录可能产生多个副本（在更新时需要新增），通过回滚指针来生组织成一条`undo log`链。在查询使用时，每次都会线查到最新的节点，判断当前节点是否满足需要，若满足直接返回；若不满足通过回滚指针查找上一版本节点

#### 4.6 如何实现一致性读-ReadView

对于MVCC读取操作，通过ReadView来维护当前已开始但未完成的事务，即活跃事务，来避免其他事务读到未提交事务的信息。ReadView中活跃的事务ID列表为m_ids，其中最小值为up_limit_id，最大值为low_limit_id，事务ID是事务开启是InnoDB分配的，决定了事务的先后顺序，具体判断流程如下

- 若被访问版本的trx_id小于m_ids中的最小值，表明事务已提交，则该事务可以被访问
- 若被访问版本的trx_id大于m_ids中的最大值，表明被访问的事务是在获取ReadView时开始的，该事务不能被访问
- 若被访问版本的trx_id介于m_ids的最小值和最大值之前，判断该trx_id是否存在m_ids中，若存在则表明访问事务还是活跃事务，不能被访问；反之则可以被访问
- 经过一系列判断得到记录后，再判断记录是否被删除，若删除则继续查找下一节点

其中ReadView是单个事务持有的，其中RU和序列化不支持ReadView，一个只读取最新数据，一个对每个操作都加锁，都能读取到最新数据。对于RC和RR支持如下

##### 4.6.1 已提交读级别

在事务中的每条查询语句执行之前生成ReadView，来获取当前的活跃事务。所以RC级别会产生重复读问题

##### 4.6.2 可重复读级别

仅在事务的**第一条查询语句执行**之前生成ReadView，避免了重复读和幻读问题

> 幻读，两次读取结果不一致，受到其他事务插入和删除数据提交影响。这里MVCC机制，在第一条语句在第一次执行查询语句之前获取ReadView，获取到排查的事务。对于新增事务插入或删除，事务id大于当前版本而不会被查询到；对于未提交的老事务 插入和删除操作，已经在ReadView中记录不会被读取到。
>
> 注意：在第一条查询语句执行之前生成而非在事务开启时就生成

#### 4.7 MVCC的争论点

##### 4.7.1 RC下MVCC的问题

![](/Users/ysc/IdeaProjects/learning/document-private/image/b3wnejlzm7.jpeg)

在RC隔离模式下，对于上述流程，事务B晚于事务A执行，事务A却读到了事务B提交的数据

##### 4.7.2 RR下MVCC的问题

![](/Users/ysc/IdeaProjects/learning/document-private/image/tgmf0zg899.jpeg)

在RR隔离级别下，对于上述流程，事务B晚于事务A执行，事务A却读到了事务B提交的数据

> 上述问题可以理解为：每次都获取最新的ReadView，不管当前事务号与被访问的事务号的大小关系，只关心ReadView中的最大最小版本号，而且理解最大事务号可以理解为一个维护的变量，新增活跃事务时事务id更新，移除时不更新或异步更新不及时导致读取到了新事务的数据。

### 5. 事务回滚和数据恢复

undo log存储不同于redo log，它保存在特殊的回滚段中。因redo log是物理日志，记录数据库页的物理修改操作，且只记录修改内容。对于undo log日志的修改也会被记录到redo log日志中。所以在进行数据恢复时，redo log用于恢复undo log，undo log用于事务回滚

![image-20210131202258264](/Users/ysc/Library/Application Support/typora-user-images/image-20210131202258264.png)

### 6 锁获取实战

> 一下为InnoDB存储引擎，隔离级别RR

#### 6.1 互相占用

```mysql
# 事务1
start transaction;
update test set create_time = NOW() where id = 1;
sleep(10);
update test set create_time = NOW() where id = 2;
commit;

# 事务2
start transaction;
# 语句0
select * from test;
# 语句1
update test2 set create_time = NOW() where id = 1;
# 语句2
update test set create_time = NOW() where id = 3;
# 语句3
update test set create_time = NOW() where id = 2;
sleep(10);
update test set create_time = NOW() where id = 1;
commit;
```

对于两个事务，首先启动事务1；再启动事务2，发现语句0，1正常执行，语句2阻塞，待事务1执行完成后，语句2才执行

个人猜想，事务根据表级获取所有的需要的锁，当遇到更新或删除语句时，获取该事务下所有需要的锁

