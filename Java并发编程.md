# Java并发编程

**1. 概述**

- 三种性质

  - **可见性**：一个线程对共享变量的修改，另一个线程能立刻看到。**缓存**可导致可见性问题。
  - **原子性**：一个或多个CPU执行操作不被中断。**线程切换**可导致原子性问题。
  - **有序性**：编译器优化可能导致指令顺序发生改变。**编译器优化**可能导致有序性问题。

- 三个问题

  - **安全性问题**：线程安全

  - **活跃性问题**：死锁、活锁、饥饿

  - 性能问题

    ：

    - **使用无锁结构**：TLS，Copy-On-Write，乐观锁；Java的原子类，Disruptor无锁队列
    - **减少锁的持有时间**：让锁**细粒**度。如ConcurrentHashmap；再如读写锁，读无锁写有锁

**2. Java内存模型**

- volatile

  - **C语言中的原意**：禁用CPU缓存，从内存中读出和写入。

  - Java语言的引申义

    ：

    - Java会将变量立刻写入内存，其他线程读取时直接从内存读（普通变量改变后，什么时候写入内存是不一定的）
    - 禁止指令重排序

  - 解决问题

    ：

    - 保证可见性
    - 保证有序性
    - 不能保证原子性

- Happens-Before规则（H-B）

  - **程序顺序性规则**：前面执行的语句对后面语句可见
  - **volatile变量规则**：volatile变量的写操作对后续的读操作可见
  - **传递性规则**：A H-B B，B H-B C，那么A H-B C
  - **管程中锁的规则**：对一个锁的解锁 H-B于 后续对这个锁的加锁

**3. 互斥锁sychronized**

- **锁对象**：非静态this，静态Class，括号Object参数

- 预防死锁：

  - **互斥**：不能破坏
  - **占有且等待**：同时申请所有资源
  - **不可抢占**：sychronized解决不了，Lock可以解决
  - **循环等待**：给资源设置id字段，每次都是按顺序申请锁

- 等待通知机制

  ：

  - wait、notify、notifyAll

**4. 线程的生命周期**

- **通用线程的生命周期**：

- ![img](https://img2018.cnblogs.com/blog/1096103/201904/1096103-20190427123816687-1018314801.png)

- **Java线程的生命周期**：

- ![img](https://img2018.cnblogs.com/blog/1096103/201904/1096103-20190427124143311-1370758951.png)

- 状态流转

  ：

  - RUNNABLE -- BLOCKED

    ：线程获取和等待sychronized隐式锁

    - ps：调用阻塞式API时，不会进入BLOCKED状态，但对于操作系统而言，线程实际上进入了休眠态，只不过JVM不关心。

  - RUNNABLE -- WAITING

    ：

    - Object.wait()
    - Thread.join()
    - LockSupport.park()

  - **RUNNABLE -- TIMED-WAITING**：调用各种带超时参数的线程方法

  - **NEW -- RUNNABLE**：Thread.start()

  - **RUNNABLE -- TERMINATED**：线程运行完毕，有异常抛出，或手动调用线程stop()

**6. 线程的性能指标**

- **延迟**：发出请求到收到响应
- **吞吐量**：单位时间内处理的请求数量
- 最佳线程数：
  - **CPU密集型**：线程数 = CPU核数 + 1
  - **IO密集型**：线程数 = （IO耗时/CPU耗时 + 1）* CPU核数
  - ![img](https://img2018.cnblogs.com/blog/1096103/201904/1096103-20190428112923654-1787921521.png)

**7. JDK并发包**

- Lock

  ：lock、unlock

  - **互斥锁**，和sychronized一样的功能，里面能保证可见性

- Condition

  ：await、signal

  - **条件**，相比于sychronized的Object.wait，Condition可以实现多条件唤醒等待机制

- Semaphore

  ：acquire、release

  - **信号量**，可以用来实现多个线程访问一个临界区，如实现对象池设计中的限流器

- ReadWriteLock

  ：readLock、writeLock

  - **写锁、读锁**，允许多线程读，一个线程写，写锁持有时所有读锁和写锁的获取都阻塞（写锁的获取要等所有读写锁释放）
  - 适用于读多写少的场景

- StampedLock

  ：tryOptimisticRead、validate

  - **写锁、读锁**（分**悲观读锁、乐观读锁**）：

- 线程同步：

  - CountDownLatch

    ：一个线程等待多个线程

    - 初始化 --> countDown（减1） --> await（等待为0）

  - CyclicBarrier

    ：一组线程之间相互等待

    - 初始化 --> 设置回调函数（为0时执行，并返回原始值） --> await（减1并等待为0）

- 并发容器：

  - List：

    - **CopyOnWriteArrayList**：适用写少的场景，要容忍可能的读不一致

  - Map：

    - **ConcurrentHashMap**：分段锁
    - **ConcurrentSkipListMap**：跳表

  - Set：

    - **CopyOnWriteArraySet**：同上
    - ***\*ConcurrentSkipListSet\****：同上

  - Queue

    ：

    - **分类**：阻塞Blocking、单端Queue、双端Deque
    - **单端阻塞（BlockingQueue）**：Array～、Linked～、Sychronized～、LinkedTransfer～、Priority～、Delay～
    - **双端阻塞（BlockingDeque）**：Linked～
    - **单端非阻塞（Queue）**：ConcurrentLinked～
    - **双端非阻塞（Deque）**：ConcurrentLinked～

- 原子类：

  - **无锁方案原理**：增加了硬件支持，即CPU的CAS指令
  - **ABA问题**：有解决ABA问题的需求时，增加一个递增的版本号纬度化解
  - **分类**：原子化基本数据类型，原子化引用类型、原子化数组、原子化对象属性更新器、原子化累加器

- Future：

  - **Future**：cancel、isCanceled、isDone、get
  - **FutureTask**：实现了Runnable和Future接口

- 强大工具类

  - **CompletableFuture**：一个强大的异步编程工具类（任务之间有聚合关系），暂时略
  - **CompletionService**：批量并行任务，暂时略

**8. 线程池**

- 设计原理：
  - 用**生产者消费**者模型，线程池是消费者，调用者是生产者。
  - 线程池对象里维护一个阻塞**队列**，一个已经跑起来的工作线程组T**hreadsList**
  - ThreadList里面循环从队列中去Runnable任务，并调用run方法

- ThreadPoolExcutor

  - 参数

    - **corePoolSize**：线程池保有的最小线程数

    - **maximumPoolSize**：线程池创建的最大线程数

    - **keepAliveTime**：工作线程多久没收到任务，被认为是闲的

    - **workQueue**：工作队列

    - **threadFactory**：通过这个参数自定义如何创建线程

    - handler

      ：任务拒绝策略

      - 默认为**AbortPolicy**，会抛出RejectedExecutionException，这是个运行时异常，要注意

  - 方法

    - void execute()
    - Future submit(Runnable task | Callable task)