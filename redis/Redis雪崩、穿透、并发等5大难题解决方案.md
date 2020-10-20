# Redis雪崩、穿透、击穿等5大难题解决方案

# **缓存雪崩**

数据未加载到缓存中，或者缓存同一时间大面积的失效，从而导致所有请求都去查数据库，导致数据库CPU和内存负载过高，甚至宕机。

与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key。

**比如一个雪崩的简单过程：**

1、redis集群大面积故障

2、缓存失效，但依然大量请求访问缓存服务redis

3、redis大量失效后，大量请求转向到mysql数据库

4、mysql的调用量暴增，很快就扛不住了，甚至直接宕机

5、由于大量的应用服务依赖mysql和redis的服务，这个时候很快会演变成各服务器集群的雪崩，最后网站彻底崩溃。

![image-20200928090559057](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928090559.png)

 

**如何预防缓存雪崩：**

**1.缓存的高可用性**

缓存层设计成高可用，防止缓存大面积故障。即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务，例如 Redis Sentinel 和 Redis Cluster 都实现了高可用。

**2.缓存降级**

可以利用ehcache等本地缓存(暂时支持)，但主要还是对源服务访问进行限流、资源隔离（熔断）、降级等。

当访问量剧增、服务出现问题仍然需要保证服务还是可用的。系统可以根据一些关键数据进行**自动降级**，也可以配置开关实现**人工降级，**这里会涉及到运维的配合。

**降级的最终目的是保证核心服务可用，即使是有损的。**

比如推荐服务中，很多都是个性化的需求，假如个性化需求不能提供服务了，可以降级补充热点数据，不至于造成前端页面是个大空白。

在进行降级之前要对系统进行梳理，比如：哪些业务是核心(必须保证)，哪些业务可以容许暂时不提供服务(利用静态页面替换)等，以及配合服务器核心指标，来后设置整体预案，比如：

（1）一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；

（2）警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；

（3）错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；

（4）严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

**3.Redis备份和快速预热**

1)Redis数据备份和恢复

2)快速缓存预热

**4.提前演练**

最后，建议还是在项目上线前，演练缓存层宕掉后，应用以及后端的负载情况以及可能出现的问题，对高可用提前预演，提前发现问题。



**随机值伪代码：**

```
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";
    //缓存标记
    String cacheSign = cacheKey + "_sign";

    String sign = CacheHelper.Get(cacheSign);
    //获取缓存值
    String cacheValue = CacheHelper.Get(cacheKey);
    if (sign != null) {
        return cacheValue; //未过期，直接返回
    } else {
        CacheHelper.Add(cacheSign, "1", cacheTime);
        ThreadPool.QueueUserWorkItem((arg) -> {
      //这里一般是 sql查询数据
            cacheValue = GetProductListFromDB(); 
          //日期设缓存时间的2倍，用于脏读
          CacheHelper.Add(cacheKey, cacheValue, cacheTime * 2);                 
        });
        return cacheValue;
    }
} 
```

**解释说明：**

- 缓存标记：记录缓存数据是否过期，如果过期会触发通知另外的线程在后台去更新实际key的缓存；
- 缓存数据：它的过期时间比缓存标记的时间延长1倍，例：标记缓存时间30分钟，数据缓存设置为60分钟。这样，当缓存标记key过期后，实际缓存还能把旧数据返回给调用端，直到另外的线程在后台更新完成后，才会返回新缓存。



## 解决方案：

1、使用锁或队列

2、设置过期标志更新缓存

3、为key设置不同的缓存失效时间

4、设置“二级缓存”。



# **缓存穿透**

缓存穿透是指查询一个一不存在的数据。例如：从缓存redis没有命中，需要从mysql数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。



## 解决方案：

1、如果查询数据库也为空，直接设置一个默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库。设置一个过期时间或者当有值的时候将缓存中的值替换掉即可。

2、可以给key设置一些格式规则，然后查询之前先过滤掉不符合规则的Key。

3、采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。

4、如果一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。



# 缓存击穿

对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key。

缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回射到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。



## 解决方案：



**1.使用互斥锁(mutex key)**

业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

SETNX，是「SET if Not eXists」的缩写，也就是只有不存在的时候才设置，可以利用它来实现锁的效果。在redis2.6.1之前版本未实现setnx的过期时间，所以这里给出两种版本代码参考：

