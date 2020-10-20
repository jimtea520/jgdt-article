# Spring事务失效和解决方案

SpringAOP是基于代理来对目标方法进行增强，但是有的时候又会出现“增强无效”的情况，比如在@Transactional下的某类中的方法内调用了该类的另一个方法，这种情况下，事务有时候会出现不生效的情况。因为，事务也是基于代理来增强目标对象的目标方法的，我们应该获取代理对象再来调用子方法。

获取代理对象的三个方法：

 

1、@Autowried 目标类注入进来，用这个注入进来的对象调用子方法即可。

2、实现ApplicationContextAware接口或者@Autowried ApplicationContext对象，再通过getBean()来获取。

3、通过SpringAOP的API来获取代理对象，这个方法的前提是在启动类上的@EnableAspectJAutoProxy的注解中加上属性exposeProxy = true；接下来通过API获取即可： AopContext.*currentProxy() 即可获取当前类的目标代理对象，记得强转成当前类，然后调用你想调用的子方法即可。



例子：

```
@Controller
public class TestController {
    

@Autowired
private TestService testService;

@GetMapping("/test")
public void test(){
    testService.methodB();
}

}
```

```
@Service
public class TestService {

@Transactional
public void methodA(){

}

/**

这里调用methodA() 的事务将会失效
*/
public void methodB(){
this.methodA();
}

}
```

## 原因

因为，spring的事务实现是使用了代理类来实现，而这里的this.methodA()，并没有走TestService的代理类，所以事务会失效。



## 解决

方法1：把methodA()和methodB()分别放到不同的类里。

方法2：自己注入自己，用注入的实例调用

```
@Service
public class TestService {

@Autowired
private TestService testService;

@Transactional
public void methodA(){

}

/**

这里调用methodA() 的事务将会生效
*/
public void methodB(){
testService.methodA();
}

}
```

方法3：获取代理类，利用代理类调用自己类的方法

```
@Service
public class TestService {

@Transactional
public void methodA(){

}

/**

这里调用methodA() 的事务将会生效
*/
public void methodB(){
((TestService)AopContext.currentProxy()).methodA();
}

}
```



## 最后

spring的aop代理有jdk代理和cglib代理实现

```


	//是否代理对象
    AopUtils.isAopProxy(AopContext.currentProxy());

    //是否cglib 代理对象
    AopUtils.isCglibProxy(AopContext.currentProxy());

    //是否jdk动态代理
    AopUtils.isJdkDynamicProxy(AopContext.currentProxy());
```

