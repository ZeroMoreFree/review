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
  - `-XX:MetaspaceSize`
  - `-XX:PrintGCDetails`
  - `-XX:SurvivorRatio`
  - `-XX:NewRatio`
  - `-XX:MaxTenuringThreshold`
