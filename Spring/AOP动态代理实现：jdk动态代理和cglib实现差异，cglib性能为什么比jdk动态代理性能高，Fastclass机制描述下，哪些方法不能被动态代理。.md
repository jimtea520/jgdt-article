**AOP动态代理实现：jdk动态代理和cglib实现差异，cglib性能为什么比jdk动态代理性能高，Fastclass机制描述下，哪些方法不能被动态代理。**



1，jdk动态代理和cglib动态代理差异

jdk通过反射来实现动态代理，生成一个类，继承Proxy实现相关接口

为什么不能代理没有接口的，因为已经继承了Proxy

cglib通过 生成一个字节码的子类，会生成子类文件，同时还会生成一个FastClass索引文件

Fastclass是索引文件，使用switchcase，fastclass是懒加载，在第一次invokeSuper方法的时候生成

cglib的唯一缺点是final方法不能代理

2，关于效率

cglib走索引所以比较快，之前是比spring aop块，但是随着jdk不断对反射的优化，1.8之后效率大幅提升

Spring 5.x 中 AOP 默认依旧使用 JDK 动态代理。

SpringBoot 2.x 开始，为了解决使用 JDK 动态代理可能导致的类型转化异常而默认使用 CGLIB。