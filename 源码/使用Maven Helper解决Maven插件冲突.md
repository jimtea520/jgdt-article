# 有了它，轻松解决Maven插件冲突?



## 1、何为依赖冲突

Maven是个很好用的依赖管理工具，但是再好的东西也不是完美的。Maven的依赖机制会导致Jar包的冲突。举个例子，现在你的项目中，使用了两个Jar包，分别是A和B。现在A需要依赖另一个Jar包C，B也需要依赖C。但是A依赖的C的版本是1.0，B依赖的C的版本是2.0。这时候，Maven会将这1.0的C和2.0的C都下载到你的项目中，这样你的项目中就存在了不同版本的C，这时Maven会依据**依赖路径最短优先原则**，来决定使用哪个版本的Jar包，而另一个无用的Jar包则未被使用，这就是所谓的**依赖冲突**。

在大多数时候，依赖冲突可能并不会对系统造成什么异常，因为Maven始终选择了一个Jar包来使用。但是，不排除在某些特定条件下，会出现类似**找不到类的异常**，所以，只要存在依赖冲突，在我看来，最好还是解决掉，不要给系统留下隐患。

## 2、解决方法

解决依赖冲突的方法，就是使用Maven提供的**<exclusion>**标签，**<exclusion>**标签需要放在**<exclusions>**标签内部，就像下面这样：

```
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.10.0</version>
    <exclusions>
        <exclusion>
        <artifactId>log4j-api</artifactId>
        <groupId>org.apache.logging.log4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

`log4j-core`本身是依赖了`log4j-api`的，但是因为一些其他的模块也依赖了`log4j-api`，并且两个`log4j-api`版本不同，所以我们使用**<exclusion>**标签排除掉`log4j-core`所依赖的`log4j-api`，这样Maven就不会下载`log4j-core`所依赖的`log4j-api`了，也就保证了我们的项目中只有一个版本的`log4j-api`。

## 3、Maven Helper

看到这里，你可能会有一个疑问。如何才能知道自己的项目中哪些依赖的Jar包冲突了呢？Maven Helper这个InteliJ IDEA的插件帮我们解决了这个问题。插件的安装方法我就不讲了，既然你都会Maven了，我相信你也是会安装插件的。

在插件安装好之后，我们打开pom.xml文件，在底部会多出一个**Dependency Analyzer**选项

![image-20200913082408572](https://gitee.com/fking86/images4typora/raw/master/imgs/20200913082700.png)

点开这个选项

![image-20200913082724957](https://gitee.com/fking86/images4typora/raw/master/imgs/20200913082725.png)

找到冲突，点击右键，然后选择**Exclude**即可排除冲突版本的Jar包。

## 4、小技巧

除了使用Maven Helper查看依赖冲突，也可以使用IDEA提供的方法——Maven依赖结构图，打开Maven窗口，选择Dependencies，然后点击那个图标（Show Dependencies）或者使用快捷键（Ctrl+Alt+Shift+U），即可打开Maven依赖关系结构图

![image-20200913082758783](https://gitee.com/fking86/images4typora/raw/master/imgs/20200913082758.png)

在图中，我们可以看到有一些红色的实线，这些红色实线就是依赖冲突，蓝色实线则是正常的依赖。

![image-20200913082823429](https://gitee.com/fking86/images4typora/raw/master/imgs/20200913082823.png)



 来源：

https://segmentfault.com/a/1190000017542396