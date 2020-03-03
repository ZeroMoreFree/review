# JUC多线程及高并发

>JUC:java.util.concurrent

## volatile是什么

- 是一种轻量级的同步机制
  - 保证可见性
  - **不保证原子性**
  - 禁止指令重排

- JMM（Java Memory Model）（JAVA内存模型）

>`主内存`是共享区域，每个线程拥有自己的`工作内存`。线程之间的通信（传值）必须通过主内存来完成  
线程先从主内存拷贝变量到自己的工作内存，完成操作后将变量写回主内存

- 三大特性
  - 可见性：主内存中变量的改动，第一时间通知到各个线程中
  - 原子性：操作不可分割
  - 有序性

- 并发和并行的区别
  - 并发：秒杀系统那样的，多个线程抢一个资源
  - 并行：同时做很多事情，比如一边煮开水，一边叠衣服

- 用于`DCL单例模式`

## CAS

- CAS是什么：compare and swap，先比较后交换，是一条CPU并发原语，由底层保证原子性 
- 动作：比较当前工作内存中的值和主内存中的值，如果相同则执行规定操作，否则继续比较直到主内存和工作内充中的值一致为之
- 底层原理
  - 自旋锁
  - `unsafe类`：CAS的核心类，在`rt.jar`里面，类中大都是`native`方法，可以直接操作内存等底层资源执行相应任务
- `synchronize`只能一个线程，而`CAS`可以多个线程进入，然后通过不断地比较，直到成功，交换值。
- `CAS`的缺点
  - 循环时间长，开销大
  - 只能保证`一个共享变量`的原子操作
  - `ABA问题`
    - 是什么？线程1工作内存中的值是A，主内存中的值，一开始是A，后来被线程2修改为B，在线程1还没与主内存比较并交换之前，线程2由把主内存中的值修改为A。此时线程1开始与主内存中的值比较，并修改成功。
    - 怎么解决：原子引用+ 时间戳：AtomicStampedReference
- `原子引用`：AtomicReference<V>

## 集合类不安全的问题

- `ArrayList`：add方法没有加锁，多线程下添加元素
  - 故障现象：`ConcurrentModificationException`
  - 导致原因：并发争抢修改导致，参考花名册签名情况。一个人正在写入，另外一个同学过来抢夺，导致数据不一致异常，也即并发修改异常
  - 解决方案
    - `Vector`：线程安全，但是并发性低
    - `Collections.synchronizeList(new ArrayList())`
    - `CopyOnWriteArrayList `
      - 写时复制，读写分离的思想
      - 在写入的地方加锁，对于要写入的，复制原先的数组，添加元素后再替换到原先的数组引用。数组本身不加锁，所以对于`读的操作，不会影响`。

- `HashSet`
  - 故障现象和导致原因，同`ArrayList`
  - 解决方案：
    - `Collections.synchronizeSet(new HashSet())`
    - `CopyOnWriteArraySet`
  - `HashSet`的底层数据结构，就是`HashMap`,set的值，就是map的key，所有的key的value都是一个叫做`PRESENT`的`Object`对象

- `HashMap`
  - 故障现象和导致原因，同`ArrayList`
  - 解决方案：
    - `ConcurrentHashMap`
    - Collections.synchronizeMap(new HashMap())

## TransferValue小练习

- 八种基本类型都是在`栈`上分配的
- `栈`管运行，`堆`管存储
- String是放在`字符串常量池`

## Java锁

- 公平锁和非公平锁

  ```java
  # 非公平锁
  Lock lock = new ReenTrantLock();

  # 公平锁
  Lock lock = new ReenTrantLock(true);
  ```

  - 公平锁：多个线程按照申请锁的顺序来加锁
  - 非公平锁：在高并发下，有可能造成优先级反转或者`饥饿现象`（某个线程一直没有获取到锁）
  - 非公平锁的`吞吐量`比公平锁大
  - `synchronized`是一种非公平锁

- 可重入锁（又名递归锁）
  - 同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码。
  - 一个线程在外层方法获取锁之后，进入内层方法会自动获取锁（method1调method2）
  - `ReentrantLock`和`synchronized`都是可重入锁
  - 作用：避免死锁

