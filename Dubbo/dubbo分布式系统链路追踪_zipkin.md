# dubbo分布式系统链路追踪_zipkin



# 基础知识储备

## 分布式跟踪的目标

> 一个分布式系统由若干分布式服务构成，每一个请求会经过多个业务系统并留下足迹，但是这些分散的数据对于问题排查，或是流程优化都很有限，要能做到追踪每个请求的完整链路调用，收集链路调用上每个服务的性能数据，计算性能数据和比对性能指标（SLA），甚至能够再反馈到服务治理中，那么这就是分布式跟踪的目标。

## 分布式跟踪的目的

> zipkin分布式跟踪系统的目的：

- zipkin为分布式链路调用监控系统，聚合各业务系统调用延迟数据，达到链路调用监控跟踪；
- zipkin通过采集跟踪数据可以帮助开发者深入了解在分布式系统中某一个特定的请求时如何执行的；
- 假如我们现在有一个用户请求超时，我们就可以将这个超时的请求调用链展示在UI当中；我们可以很快度的定位到导致响应很慢的服务究竟是什么。如果对这个服务细节也很很清晰，那么我们还可以定位是服务中的哪个问题导致超时；
- zipkin系统让开发者可通过一个Web前端轻松的收集和分析数据，例如用户每次请求服务的处理时间等，可方便的监测系统中存在的瓶颈。

## ZipKin介绍

> - Zipkin是一个致力于收集分布式服务的时间数据的分布式跟踪系统。

> - Zipkin 主要涉及四个组件：collector（数据采集）,storage（数据存储）,search（数据查询）,UI（数据展示）。

