# jvm

## 前提复习

- JVM内存结构
  - class loader
  - 运行时数据区
    - 方法区
    - 堆
    - java栈
    - 本地方法栈
    - 程序计数器
  - 执行引擎
- GC的作用域
  - 方法区
  - 堆
- 常见的垃圾回收算法
  - 引用计数（不被采纳）
    - 缺点：维护计数器有一定的消耗
    - 较难处理`循环引用`
  - 复制：用于新生代
    - 优点：没有碎片
    - 缺点：复制本身有较多的消耗
  - 标记-清除-：用于老年代
    - 优点：节省复制的消耗
    - 缺点：造成碎片空间
  - 标记-压缩：用于老年代
    - 较少的对象移动成本，没有内存碎片

## 如何确定垃圾，什么是GCRoot

- 枚举根节点做可达性分析（跟搜索路径）
- 从根节点出发，如果对象没有出现在链路上，就被判定为垃圾，等待被回收
- 可以做为`GCRoot`的对象
  - 虚拟机栈（栈帧中的本地变量表）中引用的对象
  - 方法区中的类静态属性引用的对象
  - 方法区中常量引用的对象
  - 本地方法栈中JNI（Native方法）引用的对象

## JVM调优和参数配置，查看JVM系统默认值

- JVM的参数类型
  - 标配参数
    - java -version
    - java -help
    - ......
  - X参数
    - -Xlint 解释执行
    - -Xcomp 第一次使用就编译成本地代码
    - -Xmixed 混合模式
  - **XX参数**
    - Boolean类型
      - `-xx:+ 某个属性值` 表示开启某属性
      - `-xx:- 某个属性值` 表示关闭某属性
      - 查看正在运行中的程序是否开启了某个属性

      ```java
        查看进程号：
        jps -l
        11664 sun.tools.jps.Jps
        10500
        3140 org.jetbrains.jps.cmdline.Launcher
        11004 zenolock.DeadLockDemo

        查看信息，以`PrintGCDetails`为例
        jinfo -flag PrintGCDetails 11004
        -XX:-PrintGCDetails

        显示减号，说明没有开启
      ```

    - key-value 设置类型
      - 模式：`-XX:属性key=属性value`

      ```java
      jinfo -flag MetaspaceSize 11004
      -XX:MetaspaceSize=128m
      ```

    - jinfo的使用
      - 查看某个属性值：jinfo -flag 属性值 进程号
      - 查看全部属性值：jinfo -flags 进程号
    - 两个特殊的属性
      - `-Xms` 等价于 `-XX:InitialHeapSize`
      - `-Xmx` 等价于 `-XX:MaxHeapSize`

- 查看JVM系统默认值
  - 第一种：jps + jinfo
  - 第二种
    - `java -XX:+PrintFlagsInitial` 主要查看初始值

    ```java
    bool UseLargePagesInMetaspace = false
    等于号前面有冒号的，表示这个值被修改过，不是jvm默认值
    bool UseLargePagesIndividualAllocation  := false
    ```

    - `java -XX:+PrintFlagsFinal` 主要查看最终值
    - `-XX:+PrintCommandLineFlags` 主要看默认的垃圾回收器

## JVM常用配置参数

- 基础知识复习
  - JDK1.8之后，`永久代`被取消了，由`元空间`取代。元空间使用的是本机物理内存，而不像永久代那样，使用的是JVM的堆内存，受`MaxPermSize`控制。
  - `Runtime.getRuntime().totalMemory()` 等价于 `-Xms`
  - `Runtime.getRuntime().maxMemory()` 等价于 `-Xmx`
- 常用参数
  - `-Xms`：等价于`-XX:InitialHeapSize`，初始大小内存，默认为物理内存的1/64
  - `-Xmx`：等价于`-XX:MaxHeapSize`，最大分配内存，默认为物理内存的1/4
  - `-Xss`：等价于`-XX:ThreadStackSize`,设置单个线程栈的大小，一般默认512k~1024k
  - `-Xmn`：设置`年轻代`的大小
  - `-XX:MetaspaceSize`：设置元空间的大小，默认21M左右
  - `-XX:PrintGCDetails`：输出GC的详细日志信息
    - 规律： 名称 GC前内存占用 -> GC后内存占用 总内存总大小
  - `-XX:SurvivorRatio`：设置新生代中`eden`和`s0/s1`空间的比例，默认 8:1:1
  - `-XX:NewRatio`:设置年轻代与老年代在堆结构的占比。
  - `-XX:MaxTenuringThreshold`：设置垃圾的最大年龄，默认十五次。可设置的值为0~15