- 自旋锁
  - 尝试获取锁的线程不会立刻`阻塞`，而是采用`循环`的方式去尝试获取锁
    - 好处：减少线程上下文的切换
    - 缺点：循环会消耗CPU
  - 手写一个自旋锁

    ```Java
    public class SpinLock {

        AtomicReference<Thread> lock = new AtomicReference<>();

        public void lock(){

            Thread thread = Thread.currentThread();
            while (!lock.compareAndSet(null,thread));

        }

        public void unLock(){

            Thread thread = Thread.currentThread();
            lock.compareAndSet(thread,null);

        }

    }
    ```

- 独占锁（写锁）
  - 该锁一次只能被一个线程所只有，比如reentrantlock或者synchronized
- 共享锁（读锁）
  - 该锁可被多个线程所持有
  - `ReenTrantReadWriteLock`其读锁是共享锁，其写锁是独占锁
    - 读-读能共存
    - 读-写不能共存
    - 写-写不能共存

## 几个工具

- CountDownLatch
  - 可以设置它的`count`数值，然后调用`countDown()`方法使其减一
  - 其`await()`方法会一直阻塞，直到它的`count`变为0。

- CyclicBarrier
  - 让一组线程到达一个屏障时被阻塞，直到最后一个线程到达，所以被拦截的线程才会继续干活
  - 设置一个数值，表示需要多个线程被阻塞才满足条件，线程调用CyclicBarrier的`await()`方法进行阻塞

- Semaphore（信号量）
  - 用于多个共享资源的互斥使用
  - 用于并发线程数的控制
  - 用`acquire()`方法获取信号量，用`release()`方法释放信号量
  - 例子：六个车子抢三个车位

## 阻塞队列

- 可以结合`消费线程`和`生产线程`实现生产者和消费者模式
- 如果不使用这个工具，我们在实现生产者和消费者模式的时候，我们需要自己精确控制线程的`wait()`和`notify()`。这个工具大大减少了我们实现的复杂度，我们现在不需要去关心这些细节了。

- 接口`BlockingQueue`，父接口是Colleciton，有七个实现类
  - `ArrayBlockingQueue`：数组结构，有界
  - `LinkedBlockingQueue`：链表结构，有界（Integer.MAX_VALUE）但接近无界
  - `SynchronousQueue`：单个元素的队列
  - DelayQueue
  - LinkedTransferQueue
  - PriorityBlockingQueue
  - LinkedBlocking`Deque`：双向阻塞队列

- `BlockingQueue`核心方法
  - 第一组：超过预定值会抛出异常：`add(e)`、`remove()`。检查队头元素：`element()`
  - 第二组：超过预定值不会抛出异常：`offer(e)`，`poll()`。检查队头元素：`peek()`
  - 第三组：超过预定值会阻塞：`put(e)`,`take()`
  - 第四组：可以指定阻塞的超时时间：`offer(e, time, unit)`,`poll(time, unit)`

- `SynchronousQueue`
  - 每一个put操作必须等待一个take操作，否则不能再继续添加元素，反之亦然

- 使用场景
  - 生产者消费者模式
  - 线程池
  - 消息中间件

- 生产者与消费者的实现方式
  - `synchronized`版本，使用`wait()`和`notify()`
  - `lock`版本，使用`await()`和`signalAll()`
  - 阻塞队列版本

- `synchronized`和`lock`有什么区别
  - 原始构成
    - synchronized是关键字，属于JVM层面，底层是monitor对象
    - lock是具体类，是api层面的
  - 使用方法
    - synchronize不需要用户手动释放锁
    - lock需要用户手动释放锁，否则可能造成死锁
  - 等待是否可中断
    - synchronize不可中断，只会正常完成或者抛异常
    - reentrantlock可中断
      - 设置超时方法tryLock(timeout timeunit)
      - lockInterruptibly()配合interrupt()方法可中断
  - 加锁是否公平
    - reentrantlock有公平锁选项
  - 锁绑定多个条件Condition
    - synchronized没有条件，只能随机唤醒一个线程，或者唤醒全部线程
    - reentrantlock可以使用`Condition`实现精确唤醒，也即控制condition的await()和signal()

## Callable

- 进行多线程编程的四种选择
  - 继承`Thread`类
  - 实现`Runnable`接口
  - 实现`Callable`接口
  - 线程池

