# Hadoop

> Hadoop分布式文件系统，简称 HDFS，**是 Hadoop 抽象文件系统的一种实现**，Hadoop 抽象文件系统可以与本地系统，Amazon S3 等集成，甚至可以通过 Web 协议来操作。HDFS 的文件分布在集群机器上，同时提供腐败进行容错及可靠性保证。例如客户端写入读取文件的直接操作都是分布在集群各个机器上，没有单点性能压力。

如果你从零开始搭建一个完整的集群，参考: [Hadoop集群搭建详细步骤](http://blog.csdn.net/bingduanlbd/article/details/51892750)

## 一、准备 

### 1. 安装环境

Hadoop 的运行依赖 JDK，需要预先安装

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

### 2. 配置 SSH 免密码登录

Hadoop 组件之间需要基于 SSH 进行通讯

**配置映射**

```shell
$ vim /etc/hosts

# 文件末尾增加
0.0.0.0  hadoop01
```

**生成公私钥并授权**

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

## 安装及配置

### 配置文件目录

```shell
$ mkdir /opt/data/hdfs
$ cd /opt/data/hdfs
$ mkdir name
$ mkdir data
$ mkdir tmp

# 如果是集群部署，使用 scp 命令
# scp -r hdfs/ root@192.168.1.1:/opt/data/hdfs
```

### 环境搭建

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

# 配置位置：/opt/hadoop/etc/hadoop
```

### 修改Hadoop配置

**hadoop-env.sh**

```shell
export JAVA_HOME=/opt/jdk
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
    <value>hdfs://hadoop01:9000</value>
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
    <value>hadoop01:9001</value>
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
    <value>hadoop01</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hadoop01:18040</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hadoop01:18030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>hadoop01:18088</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hadoop01:18025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>hadoop01:18141</value>
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

### 初始化

第一次启动 Hadoop 时需要进行初始化，进入 `${HADOOP_HOME}/bin/` 目录下，执行以下命令：

```shell
$ hdfs namenode -format
```

### 启动 hdfs

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
```

### 启动 yarn

```shell
# 启动 YARN 守护进程
$ yarn-daemon.sh start resourcemanager
$ yarn-daemon.sh start nodemanager

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

## 常用命令

**1. 显示当前目录结构**

```shell
# 显示当前目录结构
$ hadoop fs -ls  <path>
# 递归显示当前目录结构
$ hadoop fs -ls  -R  <path>
# 显示根目录下内容
$ hadoop fs -ls  /
```

**2. 创建目录**

```shell
# 创建目录
$ hadoop fs -mkdir  <path> 
# 递归创建目录
$ hadoop fs -mkdir -p  <path>  
```

**3. 删除操作**

```shell
# 删除文件
$ hadoop fs -rm  <path>
# 递归删除目录和文件
$ hadoop fs -rm -R  <path> 
```

**4. 从本地加载文件到 HDFS**

```shell
# 二选一执行即可
$ hadoop fs -put  [localsrc] [dst] 
$ hadoop fs - copyFromLocal [localsrc] [dst] 
```

**5. 从 HDFS 导出文件到本地**

```shell
# 二选一执行即可
$ hadoop fs -get  [dst] [localsrc] 
$ hadoop fs -copyToLocal [dst] [localsrc] 
```

**6. 查看文件内容**

```shell
# 二选一执行即可
$ hadoop fs -text  <path> 
$ hadoop fs -cat  <path>  
```

**7. 显示文件的最后一千字节**

```shell
$ hadoop fs -tail  <path> 
# 和Linux下一样，会持续监听文件内容变化 并显示文件的最后一千字节
$ hadoop fs -tail -f  <path> 
```

**8. 拷贝文件**

```shell
$ hadoop fs -cp [src] [dst]
```

**9. 移动文件**

```shell
$ hadoop fs -mv [src] [dst] 
```

**10. 统计当前目录下各文件大小**

- 默认单位字节
- -s : 显示所有文件大小总和，
- -h : 将以更友好的方式显示文件大小（例如 64.0m 而不是 67108864）

```shell
$ hadoop fs -du  <path>  
```

**11. 合并下载多个文件**

- -nl 在每个文件的末尾添加换行符（LF）
- -skip-empty-file 跳过空文件

```shell
$ hadoop fs -getmerge
# 示例 将HDFS上的hbase-policy.xml和hbase-site.xml文件合并后下载到本地的/usr/test.xml
$ hadoop fs -getmerge -nl  /test/hbase-policy.xml /test/hbase-site.xml /usr/test.xml
```

**12. 统计文件系统的可用空间信息**

```shell
$ hadoop fs -df -h /
```

**13. 更改文件复制因子**

```shell
$ hadoop fs -setrep [-R] [-w] <numReplicas> <path>
```

- 更改文件的复制因子。如果 path 是目录，则更改其下所有文件的复制因子
- -w : 请求命令是否等待复制完成

```shell
# 示例
$ hadoop fs -setrep -w 3 /user/hadoop/dir1
```

**14. 权限控制**

```shell
# 权限控制和Linux上使用方式一致
# 变更文件或目录的所属群组。 用户必须是文件的所有者或超级用户。
$ hadoop fs -chgrp [-R] GROUP URI [URI ...]
# 修改文件或目录的访问权限  用户必须是文件的所有者或超级用户。
$ hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
# 修改文件的拥有者  用户必须是超级用户。
$ hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

**15. 文件检测**

```shell
$ hadoop fs -test - [defsz]  URI
```

可选选项：

- -d：如果路径是目录，返回 0。
- -e：如果路径存在，则返回 0。
- -f：如果路径是文件，则返回 0。
- -s：如果路径不为空，则返回 0。
- -r：如果路径存在且授予读权限，则返回 0。
- -w：如果路径存在且授予写入权限，则返回 0。
- -z：如果文件长度为零，则返回 0。

```shell
# 示例
$ hadoop fs -test -e filename
```

## 问题

通常直接修改 **core-site.xml** 等配置后，format 并 restart 后无法生效，需要手动杀掉相关进程，再重新启动

```shell
$ stop-all.sh
$ ps -ef | grep java
$ kill [pid]
$ hadoop namenode -format
$ start-all.sh
```

## 参考

[HDFS Java API](https://github.com/stultuss/God-Of-BigData/blob/master/大数据框架学习/HDFS-Java-API.md)

