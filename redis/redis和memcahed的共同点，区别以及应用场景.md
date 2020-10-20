## 共同点

​    1.都是放在内存中，是内存数据库

​    2.都可以做分布式集群，可以一主多从，也可以一主一从

## 区别

1.redis不仅仅支持k/v类型的数据，还支持hash list set sortedset类型数据结构的存储 ，memcached 支持简单的key/value ,而且 memcached 还可用于缓存其他东西，例如图片、视频等

2.如果挂掉,redis可以通过aof恢复数据，而且为了数据安全，redis 可以通过定期保存到磁盘做持久化处理,memcahed 不能恢复数据

3.对于过期设置

memcached 通过set设定

​	用法：set key flags exptime bytes [noreply] 

​	参数说明如下：

- **key：**键值 key-value 结构中的 key，用于查找缓存值。

- **flags**：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。

- **exptime**：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）

- **bytes**：在缓存中存储的字节数

- **noreply（可选）**： 该参数告知服务器不需要返回数据

- **value**：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）

  

  例子：set keytest 0 900 9

  以上实例中我们设置：

  - key → runoob

  - flag → 0

  - exptime → 900 (以秒为单位)

  - bytes → 9 (数据存储的字节数)

  - value → memcached

    

  redis通过expire设定 

  例如：EXPIRE key 60 ， 单位为秒

4.Redis支持master-slave模式的数据备份

## 应用场景

1、redis：有持久化数据方面的需求以及对数据类型和处理有要求。 
2、memcache:  简单的key/value 存储。