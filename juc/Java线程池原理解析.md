# Java线程池原理解析

## 前言

线程是稀缺资源，如果被无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，合理的使用线程池对线程进行统一分配、调优和监控，有以下好处：
 1、降低资源消耗；通过重复利用已创建好的线程来降低线程创建和销毁造成的消耗;
 2、提高响应速度；当任务到达时，任务可以不需要等待线程创建就能立马执行;
 3、提高线程的可管理性。线程池时稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。



## 阿里巴巴Java开发手册中对线程池的使用规范

1.【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。
正例：

```
public class TimerTaskThread extends Thread {
public TimerTaskThread(){
    super.setName("TimerTaskThread"); 
    ...
}
}
```

2.【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
说明： 使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资
源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者
“过度切换”的问题。

3.【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样
的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

### Executors 返回的线程池对象的弊端如下

**1） FixedThreadPool 和 SingleThreadPool:**
允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
**2） CachedThreadPool 和 ScheduledThreadPool:**
允许的创建线程数量为 Integer.MAX_VALUE， 可能会创建大量的线程，从而导致 OOM。



## 线程池的 UML图



**Executor接口：**

它是线程池的基础，提供了唯一的一个方法execute用来执行线程任务。

**ExecutorService接口：**

它继承了Executor接口，提供了线程池生命周期管理的几乎所有方法，包括诸如shutdown、awaitTermination、submit、invokeAll、invokeAny等。

**AbstractExecutorService类：**

一个提供了线程池生命周期默认实现的抽象类，并且自行新增了如newTaskFor、doInvokeAny等方法。

**ThreadPoolExecutor类：**

这是线程池的核心类，也是我们常用来创建和管理线程池的类，我们使用Executors调用newFixedThreadPool、newSingleThreadExecutor和newCachedThreadPool等方法创建的线程池，都是ThreadPoolExecutor类型的。

**ScheduledExecutorService接口：**

赋予了线程池具备延迟和定期执行任务的能力，它提供了一些方法接口，使得任务能够按照给定的方式来延期或者周期性的执行任务。

**ScheduledThreadPoolExecutor类：**

继承自ThreadPoolExecutor类，同时实现了ScheduledExecutorService接口，具备了线程池所有通用能力，同时增加了延时执行和周期性执行任务的能力。



## ThreadPoolExecutor部分源码

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

这里定义的ctl，被定义为了**AtomicInteger**，使用**高3位**来表示**线程池状态**，**低29位**来表示线程池中的**任务数量**。



## 线程池状态

**RUNNING：**线程池能够接受新任务，以及对新添加的任务进行处理。

**SHUTDOWN：**线程池不可以接受新任务，但是可以对已添加的任务进行处理。

**STOP：**线程池不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。

**TIDYING：**当所有的任务已终止，ctl记录的"任务数量"为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。

**TERMINATED：**线程池彻底终止的状态。

### 状态切换示意图



## ThreadPoolExecutor线程池类参数

```
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
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

### 参数详解

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| corePoolSize    | 核心线程数量，线程池维护线程的最少数量                       |
| maximumPoolSize | 线程池维护线程的最大数量                                     |
| keepAliveTime   | 线程池除核心线程外的其他线程的最长空闲时间，超过该时间的空闲线程会被销毁 |
| unit            | keepAliveTime的单位，TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS |
| workQueue       | 线程池所使用的任务缓冲队列                                   |
| threadFactory   | 线程工厂，用于创建线程，一般用默认的即可                     |
| handler         | 线程池对拒绝任务的处理策略                                   |

## ThreadPoolExecutor提供的四种拒绝策略

1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常；也是默认的处理方式。
2. ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。
3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

如果是需要自定义处理则是需要继承RejectedExecutionHandler

# 线程池任务执行

### 1. 添加执行任务

- submit() 该方法返回一个Future对象，可执行带返回值的线程；或者执行想随时可以取消的线程。Future对象的get()方法获取返回值。Future对象的cancel(true/false)取消任务，未开始或已完成返回false，参数表示是否中断执行中的线程
- execute() 没有返回值。

### 2.线程池任务提交过程

ThreadPoolExecutor#execute方法

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
     // ctl记录了线程数量和线程状态
    int c = ctl.get();
    // 如果工作线程数小于核心线程数，创建新线程执行
    // 即时其他线程是空闲的
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 缓存任务到队列中，这里进行了double check
    // 如果线程池中运行的线程数量>=corePoolSize，
    // 且线程池处于RUNNING状态，且把提交的任务成功放入阻塞队列中，
    // 就再次检查线程池的状态。
    // 1.如果线程池不是RUNNING状态，且成功从阻塞队列中删除任务，
    //   则该任务由当前 RejectedExecutionHandler 处理。
    // 2.否则如果线程池中运行的线程数量为0，则通过
    //   addWorker(null, false)尝试新建一个线程，
    //   新建线程对应的任务为null。
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 任务无效则拒绝
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 添加新的工作线程，并在addWorker方法中的两个for循环来判断
    // 如果以上两个条件不成立，既没能将任务成功放入阻塞队列中，
    // 且addWoker新建线程失败，则该任务由当前
    // RejectedExecutionHandler 处理。
    else if (!addWorker(command, false))
        // 采用拒绝策略
        reject(command);
}
```

