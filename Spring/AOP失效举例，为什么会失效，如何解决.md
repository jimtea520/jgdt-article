**AOP失效举例，为什么会失效，如何解决**



AOP失效场景和如何解决，也是这个问题

常见的：

1，service中A 方法调用B方法，那么B方法不会走事务。因为内部调用没有走aop

2，异常被吃了

3，抛出普通异常，不是RuntimeException，事务aop中默认捕捉RuntimeException，如果抛出Exception要指定 rollBackFor="Exception.class"

不常见：

Mysiam引擎，私有方法，不是@Service等

解决方法：

1，写两个service

2，通过AopContext获取当前的代理对象，执行代理对象方法

AOP有个配置，开启暴露代理对象，开启之后，会把当前代理对象写到ThreadLocal中

<!-- aspect -->

<aop:aspectj-autoproxy proxy-target-class="true" expose-proxy="true"/>

protected final T proxy() {

return (T) AopContext.currentProxy()；

}