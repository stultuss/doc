# Zipkin

## 概述

由于微服务架构的集群部署和业务独立性，会导致在一个请求的完成至少经过三四个业务模块的调用。如何分析链路的瓶颈进行调优，快速的故障发现，这就是分布式链路跟踪系统存在的目的和意义。分布式链路追踪系统起源于Google的论文“[Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](http://research.google.com/pubs/pub36356.html)”（译文可参考[此处](http://bigbully.github.io/Dapper-translation/)），Twitter的zipkin是基于此论文上线较早的分布式链路追踪系统了。

##介绍

Zipkin是Twitter的一个开源项目，是一个致力于收集Twitter所有服务的监控数据的分布式跟踪系统，它提供了收集数据，和查询数据两大接口服务。

## Zipkin的架构

Zipkin 一共是由 4 个组件组成的，分别是：Collector，Storage，Search，WebUI。

他们之间的关系如下图1

图1：

![Zipkin1](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin1.png)

## Zipkin的特点

### 低侵入性

作为非业务组件，链路追踪系统应当尽量少侵入或者无侵入其他业务系统，对使用方透明，减少开发负担。而Zipkin 并没有提供相应的客户端，在侵入性方面，使用方完全可以根据 Zipkin 的数据结构自行开发低侵入性的中间件进行数据的处理和转发。

### 灵活性

同样，由于Zipkin并没有提供客户端的原因。可以在自己开发的 Zipkin 客户端中自行添加采样比例的代码，举例：通过消息队列发送采样比例数值，达到远程控制采样比例的结果。

### 可视化

Zipkin 拥有可视化的展示界面，并且提供丰富的查询条件，将数据在决策支持层面发挥作用。

### 故障容忍

Zipkin 分布式链路跟踪系统是完全独立的服务，当服务出现宕机，并不会影响到正常的业务。

## Zipkin的安装

安装 Zipkin 非常简单，openzipkin 直接提供了 Zipkin 以及相关组件的 docker 镜像。在 [github.com/openzipkin](https://github.com/openzipkin/docker-zipki) 上有详细的说明和安装流程。

以下例子将 ElasticSearch 作为 Storage，启动 Zipkin。

1、启动 zipkin-elasticsearch。

```bash
docker run -d \
    -p 9200:9200 \
    openzipkin/zipkin-elasticsearch
```

2、启动 zipkin。

```bash
docker run -d \
    -p 9411:9411 \
    -e STORAGE_TYPE=elasticsearch \
    -e ES_HOSTS=127.0.0.1:9200 \
    openzipkin/zipkin
```

3、启动zipkin-dependencies。

```bash
docker run -d \
    -e STORAGE_TYPE=elasticsearch \
    -e ES_HOSTS=127.0.0.1:9200 \
    openzipkin/zipkin-dependencies
```

## Zipkin的原理

### Span模型

Zipkin 的 Span 模型几乎仿造了 Dapper 的 Span 模型设计，我们知道 Span 是用来描述一次 RPC 调用的，所以一个 RPC 调用只应该关联一个 spanId（不算父spanId），Zipkin 的 Span 主要包含三个数据部分：

- 基础数据：用于跟踪树的节点关联和界面展示，包含 traceId，spanId，parentSpanId（pSpanId），name，timestamp，duration。
- Annotation数据：用来记录关键时间，只有四种：cs（Client Send），sr（Server Receive），ss（Server Send），cr（Client Receive），每种关键事件包含 value，timestamp，endpoint。
- BinaryAnnotation数据：用于记录业务数据，类似数据采集的埋点。它的结构和 Annotation 数据一样。

### Span数据流转

> SpanId 官方提供的生成算法是随机生成一个64位固定长度的十六进制格式的数字，如果有需要，可以在API Gateway 中直接使用自定义算法的值来替换 SpanId 中的值。

现在我们已经了解了一个Span的内部结构，接下来我们用一个图片来举例说明 Span 的数据流转。

![Zipkin2](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin2.png)

### 解释

#### Span 数据流转流程如下：

1. 用户访问 gateway 。
   - gateway 会生成一个全新的 traceId：bc614，并把这个 traceId 作为自己的 SpanId。
   - gateway 将 ServerRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：bc614）。
2. gateway 向 user 发起请求。
   - gateway 为 user 创建一个 child TraceId。（traceId：bc614，spanId：154f7c）
   - gateway 将 ClientSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：154f7c）
   - gateway 将 child TraceId 信息通过 grpc 元数据发送到 order。
3. user 收到 gateway 的请求。
   - user 通过 grpc 元数据组织一个 child TraceId。（traceId：bc614，spanId：154f7c）
   - user 将 ServerRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：154f7c）
4. user向 order 发起请求。
   - user为 order 创建一个 child TraceId。（traceId：bc614，spanId：1ed8e4）
   - user将 ClientSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：1ed8e4）
   - user将 child TraceId 信息通过 grpc 元数据发送到 order。
5. order 收到 user的请求。
   - order 通过 grpc 元数据组织一个 child TraceId。（traceId：bc614，spanId：1ed8e4）
   - order 将 ServerRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：1ed8e4）
6. order 执行完成后，通过 grpc callback 将结果返回给 user。
   - order 将 ServerSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：1ed8e4）
7. user 收到 order 的 grpc callback 结果。
   - user 将 ClientRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：1ed8e4）
8. user向 mysql 发起请求。
   - user为 order 创建一个 child TraceId。（traceId：bc614，spanId：28624c）
   - user将 ClientSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：28624c）
9. user 收到 mysql 的 查询结果。
   - user 将 ClientRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：28624c）
10. user 执行完成后，通过 grpc callback 将结果返回给 gateway
    - user 将 ServerSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：154f7c）
11. gateway 收到 user 的 grpc callback 结果。
    - gateway 将 ClientRecv 事件发送到 Zipkin Collector。（traceId：bc614，spanId：154f7c）
12. gateway 执行完成后，将最终结果打印给用户。
    - gateway 将 ServerSend 事件发送到 Zipkin Collector。（traceId：bc614，spanId：bc614）

####Span 数据流转的 Trace 树

```json
// traceId: bc614
{
  "name": "gateway",
  "spanId": "bc614",
  "childs": [{
  	"name": "user",
  	"spanId": "154f7c",
    "childs": [{
  	  "name": "order",
      "spanId": "1ed8e4"
    },{
  	  "name": "mysql",
      "spanId": "28624c"
    }]
  }]
}
```

## Zipkin的WebUI

### 主界面

我们通过打开 Zipkin 的主界面，查看最新的跟踪日志，跟踪日志会默认显示最近的 10 次跟踪日志，并根据执行时间进行排序，通过更改搜索条件，点击`Find Traces`可以进行精确查询。

![Zipkin3](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin3.png)

### 服务依赖树

通过点击上方  banner 上的 Dependencies，可以看到一段时间内的服务调用情况以及他们之间的关系。

![Zipkin4](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin4.png)

### 跟踪日志详情

在主界面上，点击某个日志进入，可以看到这一次调用中，每个服务执行时间和顺序。

![Zipkin5](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin5.png)

进入每个服务，我们可以看到埋点的 BinaryAnnotation 数据，目前封装的 Zipkin 客户端，记录了调用的接口名（rpc_query），调用接口的参数（rpc_query_params），返回结果类型（rpc_end）以及返回结果内容（rpc_end_response）

![Zipkin6](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin6.png)

### 错误日志查询

用户可以在查询条件中填写 BinaryAnnotation 的字段进行精确查询，例如 rpc_end=Error，这时候，就可以通过查询条件，筛选出符合条件的错误日志。

**Zipkin 不支持模糊查询，查询人必须清除的知道自己想要查什么**

![Zipkin7](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin7.png)

进入其中的某条日志，我们就可以在 rpc_end_response 中看到服务的保存讯息了。

![Zipkin8](https://github.com/stultuss/doc/blob/master/images/microservice/Zipkin8.png)

## 问题

###  关于 Zipkin 日志中的时区问题

答：zipkin 的 node 实现中，时间的采集是使用本机 node 进程的 process..hrtime 函数，因此本机的时区必须设置为 UTC 时间，否则会造成多台微服务之间的时区时间差。