- `Runnable`和`Callable`的区别
  - `Callable`有返回值
  - `Callable`的call方法有受检异常，而`Runnable`的run方法没有受检异常
  
- callable可以让多个任务并行计算，然后再汇总结果
- callable -> FutureTask -> Thread
- callable的get方法会阻塞

- 多个线程去抢`FutureTask`，只会计算一次

## 线程池

- 为什么用线程池
  - 线程复用：减少开销、提高响应速度
  - 控制最大并发数
  - 管理线程：进行统一的分配、调优、监控

- 五种常用的线程池
  - `Executors.newFixedThreadPool(int nThreads)`：固定数值的线程
  - `Executors.newSingleThreadExecutor()`：只有一个线程，保证所有任务按照指定顺序执行
  - `Executors.newCachedThreadPool()`：多个线程，可扩充
  - 以上三个线程池的底层都是`ThreadPoolExecutor`
  - Executors.newWorkStealingPool()：Java8的新线程池
  - Executors.newScheduledThreadPool(int corePoolSize)：带时间调度

- 七大参数
  - `int corePoolSize`：线程池中的常驻核心线程数
  - `int maximumPoolSize`：线程池能够容纳同时执行的最大线程数，必须大于1
  - `long keepAliveTime`：多余的空闲线程的存活时间
  - `TimeUnit unit`：keepAliveTime的单位
  - `BlockingQueue<Runnable> workQueue`：任务队列，被提交但尚未被执行的任务
  - `ThreadFactory threadFactory`：生成线程池中工作线程的工厂
  - `RejectedExecutionHandler handler`：拒绝策略，表示当队列满了并且工作线程大于等于线程池的最大线程数，拒绝接收新请求的策略

- 底层工作原理
  
  ![底层工作原理](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1583253247270&di=c1951943514eb2bd2cc35c082a585917&imgtype=0&src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F4a237fd689fcf62970614142138e1011.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100)

- 线程池的拒绝策略
  - 触发条件：线程数达到`maximumPoolSize`，而且任务队列`workQueue`也满了
  - JDK的内置策略：
    - `AbortPolicy`：默认的拒绝策略，直接抛出`RejectedExecutionException`
    - `CallerRunsPolicy`：不抛弃任务、不抛异常。而是将某些任务回退到调用者（比如main线程提交，那就回退到main线程）那里处理。
    - `DiscardOldestPolicy`：抛弃*队列中*等待最久的任务，尝试再次提交当前任务
    - `DiscardPolicy`：直接丢弃*新*任务

- 阿里巴巴实践建议
  - 线程不用手动创建，而且应该使用线程池获取
  - 线程池不允许使用`Executors`去创建，原因如下
    - `FixedThreadPool`和`SingleThreadPool`允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
    - `CachedThreadPool`和`ScheduledThreadPool`允许的创建线程数量为Integer.MAX_VALUE,可能会创建大量的线程，从而导致OOM。
  - 线程池应该通过`ThreadPoolExecutor`的方式创建

- 如何合理配置线程池的线程数量（maximumPoolSize）
  - CPU密集型
    - 特点：任务需要大量的运算，而没有阻塞，CPU一直全速运行
    - 一般公式：（CPU核数 + 1个线程）的线程池
  - I/O密集型
    - 特点：并不是一直在运行任务，会有大量的阻塞，需要多配置线程数
    - 一般公式：CPU核数 * 2
    - 大厂公式：CPU核数/(1-阻塞系数)，阻塞系数0.8~0.9
      - 举例：8核CPU：8/(1-0.9) = 80个线程数

- 死锁编码以及定位分析
  - 现象：两个线程因为争抢资源而**互相等待**
  - 产生死锁的原因
    - 系统资源不足
    - 进程运行推进的顺序不合适
    - 资源分配不当
  - 代码演示：A线程持有lockA等待lockB,B线程持有lockB等待lockA
  - 解决
    - jps查看当前运行的线程及其线程号
    - jstack 加上线程ID号，查看栈内情况

      ```java
      Found one Java-level deadlock:
      =============================
      "bbb":
        waiting to lock monitor 0x000000001d4035a8 (object 0x000000076b19cc68, a java.lang.String),
        which is held by "aaa"
      "aaa":
        waiting to lock monitor 0x000000001d400bb8 (object 0x000000076b19cca0, a java.lang.String),
        which is held by "bbb"
      ```