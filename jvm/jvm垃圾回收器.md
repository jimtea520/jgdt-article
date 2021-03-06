jvm垃圾回收器


**1.Serial收集器（新生代）**

　　(1)单线程收集器

　　(2)采用复制算法，用于新生代垃圾回收

　　(3)垃圾回收期间需要STW(Stop The World),STW表示垃圾回收线程不与用户线程并发执行

**2.Serial Old收集器（老年代）**

　　(1)与Serial相似

　　(2)采用标记整理算法，用于老年代的立即回收

**3.ParNew收集器（新生代）**

　　(1)是Serial的多线程版本

　　(2)除此之外与Serial收集相似

**4.Parallel Scavenge收集器（新生代）**

　　(1)基本功能与ParNew收集器相似

　　(2)区别在于该收集器是要达到一个可控制的吞吐量(吞吐量=运行用户代码的时间/(运行用户代码的时间)+(垃圾回收的时间))

　　(3)可以高效的利用cpu时间

　　(4)提供了参数可以精确的控制吞吐量，分别是控制最大垃圾回收停顿时间，也可以直接设置吞吐量大小

**5.Parallel Old收集器（老年代）**

　　(1)Parallel Scavenge收集器的老年代版本，采用标记整理算法

**6.CMS收集器（老年代）**

　　(1)采用标记清除算法，用户老年代的垃圾回收

　　(2)主要关注的是尽可能的缩短垃圾回收时用户线程的停顿时间

　　(3)主要有一下几个步骤：

　　　　①初始标记：简单标记一下GC ROOTS能直接关联到的节点，此阶段需要STW

　　　　②并发标记：进行GC Roots Tracing的过程，此阶段与用户线程并发执行

　　　　③重新标记：对并发标记时用于线程产生的新的节点进行标记，此阶段需要STW, 但是此阶段为多线程并行的(多个垃圾回收线程同时进行)

　　　　④并发清除：使用标记清楚算法对对象进行回收，此阶段与同户线程同时进行

　　(4)缺点：

　　　　①无法清除浮动垃圾，由于最后一个阶段并发清除是与用户线程同时进行的，所以用户线程可能会产生新的可会收的对象

　　　　②可能会产生垃圾碎片，由于该回收器采用的是标记清除算法

**7.G1收集器**

　　(1)G1(Garbage-First)

　　(2)G1收集器作用于整个JVM堆

　　(3)G1收集器将整个堆分成了大小相同的独立区域(Region)

　　(4)在后台会维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region