> - github源码地址:[https://github.com/openzipkin/zipkin](https://link.jianshu.com?t=https://github.com/openzipkin/zipkin)。

> - Zipkin提供了可插拔数据存储方式：In-Memory，MySql, Cassandra, Elasticsearch

## brave 介绍

> Brave 是用来装备 Java 程序的类库，提供了面向标准Servlet、Spring MVC、Http Client、JAX RS、Jersey、Resteasy 和 MySQL 等接口的装备能力，可以通过编写简单的配置和代码，让基于这些框架构建的应用可以向 Zipkin 报告数据。同时 Brave 也提供了非常简单且标准化的接口，在以上封装无法满足要求的时候可以方便扩展与定制。

> 本文主要介绍springmvc+dubbo下的brave使用。

## zipkin存储与启动

详情参考官网： [https://github.com/openzipkin/zipkin/tree/master/zipkin-server](https://link.jianshu.com?t=https://github.com/openzipkin/zipkin/tree/master/zipkin-server)

### （1）In-Memory方式



```shell
 nohup java -jar zipkin.jar  &
```

注意：内存存储，zipkin重启后数据会丢失，**建议测试环境使用**

### （2）MySql方式

目前只与MySQL的5.6-7。它的设计是易于理解，使用简单。但是，当数据量大时，查询很慢。性能不是很好。

- 创建数据库zipkin
- 建表



```sql
CREATE TABLE IF NOT EXISTS zipkin_spans (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL,
  `id` BIGINT NOT NULL,
  `name` VARCHAR(255) NOT NULL,
  `parent_id` BIGINT,
  `debug` BIT(1),
  `start_ts` BIGINT COMMENT 'Span.timestamp(): epoch micros used for endTs query and to implement TTL',
  `duration` BIGINT COMMENT 'Span.duration(): micros used for minDuration and maxDuration query'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_spans ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `id`) COMMENT 'ignore insert on duplicate';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`, `id`) COMMENT 'for joining with zipkin_annotations';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTracesByIds';
ALTER TABLE zipkin_spans ADD INDEX(`name`) COMMENT 'for getTraces and getSpanNames';
ALTER TABLE zipkin_spans ADD INDEX(`start_ts`) COMMENT 'for getTraces ordering and range';

CREATE TABLE IF NOT EXISTS zipkin_annotations (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.trace_id',
  `span_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.id',
  `a_key` VARCHAR(255) NOT NULL COMMENT 'BinaryAnnotation.key or Annotation.value if type == -1',
  `a_value` BLOB COMMENT 'BinaryAnnotation.value(), which must be smaller than 64KB',
  `a_type` INT NOT NULL COMMENT 'BinaryAnnotation.type() or -1 if Annotation',
  `a_timestamp` BIGINT COMMENT 'Used to implement TTL; Annotation.timestamp or zipkin_spans.timestamp',
  `endpoint_ipv4` INT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_ipv6` BINARY(16) COMMENT 'Null when Binary/Annotation.endpoint is null, or no IPv6 address',
  `endpoint_port` SMALLINT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_service_name` VARCHAR(255) COMMENT 'Null when Binary/Annotation.endpoint is null'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_annotations ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `span_id`, `a_key`, `a_timestamp`) COMMENT 'Ignore insert on duplicate';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`, `span_id`) COMMENT 'for joining with zipkin_spans';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTraces/ByIds';
ALTER TABLE zipkin_annotations ADD INDEX(`endpoint_service_name`) COMMENT 'for getTraces and getServiceNames';
ALTER TABLE zipkin_annotations ADD INDEX(`a_type`) COMMENT 'for getTraces';
ALTER TABLE zipkin_annotations ADD INDEX(`a_key`) COMMENT 'for getTraces';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id`, `span_id`, `a_key`) COMMENT 'for dependencies job';

CREATE TABLE IF NOT EXISTS zipkin_dependencies (
  `day` DATE NOT NULL,
  `parent` VARCHAR(255) NOT NULL,
  `child` VARCHAR(255) NOT NULL,
  `call_count` BIGINT
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_dependencies ADD UNIQUE KEY(`day`, `parent`, `child`);
```

- 启动zipkin命令

> $  STORAGE_TYPE=mysql MYSQL_HOST=**IP** MYSQL_TCP_PORT=3306 MYSQL_DB=**zipkin** MYSQL_USER=**username** MYSQL_PASS=**password** nohup java -jar zipkin.jar &

### （3）Elasticsearch方式

本文建议使用此方法。

- [Elasticsearch官网](https://link.jianshu.com?t=https://www.elastic.co/downloads/elasticsearch)
- 创建elasticsearch用户，安装启动Elasticsearch服务
- 官方文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html](https://link.jianshu.com?t=https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- zipkin启动命令

> $ STORAGE_TYPE=elasticsearch ES_HOSTS=http：//**IP**:9200 nohup java -jar zipkin.jar &



## 整体结构长啥样

一个独立的分布式追踪系统，客户端存在于应用中（即各服务中），应具备追踪信息生成、采集发送等功能，而服务端应该包含以下基本的三个功能：

- 信息收集：用来收集各服务端采集的信息，并对这些信息进行梳理存储、建立索引。
- 数据存储：存储追踪数据。
- 查询服务：提供查询请求链路信息的接口。

zipkin 整体结构图如下：

![image-20200928123221767](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928123222.png)

zipkin(服务端)包含四个组件，分别是collector、storage、search、web UI。

- collector 就是信息收集器,作为一个守护进程，它会时刻等待客户端传递过来的追踪数据，对这些数据进行验证、存储以及创建查询需要的索引。
- storage 是存储组件。zipkin 默认直接将数据存在内存中，此外支持使用Cassandra、ElasticSearch 和 Mysql。
- search 是一个查询进程，它提供了简单的JSON API来供外部调用查询。
- web UI 是zipkin的服务端展示平台，主要调用search提供的接口，用图表将链路信息清晰地展示给开发人员。

zipkin的客户端主要负责根据应用的调用情况生成追踪信息，并且将这些追踪信息发送至zipkin由收集器接收。各语言支持均不同，具体可以查看[zipkin官网](https://zipkin.io/pages/tracers_instrumentation.html),java语言的支持就是brave。上面结构图中，有追踪器就是指集成了brave。



## 基本概念了解下

在使用zipkin之前，先了解一下Trace和Span这两个基本概念。一个请求到达应用后所调用的所有服务所有服务组成的调用链就像一个树结构（如下图），我们**追踪**这个调用**链路**得到的这个树结构可以称之为**Trace**。

![image-20200928123350611](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928123351.png)
在一次Trace中，每个服务的**每一次调用**，就是一个**基本工作单元**，就像上图中的每一个树节点，称之为**span**。每一个span都有一个**id作为唯一标识**，同样每一次Trace都会生成一个**traceId在span中作为追踪标识**，另外再通过一个**parentId标明本次调用的发起者**（就是发起者的span-id）。当span有了上面三个标识后，就可以很清晰的将多个span进行梳理串联，最终归纳出一条完整的跟踪链路。此外，span还会有其他数据，比如：名称、节点上下文、时间戳以及K-V结构的tag信息等等（Zipkin v1核心注解如“cs”和“sr”已被Span.Kind取代，详情查看[zipkin-api](https://zipkin.io/zipkin-api/#/)，本文会在入门的demo介绍完后对具体的Span数据模型进行说明）。



## 具体怎么追踪的

追踪器位于应用程序上，负责生成相关ID、记录span需要的信息，最后通过传输层传递给服务端的收集器。我们首先思考下面几个问题：

- 每个span需要的基本信息何时生成？
- 哪些信息需要随着服务调用传递给服务提供方？
- 什么时候发送span至zipkin 服务端？
- 以何种方式发送span?

一个 span 表示一次服务调用，那么追踪器必定是被服务调用发起的动作触发，生成基本信息，同时为了追踪服务提供方对其他服务的调用情况，便需要传递本次追踪链路的traceId和本次调用的span-id。服务提供方完成服务将结果响应给调用方时，需要根据调用发起时记录的时间戳与当前时间戳计算本次服务的持续时间进行记录，至此这次调用的追踪span完成，就可以发送给zipkin服务端了。但是需要注意的是，发送span给zipkin collector不得影响此次业务结果，其发送成功与否跟业务无关，因此这里需要采用异步的方式发送，防止追踪系统发送延迟与发送失败导致用户系统的延迟与中断。下图就表示了一次http请求调用的追踪流程（基于zipkin官网提供的流程图）：

![image-20200928123408381](https://gitee.com/fking86/images4typora/raw/master/imgs/20200928123408.png)
可以看出服务A请求服务B时先被追踪器拦截，记录tag信息、时间戳，同时将追踪标识添加进http header中传递给服务B，在服务B响应后，记录持续时间，最终采取异步的方式发送给zipkin收集器。span从被追踪的服务传送到Zipkin收集器有三种主要的传送方式：http、Kafka以及Scribe（Facebook开源的日志收集系统）。



