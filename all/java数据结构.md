### HashMap

存储键值对的数据结构，采用数组+链表+红黑树实现

实现关注点

- size表示实际大小，threshold表示容量，cap表示哈希表大小。当未设置容量时，则哈希表大小默认16，容量默认16\*负载因子（默认0.75）；当设置了容量，哈希表大小等于容量，实际容量为哈希表容量\*负载因子；当hashMap扩容时，容量和表大小都变为原来的两倍
- new HashMap<>(int) // 传入的为map容量。实际大小为实际容量*负载因子取整
- 默认遇到重复键值，value更新
- 初始化时哈希表为空表，在第一次添加元素时初始化哈希表
- 允许键值为null，对于null键，存入table[0]中
- 链表长度大于8，链表转成红黑树。删除数据时，红黑树节点过少时转成链表（在[2,6]范围内，具体要看红黑树结构），或者在扩容rehash时新树节点小于等于6转成链表
- 仅当新增和删除数据modCount自增，扩容和更新时不变

jdk8相对于jdk7变化

- 引入了红黑树，对于长链表查询效率更高
- 在resize时，链表依然按照顺序分配，不再逆序分配（jdk7链表逆序分配，实现简单，但在并发使用时可能产生死循环）

### LinkedHashMap

继承自HashMap，同样也复用hashmap的数组+链表结构，只是新增了一条链表，来表示顺序。新增的节点都放到链表末尾，遍历时从链表头开始遍历，扫描至链表尾部

accessOrder: true表示访问列表为访问数据，false表示列表为插入顺序。LinkedHashMap默认false，表示插入和更新的节点都迁移到链表末尾；当为true时，访问过的元素会迁移到链表末尾

put() 使用hashMap put方法

```java
// 16-容器容量；0.75f-负载因子；true-访问顺序accessOrder
new LinkedHashMap<>(16, 0.75f, true);
```

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

## JUC

### ConcurrentHashMap(chm)

Jdk1.7和jdk1.8实现不同

#### jdk1.7

https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/

![](../image/169f29dca9416c8f)

ConcurrentHashMap结构，分为多个segment段，每个段类都一个HashEntry[]数组，类似于一个hashMap，各个段之间互不干扰

- key和value不能为null

核心成员变量

```
/**
 * Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
 */
final Segment<K,V>[] segments;

transient Set<K> keySet;
transient Set<Map.Entry<K,V>> entrySet;
```

```java
// 段是ConcurrentHashMap内部类
static final class Segment<K,V> extends ReentrantLock implements Serializable {
       private static final long serialVersionUID = 2249069246763182397L;
       
       // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
       transient volatile HashEntry<K,V>[] table;
       transient int count;
       transient int modCount;
       transient int threshold;
       final float loadFactor;      
}

static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
}
```

put方法

1. 通过key定位到Segment
2. 获取Segment的锁。第一步先尝试通过tryLock非公平锁方式CAS获取锁，若失败超过一定次数则lock阻塞获取锁

get方法不加锁:hashEntry中的value用volatile修饰，保证变量的可见性

size方法：先通过两次不加锁计算各段大小总和，若两次的modCount相等则计算完成；否则对所有分段加锁，计算总大小

#### jdk1.8

![](../image/5cd1d2ce33795.jpg)

结构类似于HashMap

- 无modCount
- 采用Node替换HashEntry，后续Node为加锁对象

put方法如下。键或值为空时抛出NPE

![5cd1d2cfc3293](../image/5cd1d2cfc3293.jpg)

1. 遍历表
2. 表为空则进行初始化
3. 若当前桶未初始化，则通过cas进行初始化
4. 当hash=MOVED=-1时，需要进行扩容
5. 获取表头锁
6. 当链表数量大于8则转成红黑树

get方法，无需加锁。可以通过hash<0来识别map正在扩容，需要到nextTable查询结果

扩容

https://www.cnblogs.com/sanzao/p/10792546.html

- 扩容可以并发执行，当进行扩容时标记map为扩容。后续put操作可以帮助扩容

Jdk1.8相对于jdk1.7改动

