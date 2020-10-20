cas

lock cmpxchage 指令 不是原子的

lock 指令在之行后面指令的时候锁定一个北桥信号



new 一个对象 16个字节

makeword 8字节 

klass pointer  压缩默认是4个字节 也有8个字节

m 4字节

padding 



markword 什么用？记录锁信息，gc信息，hashCode



用户态和内核态 的转化过程

![image-20200502210442544](E:\技术帖子\笔记\基础\图\image-20200502210442544.png)

轻量级锁消耗CPU资源 重量级锁不需要

偏向锁默认延迟4秒

偏向锁打开一定会比自旋锁高么？不一定，明确知道很多资源去竞争，直接启用自旋锁

为什么延迟4秒？hotspot启动过程会有很多资源请求锁



-xx：biaseLockingstartupdelay=0

偏向锁未启动->普通对象

偏向锁已启动->匿名对象

-xx查询

java -XX:+PrintFlagsfinal -version |wc -l



java -XX:+PrintFlagsfinal -version |grep BiasedLocking