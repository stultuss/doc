# CDH

> CDH（Cloudera Distribution Hadoop） 是 Hadoop 的安装套件，由 Cloudera 维护，基于稳定版本的Apache Hadoop构建，版本清晰，更新快，文档全，包含的工具有：HDFS、YARN、ZooKeeper、Oozie、Hive、Hue、HBase、Impala、Solr、Spark、Key-Value Store Indexer

[TOC]

优点：

- 适合新手入门，能够快速搭建 hadoop 集群及其相关工具。
- 拥有 CM 管理工具，可以更方便的管理 hadoop 集群。

## 一、准备 

### 1. 检查环境

```shell
# 硬件检查
$ df -h 	# 空间要求 5G 以上
$ free -m # 内存要求 4G 以上
$ cat /etc/issue # 检查系统版本，CentOS 7.X

# 设置唯一主机名, 一定要注意 hostname 不能包含下划线
$ hostname hadoop01  # 直接修改命令
$ hostname           # 查看修改后的hostname
  
# 修改网络配置文件
$ vim /etc/sysconfig/network
HOSTNAME=hadoop01

# 修改 hosts 文件
$ vim /etc/hosts     
192.168.0.1 hadoop01
192.168.0.2 hadoop02
192.168.0.3 hadoop03
```

#### 虚拟内存设置

需要调整的参数为vm.swappiness，是一个0 - 100的值，用于控制应用数据从物理内存到磁盘上的虚拟内存的交换，值越高，交换越积极，值越小，交换的次数越少

在大多数Linux系统上，该参数默认值为 60，这并不适用于Hadoop集群，因为即使有足够的内存，也可能会进行进程的交换，从而可能会影响Hadoop的性能和稳定性

```shell
# 查看当前该项参数值
$ cat /proc/sys/vm/swappiness

# 修改该项参数值（临时生效）
$ sysctl -w vm.swappiness=0

# （重启后永久生效）
$ echo vm.swappiness = 0 >> /etc/sysctl.conf
```

#### 禁用 tuned 服务

如果您的群集主机正在运行RHEL / CentOS 7.x，请通过运行以下命令禁用”tuned”服务

```shell
# 确保tuned服务已开启
$ systemctl start tuned

# 关闭tuned服务
$ tuned-adm off

# 确保没有已激活的配置, 如果输出内容中包含No current active profile表示关闭成功
$ tuned-adm list

# 关闭并且禁用tuned服务
$ systemctl stop tuned
$ systemctl disable tuned
```

#### 禁用 THP

大多数Linux平台都包含一个名为transparent hugepages的功能，该功能可能会严重降低Hadoop集群的性能

在root权限下

```shell
# 检查THP是否启用, 输出[always] madvise never意味着THP已启用，always madvise [never]意味着THP未启用
$ cat /sys/kernel/mm/transparent_hugepage/enabled
$ cat /sys/kernel/mm/transparent_hugepage/defrag

# 编辑文件，最下面添加两行配置，重启服务器生效
vim /etc/rc.d/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# 赋予/etc/rc.d/rc.local文件可执行权限
chmod +x /etc/rc.d/rc.local

# 修改GRUB配置, 仅RHEL/CentOS 7.x需要进行该项操作
# 在GRUB_CMDLINE_LINUX项目后面添加一个参数
vim /etc/default/grub 
transparent_hugepage=never

# 执行命令
grub2-mkconfig -o /boot/grub2/grub.cfg
```

#### SSH 免密登陆

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
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop01
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop02
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop03
```

### 2. 安装环境

#### MySQL

CDH 对 MySQL 有一些限制

1. MySQL 默认的 datadir 目录是`/var/lib/mysql`，要确保该目录存在的分区有足够的空间。
2. 如果 MySQL 启用了 GTID 复制，会导致 Cloudera Manager 安装失败。
3. 数据库需要使用UTF8编码。对于MySQL和MariaDB，必须使用UTF8编码，而不是utf8mb4
4. 对于MySQL5.7，必须要额外安装 MySQL-shared-compat 或者 MySQL-shared 包，因为 Cloudera Manager Agent 包的安装依赖这两个

```shell
# 首先卸载操作系统可能会自带的mariadb-libs
$ yum -y remove mariadb-libs
$ yum -y install numactl

# 解压mysql rpm-bundle tar包
$ wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
$ tar xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar

# 开始安装mysql, 一定要按照下面的顺序来安装，否则会安装不成功
$ rpm -ivh mysql-community-common-5.7.29-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-5.7.29-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-client-5.7.29-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-server-5.7.29-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-compat-5.7.29-1.el7.x86_64.rpm #（安装Cloudera Manager6需要）