- 引入了红黑树，提高查询效率
- 摒弃了段的概念，结构更像hashMap，直接锁定单个hash桶，锁粒度更小
- 加锁时直接使用synchronized（可能适用于并发度较高的场景）
- 扩容时将任务拆分，使用多线程并发执行加快扩容
- 获取map大小，引入计数器概念（CounterCell数组）直接无锁获取。而jdk1.7需要对所有段加锁。无modCount，1.7还有

### CopyOnWriteArrayList/CopyOnWriteArraySet

线程安全list，读取迭代期间不需要对容器进行加锁或复制。

对list修改时，先加锁，复制一个全新的不可变数组，加入新元素，list底层数组引用不变，因此该集合类适用于遍历远多于修改场景

### ConcurrentLinkedQueue

线程安全list，采用非阻塞的方法实现（CAS）。基于链接节点实现的一个无界安全队列，采用先进先出的方式

高并发下性能最好的队列

### ReentrantLock

可重入锁，每次只能一个线程能获取锁。可重入性，获取锁的线程可多次获取，通过计数标记，完全释放锁时也必须要释放与加锁同样的次数

当且仅当内置锁（synchronized）不能满足当前需求时，可以使用ReentrantLock

默认新建非公平性锁，若传入参数true可选择公平性锁。

**tryLock()**:  非公平性方式直接CAS获取锁，不关心等待顺序

**lock()-非公平锁 **：直接CAS获取锁（若无锁状态直接获取锁），失败再非公平性尝试获取锁，再失败将自己加入到等待队列中，并将当前线程中断等待，返回

**lock()-公平锁 **：若当前锁处于未被获取状态且等待队列内无等待中的任务，CAS获取锁；否则将任务添加到等待队列中，并将当前线程中断等待，返回

```java
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  if (c == 0) {
    if (!hasQueuedPredecessors() &&
        compareAndSetState(0, acquires)) {
      setExclusiveOwnerThread(current);
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    int nextc = c + acquires;
    if (nextc < 0)
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}
```

公平锁为了保证获取锁的顺序，阻塞几率高，唤醒的消耗的性能高。而非公平锁可能不阻塞就可获取锁，阻塞几率较低，唤醒消耗的性能就低。非公平锁虽然降低了性能开销，但可能有线程处于一直等待获取锁的状态，造成“饥饿”现象

当完全释放锁时，会唤醒等待队列中的第一个等待者，通知其获取锁

synchronized也隐式支持重入，比如一个线程对一个变量同时加锁，不会阻塞

### AbstractQueuedSynchronizer

并发编程实现同步器的一个框架，基于FIFO实现（Reentrant中的lock继承自它）

AQS（简称）中存在一个FIFO队列，存放阻塞的节点，节点状态如下

- CANCELLED：因超时或中断被放入队列
- CONDITION：表示该线程因不满足某个条件而被放入队列
- SIGNAL：该线程需要被唤醒
- PROPAGATE：传播，在共享模式下，当前节点release后，需要通知传播通知给后续节点

模式：独占模式：只能一个线程独占资源，如ReentrantLock；共享模式，资源可被多个线程同时持有，如CountDownLatch

```java
// 获取锁
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

### StampedLock

和读写锁类似。但是有以下不同

- 当使用读锁时，可以通过乐观模式获取，先记录版本号，获取完成再对比版本号，若相同直接返回无需加锁；若失败则添加读锁。在读锁操作时性能上有提升
- 读锁可以升级为写锁

```java
public class StampedLockTest {
    private static StampedLock stampedLock = new StampedLock();
    private int x;
    private int y;

    public void move(int x, int y) {
        long stamp = stampedLock.writeLock();
        try {
            this.x = x;
            this.y = y;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
    }

    public int get() {
        // 记录版本号
        long stamp = stampedLock.tryOptimisticRead();
        int currentX = x;
        int currentY = y;
        if (!stampedLock.validate(stamp)) {
            // 校验不通过尝试获取读锁
            long tmp = stampedLock.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(tmp);
            }
        }
        return currentX * currentY;
    }

