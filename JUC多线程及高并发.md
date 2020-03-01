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
  