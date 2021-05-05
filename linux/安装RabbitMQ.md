由于rabbitmq是基于erlang语言开发的，所以必须先安装erlang。

 

安装依赖

```
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
```

 

erlang官网：

https://www.erlang.org/downloads

 

下载（会比较慢，请耐心等待）

```
wget http://erlang.org/download/otp_src_22.0.tar.gz
```

 

解压

```
tar -zxvf otp_src_22.0.tar.gz
```

 

移走

```
mv otp_src_22.0 /usr/local/
```

 

切换目录

```
cd /usr/local/otp_src_22.0/
```

 

创建即将安装的目录

```
mkdir ../erlang
```

 

配置安装路径

```
./configure --prefix=/usr/local/erlang
```

 

如果遇到这个错 你就假装没看到

![img](https://img2018.cnblogs.com/blog/1181622/201907/1181622-20190704173838160-1881201010.png)

 

安装

make install

 

查看一下是否安装成功

```
ll /usr/local/erlang/bin
```

 

添加环境变量

```
echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile
```

 

刷新环境变量

```
source /etc/profile
```

 

甩一条命令

```
erl
```

 

瞬间进入了一个未知的世界

![img](https://img2018.cnblogs.com/blog/1181622/201907/1181622-20190704174008184-103160721.png)

 

在里面输入halt().命令退出来（那个点号别忘记）

![img](https://img2018.cnblogs.com/blog/1181622/201907/1181622-20190704174027151-798244865.png)

 

回到顶部

### 安装RabbitMQ

rabbitmq下载地址：

https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.7.15

 

下载

```
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-generic-unix-3.7.15.tar.xz
```

 

由于是tar.xz格式的所以需要用到xz，没有的话就先安装 

```
yum install -y xz
```

 

第一次解压

```
/bin/xz -d rabbitmq-server-generic-unix-3.7.15.tar.xz
```

 

第二次解压

```
tar -xvf rabbitmq-server-generic-unix-3.7.15.tar
```

 

移走

```
mv rabbitmq_server-3.7.15/ /usr/local/
```

 

改名

```
mv /usr/local/rabbitmq_server-3.7.15  rabbitmq
```

 

配置环境变量

```
echo 'export PATH=$PATH:/usr/local/rabbitmq/sbin' >> /etc/profile
```

 

刷新环境变量

```
source /etc/profile
```

 

创建配置目录（本博客并未使用单独的配置文件，因此本步骤纯属多余，感谢朋友特意指正。）

```
mkdir /etc/rabbitmq
```

 

回到顶部

### 启动命令

启动：

```
rabbitmq-server -detached
```

 

停止：

```
rabbitmqctl stop
```

 

状态：

```
rabbitmqctl status
```

 

防火墙之类的请自行处理（5672和15672端口），反正我是从来不开防火墙。

 

回到顶部

### WEB管理

开启web插件

```
rabbitmq-plugins enable rabbitmq_management
```

 

访问：http://127.0.0.1:15672/

![img](https://img2018.cnblogs.com/blog/1181622/201907/1181622-20190704174454266-1014572202.png)

 

默认账号密码：guest guest（这个账号只允许本机访问）

 

回到顶部

### 用户管理

查看所有用户

```
rabbitmqctl list_users
```

 

添加一个用户

```
rabbitmqctl add_user zhaobl 123456
```

 

配置权限

```
rabbitmqctl set_permissions -p "/" zhaobl ".*" ".*" ".*"
```

 

查看用户权限

```
rabbitmqctl list_user_permissions zhaobl
```

 

设置tag

```
rabbitmqctl set_user_tags zhaobl administrator
```

 

删除用户（安全起见，删除默认用户）

```
rabbitmqctl delete_user guest
```

 

回到顶部

### 登陆

配置好用户之后重启一下rabbit

然后就可以用新账号进行登陆

![img](https://img2018.cnblogs.com/blog/1181622/201907/1181622-20190704174640739-2105181498.png)





地址:

https://www.cnblogs.com/fengyumeng/p/11133924.html