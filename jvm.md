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
