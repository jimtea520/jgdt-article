**String类型** 

	可以是字符串（简单的字符串、复杂的字符串（例如JSON、XML））、数字（整数、浮点数），甚至是二进制（图片、音频、视频），但是值最大不能超过512M
	
	实现方式：一个字符串，被redisObject所引用，当遇到incr,decr等操作时会转成数值型进行计算，此时redisObject的encoding字段为int。
	
	常用命令：
	
		 > set key value ：设置key对应的string类型值。
		 > get key ：获取key对于的string类型值。
	 
	使用场景:
	 	>用户登录天数, 博客访问次数、网站访问量

**List类型** 

		list数据类型指key对应的value是一个双向链表结构。注意:一个列表最多可以包含 232 - 1 个元素

   实现方式:

		双向链表
	
	常用命令：
	
		1.lpush key value ：向key对应list头部添加一个字符串元素。 
	
		2.rpush key value ：向key对应list尾部添加一个字符串元素。 
	
		3.lrange key start end ：返回指定区间内（start~end）的元素。下标从0开始。支持负数。 
	
		4.lpop key ：从list头部返回一个元素，并删除此元素。
	
		5.4.rpop key ：从list头部返回一个元素，并删除此元素。

   使用场景:

		微博中我的粉丝列表，关注列表	



**Hash类型** 

	hash 是一个 string 类型的 field 和 value 的映射表，适合用于存储对象。
	
	实现方式：
	
		Hash的成员比较少时，采用类似一维数组的方式来紧凑存储,不会采用HashMap结构，此时encoding为	   zipmap，成员数量增大时会自动转成HashMap，此时encoding为ht
	
	常用命令：
	
		hset key field value ：设置key对应的hash对象中指定域的值。 
	
		hget key field ：返回key对应的hash对象中指定域的值。
	
		hdel key field [field ...]： 删除
	
		hlen key：计算field个数
	
	适用场景：
	
		 1.存储用户信息
		  2.原生string:直观,占用键多
		  3.序列化字符串:序列化后好操作,每次都需要反序列化和序列化所有字段 
		 4.哈希类型:简单直观,减少内存空间的使用

**Set类型**

	一种无序的不重复的集合
	
	实现方式:
	
			hashmap, 通过计算hash的方式来快速排重
	
	常用命令：
	
		sadd key member ：添加一个元素到key对于的set集合中。 
	
		sismember key member ：判断member是否在set中。 
	
		smembers key ：返回所以key对于的set元素。
	
		srem key element [element ...]：  删除元素
	
		scard key：计算元素个数

   应用场景：

		抽奖，推荐系统，好友



**Sorted Set（zset）**

			是一个有序的不重复的集合，集合的成员从大到小排序，并且每个元素都会关联一个double类型的分数，成员是唯一，但是分数可以不唯一
	
		实现方式：
	
			使用HashMap和跳跃表(SkipList)来保证数据的存储和有序
	
		常用命令：
	
			zadd key score member: 添加成员
	
			zcard key：计算成员个数
	
			zscore key member: 计算某个成员的分数
	
			zrank key member: 计算成员的排名,分数从低到高
	
			zrevrank key member： 计算成员的排名，分数从高到低
	
			zrem key member： 删除成员
	
			zincrby key increment member： 增加成员的分数
	
	    使用场景：
	
			排行榜，分页(动态分页)


​		