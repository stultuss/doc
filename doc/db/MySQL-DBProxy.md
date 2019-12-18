# DBProxy 部署文档

[TOC]

## 总结

优点：无代码侵入，无需 ORM 配置分表分库数量。

缺点：必须事先确定好分表分库数量，后期变更成本巨大。

## 安装

### FPNN

官方文档：https://github.com/highras/fpnn

检查依赖：

- gcc
- g++
- libcurl
- tcmalloc
- openssl

初始化 FPNN 编译环境：

```shell
$ yum -y groupinstall 'Development tools'
$ yum -y install libcurl-devel openssl-devel gperftools mysql-devel

# 检查 php 是否使用的是 mysql 还是 mysqlnd 模块, 如果是 mysql 则移除，并安装 php-mysqlnd
$ php -m | grep mysql
$ yum -y remove php-mysql
$ yum -y install php-mysqlnd
```

编译 FPNN 框架

```shell
$ cd /opt/lib/downloads/dbproxy/
$ wget https://github.com/highras/fpnn/archive/v0.8.1.tar.gz
$ tar -zxvf fpnn-0.8.1.tar.gz

$ mkdir -p /opt/lib/dbproxy
$ mv fpnn-0.8.1 /opt/lib/dbproxy/fpnn

# 坑1: 必须是 fpnn 这个文件夹名...
# 坑2: 必须指定版本，否则后面的工具依赖会报错，比如某个变量名变更了。
$ cd /opt/lib/dbproxy/fpnn
$ make
```

### DBProxy

官方文档：https://github.com/highras/dbproxy

编译 DBProxy 工具

```shell
$ cd /opt/lib/downloads/dbproxy/
$ wget https://github.com/highras/dbproxy/archive/v2.5.4.tar.gz
$ tar -zxvf dbproxy-2.5.4.tar.gz

$ mv dbproxy-2.5.4 /opt/lib/dbproxy/dbproxy
$ cd /opt/lib/dbproxy/dbproxy
$ make
$ make deploy

$ vim /etc/profile

export PATH=$PATH:/opt/lib/dbproxy/deployment/dbproxy/tools

$ source /etc/profile
```

安装 supervisor

```shell
$ yum -y install supervisor 
$ systemctl enable supervisord.service
$ systemctl start supervisord.service

$ mkdir /var/log/supervisord
```

配置 DBProxy

```shell
# 修改 DBProxy 全局配置
$ vim /opt/lib/dbproxy/deployment/dbproxy/conf/DBProxy.conf

FPNN.server.idle.timeout = 1800
DBProxy.ConfigureDB.host = 127.0.0.1
DBProxy.ConfigureDB.port = 3306
DBProxy.ConfigureDB.timeout = 3
DBProxy.ConfigureDB.databaseName = dbproxy
DBProxy.ConfigureDB.username = root
DBProxy.ConfigureDB.password = 123456

# 创建 dbproxy.ini
$ vim /etc/supervisord.d/dbproxy.ini

[program:dbproxy]
directory=/opt/lib/dbproxy/deployment/dbproxy
command=/opt/lib/dbproxy/deployment/dbproxy/bin/DBProxy /opt/lib/dbproxy/deployment/dbproxy/conf/DBProxy.conf
priority=1
numprocs=1
autostart=true
autorestart=true
stderr_logfile=/var/log/supervisord/dbproxy-err.log
stdout_logfile=/var/log/supervisord/dbproxy-out.log

$ supervisorctl update
$ supervisorctl start dbproxy

$ netstat -tunlp|grep 12321
```

配置 DBProxyMgr