## 强软弱虚引用

![强软弱虚引用](http://dl.iteye.com/upload/attachment/193935/76e46aa5-1ffe-3111-9f16-c925092176f6.gif)

- 强引用Reference
  - 把一个对象赋值给一个引用变量，该变量就是强引用
  - 就算是出现了`OOM`也不会对强引用进行回收
- 软引用SoftReference
  - 当系统内存充足时不会被回收，反之会被回收
  - 适用场景：读取大量的图片，图片对象可以使用软引用，避免OOM
- 弱引用WeakReference
  - 只要发生GC，就会被回收
  - `WeakHashMap`
    - 当key被设置为null，那么那个`entry`就会被GC回收
- 虚引用PhantomReference
  - 形同虚设，任何时候都可能被垃圾回收器回收掉
  - 必须配合引用队列（ReferenceQueue）进行使用
  - 意义：在对象被回收的时候，收到一个通知，或添加后续处理。
- 引用队列ReferenceQueue
  - 可以让引用进行注册，引用被GC回收之后，该队列会保留引用的信息

```Java
  //强引用
  Object obj = new Object();

  //软引用
  SoftReference<Object> softReference = new SoftReference<>(obj);
  //弱引用
  WeakReference<Object> weakReference = new WeakReference<>(obj);

  //软引用和弱引用的适用场景
  //用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系
  Map<String,SoftReference<Object>> imageCache = new HashMap<>();

  //引用队列
  ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
  //虚引用
  PhantomReference<Object> phantomReference = new PhantomReference<>(obj,referenceQueue);
```

## OOM

- `StackOverflowError`：栈溢出，方法递归调用并且没有出口时，会撑爆栈
- `java heap space`：堆溢出，大量创建对象，堆放不下
- `GC overhead limit exceeded`：超过98%的时间用来做GC，并且回收成功很糟糕，继续触发GC，恶性循环
- `Direct buffer memory`：比如NIO之类的直接使用本地内存，所以没有垃圾回收管理，把本地内存（默认是全部内存的1/4）给撑爆了

  ```Java

  //设置直接内存的大小为5m
  -XX:MaxDirectMemorySize=5m

  //分配6m的直接内存
  System.out.println(sun.misc.VM.maxDirectMemory() / ((double)1024 * 1024));
  ByteBuffer.allocateDirect(6 * 1024 *2024);

  ```

- `unable to create new native thread`：应用创建的线程数超过操作系统的规定。linux默认单个线程允许创建的线程数是1024个
  - 降低线程数量
  - 修改操作系统的设置

- `Metaspace`
  - 该区域是Java8之后才有的，取代了原先的永久代，使用的是机器的内存，不是JVM的内存。
  - 存放的信息
    - 虚拟机加载的类信息
    - 常量池
    - 静态变量
    - 即时编译后的代码

## 垃圾收集器

- 四种主要的垃圾回收器类型
  - `Serial`：
    - 串行执行
    - 只使用一个线程进行垃圾回收，工作时会暂停所有的用户线程，不适合生产环境
  - `Parallel`：
    - 并行执行
    - 多个线程并行进行回收工作，工作时会暂停所有的用户线程，适合大数据处理等弱交互场景
    - 可以使用`-XX:ParallelGCThreads`来控制收集器的线程数量
  - `CMS`：
    - 并发执行
    - 工作时不需要停顿用户线程，适用于对响应时间有要求的场景
  - `G1`
    - 将堆内存分割成不同的区域，然后并发地对其进行垃圾回收

- 垃圾收集器的查看、配置
  - 查看默认的垃圾回收器：可以使用`-XX:+PrintCommandLineFlags`命令，如果显示的是`-XX:+UseParallelGC`则是并行垃圾回收器，如果显示的是`-XX:UseSerialGC`，则是串行垃圾回收器，以此类推
  - JVM默认的垃圾收集器有哪些
    - `UseSerialGC`：使用复制算法
    - `UseSerialOldGC`(已废弃)：使用标记-整理算法
    - `UseParallelGC`（Java8的默认垃圾收集器）：使用复制算法
    - `UseParallelOldGC`：使用标记-整理算法
    - `UseConcMarkSweepGC`：使用标记-清除算法
    - `UseParNewGC`：使用复制算法
    - `UseG1GC`

- 七种垃圾收集器概述

  ![使用区域](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1583578113475&di=afec0e530255ff06482ca963c9fd2ec4&imgtype=0&src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F01baa3f2adcda6e6141cd212b1ecb7a0.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100)

  - young区
    - `Serial`收集器
      - 如何开启:`-XX:+UseSerialGC`
      - 开启后的收集器组合：Serial（Young区使用）+Serial Old（Old区使用）
    - `ParNew`收集器
      - 如何开启：`-XX:+UseParNewGC`
      - 开启后的收集器组合，ParNew（Young）+Serial Old（Old）
    - `Parallel Scavenge`收集器
      - 如何开启：`-XX:+UseParallelGC`或`-XX:+UseParallelOldGC`
      - 收集器组合：`Parallel Scavenge`(young)+`Parallel Old`（old）

  - old区
    - `Parallel Old`
      - JDK1.6才开始提供
      - 开启参数和收集器组合同`Parallel Scavenge`
    - `CMS`
      - 如何开启：`-XX:+UseConcMarkSweepGC`,开启之后，会自动开启`-XX:+UseParNewGC`
      - 收集器组合：ParNew（Young）+CMS（Old）+Serial Old（以防CMS出错）
      - 工作的四个步骤
        - 初始标记：需要暂停所有的工作线程
        - 并发标记
        - 重新标记：需要暂停所有的工作线程
        - 并发清除
      - 优点：并发收集、低停顿
      - 缺点：
        - 并发执行，CPU压力大
        - 会产生大量的碎片
      - 可以指定`-XX:CMSFullGCsBeForeCompaction`参数来指示多少次GC之后，进行一个碎片压缩的`Full GC`
    - `Serial Old`
      - 如何开启`-XX:+UseSerialOldGC`
      - 收集器组合：`Parallel Scavenge`（Young）+`Serial Old`（Old）
      - java8开始，这个收集器不会被使用了、
    - 如何选择合适的垃圾收集器
      - 单CPU或小内存，使用`-XX:UseSerialGC`
      - 多CPU，追求`大吞吐量`，比如后台计算型应用，使用`-XX:UseParallelGC`或者`-XX:UseParallelOldGC`
      - 多CPU，追求`低停顿时间`，比如需要快速响应的互联网应用，使用`-XX:UseConcMarkSweepGC`
  
- `G1`垃圾收集器
  - 设计目的：CMS已经不错了，但是会产生内存碎片，所以设计G1来替代它，Java9之后为默认的垃圾回收器
  - 之前的收集器的特点
    - 年轻代和年老代是各自独立且连续的内存块
    - 年轻代收集使用单eden+S0+S进行复制算法
    - 老年代收集必须扫描整个老年代区域
    - 尽可能**少**而**快**速地执行GC为原则
  - `G1`的特点
    - 能充分利用多CPU等硬件优势，尽量缩短STW(stop the world)
    - 整体采用标记-整理算法，局部采用复制算法，不会产生内存碎片
    - 把内存划分为多个独立的子区域（Region），不再按原先的方式划分年轻代和年老代，而是在更小的范围内区分
    - 化整为零，避免全内存扫描，只需要按照Region来扫描就好
  - 底层原理
    - 小区域收集+形成连续的内存块，避免内存碎片
    - 回收过程
      - 初始标记
      - 并发标记
      - 最终标记
      - 筛选回收
  - 参数配置
    - `-XX:+UseG1GC`：开启收集器
    - `-XX:G1HeapRegionSize=n`：设置每个Region的大小
    - `-XX:MaxGCPauseMillis=n`：最大GC停顿时间，JVM将尽可能停顿小于这个时间
    - `-XX:InitialtingHeapOccupancyPercent=n`：堆占用多少比例就触发GC，默认为45%
    - `-XX:ConcGCThreads=n`：并发GC使用的线程数
    - `-XX:G1ReservePercent=n`：设置预留空闲空间的百分比，默认为10%
  - 与`CMS`相比的优势
    - 没有内存碎片
    - 可以设置预想停顿时间