```
//2.6.1前单机版本锁
String get(String key) {
   String value = redis.get(key);
   if (value  == null) {
    if (redis.setnx(key_mutex, "1")) {
        // 3 min timeout to avoid mutex holder crash
        redis.expire(key_mutex, 3 * 60)
        value = db.get(key);
        redis.set(key, value);
        redis.delete(key_mutex);
    } else {
        //其他线程休息50毫秒后重试
        Thread.sleep(50);
        get(key);
    }
  }
}
```

新版本代码：

```
public String get(key) {
      String value = redis.get(key);
      if (value == null) { //代表缓存值过期
          //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
          if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
               value = db.get(key);
                      redis.set(key, value, expire_secs);
                      redis.del(key_mutex);
              } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                      sleep(50);
                      get(key);  //重试
              }
          } else {
              return value;
          }
 }
```

memcache代码：

```
if (memcache.get(key) == null) {
    // 3 min timeout to avoid mutex holder crash
    if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
        value = db.get(key);
        memcache.set(key, value);
        memcache.delete(key_mutex);
    } else {
        sleep(50);
        retry();
    }
}
```

**2. "提前"使用互斥锁(mutex key)**

在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。

当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。

伪代码如下：

```
v = memcache.get(key);
if (v == null) {
    if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
        value = db.get(key);
        memcache.set(key, value);
        memcache.delete(key_mutex);
    } else {
        sleep(50);
        retry();
    }
} else {
    if (v.timeout <= now()) {
        if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
            // extend the timeout for other threads
            v.timeout += 3 * 60 * 1000;
            memcache.set(key, v, KEY_TIMEOUT * 2);

            // load the latest value from db
            v = db.get(key);
            v.timeout = KEY_TIMEOUT;
            memcache.set(key, value, KEY_TIMEOUT * 2);
            memcache.delete(key_mutex);
        } else {
            sleep(50);
            retry();
        }
    }
}
```

**3. "永远不过期"**

这里的“永远不过期”包含两层意思：

**(1)** 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

**(2)** 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

从实战看，这种方法对于性能非常友好，唯一不足的就是构建缓存时候，其余线程(非构建缓存的线程)可能访问的是老数据，但是对于一般的互联网功能来说这个还是可以忍受。

```
String get(final String key) {
        V v = redis.get(key);
        String value = v.getValue();
        long timeout = v.getTimeout();
        if (v.timeout <= System.currentTimeMillis()) {
            // 异步更新后台异常执行
            threadPool.execute(new Runnable() {
                public void run() {
                    String keyMutex = "mutex:" + key;
                    if (redis.setnx(keyMutex, "1")) {
                        // 3 min timeout to avoid mutex holder crash
                        redis.expire(keyMutex, 3 * 60);
                        String dbValue = db.get(key);
                        redis.set(key, dbValue);
                        redis.delete(keyMutex);
                    }
                }
            });
        }
        return value;
}
```

**4. 资源保护**

采用netflix的hystrix，可以做资源的隔离保护主线程池，如果把这个应用到缓存的构建也未尝不可。

四种解决方案：没有最佳只有最合适

|           解决方案           |                          优点                          |                             缺点                             |
| :--------------------------: | :----------------------------------------------------: | :----------------------------------------------------------: |
|    简单分布式锁(Tim yang)    |                1. 思路简单2. 保证一致性                |  1. 代码复杂度增大2. 存在死锁的风险3. 存在线程池阻塞的风险   |
| 加另外一个过期时间(Tim yang) |                     1. 保证一致性                      |                             同上                             |
|         不过期(本文)         |            1. 异步构建缓存，不会阻塞线程池             | 1. 不保证一致性。2. 代码复杂度增大(每个value都要维护一个timekey)。3. 占用一定的内存空间(每个value都要维护一个timekey)。 |
|  资源隔离组件hystrix(本文)   | 1. hystrix技术成熟，有效保证后端。2. hystrix监控强大。 |                  1. 部分访问存在降级策略。                   |



# **缓存并发**

这里的并发指的是多个redis的client同时set key引起的并发问题。



## 解决方案：

其实redis自身就是单线程操作，多个client并发操作，按照先到先执行的原则，先到的先执行，其余的阻塞。当然，另外的解决方案是把redis.set操作放在队列中使其串行化，必须的一个一个执行。



# **缓存预热**

缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。

这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！



## 解决思路：

1、直接写个缓存刷新页面，上线时手工操作下；

2、数据量不大，可以在项目启动的时候自动进行加载；

目的就是在系统上线前，将数据加载到缓存中。