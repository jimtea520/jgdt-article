鸿蒙OS尝鲜, 居然还有这种事情！

# 1.前序部分



## 1.1 官网：

[https://www.harmonyos.com](https://www.harmonyos.com/) 网上搜索第一个不是官方是三方的



## 1.2.IDE下载位置：

源码编译的下载： https://device.harmonyos.com/cn/ide 开发应用的下载：https://developer.harmonyos.com/cn/develop/deveco-studio#download



## 1.3.源码位置：

https://device.harmonyos.com/cn/docs/start/get-code/oem_sourcecode_guide-0000001050769927



# 2. 开发应用部分（源码IDE后续我在补充，下面都是应用开发部分）



## 2.1 安装时候启动会提示下载SDK，点击取消，搜索SDK，重新自定义SDK路径。

![image-20200926100409838](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100410.png)

![image-20200926100443260](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100443.png)

我把所有的SDK都按照上，打勾会自动下载。



# 2.2 创建的项目

点击创建项目，发现现在支持有三种类型



## 2.2.1 TV 设备应用

![image-20200926100458479](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100458.png)



## 2.2.2 Wearable 可穿戴设备应用

![image-20200926100514699](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100515.png)



## 2.2.3 Lite Wearable 可穿戴设备(Lite)应用

![image-20200926100536432](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100536.png)



## 2.2.4 创建TV项目 （Java），选了一个列表模板。

![image-20200926100550780](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100551.png)

创建中：

![image-20200926100604480](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100604.png)

下载gradle-5.4.1-all.zip慢得等等。 （ 默认下载到： 

![image-20200926100637303](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100637.png)

这个应该有办法直接下载好

![image-20200926100653020](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100653.png)

设置里面很全面都能进行设置，可以探索探索 ） grade 源已经正确的切换到华为国内

![image-20200926100705795](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100706.png)



## 2.2.5 下载模拟器

![image-20200926100723051](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100723.png)

弹窗点击确认就可以下载了 （此处下载速度略慢，下载失败点击 downloadagain 继续下载 估计下载人太多，失败了十几次把）

![image-20200926100746032](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100746.png)



刷新以后：

![image-20200926100805717](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100806.png)

使用自己的华为账号登录（**此处注意有坑，如果默认使用Chrome可能授权会失败，使用Windows自带的浏览器进行登录**）

![image-20200926100822098](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100822.png)

（开发者账号实名认证，我填写的是银行卡验证速度很快） 

![image-20200926100841933](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100842.png)

进行授权：

![image-20200926100856006](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100856.png)

授权成功：

![image-20200926100915984](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100916.png)

同意协议：

![image-20200926100929341](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100929.png)

罗列了可用的模拟器：

![image-20200926100945070](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100945.png)

TV模拟器启动了

![image-20200926100958955](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926100959.png)

运行就可以看到模拟器了

![img](https://oscimg.oschina.net/oscnet/up-073ed855a67aa8394cf6abf2f8b1788ea47.png)

运行成功：

![image-20200926101034177](https://gitee.com/fking86/images4typora/raw/master/imgs/20200926101034.png)