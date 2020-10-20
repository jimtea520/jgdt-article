## 概念

Semaphore 是一个计数信号量，必须由获取它的线程释放。

常用于限制可以访问某些资源的线程数量，例如通过 Semaphore 限流。



## 主要方法摘要

```
// 非公平构造函数 创建具有给定的许可数和非公平的公平设置的 Semaphore。
Semaphore(int permits)
// 公平构造函数 创建具有给定的许可数和给定的公平设置的 Semaphore。
Semaphore(int permits, boolean fair)

// 从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
void acquire()
// 从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。
void acquire(int permits)
// 从此信号量中获取许可，在有可用的许可前将其阻塞。
void acquireUninterruptibly()
// 从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞。
void acquireUninterruptibly(int permits)
// 返回此信号量中当前可用的许可数。
int availablePermits()
// 获取并返回立即可用的所有许可。
int drainPermits()
// 返回一个 collection，包含可能等待获取的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正在等待获取的线程的估计数目。
int getQueueLength()
// 查询是否有线程正在等待获取。
boolean hasQueuedThreads()
// 如果此信号量的公平设置为 true，则返回 true。
boolean isFair()
// 根据指定的缩减量减小可用许可的数目。
protected void reducePermits(int reduction)
// 释放一个许可，将其返回给信号量。在释放许可之前，必须先获获得许可
void release()
// 释放给定数目的许可，将其返回到信号量。
void release(int permits)
// 返回标识此信号量的字符串，以及信号量的状态。
String toString()
// 尝试获取一个许可，若获取成功返回true，若获取失败返回false, 仅在调用时此信号量存在一个可用许可，才从信号量获取许可。
boolean tryAcquire()
// 仅在调用时此信号量中有给定数目的许可时，才从此信号量中获取这些许可。
boolean tryAcquire(int permits)
// 如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。
boolean tryAcquire(int permits, long timeout, TimeUnit unit)
// 如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。
boolean tryAcquire(long timeout, TimeUnit unit)
```

上面是基本常见的方法，Semaphore底层是由AQS和Uasafe完成的,调用的是

```
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```



## 步骤

1. 初始化
2. 增加
3. 减少

## 示例

```
public class SemaphoreDemo {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();

        //信号量，只允许 3个线程同时访问
        final Semaphore semaphore = new Semaphore(3);

        for (int i=0;i<10;i++){
            final long num = i;
            executorService.submit(new Runnable() {
                public void run() {
                    try {
                        //获取许可
                        semaphore.acquire();
                        //执行
                        System.out.println("执行: " + num);
                        Thread.sleep(new Random().nextInt(5000)); // 模拟随机执行时长
                        //释放
                        semaphore.release();
                        System.out.println("释放..." + num);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

        executorService.shutdown();
    }
}
```



## 总结

​    Semaphore主要用于控制当前活动线程数目，就如同停车场系统一般，而Semaphore则相当于看守的人，用于控制总共允许停车的停车位的个数，而对于每辆车来说就如同一个线程，线程需要通过acquire()方法获取许可，而release()释放许可。如果许可数达到最大活动数，那么调用acquire()之后，便进入等待队列，等待已获得许可的线程释放许可，从而使得多线程能够合理的运行。

