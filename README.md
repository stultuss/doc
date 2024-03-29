Developer Doc
=========================

> 文档的时效性无法保证，此处仅作保存和参考。

---

## 开发基础

- [网络](https://github.com/stultuss/doc/blob/master/doc/net/HTTP.md)
- [算法](https://github.com/stultuss/doc/blob/master/doc/algorithm/Base.md)
  - [排序算法](https://github.com/stultuss/doc/blob/master/doc/algorithm/Sort.md)
  - [搜索算法](https://github.com/stultuss/doc/blob/master/doc/algorithm/Search.md)
  - [哈希算法](https://github.com/stultuss/doc/blob/master/doc/algorithm/Hash.md)
  - [算法例题](https://github.com/stultuss/doc/blob/master/doc/algorithm/Example.md)
- 数据结构
  - 一维：
    - 数组 Array
    - 链表 Linked List
    - 栈 Stack
    - 队列 Queue
    - 集合 Deque
    - 映射 Map
  - 二维：
    - [树](https://github.com/stultuss/doc/blob/master/doc/structure/SearchTree.md)
      - [平衡树](https://github.com/stultuss/doc/blob/master/doc/structure/BalancedTree.md)
      - [字典树](https://github.com/stultuss/doc/blob/master/doc/structure/TrieTree.md)
      - B-Tree/B+Tree
    - 图 Graph
    - 堆 Heap
    - 并查集 Disjoint Set
    - 字典集 Trie
  - 其他：
    - 位运算 Bitwise
    - 布隆过滤器 Bllom Filter
    - LRU Cache

## 开发语言

- [PHP](https://github.com/stultuss/doc/blob/master/doc/language/PHP.md)
  - [MacOS 环境部署](https://github.com/stultuss/doc/blob/master/doc/deploy/MacOS.md)
  - [Centos 环境部署](https://github.com/stultuss/doc/blob/master/doc/deploy/Centos.md)
- [JavaScript](https://github.com/stultuss/doc/blob/master/doc/language/JavaScript.md)
  - [FLUX](https://github.com/stultuss/doc/blob/master/doc/language/JavaScript-FLUS.md)
  - [React](https://github.com/stultuss/doc/blob/master/doc/language/JavaScript-React.md)
  - [ReactNative](https://github.com/niklaus0823/demo-react-native)
- Node.js
  - [进程](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Process.md)
  - [模块](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Module.md)
  - [异步](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Async.md)
  - [事件](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Event.md)
  - [v8 引擎](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-v8.md)
  - [Debug & Profile](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Profile.md)
  - [编码规范](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-CodeStyle.md)
  - [其他](https://github.com/stultuss/doc/blob/master/doc/language/Node.js-Others.md)
- [Golang](https://github.com/Unknwon/the-way-to-go_ZH_CN/)
  - [基础](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base.md)
    - [更新](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Changelog.md)
    - [协程](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Goroutine.md)
    - [通道](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Channel.md)
    - [并发](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Concurrency.md)
    - [结构体](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Struct.md)
  - [性能](https://github.com/stultuss/doc/blob/master/doc/language/Go-Base-Profiler.md)
  - 异常
  - 调试
  - 监控
  - 工具
  - 进阶
  - 其他

## 通信

- [gRPC](https://github.com/stultuss/doc/blob/master/doc/rpc/gRPC.md)
- WebSocket

## 数据库

- [MySQL](https://github.com/stultuss/doc/blob/master/doc/db/MySQL.md)
  - [DBProxy](https://github.com/stultuss/doc/blob/master/doc/db/MySQL-DBProxy.md)
- [Redis](https://github.com/stultuss/doc/blob/master/doc/db/Redis.md)
- [Memcache](https://github.com/stultuss/doc/blob/master/doc/db/Memcache.md)
- Mongodb
- NoSQL数据库集群算法
  - 一致性哈希

## 运维工具

- Docker
- Kubernetes
- Istio

## 接口规范

- [RESTful API](https://github.com/stultuss/doc/blob/master/doc/protocol/RESTfulAPI.md)
- [Protocol Buffers](https://github.com/stultuss/doc/blob/master/doc/protocol/ProtocolBuffers.md)

## 分布式

- 原理
  - 强一致性
  - 弱一致性
  - 最终一致性
  - 读写一致性
  - 因果一致性
  - 单调读
  - 事物补偿
  - CAP 原理
- 算法
  - Raft 算法
  - Paxos 算法
- 工具
  - [ETCD](https://github.com/stultuss/doc/blob/master/doc/distributed/ETCD.md)
  - Zookeeper
  - Consul
- [微服务](https://github.com/stultuss/doc/blob/master/doc/microservice/Base.md)
  - [链路追踪：Zipkin](https://github.com/stultuss/doc/blob/master/doc/microservice/Zipkin.md)

## 监控

- Prometheus
- Grafana
- Jaeger
- Zipkin

## 工具

- 代理
  - Nginx
  - Envoy
- 消息队列
  - Kafka
  - Pulsar

## 大数据

- 环境搭建
  - [Kafka](https://github.com/stultuss/doc/blob/master/doc/bigdata/Kafka.md)
  - [Hadoop](https://github.com/stultuss/doc/blob/master/doc/bigdata/Hadoop.md)
  - [Kylin](https://github.com/stultuss/doc/blob/master/doc/bigdata/Kylin.md)

