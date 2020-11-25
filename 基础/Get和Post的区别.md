



# 概念

GET和POST是什么？

HTTP协议中的两种发送请求的方法。

HTTP是什么？

HTTP是基于TCP/IP的关于数据如何在万维网中如何通信的协议。

HTTP的底层是TCP/IP。所以GET和POST的底层也是TCP/IP，都是TCP链接。GET和POST能做的事情是一样一样的。如果给GET加上request body，或着给POST带上url参数，技术上是完全行的通的。 

# 区别

1.Get是不安全的，因为在传输过程，数据被放在请求的URL中；Post的所有操作对用户来说都是不可见的,通过request body传递参数。

2.Get传送的数据量较小，这主要是因为受URL长度限制；Post传送的数据量较大，一般被默认为不受限制。

3.Get限制Form表单的数据集的值必须为ASCII字符；而Post支持整个ISO10646字符集。

4.Get执行效率却比Post方法好。Get是form提交的默认方法。

5.GET产生一个TCP数据包；POST产生两个TCP数据包。

6.对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；

而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

7.GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。

8.GET请求只能进行url编码，而POST支持多种编码方式。

9.GET请求会被浏览器主动cache，而POST不会，除非手动设置。



# 注意

1.在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。

2.在网络环境差的情况下，两次包的TCP在验证数据包完整性上，有很好的体验效果

3.并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。