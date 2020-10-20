# spring扩展点总结

## BeanFactory构造阶段

有一个NamespaceHandler可扩展的地方。

### NamespaceHandler

通过自定义的NamespaceHandler，配合BeanDefinitionParser，可以完成自定义Bean的组装操作，对于BeanDefinition的数据结构，进行个性化创建。

![image-20200911000150321](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911000150.png)

红线处是MyBatis的NamespaceHandler，和原始的接口名重了而已。（MyBatis的Mapper接口BeanDefination扫描，还可以使用MapperScannerConfigurer，在BeanFactoryPostProcessor执行中、Bean实例化前实现）

Dubbo在spring中的使用，就有NamespaceHandler的身影。

还有目前常用的component-scan，也是利用ComponentScanBeanDefinitionParser完成了指定包下所有类的扫描，BeanDefinition创建的过程。

## ApplicationContext构造阶段

有两个可扩展的地方：BeanFactoryPostProcessor，BeanPostProcessor。

### BeanFactoryPostProcessor

拥有一个接口方法：

```
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
```

实现BeanFactoryPostProcessor的Bean，ApplicationContext会提前实例化，并使用BeanFactoryPostProcessor对BeanFactory中的BeanDefinition进行变化修改，或者创建BeanDefinition到BeanFactory中。

![image-20200911000224082](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911000224.png)

![image-20200911000250935](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911000251.png)

![image-20200911000306083](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911000306.png)

 

比如PropertyPlaceholderConfigurer可将property-placeholder中的配置文件信息替换到BeanDefinition中。

比如ConfigurationClassPostProcessor发现BeanFactory中使用了@Configuration注解的Bean，并根据此Bean在BeanFactory中增加Bean。

比如MyBatis中使用的MapperScannerConfigurer，扫描basePackage配置下的接口，生成多个Bean并注册到BeanFactory中。

### BeanPostProcessor

拥有两个接口方法：

```
Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
```

分别在Bean实例初始化之前和之后执行，可自定义修改Bean实例，甚至替换Bean实例为自定义实例。

![image-20200911000326267](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911000326.png)

此外还有四个接口继承了BeanPostProcessor接口。

### DestructionAwareBeanPostProcessor

接口有一个方法，在实例销毁前使用。

### InstantiationAwareBeanPostProcessor

接口有三个方法，两个在实例创建前后（不是初始化前后）使用，一个是在populateBean变量注入时使用。

```
Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException;
boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException;
PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException;
```

- 其中实例创建前后使用的接口，可检查是否已使用AOP代理创建了实例。并返回创建的实例，之后的Bean实例创建步骤便不会继续执行了。
- 变量注入使用的接口，可实现@Autowired注解的注入。

注：常见的AbstractAutoProxyCreator真正产生代理实例的地方是在postProcessAfterInitialization中，**AOP**也是通过此种方式和**IOC**过程连接起来的。

### SmartInstantiationAwareBeanPostProcessor

接口有三个方法，在实例创建前智能判断实例类型，智能判断构造函数，获取预创建实例，防止循环依赖。

实现了InstantiationAwareBeanPostProcessor接口的类，基本都实现了SmartInstantiationAwareBeanPostProcessor接口。

### MergedBeanDefinitionPostProcessor

接口有一个方法，在实例创建前，修改合并BeanDefinition。

看了一下几个实现类，除了调用BeanDefinition.registerExternallyManagedDestroyMethod方法外，没有修改过BeanDefinition的其他内容。