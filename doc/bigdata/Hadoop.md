# Hadoop

> Hadoop分布式文件系统，简称 HDFS，**是 Hadoop 抽象文件系统的一种实现**，Hadoop 抽象文件系统可以与本地系统，Amazon S3 等集成，甚至可以通过 Web 协议来操作。HDFS 的文件分布在集群机器上，同时提供腐败进行容错及可靠性保证。例如客户端写入读取文件的直接操作都是分布在集群各个机器上，没有单点性能压力。

如果你从零开始搭建一个完整的集群，参考: [Hadoop集群搭建详细步骤](http://blog.csdn.net/bingduanlbd/article/details/51892750)

## 准备 

### 安装 Java SE 环境

```shell
## 下载安装
#https://download.oracle.com/otn/java/jdk/8u261-b12/a4634525489241b9a9e1aa73d9e118e6/jdk-8u261-linux-x64.tar.gz
$ tar -C /opt -zxvf jdk-8u231-linux-x64.tar.gz
$ sudo mv /opt/jdk1.8.0_261/ /opt/jdk/

## 配置全局变量
$ vim /etc/profile
export JAVA_HOME=/opt/jdk
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
$ source /etc/profile

## 安装校验
$ java -version

java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

### 配置 SSH 

```shell
## 配置免密码登录
# 1.首先在设置中开启远程登录：系统偏好设置--->共享--->勾选“远程登录”
# 2.配置私钥
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub>>~/.ssh/authorized_keys
$ chmod og-wx ~/.ssh/authorized_keys
# 3.测试 SSH 登录
$ ssh localhost

# 如果是集群部署，使用 ssh-copy-id 命令
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.1
```

### Hadoop

配置文件目录

```shell
$ mkdir /opt/data/hdfs
$ cd /opt/data/hdfs
$ mkdir name
$ mkdir data
$ mkdir tmp

# 如果是集群部署，使用 scp 命令
# scp -r hdfs/ root@192.168.1.1:/opt/data/hdfs
```

## 安装及配置

```shell
## 下载安装
$ wget https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
$ tar -C /opt -zxvf hadoop-2.9.2.tar.gz
$ sudo mv /opt/hadoop-2.9.2/ /opt/hadoop/

## 配置全局变量
$ vim /etc/profile
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
$ source /etc/profile

# 开箱测试 hadoop 
$ hadoop version

Hadoop 2.9.2
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 826afbeae31ca687bc2f8471dc841b66ed2c6704
Compiled by ajisaka on 2018-11-13T12:42Z
Compiled with protoc 2.5.0
From source with checksum 3a9939967262218aa556c684d107985
This command was run using /opt/hadoop/share/hadoop/common/hadoop-common-2.9.2.jar

# 修改基础配置文件
$ cd /opt/hadoop
$ find . | grep site.xml 

./etc/hadoop/mapred-site.xml.template
./etc/hadoop/yarn-site.xml
./etc/hadoop/hdfs-site.xml
./etc/hadoop/kms-site.xml
./etc/hadoop/core-site.xml
./etc/hadoop/httpfs-site.xml
...

# 配置位置：/opt/hadoop/etc/hadoop
```

**core-site.xml**

```xml
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/opt/data/hdfs/tmp</value>
    <description>A base for other temporary directories.</description>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
  </property>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
  </property>
  <property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
  </property>
</configuration>
```

**hdfs-site.xml**

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/opt/data/hdfsname</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/opt/data/hdfs/data</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>localhost:9001</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
</configuration>
```

**yarn-site.xml**

```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>localhost</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>localhost:18040</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>localhost:18030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>0.0.0.0:18088</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>localhost:18025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>localhost:18141</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
</configuration>
```

**mapred-site.xml**

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

**运行  Hadoop**

```shell
# 启动 hadoop 的守护进程
$ hadoop-daemon.sh start namenode
$ hadoop-daemon.sh start datanode

# 启动 namenode 和 datanode 的步骤可以合并为
$ start-dfs.sh

# 检查进程
$ ps -ef | grep hadoop

# 使用jps命令查看JVM进程：
$ jps

14883 Jps
14792 NameNode

# 如果遇到 JAVA_HOME 无法找到的报错，则需要在 hadoop-env.sh 中配置 JAVA_HOME
$ vim ./etc/hadoop/hadoop-env.sh
```

**启动 Yarn**

```shell
# 启动 YARN 守护进程
$ yarn-daemon.sh start resourcemanager
$ yarm-daemon.sh start nodemanager

# 以上命令可以合并为
$ start-yarn.sh
```

## 管理界面

查看 hdfs 管理界面：localhost:50070/dfshealth.html

查看 yarn 管理界面：localhost:18088/cluster

## 测试

```shell
$ hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar pi 5 10
```

## 参考

1. [深入理解 Hadoop HDFS](https://blog.csdn.net/sjmz30071360/article/details/79877846)

