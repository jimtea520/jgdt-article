##  原来 AQS实现原理还能如此总结

# **AQS简介**

## **什么是AQS**

**AQS全称为AbstractQueuedSynchronizer**，就是**抽象队列同步器**。AQS是**一个用来构建锁和其他同步组件的基础框架**，使用`AQS`可以简单且高效地构造出应用广泛的同步器，它提供了一个FIFO队列，可以看成是一个用来实现同步锁以及其他涉及到同步功能的核心组件。



## **AQS的核心思想**

如果被请求的共享资源**空闲**，则将当前请求资源的线程设置为**有效的工作线程**，并且将共享资源设置为**锁定状态**。如果被请求的共享资源**被占用**，那么就需要一套**线程阻塞等待以及被唤醒时锁分配的机制**，这个机制AQS是用**CLH队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。



### **什么是CLH队列**

CLH(Craig,Landin,and Hagersten)队列是一个**虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)**。**AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配**。



### 同步状态

AQS中的共享资源是使用一个**int成员变量来表示同步状态**，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。状态信息通过procted类型的getState，setState，compareAndSetState进行操作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。状态信息通过procted类型的getState，setState，compareAndSetState进行操作。

源码：

```java
//同步状态，使用volatile来保证其可见性
private volatile int state;

//获取同步状态
protected final int getState() {
    return state;
}

//设置同步状态
protected final void setState(int newState) {
    state = newState;
}

//原子性地修改同步状态
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```



## **资源的共享方式**

AQS定义**两种**资源共享方式

- **Exclusive(独占)**：只能有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：

  - **公平锁**：按照线程在队列中的排队顺序，先到者先拿到锁
  - **非公平锁**：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- **Share(共享)**：多个线程可同时执行，如Semaphore、CountDownLatch等。

  

## **设计模式**

AQS的设计是使用模板方法设计模式，它将**一些方法开放给子类进行重写，而同步器给同步组件所提供模板方法又会重新调用被子类所重写的方法**。

自定义同步器时需要重写下面几个AQS提供的模板方法

```java
//独占式的获取同步状态，实现该方法需要查询当前状态并判断同步状态是否符合预期，然后再进行CAS设置同步状态 
protected boolean tryAcquire(int arg) { throw new UnsupportedOperationException();} 
//独占式的释放同步状态，等待获取同步状态的线程可以有机会获取同步状态 
protected boolean tryRelease(int arg) { throw new UnsupportedOperationException();} 
//共享式的获取同步状态 
protected int tryAcquireShared(int arg) { throw new UnsupportedOperationException();} 
//尝试将状态设置为以共享模式释放同步状态。 该方法总是由执行释放的线程调用。 
protected int tryReleaseShared(int arg) { throw new UnsupportedOperationException(); } 
//当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占 
protected int isHeldExclusively(int arg) { throw new UnsupportedOperationException();}

```

这些方法的实现必须是内部线程安全的。AQS类中的其他方法都是final ，所以无法被其他类覆盖。



# **CLH队列**

**当共享资源被某个线程占有，其他请求该资源的线程会被阻塞，从而进入同步队列**。AQS底层的数据结构采用**CLH队列**，它是一个虚拟的双向队列，即**不存在队列的实例，仅存在节点之间的关联关系**。**AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配**。注：**Sync queue，即同步队列**，是**双向链表**，包括`head`结点和`tail`结点，`head`结点主要用作后续的调度。而**Condition queue**不是必须的，它是一个**单向链表**，只有当使用`Condition`时，才会存在此单向链表，并且可能会有多个`Condition queue`。

![image-20200812101354653](E:%5C%E6%8A%80%E6%9C%AF%E5%B8%96%E5%AD%90%5C%E7%AC%94%E8%AE%B0%5C%E5%9F%BA%E7%A1%80%5CAQS%5C20200812101354.png)

**节点的类型是AQS的静态内部类Node**，代码：

```java
static final class Node {
    //模式，分为共享与独占
    //共享模式
    static final Node SHARED = new Node();
    //独占模式
    static final Node EXCLUSIVE = null;

    //节点状态值：1，-1，-2，-3
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;

    //节点状态
    volatile int waitStatus;

    //前驱节点
    volatile Node prev;

    //后继节点
    volatile Node next;

    //节点对应的线程
    volatile Thread thread;

    //下一个等待者
    Node nextWaiter;

    //判断节点是否在共享模式下等待
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    //获取前驱节点，如果为null，则抛异常
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
	
    //无参构造函数
    Node() {    // Used to establish initial head or SHARED marker
    }

    //有参构造函数1
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    //有参构造函数2
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

**Node节点的状态有如下四种**

- **CANCELLED = 1**

  表示当前节点从同步队列中取消，即当前线程被取消

- **SIGNAL = -1**

  表示后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行

- **CONDITION = -2**

  表示当前节点在等待condition，也就是在condition queue中

- **PROPAGATE = -3**

  表示下一次共享式同步状态获取将会无条件传播下去



# AQS类图

![image-20200812103251866](E:%5C%E6%8A%80%E6%9C%AF%E5%B8%96%E5%AD%90%5C%E7%AC%94%E8%AE%B0%5C%E5%9F%BA%E7%A1%80%5CAQS%5C20200812103252.png)

![image-20200812103329911](E:%5C%E6%8A%80%E6%9C%AF%E5%B8%96%E5%AD%90%5C%E7%AC%94%E8%AE%B0%5C%E5%9F%BA%E7%A1%80%5CAQS%5C20200812103330.png)

**AbstractQueuedSynchronizer类继承AbstractOwnableSynchronizer抽象类，实现Serializable接口**。

## **AbstractOwnableSynchronizer源码**

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    //序列化版本号
    private static final long serialVersionUID = 3737899427754241961L;

    //无参构造方法
    protected AbstractOwnableSynchronizer() { }

    //独占模式下的线程
    private transient Thread exclusiveOwnerThread;

    //设置独占线程，final修饰，不可被重写
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    //获取独占线程，final修饰，不可被重写
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