# 启动mysql服务
$ systemctl start mysqld
# 如果无法启动，则需要修改mysql数据目录所有者：chown -R mysql:mysql /var/lib/mysql/

# 查看root用户初始密码
$ grep password /var/log/mysqld.log

# 登录mysql修改root密码
$ mysql -uroot -p
# 如果密码复杂度不够，则会禁止修改，默认密码规则为：包含数字、大小写字母、特殊字符，同时还有长度要求
# 可以通过修改全局参数来解决，但是还是要求密码长度至少为8位
$ mysql> set global validate_password_policy=0;
$ mysql> set password = password('12345678');

# 设置远程登录权限
$ mysql> grant all privileges on *.* to 'root'@'%' identified by '12345678';
$ mysql> flush privileges;

# 修改MySQL数据库默认编码
$ mysql> SHOW VARIABLES LIKE 'char%';可以看到数据库和服务端的编码都还不是utf8：

# 创建CDH的元数据库和⽤户、amon服务的数据库及⽤户
$ mysql> create database cmf DEFAULT CHARACTER SET utf8;
$ mysql> create database amon DEFAULT CHARACTER SET utf8;
$ mysql> grant all on cmf.* TO 'cmf'@'%' IDENTIFIED BY '123456';
$ mysql> grant all on amon.* TO 'amon'@'%' IDENTIFIED BY '123456';
$ mysql> flush privileges;
```

#### JDBC

```shell
$ wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
$ tar zxvf mysql-connector-java-5.1.46.tar.gz
$ sudo mkdir -p /usr/share/java/
$ cd mysql-connector-java-5.1.46
$ sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```

## 二、安装

#### 文件下载

准备 CM 和 CDH 文件

CM6.3.2

- https://archive.cloudera.com/cm6/6.3.1/repo-as-tarball/cm6.3.1-redhat7.tar.gz
- https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/cloudera-manager.repo

ASC文件

- https://archive.cloudera.com/cm6/6.3.1/allkeys.asc

CDH6.3.2

- https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel
- https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1
- https://archive.cloudera.com/cdh6/6.3.2/parcels/manifest.json

#### 安装 JDK

```shell
$ cd /tmp
$ cp ./cloudera-manager.repo /etc/yum.repos.d/
$ cd /etc/yum.repos.d/
$ yum clean all && yum makecache
$ yum install oracle-j2sdk1.8
$ cd /usr/java/jdk1.8.0_181-cloudera/

$ vim /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH:

$ source /etc/profile
```

#### 安装 CM

```shell
# 解压缩
$ mkdir /opt/cloudera-manager
$ cd /tmp
$ tar -xzvf cm6.3.1-redhat7.tar.gz -C /opt/cloudera-manager/

# 安装 RPM
$ cd /opt/cloudera-manager/cm6.3.1/RPMS/x86_64
$ rpm -ivh cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm --nodeps --force
$ rpm -ivh cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm --nodeps --force
$ rpm -ivh cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm --nodeps --force

$ vim /etc/cloudera-scm-agent/config.ini
server_host=hadoop01

# 配置 cm 数据库链接
$ vim /etc/cloudera-scm-server/db.properties
com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=hadoop01:3306
com.cloudera.cmf.db.name=cmf
com.cloudera.cmf.db.user=cmf
com.cloudera.cmf.db.password=1q2w3e4r
com.cloudera.cmf.db.setupType=EXTERNAL

# 初始化 cm 数据库
$ /opt/cloudera/cm/schema/scm_prepare_database.sh mysql cmf cmf
```

#### 安装 httpd 服务

```shell
$ yum install -y httpd
$ cd tmp/
$ mkdir -p /var/www/html/cdh6_parcel
$ mv CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel /var/www/html/cdh6_parcel/
$ mv CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1 /var/www/html/cdh6_parcel/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha
$ mv manifest.json /var/www/html/cdh6_parcel/
$ cd /var/www/html/cdh6_parcel/
$ chown -R cloudera-scm:cloudera-scm /var/www/html/cdh6_parcel/*
$ systemctl start httpd
```

> 浏览器访问 http://129.226.181.95/cdh6_parcel/ 测试

#### 启动 CM 服务

```shell
$ systemctl start cloudera-scm-server
$ systemctl status cloudera-scm-server
$ tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
# 需要等待1分钟，日志中出现7180端口才表示启动完成，才能打开 WEB 界面，
```

打开浏览器，访问地址：http://:7180，默认账号和密码都为 admin，选择免费版安装。