```shell
$ cd /opt/lib/dbproxy/deployment/dbproxy/conf/
$ cp DBProxy.conf DBProxyMgr.conf
$ sed -i 's/FPNN.server.listening.port = 12321/FPNN.server.listening.port = 12322/g' DBProxyMgr.conf
$ sed -n '/FPNN.server.listening.port/p' DBProxyMgr.conf

# 修改 DBProxyMgr 全局配置
$ vim DBProxyMgr.conf

FPNN.server.idle.timeout = 1800
DBProxy.ConfigureDB.host = 127.0.0.1
DBProxy.ConfigureDB.port = 3306
DBProxy.ConfigureDB.timeout = 3
DBProxy.ConfigureDB.databaseName = dbproxy
DBProxy.ConfigureDB.username = root
DBProxy.ConfigureDB.password = 123456

# 创建 dbproxymgr.ini
$ vim /etc/supervisord.d/dbproxymgr.ini

[program:dbproxymgr]
directory=/opt/lib/dbproxy/deployment/dbproxy
command=/opt/lib/dbproxy/deployment/dbproxy/bin/DBProxyMgr /opt/lib/dbproxy/deployment/dbproxy/conf/DBProxyMgr.conf
priority=1
numprocs=1
autostart=true
autorestart=true
stderr_logfile=/var/log/supervisord/dbproxymgr-err.log
stdout_logfile=/var/log/supervisord/dbproxymgr-out.log

$ supervisorctl update
$ supervisorctl start dbproxymgr
$ supervisorctl status

$ netstat -tunlp|grep 12322
```

