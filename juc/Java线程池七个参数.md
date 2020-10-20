# Java线程池七个参数

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

**一、corePoolSize 核心线程大小**
线程池中最小的线程数量，即使处理空闲状态，也不会被销毁，除非设置了allowCoreThreadTimeOut。

CPU密集型：核心线程数 = CPU核数 + 1
IO密集型：核心线程数 = CPU核数 * 2+1
注：IO密集型（某大厂实践经验）
核心线程数 = CPU核数 / （1-阻塞系数）
例如阻塞系数 0.8，CPU核数为4，则核心线程数为20

**二、maximumPoolSize 线程池最大线程数量**
一个任务被提交后，首先会被缓存到工作队列中，等工作队列满了，则会创建一个新线程，处理从工作队列中的取出一个任务。

**三、keepAliveTime 空闲线程存活时间**
当线程数量大于corePoolSize时，一个处于空闲状态的线程，在指定的时间后会被销毁。

**四、unit 空间线程存活时间单位**
keepAliveTime的计量单位

**五、workQueue 工作队列**，jdk中提供了四种工作队列
新任务被提交后，会先进入到此工作队列中，任务调度时再从队列中取出任务。
①ArrayBlockingQueue
基于数组的有界阻塞队列，按FIFO排序。
②LinkedBlockingQuene
基于链表的无界阻塞队列（其实最大容量为Interger.MAX），按照FIFO排序。
④PriorityBlockingQueue
具有优先级的无界阻塞队列，优先级通过参数Comparator实现。

**六、threadFactory 线程工厂**
创建一个新线程时使用的工厂，可以用来设定线程名、是否为daemon线程等等
**七、handler 拒绝策略**
当工作队列中的任务已满并且线程池中的线程数量也达到最大，这时如果有新任务提交进来，拒绝策略就是解决这个问题的，jdk中提供了4中拒绝策略：
①CallerRunsPolicy
该策略下，在调用者线程中直接执行被拒绝任务的run方法，除非线程池已经shutdown，则直接抛弃任务。
②AbortPolicy
该策略下，直接丢弃任务，并抛出RejectedExecutionException异常。默认是AbortPolicy 策略

```
private static final RejectedExecutionHandler defaultHandler =
    new AbortPolicy();
```

③DiscardPolicy
该策略下，直接丢弃任务，什么都不做。
④DiscardOldestPolicy
该策略下，抛弃最早进入队列的那个任务，然后尝试把这次拒绝的任务放入队列。