    // 读锁升级为写锁
    public void moveIfAtOrigin(int newX, int newY) {
        long stamp = stampedLock.readLock();
        try {
            while (x == 0 && y == 0) {
                long stamp2 = stampedLock.tryConvertToWriteLock(stamp);
                if (stamp2 != 0) {
                    // 转换成功
                    x = newX;
                    y = newY;
                    break;
                } else {
                    stampedLock.unlockRead(stamp);
                    stamp = stampedLock.writeLock();
                }
            }
        } finally {
            stampedLock.unlock(stamp);
        }
    }
}
```

特点

- 所有获取锁的方法，都返回一个邮戳（stamp），为0表示获取失败，否则获取成功
- 所有释放锁的方法，都需要一个邮戳，且必须和获取锁时得到的stamp一致
- 不可重入，写锁不可重复获取
- 支持读写锁之间的互相转换

### wait和notify

二者都是Object类的方法，都是针对对象调用的。wait使线程释放对象锁资源，进入等待态，等他其他线程唤醒。热点如下

- 必须持有对象锁的情况下才能调用wait
- 必须持有对象锁的情况下才能调用notify和notifyAll
- 唤醒处于wait的线程后，仍需要获取锁才能继续执行

### Condition

condition提供了类似于Object的监听器接口，与Lock接口配合实现等待、通知模式。等待之后需要被唤醒。

> 与wait和notify相比，Condition就是一个锁对象，不过是lock的，使用时先获取锁，然后直接wait或notify，避免synchronized

await()等待：调用时线程处于阻塞状态，等待唤醒

signal()：唤醒一个线程，将条件队列（Condition类中）头节点移动到同步队列中（AQS类中），状态设置为signal；若设置失败，直接调用唤醒的线程

```java
class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```

### Semaphore

信号量，一个控制多个共享资源的计数器。state表示剩余可获取的许可数

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

当信号量为1时，可被当作互斥锁使用。

当信号量为0时，仍然可以release变成1，然后再获取

acquire(): 获取锁，若获取不到则阻塞，直到取得锁为止

release(): 释放锁，信号量加1。释放成功后唤醒等待队列中的线程，使得线程继续运行，尝试获得锁

### CountDownLatch

可拦截一组线程，判断该组线程是否执行完毕。只能使用一次

```java
static void countDownLatch() throws InterruptedException {
        int count = 10;
        CountDownLatch countDownLatch = new CountDownLatch(count);
        for (int i = 0; i < count; ++i) {
            new Thread() {
                @SneakyThrows
                @Override
                public void run() {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000));
                System.out.println(Thread.currentThread().getName() + " ing");
                countDownLatch.countDown();
                countDownLatch.await();
                System.out.println(Thread.currentThread().getName() + " finish");
            }}.start();
        }
    }
```

### CyclicBarrier

可循环使用的屏障，主要功能是拦截一组线程，当所有线程都到达屏障（计数器为0），唤醒所有等待线程

await(): 一个线程到达屏障，屏障计数器减一。当屏障计数器不为0时，线程自旋调用wait；当屏障计数器为0时，唤醒所有等待的线程，并重置屏障

await(long timeout, TimeUnit unit): 超时等待，超过指定时间便放行

reset: 重置屏障状态。具体实现为先破坏屏障，使得唤醒所有线程并抛出异常，再重新初始化屏障

```java
static void cyclicBarrier() throws InterruptedException {
        int count = 10;
        CyclicBarrier barrier = new CyclicBarrier(count);

        for (int i = 0; i < count; ++i) {
            new Thread() {
                @SneakyThrows
                @Override
                public void run() {
                    Thread.sleep(ThreadLocalRandom.current().nextInt(1000));
                    System.out.println(Thread.currentThread().getName() + " 正在写入数据");
                    barrier.await();
                    System.out.println(Thread.currentThread().getName() + " finish");
                }
            }.start();
        }
    }
