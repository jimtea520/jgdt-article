# CountDownLatch的原理以及使用方法

概述

​	1 ountDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。

​	2 是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。



源码解析

构造函数

当我们调用CountDownLatch countDownLatch=new CountDownLatch(4) 时候，此时会创建一个AQS的同步队列，并把创建CountDownLatch 传进来的计数器赋值给AQS队列的 state，所以state的值也代表CountDownLatch所剩余的计数次数

```
//参数count为计数值
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);//创建同步队列，并设置初始计数器值
}
```

await() 

创建一个节点，加入到AQS阻塞队列，并同时把当前线程挂起。

```
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException {
	sync.acquireSharedInterruptibly(1);
}

//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit)
throws InterruptedException {
	return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

acquireSharedInterruptibly(int arg)

```
//1.判断当前线程是否有被中断
//2.如果没有的话，就调用tryAcquireShared(int acquires)方法，判断当前线程是否还需要“阻塞”。
 public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
}
```

doAcquireSharedInterruptibly(int arg)

```
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
    	//加入等待队列 				      
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
    	// 进入 CAS 循环
        try {
            for (;;) {
                //当一个节点(关联一个线程)进入等待队列后， 获取此节点的 prev 节点 
                final Node p = node.predecessor();
                // 如果获取到的 prev 是 head，也就是队列中第一个等待线程
                if (p == head) {
                    // 再次尝试申请 反应到 CountDownLatch 就是查看是否还有线程需要等待(state是否为0)
                    int r = tryAcquireShared(arg);
                    // 如果 r >=0 说明 没有线程需要等待了 state==0
                    if (r >= 0) {
                        //尝试将第一个线程关联的节点设置为 head 
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                //经过自旋tryAcquireShared后，state还不为0，就会到这里，第一次的时候，waitStatus是0，那么node的waitStatus就会被置为SIGNAL，第二次再走到这里，就会用LockSupport的park方法把当前线程阻塞住
                //重组双向链表，清空无效节点，挂起当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
 }
```

总结

1 当前线程就会进入了一个死循环当中，在这个死循环里面，会不断的进行判断

2 通过调用tryAcquireShared方法，不断判断我们上面说的那个计数器，

3 看看它的值是否为0了（为0的时候，其实就是我们调用了足够多次数的countDownLatch.countDown（）方法的时候）

4 如果是为0的话，tryAcquireShared就会返回1，设置第一个线程关联的借点未head，然后跳出了循环，不再“阻塞”当前线程了。

5 需要注意的是，说是在不停的循环，其实也并非在不停的执行for循环里面的内容，因为在后面调用parkAndCheckInterrupt（）方法时，在这个方法里面是会调用 LockSupport.park(this)来禁用当前线程的。

parkAndCheckInterrupt()

```
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```



addWaiter(Node mode)

将当前线程加入等待队列

```
static final Node SHARED = new Node();

static final Node EXCLUSIVE = null;

private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // 尝试快速入队操作，因为大多数时候尾节点不为 null
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
    	//如果尾节点为空(也就是队列为空) 或者尝试CAS入队失败(由于并发原因)，进入enq方法
        enq(node);
        return node;
}
```

步骤：

1.构造Node实体，参数为当前线程和mode，mode是SHARED 或者 EXCLUSIVE

2.尝试快速入队列操作

3.如果尾节点为空(也就是队列为空) 或者尝试CAS入队失败(由于并发原因)，进入enq方法



enq(final Node node)

```
private Node enq(final Node node) {
    	// 死循环+CAS保证所有节点都入队
        for (;;) {
            Node t = tail;
            // 如果队列为空 设置一个空节点作为 head
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                //加入队尾
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
 }
```

说明：

1.死循环+CAS 乐观锁

2.CAS是实现原子性，compareAndSetTail底层调用的是unsafe类，直接操作内存，在cpu层上加锁，直接对内存进行操作



setHeadAndPropagate(node, r)

```
private void setHeadAndPropagate(Node node, int propagate) {
    //备份head
    Node h = head; 
    //抢到锁的线程被唤醒 将这个节点设置为head
    setHead(node);
    // propagate 一般都会大于0 或者存在可被唤醒的线程
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
         // 只有一个节点 或者是共享模式 释放所有等待线程 各自尝试抢占锁
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

Node 中的waitStatus四种状态

```
//线程已被 cancelled ，这种状态的节点将会被忽略，并移出队列
static final int CANCELLED =  1;
// 表示当前线程已被挂起，并且后继节点可以尝试抢占锁
static final int SIGNAL    = -1;
//线程正在等待某些条件
static final int CONDITION = -2;
//共享模式下 无条件所有等待线程尝试抢占锁
static final int PROPAGATE = -3;
```

countDown()

```
//将计数值减一
//递减锁重入次数，当state=0时唤醒所有阻塞线程
public void countDown() {
        sync.releaseShared(1);
}
```

releaseShared(1)

```
// AQS类
public final boolean releaseShared(int arg) {
    	// arg 为固定值 1
    	// 如果计数器state 为0 返回true，前提是调用 countDown() 之前不能已经为0
        if (tryReleaseShared(arg)) {
            // 唤醒等待队列的线程
            doReleaseShared();
            return true;
        }
        return false;
}

// CountDownLatch 重写的方法
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
    		// 依然是循环+CAS配合 实现计数器减1
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
}

/// AQS类
 private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                // 如果节点状态为SIGNAL，则他的next节点也可以尝试被唤醒
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);//成功则唤醒线程
                }
                // 将节点状态设置为PROPAGATE，表示要向下传播，依次唤醒
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
}
```



## **总结**

1、AQS 分为独占模式和共享模式，CountDownLatch 使用了它的共享模式。

2、AQS 当第一个等待线程（被包装为 Node）要入队的时候，要保证存在一个 head 节点，这个 head 节点不关联线程，也就是一个虚节点。

3、当队列中的等待节点（关联线程的，非 head 节点）抢到锁，将这个节点设置为 head 节点。

4、第一次自旋抢锁失败后，waitStatus 会被设置为 -1（SIGNAL），第二次再失败，就会被 LockSupport 阻塞挂起。

5、如果一个节点的前置节点为 SIGNAL 状态，则这个节点可以尝试抢占锁。



实现逻辑

1、初始化CountDownLatch实际就是设置了AQS的state为计数的值

2、调用CountDownLatch的countDown方法时实际就是调用AQS的释放同步状态的方法，每调用一次就自减一次state值

3、调用await方法实际就调用AQS的共享式获取同步状态的方法acquireSharedInterruptibly(1)，这个方法的实现逻辑就调用子类Sync的tryAcquireShared方法，只有当子类Sync的tryAcquireShared方法返回大于0的值时才算获取同步状态成功，否则就会一直在死循环中不断重试，直到tryAcquireShared方法返回大于等于0的值，而Sync的tryAcquireShared方法只有当AQS中的state值为0时才会返回1，否则都返回-1，也就相当于只有当AQS的state值为0时，await方法才会执行成功，否则就会一直处于死循环中不断重试。



示例

```
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        /*创建CountDownLatch实例,计数器的值初始化为5*/
        final CountDownLatch downLatch = new CountDownLatch(5);

        /*每个线程等待1s,表示执行比较耗时的任务*/
        for(int i = 0;i < 5;i++){
            final int num = i;
            new Thread(new Runnable() {
                public void run() {
                    try {
                        Thread.sleep(1000);

                    }catch (InterruptedException e){
                        e.printStackTrace();

                    }

                    System.out.println(String.format("thread %d has finished",num));

                    /*任务完成后调用CountDownLatch的countDown()方法*/
                    downLatch.countDown();
                }
            }).start();
        }

        /*主线程调用await()方法,等到其他5个线程执行完后才继续执行*/
        downLatch.await();

        System.out.println("all threads have finished,main thread will continue run");
    }
}
```

输出

```
thread 1 has finished
thread 2 has finished
thread 0 has finished
thread 3 has finished
thread 4 has finished
all threads have finished,main thread will continue run
```



**使用场景**

1 需要等待某个条件达到要求后才能做后面的事情

2 同时当线程都完成后也会触发事件，以便进行后面的操作



