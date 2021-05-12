### 字符

java中默认使用unicode编码，一个char类型2字节，中文或英文都是

当使用GBK或GB2312编码时，一个英文字母一个字节，一个汉字2个字节

当使用UTF-8编码时，一个英文字符一字节，一个汉字3或4字节（表情符号4字节）

当使用UTF-16编码时，一个英文字符2字节，一个汉字3或4字节

当使用UTF-32编码时，所有字符都是4字节





### 运算符

#### 左右移

- \>\>：带符号右移，转化为二进制整体向右移动一位，前面补1或0。当为整数时补0，为负数时补1
- \>\>\>：无符号右移，转化为二进制整体向右移动一位，前面补1
- <<：左移，转化为二进制整体向左移动一位，后面补0。移动32位等于原数，1<<31为负数



## 哈希

### 1 解决哈希冲突使用的方法

##### 1.1开放地址法

将哈希冲突的数据保存到后面为空的哈希表中

**线程探查法**

当出现哈希冲突时，按照顺序寻找下一个为空的位置，保存。若遍历一圈仍未找到合适位置则插入失败

问题

- 删除比较麻烦，不能直接删除键，会影响后续查询
- 容易产生堆聚现象，大部分数据保存在哈希表中一块连续的空间

**线性补偿探测法**：类似线性探查法，补偿改为变量K

**随机探测法**：补偿改为随机数RN，预先提供数据数序列

##### 1.2 链地址法

将哈希冲突的数据添加到链表中

##### 1.3 再哈希法

构建多个哈希算法，当发生哈希冲突时，使用其他哈希算法直至不产生哈希冲突

### 2 一致性哈希

https://segmentfault.com/a/1190000021199728

为解决分布式缓存提出，为了解决因特网中的热点问题

当数据量太大，无法存储在一个节点或机器上，需要多个节点存储。这是一般选择一个key，对key进行哈希对节点数量取模，确定存储节点。但分布式系统中需要考虑数据分配平衡性，节点的动态扩展性，数据分散性

#### 2.1 传统哈希算法的局限性

posi = Hash(key) % size，实现简单

**当节点增加或减少时**：新节点映射不变。对于老节点，需要重新进行哈希。

对于分布式系统，数据量巨大，重新哈希时资源消耗高，且哈希过程中可能造成缓存失效，导致“缓存雪崩”

#### 2.2 一致性哈希算法

- 平衡性：对象相对均匀地分配到各服务器上
- 单调：当节点增加或删除时，只移动部分数据，不影响整体系统
- 分散性：每个数据只分散保存在一个服务器上（服务器可以有备份），不必每个节点都存储所有数据

#### 2.3 一致性哈希算法原理

1. 设置一个哈希环，环大小为[0, 2^32-1]
2. 将对象（key）hash放入到哈希环中，也将服务器地址（取哈希）放入到哈希环中
3. 为对象选择服务器：对象按照顺时针选择第一个大于等于本兑现哈希值的服务器
4. 当服务器节点增加时：扫描新增节点服务器至上一个服务器节点之间的对象，将这些对象重新哈希到新增服务器上
5. 当服务器节点减少时：扫描移除节点服务器至上一服务器节点之间的对象，将这些对象重新哈希到移除节点的下一节点上

#### 2.4 虚拟节点

当服务器节点数量较少时，比如两个，可能造成数据倾斜于其中一个节点（hash值差距不大），造成数据分配的不均匀

解决办法：采用多次hash，为每个服务器生成多个虚拟节点，这些虚拟节点存储方式和真实节点类似，不过虚拟节点最终还要定位到正式节点上，一般每台机器的虚拟节点数32甚至更大，使数据分配的更加均匀。比如服务器A，生成三个虚拟节点A1，A2，A3

## 流

> java8新特性

stream操作分类

**中间操作**：指中间操作

