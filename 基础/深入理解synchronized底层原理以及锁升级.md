# synchronized底层原理以及锁升级

什么是synchronized?

`synchronized`是Java提供的一个并发控制的关键字，作用于对象上。主要有两种用法，分别是同步方法(访问对象和clss对象）和同步代码块（需要加入对象），保证了代码的原子性和可见性以及有序性，但是不会处理重排序以及代码优化的过程，但是在一个线程中执行肯定是有序的，因此是有序的。



# synchronized 的特性

## **1 原子性**

**所谓原子性就是指一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行**

被synchronized修饰的类或对象的所有操作都是原子的，因为在执行操作之前必须先获得类或对象的锁，直到执行完才能释放，这中间的过程无法被中断（除了已经废弃的stop()方法），即保证了原子性。

## **2 可见性**

**可见性是指多个线程访问一个资源时，该资源的状态、值信息等对于其他线程都是可见的**

synchronized对一个类或对象加锁时，一个线程如果要访问该类或对象必须先获得它的锁，而这个锁的状态对于其他任何线程都是可见的，并且在释放锁之前会将对变量的修改刷新到主存当中，保证资源变量的可见性，如果某个线程占用了该锁，其他线程就必须在锁池中等待锁的释放。

## **3 有序性**

**有序性值程序执行的顺序按照代码先后执行**

Java允许编译器和处理器对指令进行重排，但是指令重排并不会影响单线程的顺序，它影响的是多线程并发执行的顺序性。synchronized保证了每个时刻都只有一个线程访问同步代码块，也就确定了线程执行同步代码块是分先后顺序的，保证了有序性。

## **4 可重入性**

当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁。通俗一点讲就是说一个线程拥有了锁仍然还可以重复申请锁。



# **synchronized的使用场景**

一般归结为三种:

 1.修饰静态方法，给当前类对象加锁，进入同步方法时需要获得类对象的锁

```
public class SynchronizedStaticFunDemo {
    private static int count;

    public static synchronized void addCount(){
        count = count + 1;
    }
}
```

2.修饰实例方法，给当前实例变量加锁，进入同步方法时需要获得当前实例的锁

```
public class SynchronizedMemberDemo {
    private static int count;

    public synchronized void addCount(){
        count = count + 1;
    }
}
```

3.修饰同步方法块，指定加锁对象（可以是实例对象，也可以是类变量），对给定对象加锁，进入同步方法块时需要获得加锁对象的锁

```
public class SynchronizedCodeBlockDemo {
    private static int count;

    public  void addCount(){
        synchronized (this){
            count = count + 1;
        }
    }
}
```

或者

```
public class SynchronizedCodeBlockDemo {
    private static int count;

    public  void addCount(){
        synchronized (SynchronizedCodeBlockDemo.class){
            count = count + 1;
        }
    }
}
```

## **synchronized对锁的底层实现**

在理解锁实现原理之前先了解一下Java的对象头和Monitor，首先我们得先了解java对象结构，在对象头里面的Mark word中, 存储对象的hashCode、锁信息或分代年龄或GC标志等信息。

## Monitor

什么是Monitor?

1.Monitor是一种用来实现同步的工具

2.与每个java对象相关联，所有的 Java 对象是天生携带 monitor

3.Monitor是实现Sychronized(内置锁)的基础

对象的监视器（monitor）由ObjectMonitor对象实现（C++），其跟同步相关的数据结构如下：

```
ObjectMonitor() {
    _count        = 0; //用来记录该对象被线程获取锁的次数
    _waiters      = 0;
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
}
```

下面通过两个列子:

代码:

```
public class SynchronizedDemo {
    public void method(){
        synchronized (this){
            System.out.println("Method 1 start");
        }
    }
}
```

反编译查看如图：



通过上面代码看

monitorenter：

每个对象有一个监视器锁(monitor)。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

monitorexit:

1.执行monitorexit的线程必须是objectref所对应的monitor的所有者。

2.指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

代码修改下：

```
public class SynchronizedDemo {
    public synchronized void method(){
            System.out.println("Method 1 start");
    }
}
```

反编译如图：

从反编译的结果来看,其常量池中多了ACC_SYNCHRONIZED标示符，JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。

总结

Synchronized的底层是通过一个monitor的对象来完成，wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

## Synchronized 锁升级过程

锁也分不同状态，JDK6之前只有两个状态：无锁、有锁（重量级锁），而在JDK6之后对synchronized进行了优化，新增了两种状态，总共就是四个状态：**无锁状态、偏向锁、轻量级锁、重量级锁**，其中无锁就是一种状态了。锁的类型和状态在对象头`Mark Word`中都有记录，在申请锁、锁升级等过程中JVM都需要读取对象的`Mark Word`数据。

synchronized同步锁升级步骤是：

**偏向锁 -> 轻量级锁 -> 重量级锁**

### 偏向锁

偏向锁他会偏向第一次访问的线程，当线程获取锁对象时，会在java对象头markword中记录偏向锁的threadID，并不会主动释放偏向锁。当同一个线程再次获取锁时会比较当前的threadID与对象头中的threadID是否一致。如果一致则不需要通过CAS来加锁、解锁。如果不一致并且线程还需要持续持有锁，则暂停当前线程撤销偏向锁，升级为轻量级锁。如果不在需要持续持有锁则锁对象头设为无锁状态，重新设置偏向锁。

偏向锁过程：

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标识位是否为01，确认偏向状态
2. 如果为可偏向状态，则判断当前线程ID是否为偏向线程
3. 如果偏向线程未只想当前线程，则通过cas操作竞争锁，如果竞争成功则操作Mark Word中线程ID设置为当前线程ID
4. 如果cas偏向锁获取失败，则挂起当前偏向锁线程，偏向锁升级为轻量级锁。

### 轻量级锁（自旋锁）

轻量级锁由偏向锁升级而来，偏向锁运行在一个线程同步块时，第二个线程加入锁竞争的时候，偏向锁就会升级为轻量级锁。

轻量级锁过程：

1. 线程由偏向锁升级为轻量级锁时，会先把锁的对象头MarkWord复制一份到线程的栈帧中，建立一个名为锁记录空间（Lock Record），用于存储当前Mark Word的拷贝。
2. 虚拟机使用cas操作尝试将对象的Mark Word指向Lock Record的指针，并将Lock record里的owner指针指对象的Mark Word。
3. 如果cas操作成功，则该线程拥有了对象的轻量级锁。第二个线程cas自选锁等待锁线程释放锁。
4. 如果多个线程竞争锁，轻量级锁要膨胀为重量级锁，Mark Word中存储的就是指向重量级锁（互斥量）的指针。其他等待线程进入阻塞状态。

### 重量级锁

自旋失败，很大概率 再一次自旋也是失败，因此直接升级成重量级锁，进行线程阻塞，减少cpu消耗。

当锁升级为重量级锁后，未抢到锁的线程都会被阻塞，进入阻塞队列。

### synchronized的执行过程

1. 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁
2. 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1
3. 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。
4. 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁
5. 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
6. 如果自旋成功则依然处于轻量级状态。
7. 如果自旋失败，则升级为重量级锁。

### 用户态和内核态

- 内核态

CPU可以访问内存所有数据，包括外围设备，例如硬盘，网卡。CPU也可以将自己从一个程序切换到另一个程序

- 用户态

只能受限的访问内存，切不允许访问外围设备。占用CPU的能力被剥夺，CPU资源可以被其他程序获取。

之所以会有这样的却分是为了防止用户进程获取别的程序的内存数据，，或者获取外围设备的数据。

### 用户态和内核态什么时候进行切换

用户程序都是运行在用户态的，但是有时候程序确实需要做一些内核的事情，例如从硬盘读取数据，或者从硬盘获取输入，而唯一可以做这些事情的就是操作系统（synchronized中依赖的monitor也需要依赖操作系统完成，因此需要用户态到内核态的切换）所以程序就需要先操作系统请求以程序的名义来执行这些操作。

### 总结

synchronized锁升级实际上是把本来的悲观锁变成了 在一定条件下 使用无所(同样线程获取相同资源的偏向锁)，以及使用乐观(自旋锁 cas)和一定条件下悲观(重量级锁)的形式。

偏向锁: 适用于单线程适用锁的情况

轻量级锁：适用于竞争较不激烈的情况(这和乐观锁的使用范围类似)

重量级锁：适用于竞争激烈的情况

