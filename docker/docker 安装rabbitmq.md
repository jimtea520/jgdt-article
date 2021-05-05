### 查找镜像

带有可视化界面



```shell
[root@VM-0-15-centos ~]# docker search rabbitmq:management
NAME                                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
macintoshplus/rabbitmq-management   Based on rabbitmq:management whit python and…   6                                       [OK]
xiaochunping/rabbitmq               xiaochunping/rabbitmq:management   2018-06-30   4                                       
transmitsms/rabbitmq-sharded        Fork of rabbitmq:management with sharded_exc…   0                                       
yunyan2140/rabbitmq                 docker pull rabbitmq:management                 0  
```

### 拉取镜像



```shell
docker pull docker.io/macintoshplus/rabbitmq-management
```

### 查看镜像



```shell
[root@VM-0-15-centos ~]# docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
macintoshplus/rabbitmq-management   latest              c20a8529776b        4 years ago         216MB
```

### 制作容器并启动



```shell
docker run -d --hostname fuyi-rabbit --name rabbitmq -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest -p 15672:15672 -p 5672:5672 c20
```

参数说明



```text
-d                       #后台运行
-- homename fuyi-rabbit     #主机名
RABBITMQ_DEFAULT_USER=guest #可视化界面登录用户名
RABBITMQ_DEFAULT_PASS=guest #可视化界面登录密码
-p 15672:15672              #端口映射
c20                         #镜像ID
```

### 启动成功



```shell
[root@VM-0-15-centos ~]# docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                      PORTS                                                                     NAMES
a9868b676705        c20                   "/docker-entrypoint.…"   5 seconds ago       Up 3 seconds                4369/tcp, 5671-5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbitmq
```

### 登录访问



```shell
http://ip:15672/ 
```