# 线程池

**线程池的优势**

线程池的工作主要是控制运行的线程数量 处理过程中将任务放入队列 然后在线程创建后启动这些任务 如果线程数量超过了最大数量 超出数量的线程排队等候 等其他线程执行完毕 再从队列中取出任务来执行

**主要特点**

线程复用 控制最大并发数 管理线程

***

```java
public static void main(String[] args) {
    //    Executors 线程池工具类
    // 一池五个线程 固定线程数
    ExecutorService threadPool = Executors.newFixedThreadPool(5);

    // 一池一个工作线程
    ExecutorService threadPool2 = Executors.newSingleThreadExecutor();

    // 一池N线程
    ExecutorService threadPool3 = Executors.newCachedThreadPool();
    try {
        for (int i = 0; i < 10; i++) {
            threadPool.execute(() -> {
                System.out.println(Thread.currentThread().getName()+"\t线程工作");
            });
            TimeUnit.MILLISECONDS.sleep(300);
        }

    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        threadPool.shutdown();
    }

}
```

***

**超级重点------**

**ThreadPoolExecutor**

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

**线程池的几个重要参数**

1. corePoolSize 线程池中的常驻核心线程数
2. maximumPoolSize 线程池中能够容纳同时执行的最大线程数 此值必须大于等于1
3. keepAliveTime 多余的空闲线程的存活时间 当前线程池中线程数量超过corePoolSize时 当空闲时间达到keepAliveTime时 多余线程会被销毁直到只剩下corePoolSize个线程为止
4. unit keepAliveTime的单位
5. workQueue 任务队列 被提交但尚未被执行的任务
6. threadFactory 表示生成线程池中的工作线程的线程工厂 用于创建线程 一般默认的即可
7. handler 拒绝策略 表示当队列满了 并且工作线程大于等于线程池的最大线程数（maximumPoolSize）时如何来拒绝请求执行的runnable的策略

***

**线程池工作原理**

<img src="http://img.tomato530.com/ThreadPool2.png" style="zoom:50%;" />

![](http://img.tomato530.com/ThreadPool1.png)

1. 在创建了线程池后 开始等待请求
2. 当调用execute()方法添加一个请求任务时 线程池会做出如下判断
   1. 如果正在运行的线程数量少于corePoolSize 那么马上创建线程运行这个任务
   2. 如果正在运行的线程数量大于或者等于corePoolSize 那么将这个任务放入队列
   3. 如果这个时候队列满了且正在运行的线程数量还少于maximumPoolSize 那么还是要创建非核心线程立刻运行这个任务
   4. 如果队列满了且正在运行的线程数量大于或者等于maximumPoolSize 那么线程池会启动饱和拒绝策略来执行
3. 当一个线程完成任务时 他会从队列中取下一个任务来执行
4. 当一个线程无事可做超过一定时间（keepAliveTime）时 线程会判断：如果当前运行的线程数大于corePoolSize 那么这个线程就被停掉 所以线程池的所有任务完成后 它最终后收缩到corePoolSize大小

***

**不要使用Executors去创建线程池 通过ThreadPoolExecutor手动创建**

![](http://img.tomato530.com/NotExecutors.png)

```java
public static void initPool() {
        //    Executors 线程池工具类
        // 一池五个线程 固定线程数
//        ExecutorService threadPool = Executors.newFixedThreadPool(5);
//
//        // 一池一个工作线程
//        ExecutorService threadPool2 = Executors.newSingleThreadExecutor();
//
//        // 一池N线程
//        ExecutorService threadPool3 = Executors.newCachedThreadPool();

        ExecutorService threadPool = new ThreadPoolExecutor(2, 5, 2L,
                TimeUnit.SECONDS, new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());

        // 拒绝策略 挨个试
        //ThreadPoolExecutor.AbortPolicy 默认
        //ThreadPoolExecutor.CallerRunsPolicy
        //ThreadPoolExecutor.DiscardOldestPolicy
        //ThreadPoolExecutor.DiscardPolicy
        try {
            for (int i = 0; i < 100; i++) {
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName()+"\t线程工作");
                });
//                TimeUnit.MILLISECONDS.sleep(300);
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
```

***

线程池最大容纳数是 max + 队列数

***

**拒绝策略**

- ThreadPoolExecutor.AbortPolicy （默认 丢弃任务并抛出RejectedExecutionException异常）
- ThreadPoolExecutor.CallerRunsPolicy （回退 该策略不会抛出异常 也不会抛弃任务 而是将某些任务退回到调用者 从而降低新任务的流量）
- ThreadPoolExecutor.DiscardPolicy （抛弃 该任务默默丢弃无法处理的任务 不予任何处理也不抛异常 如果允许任务丢失 这是最好的一种策略）
- ThreadPoolExecutor.DiscardOldestPolicy （抛弃队列中等待最久的任务 然后把当前任务加入队列中 尝试再次提交当前任务）

***

**HOW TO 配线程池**

先看CPU什么类型

+ CPU密集型（计算密集型）
  + CPU核数 + 1 是 = 最大线程数
+ I/O密集型

```java
// 查当前系统内核数
Runtime.getRuntime().availableProcessors();
```

