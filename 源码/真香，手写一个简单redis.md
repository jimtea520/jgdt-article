

# 前言

今天主要介绍两个开源项目，然后创建应用最终实现的效果就像简版的redis服务那样，通过http的get请求，能够插入和获取数据，项目暂取名为kedis，源码后面会上传到git仓库。他们分别是Facebook开源的Rocksdb和netty实现的http容器RestExpress。通过实现这样的一个key/value系统实例来学习这两个框架的使用。



# rocksdb

项目地址：https://github.com/facebook/rocksdb

RocksDB是一个带key/value接口的存储引擎，其中键和值是任意字节流。它是一个C ++库。它是在Facebook基于google开源的LevelDB（https://github.com/google/LevelDB）开发的，并为LevelDB API提供向后兼容的支持。

RocksDB支持各种存储硬件，最初的重点是快速闪存。它使用日志结构化数据库引擎进行存储，完全用C ++编写，并有一个名为RocksJava的Java包装器。

RocksDB可以适应各种生产环境，包括纯内存，闪存，硬盘或远程存储。在RocksDB无法自动适应的情况下，提供了高度灵活的配置设置，以允许用户为其进行调整。它支持各种压缩算法和生产支持和调试的好工具。



## 特征

- 专为希望在本地或远程存储系统上存储多达数TB数据的应用程序服务器而设计。
- 优化用于在快速存储 - 闪存设备或内存中存储中小尺寸键值
- 它适用于具有多个内核的处理器

RocksDB就是这样的一个key/value存储引擎，facebook基于RocksDB这个项目写了MyRocks，一个使用RocksDB实现的msyql数据库引擎。通过RocksDB的压缩技术相比InnoDB能够节省很大的存储空间。newsql数据库tidb组件tikv也使用了RocksDB作为底层数据存储。



# RestExpress

项目地址：https://github.com/RestExpress/RestExpress

RESTExpress是一个非常高效的小型http容器，可以在Java中创建性能非常高，可扩展的RESTful服务。使用牛逼的Netty框架编写，RESTExpress使用非阻塞I / O来处理请求，同时利用Executor来服务后端逻辑服务（可能是阻塞）操作。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfdxtpNFC5vvvrWUlzCxe3PtxcdJx795Kp6Od306Hs3P8QTa5WRRfglcIfs7o5WHTbHeX9F7OiaNbCA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 实现kedis

创建服务并绑定端口

```
/**
 * @author: kl @kailing.pub
 * @date: 2019/4/12
 */
public class Main {

    public static void main(String[] args) {
        Configs configs = new Configs();
        configs.fromArgs(args);
        RestExpress server = new RestExpress()
                .setName("kedis-server")
                .setBaseUrl("http://localhost:" +configs.getPort());
        KedisCore core =new KedisCore(configs.getDbPath());
        Routes.define(server,core);
        server.bind(configs.getPort());
        server.awaitShutdown();
    }
}
```

创建RocksDB引擎api操作类

```
/**
 * @author: kl @kailing.pub
 * @date: 2019/4/12
 */
public class KedisCore {

    private RocksDB db;

    public KedisCore(String path) {
        RocksDB.loadLibrary();
        try {
            final Options options = new Options().setCreateIfMissing(true);
            this.db = RocksDB.open(options, path);
        } catch (RocksDBException ex) {
            ex.printStackTrace();
        }
    }

    public String put(Request request, Response response) throws Exception {
        Map<String, String> map = request.getQueryStringMap();
        String key = map.get("key");
        String value = map.get("value");
        db.put(key.getBytes(), value.getBytes());
        return "ok";
    }

    public String get(Request request, Response response) throws Exception {
        Map<String, String> map = request.getQueryStringMap();
        String key = map.get("key");
        byte[] values = db.get(key.getBytes());
        if(values != null){
            return new String(values,"utf-8");
        }else {
           return null;
        }
    }
}
```

设置请求路由

```
/**
 * @author: kl @kailing.pub
 * @date: 2019/4/12
 */
public abstract class Routes {
   public static void define(RestExpress server,KedisCore core){
       server.uri("/put", core).action("put", HttpMethod.GET).noSerialization();
       server.uri("/get", core).action("get", HttpMethod.GET).noSerialization();
   }
}
```

代码地址：https://gitee.com/kailing/kedis

mvn install打包后，进入target目录会有kedis-1.0.jar。CMD下分别执行如下脚本启动验证

启动

```
java -jar kedis-1.0.jar --port 8081
```

插入数据

```
curl http://localhost:8081/put?key=name&value=ckl
```

获取数据

```
curl http://localhost:8081/get?key=name
```

# 结语

RocksDB和RestExpress这两个项目都很有特点，RocksDB作为嵌入式的微存储引擎java包装器的大小仅有10M左右，主要是C++编译后的dll和so文件，其本身功能非常强大，强大到可以作为mysql的底层存储引擎，对底层存储做了很多的优化。RestExpress虽很轻量但五脏俱全，非常适合一些小工具暴露http的服务。