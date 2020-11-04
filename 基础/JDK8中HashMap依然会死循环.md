#### 

 是否你听说过JDK8之后HashMap已经解决的扩容死循环的问题，虽然HashMap依然说线程不安全，但是不会造成服务器load飙升的问题。

 然而事实并非如此。少年可曾了解一种红黑树成环的场景，=v=

 今日在查看监控时候发现，某一台机器load飙升
![image-20201021132914553](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021132914.png)

感觉问题不对劲，ssh大法登陆机器，top，top -Hp,jstack，jmap四连击保存下来堆栈，cpu使用最高的线程，内存信息准备分析。

首先查看使用最耗费cpu的线程堆栈信息

```shell
cat stack | grep -i 34670 -C10 --color
1
```

![image-20201021132937872](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021132937.png)

我勒个去，HashMap，猜测八成死循环了，但是我们使用的JDK8，在8中通过栈封闭的链表替换，解决了扩容死循环的问题。疑惑，继续往下看。

根据堆栈信息，root方法是问题所在，点开HashMap源码

![image-20201021133001969](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021133002.png)

好嘛，load飙高，代码有个for语句，我觉得铁定死循环了，看代码情况只可能是两个红黑树节点的父亲节点相互引用才可以导致无法走出这个for语句。

然而这都是我的猜测，我没有证据。而且让我追红黑树的代码，也是需要耗费大量时间的事情，我需要快速验证我的猜测。

我之前dump下来了堆内存信息，我通过jhat 命令生成html的内存信息页面
![image-20201021133031070](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021133031.png)
然后输入http://localhost:7000查看

我先找业务代码中持有这个HashMap的对象，然后点进去查询内部信息

![image-20201021133047536](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021133047.png)

因为数据都放在table中，点击Table字段，查看其内容

![image-20201021133104362](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021133104.png)

table中存在唯一的一个TreeNode节点，这肯定是已经变成了红黑树了

点进去查看

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190926214912105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzMzMwNjg3,size_16,color_FFFFFF,t_70)
点击parent字段信息

![image-20201021133136550](https://gitee.com/fking86/images4typora/raw/master/imgs/20201021133136.png)

**0x72745d828与0x72745d7b8两个TreeNode节点的Parent引用都是对方**。

后续打算深入研究一下红黑树什么场景会造成这个原因。

最后，无论什么并发场景请别使用HashMap，**ConcurrentHashmap大法好**



转自：https://blog.csdn.net/qq_33330687/article/details/101479385