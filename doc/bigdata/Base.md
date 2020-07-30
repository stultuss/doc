# Hadoop、HBase、Hive、Spark分布式系统搭建

Hadoop、HBase、Hive、Spark分布式系统架构，各系统所承担的功能如下：

- Hadoop
  - yarn：负责资源管理和任务调度
    - ResourceManager（RM）负责资源的仲裁，任务调度也需要 RM 负责接受与调度。
    - NodeManager 负责每个节点的资源监控，状态回报和容器管理
  - hdfs：负责分布式存储
    - NameNode 是对全局数据的名字信息作管理的模块
    - SecondaryNameNode 是 NameNode 的从节点（备份）
    - DataNode 是真正的在每个存储节点上管理数据的模块
  - map-reduce 负责分布式计算，依赖于 yarn 和 hdfs
    - JobHistoryServer 用来查看任务运行历史
- Spark：用于分布式机器学习
- Hive：用于分布式数据库
- HBase：用于分布式KV系统

其中 Spark，Hive，HBase 都依赖 Hadoop 的 yarn 资源管理和 hdfs 存储。