```

和countDownLatch对比

- 侧重点不同。CountDownLatch一般用于一个线程等待其他线程执行完毕再执行。而cyclicBarrier一般用于等待所有线程到达某个状态，再同时执行
- 可以循环使用，countDownLatch只能使用一次

### Exchanger

用于两个线程之间交换数据，当一个线程提前执行exchanger后会等待第二个线程执行到exchanger，之后交换数据

```java
// thread-1
int data = 1;
data = exchanger.exchange(data);
// thread-2
int data2 = 2;
data2 = exchanger.exchange(data2);
// 交换后，数据对调
```

## 线程池

### ThreadPoolExecutor

参考：https://www.jianshu.com/p/ade771d2c9c0

```
ctl是一个原子变量，前3位表示线程池状态runState（简称rs），后29位表示当前有效的线程数workerCount（简称wc），即worker的数量
runState装填
1. RUNNING，可以新增线程，同时处理queue的任务。值111
2. SHUTDOWN，不可以新增线程，但是处理queue的任务。值000
3. STOP，不可以新增线程，同时不处理queue的任务。值001
4. TIDYING，所有线程的终止了，同时workerCount为0，中间态，最后会转化为TERIMINATED。值010
5. terminated()方法结束，状态变为TERIMINATED。值011
```

- corePoolSize：核心线程数量

- maximumPoolSize：最大线程数量

- keepAliveTime：worker最长闲置时间，超过改时间worker过期被清理。核心线程默认永不过期，allowCoreThreadTimeOut控制，默认false

- workQueue：任务队列，阻塞队列，BlockingQueue

- threadFactory：线程工厂

- RejectedExecutionHandler：拒绝策略。默认拒绝任务，抛出异常

- Worker：线程池任务类，可持有一个任务运行

#### addWorker

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  			// CAS循环新增有效线程
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            // 当状态大于shutdown时，拒绝新增任务
            // 当状态等于shutdown时，只接受当任务队列非空时的null任务。其他任务拒绝
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

          	// CAS增加wc
            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
              	// CAS新增有效线程数，若成功则跳出循环
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
              	// 判断状态是否改变，不改变继续循环增加wc
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
              	// 获取锁，往workers中添加worker
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```

#### runWroker

Worker运行方法

```java
final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
  			// 标记任务是否异常退出
        boolean completedAbruptly = true;
        try {
          	// worker一直运行，若worker只有task不为空直接处理，若为空通过getTask()从任务队列中获取任务
          	// 若getTask也返回，则该worker退出（过期或异常）
            while (task != null || (task = getTask()) != null) {
              	// 获取锁
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
          	// 处理结束worker
            processWorkerExit(w, completedAbruptly);
        }
    }
```

#### getTask

从任务队列中获取任务，当线程池允许核心线程过期或worker数量大于核心线程数量，则从工作队列获取任务为超时获取，否则为阻塞获取；当线程池任务过多或worker闲置超时，则CAS减少worker数量，成功则返回null，通过外层停止worker

```java
private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
          	// 是否允许超时获取任务
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

          	// 当超过最大线程数或超时，且线程池中线程大于1或任务队列为空都满足时，（超时或任务过多）尝试减少worker，返回null
            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
              	// 根据超时类型决定超时获取任务或阻塞获取任务
                Runnable r = timed ?
                  	// 超时返回null，发送中断则抛出中断异常
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
              	// 超时
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
```

#### shutdown

停止线程池，中断所有空闲线程，其他处于运行中的线程仍可以进行运行。共分为三步：1、检查是否能操作目标线程；2、将线程池状态改为SHUTDOWN；3、中断所有空闲线程

```java
public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          	// 判断是否可以操作目标线程
            checkShutdownAccess();
          	// 线程池状态改为SHUTDOWN，不会再新建worker
            advanceRunState(SHUTDOWN);
          	// 中断所有闲置线程（不在运行中）
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }

private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          	// 对所有的worker进行遍历，尝试加锁，不会中断正在运行的线程。运行中的线程会加锁
            for (Worker w : workers) {
                Thread t = w.thread;
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
```

#### ShutDownNow

停止线程池，中断所有线程，不管是否处于运行中，风险较高

```java
public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
```

### FixedThreadPool

```java
Executors.newFixedThreadPool(10);
new ThreadPoolExecutor(nThreads,nThreads,0L, TimeUnit.MillSeconds, new LinkedBlockingQueue<Runnable>());
```

固定大小的线程池。核心线程数等于最大线程数，使用无界阻塞队列，线程不会过期

