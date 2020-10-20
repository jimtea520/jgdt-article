# OOM问题定位方法



# 1. 背景

线上内存OOM问题是最难定位的问题，最常见的原因：

（1）本身资源不够

（2）申请的太多

（3）资源耗尽

某服务器上部署了Java服务，出现OutOfMemoryError，请问有可能是什么原因，问题应该如何定位？

解决思路：

Java服务OOM，最常见的原因为：

（1）有可能是内存分配确实过小，而正常业务需要使用更大的内存；

（2）某一个对象被频繁申请，却没有释放，内存不断泄露，导致内存耗尽；

（3）某一个资源被不断申请，系统资源耗尽，例如：不断创建线程，不断发起网络连接

# 2. 排查过程

## 2.1 确认是不是内存本身就分配过小

jmap -heap pid

![image-20200902115152675](https://gitee.com/fking86/images4typora/raw/master/imgs/20200902115154.png)

如图，可以查看新生代，老年代堆内存的分配大小以及使用情况，看是否本身分配过小。

## 2.2 找到最耗内存的对象

jmap -histo:live pid | more

![image-20200902115224370](https://gitee.com/fking86/images4typora/raw/master/imgs/20200902115226.png)

如图，结果以表格的形式显示存活对象的信息，并按照所占内存大小排序：

实例数，所占内存大小，类名

如果发现某类对象占用内存很大，很可能是类对象创建太多，且一直未释放。例如：

（1）申请完资源后，未调用close释放资源

（2）消费者消费速度慢，生产者不断往队列中投递任务，导致队列中任务累积过多

## 2.3 确认释放是资源耗尽

pstree：查看进程创建的线程数

netstat：网络连接数

还有另一种方法，通过

ll /proc/pid/fd 查看占用句柄

ll /proc/pid/task 查看线程数

例如，某一台显示服务器的sshd进程是1041，查看：

![image-20200902115254974](https://gitee.com/fking86/images4typora/raw/master/imgs/20200902115256.png)

sshd共占用了5个句柄。

![image-20200902115314586](https://gitee.com/fking86/images4typora/raw/master/imgs/20200902115316.png)

sshd只有一个主线程为1041，并没有多线程。