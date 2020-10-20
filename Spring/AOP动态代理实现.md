AOP动态代理实现



**案例：**

```
public interface ForumService {
 void removeTopic(int topicId);
 void removeForum(int forumId);
}
```

对相关方法进行性能监控:

```
public class ForumServiceImpl implements ForumService {
 public void removeTopic(int topicId) {
 // PerformanceMonitor.begin("com.hand.proxy.ForumServiceImpl.removeTopic");
 System.out.println("模拟删除Topic记录:" + topicId);
 try {
  Thread.sleep(20);
 } catch (InterruptedException e) {
  e.printStackTrace();
 }
 // PerformanceMonitor.end();
 }
 public void removeForum(int forumId) {
 // PerformanceMonitor.begin("com.hand.proxy.ForumServiceImpl.removeForum");
 System.out.println("模拟删除Forum记录:" + forumId);
 try {
  Thread.sleep(20);
 } catch (InterruptedException e) {
  e.printStackTrace();
 }
 // PerformanceMonitor.end();
 }
}
```

性能监控实现类：

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
```

用于记录性能监控信息：

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
```

**JDK动态代理**

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
```

```
public class ForumServiceTest {
 @Test
 public void proxy() {
 ForumService forumService = new ForumServiceImpl();
 PerformanceHandler handler = new PerformanceHandler(forumService);
 ForumService proxy = (ForumService) Proxy.newProxyInstance(forumService.getClass().getClassLoader(),
  forumService.getClass().getInterfaces(), handler);
 proxy.removeForum(10);
 proxy.removeTopic(1012);
 }
}
```

# JDK动态代理与CGLib动态代理的区别对比

时间:2019-02-11

今天小编就为大家分享一篇关于JDK动态代理与CGLib动态代理的区别对比，小编觉得内容挺不错的，现在分享给大家，具有很好的参考价值，需要的朋友一起跟随小编来看看吧

**案例：**

```
public interface ForumService {
 void removeTopic(int topicId);
 void removeForum(int forumId);
}
```

对相关方法进行性能监控

```
public class ForumServiceImpl implements ForumService {
 public void removeTopic(int topicId) {
 // PerformanceMonitor.begin("com.hand.proxy.ForumServiceImpl.removeTopic");
 System.out.println("模拟删除Topic记录:" + topicId);
 try {
  Thread.sleep(20);
 } catch (InterruptedException e) {
  e.printStackTrace();
 }
 // PerformanceMonitor.end();
 }
 public void removeForum(int forumId) {
 // PerformanceMonitor.begin("com.hand.proxy.ForumServiceImpl.removeForum");
 System.out.println("模拟删除Forum记录:" + forumId);
 try {
  Thread.sleep(20);
 } catch (InterruptedException e) {
  e.printStackTrace();
 }
 // PerformanceMonitor.end();
 }
}
```

性能监控实现类：

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
```

用于记录性能监控信息：

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
```

**1、JDK动态代理**

```
public class PerformanceMonitor {
 // 通过一个ThreadLocal保存与调用线程相关的性能监视信息
 private static ThreadLocal<MethodPerformance> performanceRecord = new ThreadLocal<MethodPerformance>();
 // 启动对某一目标方法的性能监视
 public static void begin(String method) {
 System.out.println("begin monitor...");
 MethodPerformance mp = new MethodPerformance(method);
 performanceRecord.set(mp);
 }
 public static void end() {
 System.out.println("end monitor...");
 MethodPerformance mp = performanceRecord.get();
 // 打印出方法性能监视的结果信息
 mp.printPerformance();
 }
}
public class ForumServiceTest {
 @Test
 public void proxy() {
 ForumService forumService = new ForumServiceImpl();
 PerformanceHandler handler = new PerformanceHandler(forumService);
 ForumService proxy = (ForumService) Proxy.newProxyInstance(forumService.getClass().getClassLoader(),
  forumService.getClass().getInterfaces(), handler);
 proxy.removeForum(10);
 proxy.removeTopic(1012);
 }
}
```

**得到以下输出信息：**

```
begin monitor...
模拟删除Forum记录:10
end monitor...
com.hand.proxy.ForumServiceImpl.removeForum花费21毫秒
begin monitor...
模拟删除Topic记录:1012
end monitor...
com.hand.proxy.ForumServiceImpl.removeTopic花费21毫秒
```

**2、CGLib动态代理**

```
 <!-- https://mvnrepository.com/artifact/cglib/cglib -->
 <dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>2.2.2</version>
 </dependency>
```

```
public class CglibProxy implements MethodInterceptor {
 private Enhancer enhancer = new Enhancer();
 public Object getProxy(Class clazz) {
 enhancer.setSuperclass(clazz);
 enhancer.setCallback(this);
 return enhancer.create();
 }
 public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
 PerformanceMonitor.begin(obj.getClass().getName() + "." + method.getName());
 Object result = proxy.invokeSuper(obj, args);
 PerformanceMonitor.end();
 return result;
 }
}
```

```
public class ForumServiceTest2 {
 @Test
 public void proxy() {
 CglibProxy proxy = new CglibProxy();
 ForumServiceImpl forumService = (ForumServiceImpl) proxy.getProxy(ForumServiceImpl.class);
 forumService.removeForum(10);
 forumService.removeTopic(1023);
 }
}
```



# 总结：



**1）JDK和CGLib的区别**

- JDK动态代理只能对实现了接口的类生成代理，而不能针对类
- CGLib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法（继承）

**2）Spring在选择用JDK还是CGLib的依据**

- 当Bean实现接口时，Spring就会用JDK的动态代理
- 当Bean没有实现接口时，Spring使用CGLib来实现
- 可以强制使用CGLib（在Spring配置中加入<aop:aspectj-autoproxy proxy-target-class=“true”/>）

**3）JDK和CGLib的性能对比**

- 使用CGLib实现动态代理，**CGLib底层采用ASM字节码生成框架**，使用字节码技术生成代理类，在JDK1.6之前比使用Java反射效率要高。唯一需要注意的是，**CGLib不能对声明为final的方法进行代理**，因为CGLib原理是动态生成被代理类的子类。
- 在JDK1.6、JDK1.7、JDK1.8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLib代理效率，只有当进行大量调用的时候，JDK1.6和JDK1.7比CGLib代理效率低一点，但是到JDK1.8的时候，JDK代理效率高于CGLib代理