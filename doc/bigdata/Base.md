## 大数据框架

> 引用了Bigdata-Notes的文章

## 简介

Hadoop、HBase、Hive、Spark分布式系统架构，各系统所承担的功能如下：

- Hadoop
  - yarn
    - ResourceManager（RM）负责资源的仲裁，任务调度也需要 RM 负责接受与调度。
    - NodeManager 负责每个节点的资源监控，状态回报和容器管理
  - hdfs 是 Hadoop 下的分布式文件系统，由一个 NameNode 和多个 DataNode 组成
    - NameNode 负责执行有关 **文件系统命名空间** 的操作，同时还负责集群元数据的存储，记录文件中各个数据库的位置信息。SecondaryNameNode 是 NameNode 的从节点（备份）
    - DataNode 负责提供来自文件系统客户端的读写请求，执行块的创建，删除等操作。
  - map-reduce 负责分布式计算，依赖于 yarn 和 hdfs
    - JobHistoryServer 用来查看任务运行历史
- Hive 是一个构建在 Hadoop 之上的数据仓库，它可以将结构化的数据文件映射成表，并提供类 SQL 查询功能，用于查询的 SQL 语句会被转化为 MapReduce 作业，然后提交到 Hadoop 上运行。
- HBase：用于分布式KV系统
- Spark：用于分布式机器学习

其中 Spark，Hive，HBase 都依赖 Hadoop 的 yarn 资源管理和 hdfs 存储。



## 练习

### 词频统计

统计如下样本数据中每个单词出现的次数。

```
Spark	HBase
Hive	Flink	Storm	Hadoop	HBase	Spark
Flink
HBase	Storm
HBase	Hadoop	Hive	Flink
HBase	Flink	Hive	Storm
Hive	Flink	Hadoop
HBase	Hive
Hadoop	Spark	HBase	Storm
HBase	Hadoop	Hive	Flink
HBase	Flink	Hive	Storm
Hive	Flink	Hadoop
HBase	Hive
```

> 工具源码下载地址：[hadoop-word-count](https://github.com/heibaiying/BigData-Notes/tree/master/code/Hadoop/hadoop-word-count) 用于模拟产生词频统计的样本，生成的文件支持输出到本地或者直接写到 HDFS 上。

## 参考

[**大数据框架学习**](https://github.com/stultuss/God-Of-BigData/tree/master/大数据框架学习)

