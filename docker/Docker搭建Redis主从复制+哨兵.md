### 一、安装Redis

#### 1.拉取官方的镜像,标签为3.2

```shell
    [root@localhost /]# docker pull  redis:3.2
1
```

#### 2.下载完成后，我们就可以在本地镜像列表里查到REPOSITORY为redis,标签为3.2的镜像。

```shell
    [root@localhost /]# docker images redis 
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    redis               3.2                 43c923d57784        2 weeks ago         193.9 MB
123
```

#### 3.运行容器

```shell
    [root@localhost /]# docker run -p 6379:6379 -v $PWD/data:/data  -d redis:3.2 redis-server --appendonly yes
    43f7a65ec7f8bd64eb1c5d82bc4fb60e5eb31915979c4e7821759aac3b62f330
12
```

命令说明：
**-p 6379:6379** : 将容器的6379端口映射到主机的6379端口
**-v $PWD/data:/data** : 将主机中当前目录下的data挂载到容器的/data
**redis-server --appendonly yes** : 在容器执行redis-server启动命令，并打开redis持久化配置



#### 4.连接、查看容器

```shell
   [root@localhost /]# docker exec -it 43f7a65ec7f8 redis-cli
    172.17.0.1:6379> info
    # Server
    redis_version:3.2.0
    redis_git_sha1:00000000
    redis_git_dirty:0
    redis_build_id:f449541256e7d446
    redis_mode:standalone
    os:Linux 4.2.0-16-generic x86_64
    arch_bits:64
    multiplexing_api:epoll
    ...
123456789101112
```

### 二、主从复制

#### 1.运行redis镜像

**首先使用docker启动3个redis容器服务，分别使用到6379、6380、6381端口**

```
docker run -d --name redis -p 6379:6379 redis --requirepass "password"
```

```shell
docker run --name redis-6379 -p 6379:6379 -d redis:3.2 redis-server --requirepass "qyn123#@!"
docker run --name redis-6380 -p 6380:6379 -d redis:3.2 redis-server --requirepass "qyn123#@!"
docker run --name redis-6381 -p 6381:6379 -d redis:3.2 redis-server --requirepass "qyn123#@!"
123
```

#### 2.配置redis集群

使用如下命令查看容器内网的ip地址等信息

```
docker inspect containerid（容器ID）
1
```

![image-20201106012502073](https://gitee.com/fking86/images4typora/raw/master/imgs/image-20201106012502073.png)

3个redis的内网ip地址为：

```
redis-6379：172.17.0.3:6379
redis-6380：172.17.0.4:6379
redis-6381：172.17.0.5:6379
123
```

进入docker容器内部，查看当前redis角色（主master还是从slave）（命令：info replication）

```shell
    [root@localhost /]# docker exec -it 007f7ab412b9 redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:3860
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:3859

127.0.0.1:6379> 

12345678910111213
```

可以看到当前3台redis都是master角色，使用redis-cli命令修改redis-6380、redis-6381的主机为172.17.0.3:6379

`slaveof <master-ip> <master-port>`。`<master-ip>`为主库服务ip，`<master-port>`表示主库所在端口，默认6379

```java
SLAVEOF 172.17.0.3 6379
1
```

主库 密码认证

```
config set masterauth <master-password>。<master-password>即为主库访问密码
```



再次查看主机info，已经有两个从机了（.4 和 .5）

```shell
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.4,port=6379,state=online,offset=3860,lag=0
slave1:ip=172.17.0.5,port=6379,state=online,offset=3860,lag=0
master_repl_offset:3860
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:3859
1234567891011
```

至此，redis下的主从配置就ok了。

### 三、Sentinel哨兵(一)

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务（具体介绍可参考 [链接](http://redisdoc.com/topic/sentinel.html)）：

```
监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。
123
```

接下来直接进入3台redis容器内部进行配置

```shell
[root@localhost /]# docker exec -ti 容器id /bin/bash
1
```

进入根目录创建sentinel.conf文件

```shell
[root@localhost /]# cd / && touch sentinel.conf && touch log.txt
1
```



#### docker 安装vim和yum命令

```
apt-get update
apt-get install vim -y
apt-get install yum -y

apt-get update && apt-get install lrzsz
```

修改sentinel.conf文件内容为：

```shell
sentinel monitor mymaster 172.17.0.3 6379 1
#添加为后台运行
daemonize yes
#指定日志目录
logfile "/log.txt"
12345
```

最后，启动Redis哨兵：

```shell
[root@localhost /]# redis-sentinel /sentinel.conf
1
```

至此，Sentinel哨兵配置完毕。

测试验证，如图

![image-20201106012519051](https://gitee.com/fking86/images4typora/raw/master/imgs/image-20201106012519051.png)



四、Sentinel哨兵(二)



