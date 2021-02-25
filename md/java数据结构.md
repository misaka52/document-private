### HashMap

只有在添加元素才会开始初始化表。

new HashMap<>(int) // 传入的为map容量。实际大小为实际容量*负载因子取整

链表长度大于8，链表转成红黑树。删除数据时，红黑树可能转成链表（在节点数在[2,6]时触发，具体大小和树结构有关）

默认遇到重复键值，value更新

### LinkedHashMap

继承自HashMap，同样也复用hashmap的数组+链表结构，只是新增了一条链表，来表示顺序。新增的节点都放到链表末尾

accessOrder: true表示访问列表为访问数据，false表示列表为插入顺序

put() 使用hashMap put方法

当表示访问顺序时，最新的访问的节点移至链表末尾

遍历时，从链表头开始遍历

### ThreadLocal

java.lang.ThreadLocal

ThreadLocal提供的线程的局部变量，其他线程不可访问。可能方便存取变量

实际上，ThreadLocal不保存任何属性，而是通过Thread中threadLocals保存线程的变量，threadLocal是ThreadLocalMap类

```java
static class ThreadLocalMap {
  static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
      super(k);
      value = v;
    }
  }
  // 变量表
  private Entry[] table;
  // map实际保存变量个数
  private int size = 0;
 	// map容量，当实际大小超过该容量时需要扩容
  private int threshold;
}
```

ThreadLocalMap包含实体组数，key为ThreadLocal变量，value为需要存储的对象，结构图如下

![](../image/5959612-df3da0d24c26271b.png)

**内存泄露**

当ThreadLocal的直接引用被回收后，仍存在Entry的key引用，Entry的生命周期和Thread生命周期一样，若key为强引用，Thread为回收key则ThreadLocal就无法回收，造成内存泄露

所以key设计为弱引用，当ThreadLocal的直接引用被回收，仅存在key的弱引用，在下次gc的时候，ThreadLocal就会被回收。当引用key被回收后，ThreadLocal可以通过remove， get，set来回收key=null的Entry。

> jvm是按照对象来回收的，当对象不存在强引用后，就会进行回收

当线程退出后，线程ThreadLocalMap会被回收

注意使用ThreadLocal变量时，如果使用后未remove，可能产生很多问题

1. 当使用线程池时，线程循环使用，ThreadLocal不remove就不会丢
2. 内存泄露，Entry不能及时被清理

**在使用完ThreadLocal后及时remove**

