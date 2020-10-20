# Spring,Spring MVC及Spring Boot区别



# 什么是Spring？它解决了什么问题？

我们说到Spring，一般指代的是Spring Framework，它是一个开源的应用程序框架，提供了一个简易的开发方式，**通过这种开发方式，将避免那些可能致使代码变得繁杂混乱的大量的业务/工具对象，说的更通俗一点就是由框架来帮你管理这些对象，包括它的创建，销毁等**，比如基于Spring的项目里经常能看到的`Bean`，它代表的就是由Spring管辖的对象。



# 什么是Spring MVC？它解决了什么问题？

Spring MVC是Spring的一部分，Spring 出来以后，大家觉得很好用，于是按照这种模式设计了一个 MVC框架（一些用Spring 解耦的组件），**主要用于开发WEB应用和网络接口，它是Spring的一个模块，通过Dispatcher Servlet, ModelAndView 和 View Resolver，让应用开发变得很容易**



# 什么是Spring Boot？它解决了什么问题？

初期的Spring通过代码加配置的形式为项目提供了良好的灵活性和扩展性，但随着Spring越来越庞大，其配置文件也越来越繁琐，太多复杂的xml文件也一直是Spring被人诟病的地方，特别是近些年其他简洁的WEB方案层出不穷，如基于Python或Node.Js，几行代码就能实现一个WEB服务器，对比起来，大家渐渐觉得Spring那一套太过繁琐，此时，Spring社区推出了Spring Boot，它的目的在于**实现自动配置，降低项目搭建的复杂度**，如需要搭建一个接口服务，通过Spring Boot，几行代码即可实现



# Spring，Spring MVC，Spring Boot 三者比较

Spring 就像一个大家族，有众多衍生产品例如 Boot，Security，JPA等等。但他们的基础都是Spring 的 IOC 和 AOP，IOC提供了依赖注入的容器，而AOP解决了面向切面的编程，然后在此两者的基础上实现了其他衍生产品的高级功能；

Spring MVC是基于 Servlet 的一个 MVC 框架，主要解决 WEB 开发的问题，因为 Spring 的配置非常复杂，各种xml，properties处理起来比较繁琐。于是为了简化开发者的使用，Spring社区创造性地推出了Spring Boot，它遵循约定优于配置，极大降低了Spring使用门槛，但又不失Spring原本灵活强大的功能，下面用一张图来描述三者的关系：



![image-20200915001214274](https://gitee.com/fking86/images4typora/raw/master/imgs/20200915001214.png)



最后一句话总结：**Spring MVC和Spring Boot都属于Spring，Spring MVC 是基于Spring的一个 MVC 框架，而Spring Boot 是基于Spring的一套快速开发整合包**