### SingleThreadPool

```java
Executors.newSingleThreadPool();
new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
```

和固定大小线程池相似，只有一个核心线程

### CachedThreadPool

```java
Executors.newCachedThreadPool();
new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
```

缓存线程池，无核心线程，当处理任务的线程不足会创建新线程。线程闲置超过60s则释放。当主线程提交任务的速率大于后台线程处理任务的速度，会不断地创建线程

SynchronousQueue：每个操作必须等待另一个线程的移除操作

缓存线程池的步骤

1. 首先执行SynchronousQueue.offer(Runnable task)。如果当前线程池中存在空闲线程正在执行SynchronousQueue.poll(keeyAliveTime, TimeUnit.NANOSECONDS)，则配对成功取出线程执行目标任务。否则跳至步骤2
2. 当线程池中无空闲线程，则创建一个新线程，执行目标任务
3. 线程执行完毕后，SynchronousQueue.poll(keeyAliveTime, TimeUnit.NANOSECONDS)。这个poll操作会让线程在SynchronousQueue中最多等待60s。若主线程提交了任务且与poll配对成功，取出线程执行；否则线程闲置超过60s，线程终止

### ScheduleThreadPool

可以延迟执行任务，或定时执行任务。和Timer类似，但ScheduleThreadPool功能更强大，更灵活，可以在构造函数中指定多个对应的后台线程

### ForkJoinPool

forkJoinPool是jdk1.7引入的线程池，可以将一个任务拆成多个小任务，然后将多个小任务结果汇总（即join）。

work-stealing：线程池分配了与线程数相等的队列，初始时任务平均分配。当有线程提前执行完队列中所有任务时，会随机从其他队列拉取任务执行

常用方法：使用forkJoin框架，需要创建ForkJoinTask任务，然后通过fork和join的机制实现。ForkJoinTask有两个实现类

- RecursiveAction：用于没有返回结果的任务
- RecursiveTask：用于有返回结果的任务

```java

import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

/**
 * Created by TF016591 on 2017/11/8.
 */
public class CountTaskTmp extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 2;
    private int start;
    private int end;

    public CountTaskTmp(int start, int end) {
        this.start = start;
        this.end = end;
    }

    //实现compute 方法来实现任务切分和计算
    protected Integer compute() {
        int sum = 0;
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++)
                sum += i;
        } else {
            //如果任务大于阀值，就分裂成两个子任务计算
            int mid = (start + end) / 2;
            CountTaskTmp leftTask = new CountTaskTmp(start, mid);
            CountTaskTmp rightTask = new CountTaskTmp(mid + 1, end);

            //执行子任务
            leftTask.fork();
            rightTask.fork();

            //等待子任务执行完，并得到结果
            int leftResult = (int) leftTask.join();
            int rightResult = (int) rightTask.join();

            sum = leftResult + rightResult;
        }

        return sum;
    }

    public static void main(String[] args) {
        //使用ForkJoinPool来执行任务
        ForkJoinPool forkJoinPool = new ForkJoinPool();

        //生成一个计算资格，负责计算1+2+3+4
        CountTaskTmp task = new CountTaskTmp(1, 4);

        Integer r = forkJoinPool.invoke(task);
        System.out.println(r);
        //  或者可以这样写
        //        Future<Integer> result = forkJoinPool.submit(task);
        //        try {
        //            System.out.println(result.get());
        //        } catch (Exception e) {
        //        }
    }
}
```

```java
// ForkJoinPool默认线程数为cpu核数
public ForkJoinPool() {
        this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
             defaultForkJoinWorkerThreadFactory, null, false);
    }

public ForkJoinPool(int parallelism,
                        ForkJoinWorkerThreadFactory factory,
                        UncaughtExceptionHandler handler,
                        boolean asyncMode) {
        this(checkParallelism(parallelism),
             checkFactory(factory),
             handler,
             // asyncMode=true 先进先出；为false时后进先出
             asyncMode ? FIFO_QUEUE : LIFO_QUEUE,
             "ForkJoinPool-" + nextPoolId() + "-worker-");
        checkPermission();
    }
```

```java
Executors.newWorkStealingPool();
public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

