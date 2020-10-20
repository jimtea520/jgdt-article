Spring是如何解决循环依赖的?



# **1. 什么是循环依赖？**

循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。如下图：![image-20200909004053379](https://gitee.com/fking86/images4typora/raw/master/imgs/20200909004056.png)

这里不是函数的循环调用，是对象的相互依赖关系。循环调用其实就是一个死循环，除非有终结条件。

Spring中循环依赖场景有： 
（1）构造器的循环依赖 
（2）field属性的循环依赖。



循环依赖的产生可能有很多种情况，例如：

- A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象
- A的构造方法中依赖了B的实例对象，同时B的某个field或者setter需要A的实例对象，以及反之
- A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象，以及反之



### **循环依赖会有3种情况： **

**1.构造器循环依赖**   

构造器的循环依赖是不可以解决的，spring容器将每一个正在创建的bean标识符放在一个当前创建bean池中，在创建的过程一直在里面，如果在创建的过程中发现已经存在这个池里面了，这时就会抛出异常表示循环依赖了。 

**2.setter循环依赖**   

对于setter的循环依赖是通过spring容器提前暴露刚完成构造器注入，但并未完成其他步骤（如setter注入）的bean来完成的，而且只能决定单例作用域的bean循环依赖，通过提前暴露一个单例工厂方法，从而使其他的bean能引用到该bean.当你依赖到了该Bean而单例缓存里面有没有该Bean的时候就会调用该工厂方法生产Bean， Spring是先将Bean对象实例化之后再设置对象属性的

Spring先是用构造实例化Bean对象，此时Spring会将这个实例化结束的对象放到一个Map中，并且Spring提供了获取这个未设置属性的实例化对象引用的方法。

为什么不把Bean暴露出去，而是暴露个Factory呢？因为有些Bean是需要被代理的

**3.prototype范围的依赖**

对于“prototype”作用域bean，Spring容器无法完成依赖注入，因为“prototype”作用域的bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。



# **2. 怎么检测是否存在循环依赖**

Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明了循环依赖了。



# **3. Spring如何解决**

Spring的单例对象的初始化主要分为三步： 

![img](https://img.jbzj.com/file_images/article/201809/201809051012215.jpg) 

（1）createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象

（2）populateBean：填充属性，这一步主要是多bean的依赖属性进行填充

（3）initializeBean：调用spring xml中的init 方法。

从上面讲述的单例bean初始化步骤可以知道，循环依赖主要发生在第一、第二步。也就是构造器循环依赖和field循环依赖。

对于单例来说，在Spring容器整个生命周期内，有且只有一个对象，所以很容易想到这个对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。

首先我们看源码 SimpleJndiBeanFactory，三级缓存主要指：

```
/* Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>();
/* Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>();
/*Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>();
```

这三级缓存分别指：

- singletonFactories（三级缓存） ： 单例对象工厂的cache
- earlySingletonObjects (二级缓存)：提前暴光的单例对象的Cache
- singletonObjects(一级缓存)：单例对象的cache

我们在创建bean的时候，首先想到的是从cache中获取这个单例的bean，这个缓存就是singletonObjects。主要调用方法就就是：



![image-20200909010634645](https://gitee.com/fking86/images4typora/raw/master/imgs/20200909010636.png)

上面的代码需要解释两个参数：

1. isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象， 或则在A的populateBean过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)
2. allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象

分析getSingleton()的整个过程，Spring首先从一级缓存singletonObjects中获取。如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取。如果还是获取不到且允许singletonFactories通过getObject()获取，就从三级缓存singletonFactory.getObject()(三级缓存)获取，如果获取到了则：

```
this.earlySingletonObjects.put(beanName, singletonObject);
            this.singletonFactories.remove(beanName);
```

从singletonFactories中移除，并放入earlySingletonObjects中。其实也就是从三级缓存移动到了二级缓存。

从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于singletonFactories这个三级cache。这个cache的类型是ObjectFactory，定义如下：

```
public interface ObjectFactory<T> {
  T getObject() throws BeansException;
}
```

这个接口在下面被引用

![image-20200909010925175](https://gitee.com/fking86/images4typora/raw/master/imgs/20200909010926.png)

这里就是解决循环依赖的关键，这段代码发生在createBeanInstance之后，也就是说单例对象此时已经被创建出来(调用了构造器)。这个对象已经被生产出来了，虽然还不完美（还没有进行初始化的第二步和第三步），但是已经能被人认出来了（根据对象引用能定位到堆中的对象），所以Spring此时将这个对象提前曝光出来让大家认识，让大家使用。

addSingleton：

![image-20200909011017998](https://gitee.com/fking86/images4typora/raw/master/imgs/20200909011023.png)





# 扩展：

## **为什么Spring不能解决构造器的循环依赖？**

在Bean调用构造器实例化之前，一二三级缓存并没有Bean的任何相关信息，在实例化之后才放入三级缓存中，因此当getBean的时候缓存并没有命中，这样就抛出了循环依赖的异常了。

## 为什么多实例Bean不能解决循环依赖？

多实例Bean是每次创建都会调用doGetBean方法，根本没有使用一二三级缓存，肯定不能解决循环依赖。



## singletonBeanFactory只是一个三级缓存，那么一级缓存和二级缓存有什么用呢？

Spring在初始化Singleton的时候大致可以分几步，初始化——设值——销毁，循环依赖的场景下只有A——B——A这样的顺序，但在并发的场景下，每一步在执行时，都有可能调用getBean方法，而单例的Bean需要保证只有一个instance，那么Spring就是通过这些个缓存外加对象锁去解决这类问题，同时也可以省去不必要的重复操作。

