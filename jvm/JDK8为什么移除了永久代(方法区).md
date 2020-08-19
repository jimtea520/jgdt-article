## JDK8 的10个新特性总结如下：

1. Lambda Expressions
2. Pipelines and Streams
3. Date and Time API
4. Default Methods
5. Type Annotations
6. Nashorn JavaScript Engine
7. Concurrent Accumulators
8. Parallel operations
9. PermGen Space Removed
10. TLS SNI

其中，第九个就是移除了永久代，其目的是Hotspot JVM和JRockit JVM相融合的设计思路。

转移位置：  将java类部分放到java heap里，将字符串常量和类中的静态变量放到内存里面。



在JDK1.7中, 已经把原本放在永久代的字符串常量池移出, 放在堆中. 为什么这样做呢? 因为使用永久代来实现方法区不是个好主意, 很容易遇到内存溢出的问题. 我们通常使用PermSize和MaxPermSize设置永久代的大小, 这个大小就决定了永久代的上限, 但是我们不是总是知道应该设置为多大的, 如果使用默认值容易遇到OOM错误.

类的元数据, 字符串池, 类的静态变量将会从永久代移除, 放入Java heap或者native memory. 其中建议JVM的实现中将类的元数据放入 native memory, 将字符串池和类的静态变量放入java堆中. 这样可以加载多少类的元数据就不在由MaxPermSize控制, 而由系统的实际可用空间来控制.

为什么这么做呢? 减少OOM只是表因, 更深层的原因还是要合并HotSpot和JRockit的代码, JRockit从来没有一个叫永久代的东西, 但是运行良好, 也不需要开发运维人员设置这么一个永久代的大小.

当然不用担心运行性能问题了, 在覆盖到的测试中, 程序启动和运行速度降低不超过1%, 但是这一点性能损失换来了更大的安全保障.



总结原因：

(1)字符串存在永久代中，容易出现性能问题和内存溢出
(2)永久代大小不容易确定. PermSize指定了大小容易造成OOM（内存用完）
(3)给 GC(垃圾回收机制) 带来不必要的复杂度,且回收效率低