- 有状态：必须等待上一步执行完成得到全部的结果才能执行，比如sort
- 无状态：无需等待上一步执行的全部结果，来一个处理一个

**终止操作**：数据流的终止，数据收集

- 短路操作：在满足条件时即可终止，无需等待上一步的全部元素。如anyMatch，findFirst等
- 非短路操作：必须等待上一步执行的全部元素，如collect，max，foreach等

### 1. 理论

对于实现一个列表一连串的功能，比如过滤，排序，转化，结果收集，如果一个功能一个功能写，需要执行多次迭代，并且会频繁生成中间结果

通过stream操作，可以实现，流水线操作，针对每个元素，记录中间操作，直到遇到终止操作再停止执行记录的操作。解决上述两个问题

stream流优势

1. 代码精简，解耦
2. 减少循环迭代
3. 减少中间的生成

```java
List<Integer> list = new ArrayList<Integer>(){{
            add(212);
            add(22);
            add(2);
            add(2112);
        }};
        list.stream()
                .map(a -> {
                    System.out.println("map");
                    return a + 1;
                })
                .filter(a -> {
                    System.out.println("filter");
                    return a > 100;
                })
                .forEach(System.out::println);
//// 输出
map
filter
213
map
filter
map
filter
map
filter
2113
```

### 2. 实践

#### sorted

```java
List<Integer> list = new ArrayList<Integer>(){{
            add(212);
            add(22);
            add(2);
            add(2112);
        }};
        list.stream()
                .sorted((a, b) -> a - b)
                .forEach(System.out::println);
```

#### reduce

对流中元素按照运算规则计算。第一个入参为初始值

```java
System.out.println(Stream.of("A", "B", "C", "D")
                .reduce("-", String::concat));
System.out.println(Stream.of(1, 0, 29, 5)
                   .reduce(0, (a, b) -> a - b));
System.out.println(Stream.of(1, 0, 29, 5)
                   .reduce(0, (a, b) -> Math.max(a, b)));

//结果
-ABCD
-35
29
```

#### map/flatmap

Map: 按照输入元素一一映射为新元素

flagMap: 把多个流集合在一起

```java
// map
List<Integer> list = new ArrayList<Integer>(){{
            add(212);
            add(22);
            add(2);
            add(2112);
        }};
list.stream()
  .map(a -> -a)
  .forEach(System.out::println);

// flatmap
Stream<List<Integer>> stream = Stream.of(
                Arrays.asList(1, 2, 3),
                Arrays.asList(1, 3),
                Arrays.asList(11, 3)
        );
        stream.flatMap(a -> a.stream()).forEach(System.out::println);
```

## NIO

### ByteBuffer

负责存储数据，处理数据

**属性**

- capacity：缓存容量，初始化时分配
- limit：缓存区中数据总数，limit后数据不能进行读写
- position：读或写操作的元素位置

**方法**

- put：往缓存中添加元素
- flip：切换为读模式
- get(byte[])：读取元素

```java
// 1、分配内存
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        print(byteBuffer);
        String str = "ysc";
        // 2、往缓冲中添加元素
        byteBuffer.put(str.getBytes());
        print(byteBuffer);
        // 3、开启读模式。从position读到limit，position随着读取指针移动
        byteBuffer.flip();
        print(byteBuffer);
        byte[] bytes = new byte[byteBuffer.limit()];
        byteBuffer.get(bytes);
        System.out.println(new String(bytes));
        print(byteBuffer);
        // 4、清理缓存，切换回写模式
        byteBuffer.clear();
        print(byteBuffer);

// 输出
mark:java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
mark:java.nio.HeapByteBuffer[pos=3 lim=1024 cap=1024]
mark:java.nio.HeapByteBuffer[pos=0 lim=3 cap=1024]
ysc
mark:java.nio.HeapByteBuffer[pos=3 lim=3 cap=1024]
mark:java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
```

### FileChannel

通道，负责传输数据

### IO模型

https://zhuanlan.zhihu.com/p/115912936