DBProxy.conf & DBProxyMgr.conf：[配置说明](https://github.com/highras/dbproxy/blob/master/doc/zh-cn/DBProxy-Configurations.md)

### TableCache

官方文档：https://github.com/highras/tableCache

编译 tableCache 工具

```shell
$ cd /opt/lib/downloads/dbproxy/
$ wget https://github.com/highras/tableCache/archive/v0.1.2.tar.gz
$ tar -zxvf tableCache-0.1.2.tar.gz

$ mv tableCache-0.1.2 /opt/lib/dbproxy/tableCache
$ cd /opt/lib/dbproxy/tableCache
$ make
$ make deploy
```

## 部署

### 基础配置

使用 DBDeploy 操作，基础命令如下：

```shell
$ cd /opt/lib/dbproxy/deployment/dbproxy/tools

# 交互模式, 通过 help 了解语法
$ DBDeployer 127.0.0.1 3306 root 'root'

# 脚本模式
$ DBDeployer 127.0.0.1 3306 root 'root' /opt/lib/downloads/init_dbproxy.sql
```

初始化配置库: init_dbproxy.sql

```sql
create config database <database_name>
config DBProxy account <user_name> <password> [options]
config business account <user> <password> [options]
add master instance <host> <port> [timeout_in_sec]
show master instances
add slave instance <host> <port> <master_server_id> [timeout_in_sec]
grant config database
add deploy server <master_server_id>
add hash table from <sql_file_path> <table_name> <table_count> [with hintId field <fiedl_name>] [to cluster <cluster_name>] in database <basic_database_name> <database_count>
update config time
```

更新配置库: update_dbproxy.sql

```sql
load config database <database_name>
show master instances
add deploy server <master_server_id>
add hash table from <sql_file_path> <table_name> <table_count> [with hintId field <fiedl_name>] [to cluster <cluster_name>] in database <basic_database_name> <database_count>
update config time
```

**注意**

- 建议使用 hash 分库分表：hash ｜ 区段

> 因为区段分库在运行一段时间后，会导致数据库压力和热点不均匀，因此强烈建议使用 hash 分库分表。

- 创建数据表要使用工具检查

> 对于新创建的数据表，请使用 DBTableChecker 或 DBTableStrictChecker 进行检查。

- 新增数据表生效

> 1、请使用 DBDeployer 的update config time  命令，更新配置库更新时间。
>
> 2、等待配置项 DBProxy.ConfigureDB.checkInterval(默认900秒) 指定的时间过后，DBProxy 自动加载新的数据表。或使用 DBRefresher 强制每个 DBProxy 立刻加载新的数据表。(DBRefresher 127.0.0.1:12321 )

### 基础部署

设计三张表：user，game，system

| db_name | hintId | Shard_count | memo           |
| ------- | ------ | ----------- | -------------- |
| user    | uid    | 10          | 用户表         |
| game    | uid    | 10          | 游戏表         |
| system  | k      | 1           | 系统表，不分表 |

/opt/lib/downloads/schema.sql

```sql
DROP TABLE IF EXISTS user;
CREATE TABLE user (
  uid int(11) unsigned NOT NULL COMMENT '玩家ID',
  extra varchar(255) NOT NULL DEFAULT '{}' COMMENT '玩家数据',
  PRIMARY KEY (uid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS game;
CREATE TABLE game (
  uid int(11) unsigned NOT NULL COMMENT '玩家ID',
  extra varchar(255) NOT NULL DEFAULT '{}' COMMENT '游戏数据',
  PRIMARY KEY (uid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS system;
CREATE TABLE system (
  k int(11) unsigned NOT NULL COMMENT '键',
  v varchar(255) NOT NULL DEFAULT '' COMMENT '值',
  PRIMARY KEY (k)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 初始化数据库

通过 DBDeployer 部署数据库

```shell
# 初始化
$ DBDeployer 127.0.0.1 3306 root 'root' /opt/server/mysql/init_dbproxy

# 完成 deploy 后，会创建三个数据库：
# db_test_dbproxy (system)
# db_test_dbproxy_0 (user_0, user_2, user_4, user_6, user_8)
# db_test_dbproxy_1 (user_1, user_3, user_5, user_7, user_9)
```

/opt/server/mysql/init_dbproxy

```
create config database dbproxy
config business account root 123456
add master instance 127.0.0.1 3306
show master instances

load config db dbproxy
add deploy server 1
show deploy servers
add hash table from /opt/server/mysql/schema.sql system 1 with hintId field k in database db_test_dbproxy 1
clear deploy servers

load config db dbproxy
add deploy server 1
show deploy servers
add hash table from /opt/server/mysql/schema.sql user 10 with hintId field uid in database db_test_dbproxy 2

update config time
```

**注：DBProxy 不支持弹性扩容，一旦决定了分表和分库的数量，后期变更分库分表数量成本巨大！**

> 只能通过复制表到新的表上，并 **手动修改** dbproxy 里相应的分表分库的配置结构：
>
> 1. 在 `server_info` 中添加新的数据库信息【不添加新数据库地址的情况下，可忽略】
> 2. 在 `table_info` 中更新的表的 `table_count` 的数量
> 3. 在 `split_table_info` 中添加并更新 `table_num` 和 `database_name`

#### 添加数据库

如果在此基础上要添加新的表，则创建一个新的 script, 直接 `load config db sys_dbproxy` 即可。

```shell
# 添加数据库 game
$ DBDeployer 127.0.0.1 3306 root 'root' /opt/server/mysql/patch/add_dbproxy_20191217
# 检查一致性
$ DBTableStrictChecker 127.0.0.1 12322 game ''
```

/opt/server/mysql/patch/add_dbproxy_20191217

```
load config db dbproxy
config business account root 123456

add deploy server 1
show deploy servers
add hash table from /opt/server/mysql/schema.sql game 10 with hintId field uid in database db_test_dbproxy 2

update config time
```

#### 添加字段  & 修改字段 

使用 DBParamsQuery 添加和修改数据表字段

``` bash
$ DBParamsQuery -h

Usage:
        ./DBParamsQuery [-h host] [-p port] [-t table_name] [-timeout timeout_seconds] <hintId | -i int_hintIds | -s string_hintIds> sql [param1 param2 ...]

        Notes: host default is localhost, and port default is 12321.
```

 DBParamsQuery 使用配置库里面的账号去操作表结构，默认权限不足，解决方案有两种：

1. 可以复制一个配置库，用有权限的业务库账号替换现有的业务库账号，作为 DBA 的配置库。然后 DBProxyManager 指向这个配置库（推荐）
2. 临时给业务库的账号 alter 权限，处理完后再删除该权限

```shell
#!/bin/env bash

mysqldump -udump -p'123456' -h127.0.0.1 dbproxy --single-transaction --opt > dbproxy.sql
mysql -uroot -p123456 -h127.0.0.1 -e "CREATE DATABASE dbproxymgr;"
mysql -uroot -p123456 -h127.0.0.1 dbproxymgr < dbproxy.sql
mysql -uroot -p123456 -h127.0.0.1 -e "UPDATE dbproxymgr.server_info SET user='dba';"

sed -i 's/DBProxy.ConfigureDB.databaseName = dbproxy/DBProxy.ConfigureDB.databaseName = dbproxymgr/g' DBProxyMgr.conf

# 重启 DBProxyMgr
supervisorctl status
supervisorctl restart DBProxyMgr

DBParamsQuery -h 127.0.0.1 -p 12322 -i '' 'DESC user;'
DBParamsQuery -h 127.0.0.1 -p 12322 -i '' "ALTER TABLE user ADD COLUMN createTime int(10)  unsigned NOT NULL COMMENT '创建时间' AFTER extra;"

# 检查一致性
DBTableStrictChecker 127.0.0.1 12322 user ''

Query interface 'splitInfo' ...
----------------------------------
Split type: Hash
Table Count: 10
Split Hint: uid
----------------------------------
Query interface 'allSplitHintIds' ...
... checking sub-table 0 by hintId: 0 ... [OK]
... checking sub-table 1 by hintId: 1 ... [OK]
... checking sub-table 2 by hintId: 2 ... [OK]
... checking sub-table 3 by hintId: 3 ... [OK]
... checking sub-table 4 by hintId: 4 ... [OK]
... checking sub-table 5 by hintId: 5 ... [OK]
... checking sub-table 6 by hintId: 6 ... [OK]
... checking sub-table 7 by hintId: 7 ... [OK]
... checking sub-table 8 by hintId: 8 ... [OK]
... checking sub-table 9 by hintId: 9 ... [OK]
----------------------------------
All sub-tables are OK!

time cost 5.692 ms
```

#### 查找数据库

```shell
# 查找数据库
$ DBQuery 127.0.0.1 12321 35946507482475 user "select * from user"
```

# FAQ

### 1. 报错信息

```
问：
ERROR ANSWER: code(20003), exception(no msg, please refer to log.:)), QUEST(Quest, seqID(1),TCP(1),Method(sQuery),body({"hintIds":[], "sql":"ALTER TABLE user_statistics MODIFY COLUMN extraValue text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL;"}))
Query error!

