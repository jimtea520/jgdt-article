# MySQL宕机可能导致2种丢数据的场景

1、引擎层提交了，但BINLOG没写入，备库丢事务

2、引擎层没有PREPARE，但BINLOG写入，主库丢事务

即使我们将参数设置成`innodb_flush_log_at_trx_commit =1` 和 `sync_binlog = 1`，也还会面临这样一种情况：主库crash时还有binlog没传递到备库，如果我们直接提升备库为主库，同样会导致主备不一致，老主库必须根据新主库重做，才能恢复到一致的状态。针对这种场景，我们可以通过开启semisync的方式来解决，一种可行的方案描述如下：

1. 设置双1强持久化配置;
2. 我们将semisync的超时时间设到极大值，同时使用semisync AFTER_SYNC模式，即用户线程在写入binlog后，引擎层提交前等待备库ACK；
3. 基于步骤1的配置，我们可以保证在主库crash时，所有老主库比备库多出来的事务都处于prepare状态；
4. 备库完全apply日志后，记下其执行到的relay log对应的位点，然后将备库提升为新主库；
5. 将老主库的最后一个binlog进行截断，截断的位点即为步骤3记录的位点;
6. 启动老主库，那些已经传递到备库的事务都会提交掉，未传递到备库的binlog都会回滚掉。



#### 主库宕机解决

假设发生了突发事件，master宕机，现在的需求是要将192.168.1.102提升为主库，另外一个为从库

步骤：

1.确保所有的relay log全部更新完毕，在每个从库上执行：

  stop slave io_thread; 

  show processlist;

  直到看到Has read all relay log,则表示从库更新都执行完毕了

2.登陆所有从库，查看master.info文件，对比选择pos最大的作为新的主库，这里我们选择192.168.1.102为新的主库

3.登陆192.168.1.102，执行stop slave; 

并进入数据库目录，**删除master.info和relay-log.info文件, 配置my.cnf文件，开启log-bin**,如果有log-slaves-updates和read-only则要注释掉，执行reset master

4.创建用于同步的用户并授权slave，同第3.7步骤

5.登录另外一台从库，执行stop slave停止同步

6.根据第3.7步骤连接到新的主库

7.执行start slave;

8.修改新的master数据，测试slave是否同步更新