**文件描述符**：linux内核对所有的外围设备都当成一个文件来处理，并返回一个file descriptor（fd，文件描述符）。而socket也有像一个的描述符socket fd，描述符是一个数组，指向linux内核中的一个结构体

**IO运行过程**：当应用程序调用read方法时，次数系统会先等待数据准备，先将数据从网卡中拷贝到内核，再从内核中拷贝到进程中（将内核空间数据拷贝到用户空间）。写操作相反

**应用程序键数据发送**

以两个应用进程通讯为例，A向B发送一条消息

1. 应用A将消息发送TCP缓冲区
2. TCP将缓冲区的消息发送出去，B服务接收并保存到TCP接收缓冲区中
3. B将数据从TCP缓冲区读到自己的用户空间中

**应用通过调用recvfrom读取数据**

#### 阻塞IO

阻塞咨询数据是否准备完毕，是否复制完成

1. 应用进程向内核发起recvfrom读取数据
2. 准备数据报（应用进程阻塞）
3. 将数据从内核空间拷贝到用户空间
4. 复制完成后，返回成功提示

#### 非阻塞IO

循环数据是否准备完毕，准备完毕后进行数据复制

1. 应用进程向内核发起recvfrom读取数据
2. 当数据未准备好时，返回EWOULDBLOCK错误
3. 应用进程再次（多次循环咨询）发起recvfrom读取数据
4. 当数据包准备好时，进行数据复制
5. 将数据从内核空间拷贝到用户空间
6. 复制完成后，返回成功提示

#### IO多路复用模型

采用轮询的方式，调用select/poll/epoll/pselect其中一个函数监听传入多个文件描述符，判断是否存在fd准备就绪，当准备就绪进行处理，否则阻塞等待

多路复用适用于高并发处理海量连接，在连接数小于1000时，并没有明显的性能优势

##### select

多路复用程序通过select/poll/epoll/pselect来管理客户端连接。比如select，对某些时间监听（连接、读、写），不停的轮询，当select()返回大于0表示存在新事件则处理，否则阻塞等待

```c
interface ChannelHandler{
      void channelReadable(Channel channel);
      void channelWritable(Channel channel);
   }
   class Channel{
     Socket socket;
     Event event;//读，写或者连接
   }

   //IO线程主循环:
   class IoThread extends Thread{
   public void run(){
   Channel channel;
   while(channel=Selector.select()){//选择就绪的事件和对应的连接
      if(channel.event==accept){
         registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
      }
      if(channel.event==write){
         getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
      }
      if(channel.event==read){
          getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
      }
    }
   }
   Map<Channel，ChannelHandler> handlerMap;//所有channel的对应事件处理器
  }
```

#### 信号驱动IO模型

IO复用模型通过select监听多个fd，但是通过轮询的方式查询的，可能产生多余的轮询操作。

信号驱动IO模型通过建立信号关联的方式，在发出请求后等待数据的就绪通知即可，避免大量无效轮询操作

1. 系统调用sigaction，建立sigio信号处理程序，内核返回成功
2. 内核数据准备完毕发送信号通知应用，应用发送recvfrom命令，请求数据复制
3. 数据复制完成，操作完成

#### 异步IO模型

IO复用模型依赖select询问和recvfrom请求拷贝数据两步完成整个操作（信号驱动依赖于信号建立+recvfrom）。异步IO模型整合成一步，只需向内核发送read操作

1. 应用向内核发送read请求，直接结束
2. 内核收到请求后建立一个信号联系，在数据准备就绪后主动将数据复制到用户空间
3. 等待所有数据拷贝完成，内核发送信号通知应用操作完成

#### 概念

同步阻塞、同步不阻塞：对于同步操作阻塞（阻塞IO、多路复用在无fd准备就绪时阻塞）或不阻塞（非阻塞IO）

异步不阻塞：信号驱动IO模型、异步IO模型



