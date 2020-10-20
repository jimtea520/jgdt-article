## CyclicBarrier源码解析以及示例

# 概念

CyclicBarrier允许一组线程在到达某个栅栏点(common barrier point)互相等待，直到最后一个线程到达栅栏点，栅栏才会打开，处于阻塞状态的线程恢复继续执行。



# 核心方法

构造函数

设置parties、count及barrierCommand属性

```
public CyclicBarrier(int parties) {
    this(parties, null);
}
```

```
//当await的数量到达了设定的数量后，首先执行该Runnable对象。   
public CyclicBarrier(int parties, Runnable barrierAction) {
	if (parties <= 0) throw new IllegalArgumentException();
	this.parties = parties;
	this.count = parties;
	this.barrierCommand = barrierAction;
}
```

await()

```
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L); //不超时等待
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
            BrokenBarrierException,
            TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}

```

dowait(boolean timed, long nanos)

    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
                TimeoutException {
        // 获取独占锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 当前代
            final Generation g = generation;
            // 如果这代损坏了，抛出异常
            if (g.broken)
                throw new BrokenBarrierException();
        // 如果线程中断了，抛出异常
        if (Thread.interrupted()) {
            // 将损坏状态设置为true
            // 并通知其他阻塞在此栅栏上的线程
            breakBarrier();
            throw new InterruptedException();
        }
     
        // 获取下标
        int index = --count;
        // 如果是 0，说明最后一个线程调用了该方法
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                // 执行栅栏任务
                if (command != null)
                    command.run();
                ranAction = true;
                // 更新一代，将count重置，将generation重置
                // 唤醒之前等待的线程
                nextGeneration();
                return 0;
            } finally {
                // 如果执行栅栏任务的时候失败了，就将损坏状态设置为true
                if (!ranAction)
                    breakBarrier();
            }
        }
     
        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                 // 如果没有时间限制，则直接等待，直到被唤醒
                if (!timed)
                    trip.await();
                // 如果有时间限制，则等待指定时间
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                // 当前代没有损坏
                if (g == generation && ! g.broken) {
                    // 让栅栏失效
                    breakBarrier();
                    throw ie;
                } else {
                    // 上面条件不满足，说明这个线程不是这代的
                    // 就不会影响当前这代栅栏的执行，所以，就打个中断标记
                    Thread.currentThread().interrupt();
                }
            }
     
            // 当有任何一个线程中断了，就会调用breakBarrier方法
            // 就会唤醒其他的线程，其他线程醒来后，也要抛出异常
            if (g.broken)
                throw new BrokenBarrierException();
     
            // g != generation表示正常换代了，返回当前线程所在栅栏的下标
            // 如果 g == generation，说明还没有换代，那为什么会醒了？
            // 因为一个线程可以使用多个栅栏，当别的栅栏唤醒了这个线程，就会走到这里，所以需要判断是否是当前代。
            // 正是因为这个原因，才需要generation来保证正确。
            if (g != generation)
                return index;
            
            // 如果有时间限制，且时间小于等于0，销毁栅栏并抛出异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        // 释放独占锁
        lock.unlock();
    }
    }


await的逻辑：如果该线程不是到达的最后一个线程，则他会一直处于等待状态，除非有如下情况：

1 最后一个线程到达，即index == 0

2 超出了指定时间（超时等待）

3 其他的某个线程中断当前线程

4 其他的某个线程中断另一个等待的线程

5 其他的某个线程在等待barrier超时

6 其他的某个线程在此barrier调用reset()方法。reset()方法用于将屏障重置为初始状态。



# 应用场景

就比如我们在打匹配游戏的时候，十个人必须全部加载到100%，才可以开局。否则只要有一个人没有加载到100%,那这个游戏就不能开始。先加载完成的玩家必须等待最后一个玩家加载成功才可以。



# 应用示例

```
public class CyclicBarrierDemo {
    private static CyclicBarrier cyclicBarrier;

    static class CyclicBarrierThread extends Thread{

        @Override
        public void run() {
            System.out.println("玩家 " + Thread.currentThread().getName() + " 加载100%");
            //等待
            try {
                cyclicBarrier.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        cyclicBarrier = new CyclicBarrier(10, new Runnable() {
            public void run() {
                System.out.println("玩家都加载好了，开始游戏....");
            }
        });

        for(int i = 0 ; i < 10 ; i++){
            new CyclicBarrierThread().start();
        }
    }
}
```

**输出**

```
玩家 Thread-0 加载100%
玩家 Thread-2 加载100%
玩家 Thread-3 加载100%
玩家 Thread-6 加载100%
玩家 Thread-1 加载100%
玩家 Thread-4 加载100%
玩家 Thread-5 加载100%
玩家 Thread-8 加载100%
玩家 Thread-7 加载100%
玩家 Thread-9 加载100%
玩家都加载好了，开始游戏....
```



# CyclicBarrier和CountDownLatch的区别

1 CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置，可以使用多次

2 CyclicBarrier还提供了一些其他有用的方法，比如getNumberWaiting()方法可以获得CyclicBarrier阻塞的线程数量，isBroken()方法用来了解阻塞的线程是否被中断

3 CountDownLatch允许一个或多个线程等待一组事件的产生，而CyclicBarrier用于等待其他线程运行到栅栏位置。