答：
错误代码20003 是超时 code(20003)，修改表结构的话，因为原始表数据量可能不小，所以可能执行时间很长。我们这有过并发修改，但也超过45分钟的情况,就是因为表的数据太多了

这时候，需要修改两个地方
- 修改操作用的客户端，DBQuery 或者 DBParamsQuery 的超时时间，默认是5秒 建议修改为 3600 秒
- DBProxyMgr的 idle 时间，默认60秒 设置为0，就是无限等待

如果 DBProxyMgr 没有退出，操作会继续执行，直到完成。20003 的错误就可以先忽略。然后看数据库压力下来，或者差不多的时候，用 DBTableStrictChecker 检查表结构是否已经修改一致，如果有部分不一致，可以等5～10分钟后，再看看，是否新的结果显示部分不一致的数量在减少。如果在减少，那就说明alert还在继续。等待，知道检查结果显示为全部一致即可
```

### 2. [运维管理](https://github.com/highras/dbproxy/blob/master/doc/zh-cn/DBProxy-Operations.md)

### 3. [账号与安全](https://github.com/highras/dbproxy/blob/master/doc/zh-cn/DBProxy-Accounts-Security.md)

## 其他

### 删库工具

```shell
#!/bin/env bash
MYSQL_HOST='127.0.0.1'
MYSQL_USER='root'
MYSQL_PASSWD='123456'
MYSQL_DBNAME='db_test_dbproxy'

mysql -h$MYSQL_HOST -u$MYSQL_USER -p$MYSQL_PASSWD -e "DROP DATABASE IF EXISTS dbproxy;"
mysql -h$MYSQL_HOST -u$MYSQL_USER -p$MYSQL_PASSWD -e "DROP DATABASE IF EXISTS $MYSQL_DBNAME;"

for i in {0..1}
do
DB_NAME=`printf '%0s_%1d' $MYSQL_DBNAME $i`
mysql -h$MYSQL_HOST -u$MYSQL_USER -p$MYSQL_PASSWD -e "DROP DATABASE IF EXISTS $DB_NAME;"
done

mysql -h$MYSQL_HOST -u$MYSQL_USER -p$MYSQL_PASSWD -e "SHOW DATABASES;"
```
