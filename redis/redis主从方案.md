## redis主从方案

redis主从模式是最简单的一种集群方案配置起来也比较简单，它的特点主要有：

- 一个master可以拥有多个slave
- 多个slave链接同一个master，也可以链接其它slave
- 主从复制不会阻塞master,在同步数据时，master可以继续处理client请求.
- slave 配置为slave-read-only on需要升级为主节点或者写入配置文件中, 而不能在默认slave情况下直接设置master与slave断开后会检测心跳, 从新建立连接.
- 可以直接copy DUMP文件从新重启master，在Master为空以后，slave同步数据会抹掉全部数据.

这种简单的主从读写分离方案的缺点也比较多，类似Mysql的主从方案，往Master节点写数据，同时Master节点会异步写入slave节点中。这种方案目前使用的越来越少，不过对于个体开发并且对缓存依赖度不高的系统还是可以使用的，毕竟搭建和维护简单。

## redis cluster方案

Redis Cluster是一种服务器Sharding技术，3.0版本开始正式提供。
 Redis Cluster中，Sharding采用slot(槽)的概念，一共分成16384个槽，这有点儿类pre sharding思路。对于每个进入Redis的键值对，根据key进行散列，分配到这16384个slot中的某一个中。使用的hash算法也比较简单，就是CRC16后16384取模。
 Redis集群中的每个node(节点)负责分摊这16384个slot中的一部分，也就是说，每个slot都对应一个node负责处理。当动态添加或减少node节点时，需要将16384个槽做个再分配，槽中的键值也要迁移。
 Redis集群，要保证16384个槽对应的node都正常工作，如果某个node发生故障，那它负责的slots也就失效，整个集群将不能工作。
 为了增加集群的可访问性，官方推荐的方案是将node配置成主从结构，即一个master主节点，挂n个slave从节点。这时，如果主节点失效，Redis Cluster会根据选举算法从slave节点中选择一个上升为主节点，整个集群继续对外提供服务，Redis Cluster本身提供了故障转移容错的能力。
 Redis Cluster的新节点识别能力、故障判断及故障转移能力是通过集群中的每个node都在和其它nodes进行通信，这被称为集群总线(cluster bus)。它们使用特殊的端口号，即对外服务端口号加10000。例如如果某个node的端口号是6379，那么它与其它nodes通信的端口号是16379。nodes之间的通信采用特殊的二进制协议。
 对客户端来说，整个cluster被看做是一个整体，客户端可以连接任意一个node进行操作，就像操作单一Redis实例一样，当客户端操作的key没有分配到该node上时，Redis会返回转向指令，指向正确的node。

![image-20200927102221977](https://gitee.com/fking86/images4typora/raw/master/imgs/20200927102222.png)

redis cluster

从这种redis cluster的架构图中可以很容易的看出首先将数据根据hash规则分配到6个slot中（这里只是举例子分成了6个槽），然后根据CRC算法和取模算法将6个slot分别存储到3个不同的Master节点中，每个master节点又配套部署了一个slave节点，当一个master出现问题后，slave节点可以顶上。这种cluster的方案对比第一种简单的主从方案的优点在于提高了读写的并发，分散了IO，在保障高可用的前提下提高了性能。

具体的redis cluster的搭建方案可以参考官方的搭建方案，链接中是中文版。
 [redis cluster官方搭建方案](https://link.jianshu.com?t=http%3A%2F%2Fwww.redis.cn%2Ftopics%2Fcluster-tutorial.html)



## codis集群方案

Codis是一个豌豆荚团队开源的使用Go语言编写的Redis Proxy使用方法和普通的redis没有任何区别，设置好下属的多个redis实例后就可以了，使用时在本需要连接redis的地方改为连接codis，它会以一个代理的身份接收请求 并使用一致性hash算法，将请求转接到具体redis，将结果再返回codis，和之前比较流行的twitter开源的Twemproxy功能类似，但是相比官方的redis cluster和twitter的Twemproxy还是有一些独到的优势，Codis官方功能对比图如下：

|                                       | Codis       | Twemproxy   | Redis Cluster                                                |
| ------------------------------------- | ----------- | ----------- | ------------------------------------------------------------ |
| resharding without restarting cluster | Yes         | No          | Yes                                                          |
| pipeline                              | Yes         | Yes         | No                                                           |
| hash tags for multi-key operations    | Yes         | Yes         | Yes                                                          |
| multi-key operations while resharding | Yes         | -           | No([details](https://link.jianshu.com?t=http%3A%2F%2Fredis.io%2Ftopics%2Fcluster-spec%23multiple-keys-operations)) |
| Redis clients supporting              | Any clients | Any clients | Clients have to support cluster protocol                     |

从表中我们可以看出Codis一个比较大的优点是可以不停机动态新增或删除数据节点，旧节点的数据也可以自动恢复到新节点。并且提供图形化的dashboard，方便集群管理。
 [Codis Github](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FCodisLabs%2Fcodis)
 下面我们看下如果使用Coids作为缓存集群方案的架构图，简单画了这么个架构图，这个架构是codis保证HA的前提下的最小级，从这张架构图可以看到我们最少需要8台机器，其中一台机器是codis的dashboard用于通过web界面可视化的配置codis group和proxy，也可以查看各个节点的状态；还有两台是用于codis的proxy代理节点，两个节点之间通过pipeline主从互备；还需要至少配置一台zk用于保存slot状态信息，也可以通过etcd存储这些状态信息，方便client请求的路由，也可以配置多台保证高可用；最后就是要配置数据节点来存储数据了，在codis中需要将数据节点都放在codis group中进行管理，每个group至少保留一个节点，该架构图中，为了保证HA，我们每个group都配置了一个master一个slave节点，这里配置了两个group，如果一个group中的master挂了，那么同一个group中的slave节点通过选举算法选出新的master节点，并通知到proxy，如果为了较好的高可用可以增加group的个数和每个group中slave节点的个数。

![image-20200927102245920](https://gitee.com/fking86/images4typora/raw/master/imgs/20200927102246.png)

codis

codis方案推出的时间比较长，而且国内很多互联网公司都已经使用了该集群方案，所以该方案还是比较适合大型互联网系统使用的，毕竟成功案例比较多，但是codis因为要实现slot切片，所以修改了redis-server的源码，对于后续的更新升级也会存在一定的隐患。，但是codis的稳定性和高可用确实是目前做的最好的，只要有足够多的机器能够做到非常好的高可用缓存系统。

Codis搭建步骤可以参考官方的 [Guide](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FCodisLabs%2Fcodis%2Fblob%2Frelease3.2%2Fdoc%2Ftutorial_zh.md)

## 总结

从各个集群方案的对比中我们也发现Codis是目前用的比较多的一种方案，Codis在高可用方面做的比较好，不需要重启节点和增加删除节点后自动Resharding，但是因为Codis是对redis-server做了改动，如果出现问题或者redis升级小团队可能应付不了，所以对于小规模应用最好还是使用官方的cluster方案，在redis3.0后官方社区也在逐步完善cluster方案，小团队可以随着官方版本的升级享受功能和稳定性的提升，也便于日后的维护