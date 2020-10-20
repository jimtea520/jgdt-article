### Synchronized与Lock的区别



# 存在层次上

**synchronized:**	 Java的关键字，在jvm层面上

**Lock:**	是一个接口



# 锁的释放

**synchronized:**	1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁

**Lock:**	在finally中必须释放锁，不然容易造成线程死锁



# 锁的获取

**synchronized:**	假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待

**Lock:**	分情况而定，Lock有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待(可以通过tryLock判断有没有锁)



# 锁的释放（死锁产生）

**synchronized:**	在发生异常时候会自动释放占有的锁，因此不会出现死锁

**Lock:**	发生异常时候，不会主动释放占有的锁，必须手动unlock来释放锁，可能引起死锁的发生



# 锁的状态

**synchronized:**	无法判断

**Lock:**	可以判断



# 锁的类型

**synchronized:**	 可重入 不可中断 非公平

**Lock:**	可重入 可判断 可公平（两者皆可）



# 性能

**synchronized:**	少量同步

**Lock:**	大量同步

- Lock可以提高多个线程进行读操作的效率。（可以通过readwritelock实现读写分离）
- 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；
- ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。



# 调度

**synchronized:**	使用Object对象本身的wait 、notify、notifyAll调度机制

**Lock:**	可以使用Condition进行线程之间的调度



# 用法

**synchronized:**	在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

**Lock:**	一般使用ReentrantLock类做为锁。在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。



# 底层实现

**synchronized:**	底层使用指令码方式来控制锁的，映射成字节码指令就是增加来两个指令：monitorenter和monitorexit。当线程执行遇到monitorenter指令时会尝试获取内置锁，如果获取锁则锁计数器+1，如果没有获取锁则阻塞；当遇到monitorexit指令时锁计数器-1，如果计数器为0则释放锁。

**Lock:**	底层是CAS乐观锁，依赖AbstractQueuedSynchronizer类，把所有的请求线程构成一个CLH队列。而对该队列的操作均通过Lock-Free（CAS）操作。