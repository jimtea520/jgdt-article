### ThreadLocalMap 和HashMap区别

HashMap 的数据结构是数组+链表

ThreadLocalMap的数据结构仅仅是数组

HashMap 是通过链地址法解决hash 冲突的问题

ThreadLocalMap 是通过开放地址法来解决hash 冲突的问题

HashMap 里面的Entry 内部类的引用都是强引用

ThreadLocalMap里面的Entry 内部类中的key 是弱引用，value 是强引用



**链地址法**

这种方法的基本思想是将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。列如对于关键字集合{12,67,56,16,25,37, 22,29,15,47,48,34}，我们用前面同样的12为除数，进行除留余数法：

![img](https:////upload-images.jianshu.io/upload_images/15595524-dd88fbd05dc16475?imageMogr2/auto-orient/strip|imageView2/2/w/541/format/webp)

**开放地址法**

这种方法的基本思想是一旦发生了冲突，就去寻找下一个空的散列地址(这非常重要，源码都是根据这个特性，必须理解这里才能往下走)，只要散列表足够大，空的散列地址总能找到，并将记录存入。

比如说，我们的关键字集合为{12,33,4,5,15,25},表长为10。 我们用散列函数f(key) = key mod l0。 当计算前S个数{12,33,4,5}时，都是没有冲突的散列地址，直接存入（蓝色代表为空的，可以存放数据）：

![img](https:////upload-images.jianshu.io/upload_images/15595524-40afd6c43cd28f80?imageMogr2/auto-orient/strip|imageView2/2/w/1010/format/webp)

计算key = 15时，发现f(15) = 5，此时就与5所在的位置冲突。于是我们应用上面的公式f(15) = (f(15)+1) mod 10 =6。于是将15存入下标为6的位置。这其实就是房子被人买了于是买下一间的作法：

![img](https:////upload-images.jianshu.io/upload_images/15595524-3b59b0a438dc262e?imageMogr2/auto-orient/strip|imageView2/2/w/1014/format/webp)

**链地址法和开放地址法的优缺点**

开放地址法：

容易产生堆积问题，不适于大规模的数据存储。

散列函数的设计对冲突会有很大的影响，插入时可能会出现多次冲突的现象。

删除的元素是多个冲突元素中的一个，需要对后面的元素作处理，实现较复杂。

链地址法：

处理冲突简单，且无堆积现象，平均查找长度短。

链表中的结点是动态申请的，适合构造表不能确定长度的情况。

删除结点的操作易于实现。只要简单地删去链表上相应的结点即可。

指针需要额外的空间，故当结点规模较小时，开放定址法较为节省空间。

ThreadLocalMap 采用开放地址法原因

ThreadLocal 中看到一个属性 HASH_INCREMENT = 0x61c88647 ，0x61c88647 是一个神奇的数字，让哈希码能均匀的分布在2的N次方的数组里, 即 Entry[] table

通过HASH_INCREMENT 可以看到，`ThreadLocal` 中使用了斐波那契散列法，来保证哈希表的离散度。而它选用的乘数值即是`2^32 * 黄金分割比`。

什么是散列?

散列（Hash）也称为哈希，就是把任意长度的输入，通过散列算法，变换成固定长度的输出，这个输出值就是散列值。

ThreadLocal 往往存放的数据量不会特别大（而且key 是弱引用又会被垃圾回收，及时让数据量更小），这个时候开放地址法简单的结构会显得更省空间，同时数组的查询效率也是非常高，加上第一点的保障，冲突概率也低.



**解决哈希冲突**

ThreadLocal中的hash code非常简单，就是调用AtomicInteger的getAndAdd方法，参数是个固定值0x61c88647。

```
private static AtomicInteger nextHashCode =
    new AtomicInteger();
```

```
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

上面说过ThreadLocalMap的结构非常简单只用一个数组存储，并没有链表结构，当出现Hash冲突时采用线性查找的方式，所谓线性查找，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。如果产生多次hash冲突，处理起来就没有HashMap的效率高，为了避免哈希冲突，使用尽量少的threadlocal变量



**内存泄漏问题**

**在JAVA里面，存在强引用、弱引用、软引用、虚引用。这里主要谈一下强引用和弱引用。**

强引用，就不必说了，类似于：

A a = new A();

B b = new B();

考虑这样的情况：

**C c = new C(b);**

**b = null;**

考虑下GC的情况。要知道b被置为null，那么是否意味着一段时间后GC工作可以回收b所分配的内存空间呢？答案是否定的，因为即便b被置为null，但是c仍然持有对b的引用，而且还是强引用，所以GC不会回收b原先所分配的空间！既不能回收利用，又不能使用，这就造成了**内存泄露**。

那么如何处理呢？

**可以c = null;也可以使用弱引用！（WeakReference w = new WeakReference(b);）**



ThreadLocal使用到了弱引用，是否意味着不会存在内存泄露呢？

把ThreadLocal置为null，那么意味着Heap中的ThreadLocal实例不在有强引用指向，只有弱引用存在，因此GC是可以回收这部分空间的，也就是key是可以回收的。但是value却存在一条从Current Thread过来的强引用链。因此只有当Current Thread销毁时，value才能得到释放。

只要这个线程对象被gc回收，就不会出现内存泄露，但在threadLocal设为null和线程结束这段时间内不会被回收的，就发生了我们认为的内存泄露。最要命的是线程对象不被回收的情况，比如使用线程池的时候，线程结束是不会销毁的，再次使用的，就可能出现内存泄露。

那么如何有效的避免呢？

在ThreadLocalMap中的set/getEntry方法中，会对key为null（也即是ThreadLocal为null）进行判断，如果为null的话，那么是会对value置为null的。我们也可以通过调用ThreadLocal的remove方法进行释放！也就是每次使用完ThreadLocal，都调用它的remove()方法，清除数据。



**ThreadLocal使用**

ThreadLocal使用的一般步骤:

1、在多线程的类（如ThreadDemo类）中。创建一个ThreadLocal对象threadXxx，用来保存线程间须要隔离处理的对象xxx。
2、在ThreadDemo类中。创建一个获取要隔离访问的数据的方法getXxx()，在方法中推断，若ThreadLocal对象为null时候，应该new()一个隔离訪问类型的对象，并强制转换为要应用的类型。
3、在ThreadDemo类的run()方法中。通过getXxx()方法获取要操作的数据。这样能够保证每一个线程相应一个数据对象，在不论什么时刻都操作的是这个对象。

使用示例：

```
private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {

        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    threadLocal.set(i);
                    System.out.println(Thread.currentThread().getName() + " = " + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal test 1").start();


        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName() + " = " + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal test 2").start();
    }
```

输出

```
threadLocal test 1 = 0
threadLocal test 2 = null
threadLocal test 2 = null
threadLocal test 1 = 1
threadLocal test 2 = null
threadLocal test 1 = 2
threadLocal test 2 = null
threadLocal test 1 = 3
threadLocal test 2 = null
threadLocal test 1 = 4
threadLocal test 2 = null
threadLocal test 1 = 5
threadLocal test 2 = null
threadLocal test 1 = 6
threadLocal test 2 = null
threadLocal test 1 = 7
threadLocal test 2 = null
threadLocal test 1 = 8
threadLocal test 2 = null
threadLocal test 1 = 9
```



与Synchonized的对照:

ThreadLocal和Synchonized都用于解决多线程并发访问。可是ThreadLocal与synchronized有本质的差别。synchronized是利用锁的机制，使变量或代码块在某一时该仅仅能被一个线程访问。而ThreadLocal为每个线程都提供了变量的副本，使得每个线程在某一时间访问到的并非同一个对象，这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通信时可以获得数据共享。

Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。



**线程隔离特性**

线程隔离特性，只有在线程内才能获取到对应的值，线程外不能访问。

（1）Synchronized是通过线程等待，牺牲时间来解决访问冲突

（1）ThreadLocal是通过每个线程单独一份存储空间，牺牲空间来解决冲突