addWorker(Runnable firstTask, boolean core)

    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);
        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;
    
        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 创建工作线程
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
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
                    // 将线程放置于HashSet中，持有mainLock才可访问
                    // workers中记录了池中真正任务线程数量
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
### 四个步骤

1、当来了新任务，如果线程池中空闲线程数量小于corePoolSize，则直接拿线程池中新的线程来处理任务；

2、如果线程池正在运行的线程数量大于等于corePoolSize，而此时workQueue队列未满，则将此任务缓存到队列中；

3、如果线程池正在运行的线程数量大于等于corePoolSize，且workQueue队列已满，但现在的线程数量还小于maximumPoolSize，则创建新的线程来执行任务。

4、如果线程数量已经大于maximumPoolSize且workQueue队列也已经满了，则使用拒绝策略来处理该任务，默认的拒绝策略就是抛出异常（AbortPolicy）。

### 简化成流程图



### 总结

处理任务判断的优先级为 核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

### 注意

1. 当workQueue使用的是×××限队列时，maximumPoolSize参数就变的无意义了，比如new LinkedBlockingQueue(),或者new ArrayBlockingQueue(Integer.MAX_VALUE);
2. 使用SynchronousQueue队列时由于该队列没有容量的特性，所以不会对任务进行排队，如果线程池中没有空闲线程，会立即创建一个新线程来接收这个任务。maximumPoolSize要设置大一点。
3. 核心线程和最大线程数量相等时keepAliveTime无作用.

# 线程池关闭

**1. shutdown()**
不接收新任务,会处理已添加任务
**2. shutdownNow()** 
不接受新任务,不处理已添加任务,中断正在处理的任务

# 常用阻塞队列介绍

**ArrayBlockingQueue：**

有界队列，基于数组结构，按照队列FIFO原则对元素排序；

**LinkedBlockingQueue：**

无界队列，基于链表结构，按照队列FIFO原则对元素排序，Executors.newFixedThreadPool()使用了这个队列；

**SynchronousQueue：**

同步队列，该队列不存储元素，每个插入操作必须等待另一个线程调用移除操作，否则插入操作会一直被阻塞，Executors.newCachedThreadPool()使用了这个队列；

**PriorityBlockingQueue：**

优先级队列，具有优先级的无限阻塞队列。

### 队列常见操作解析

| 方法    | 说明                                                   |
| ------- | ------------------------------------------------------ |
| add     | 增加一个元索; 如果队列已满，则抛出一个异常             |
| remove  | 移除并返回队列头部的元素; 如果队列为空，则抛出一个异常 |
| offer   | 添加一个元素并返回true; 如果队列已满，则返回false      |
| poll    | 移除并返回队列头部的元素; 如果队列为空，则返回null     |
| put     | 添加一个元素; 如果队列满，则阻塞                       |
| take    | 移除并返回队列头部的元素; 如果队列为空，则阻塞         |
| element | 返回队列头部的元素; 如果队列为空，则抛出一个异常       |
| peek    | 返回队列头部的元素; 如果队列为空，则返回null           |

# Executors线程工厂类

**1. Executors.newCachedThreadPool()**
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程.
内部实现：

```
new ThreadPoolExecutor(0,Integer.MAX_VALUE,60L,TimeUnit.SECONDS,new SynchronousQueue<runnable>());</runnable>
```

**2. Executors.newFixedThreadPool(int)**
创建一个给定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
内部实现：

```
new ThreadPoolExecutor(nThreads, nThreads,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<runnable>());</runnable>
```

**3. Executors.newSingleThreadExecutor()**
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照顺序执行。
内部实现：

```
new ThreadPoolExecutor(1,1,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<runnable>())</runnable>
```

**4. Executors.newScheduledThreadPool(int)**
创建一个给定长度线程池，支持定时及周期性任务执行。
内部实现：

```
new ScheduledThreadPoolExecutor(corePoolSize)
```

