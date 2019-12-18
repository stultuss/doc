PHP
=========================
> 记录日常工作中的一些问题

## 环境安装与优化

### PHP 7.1.17

通过 rpm 安装

```shell
# 安装 PHP / FPM / 扩展
$ rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
$ yum -y install epel-release php71w php71w-fpm install php71w-mbstring php71w-common php71w-gd php71w-mcrypt php71w-pecl-apcu php71w-mysqlnd php71w-xml php71w-cli php71w-devel php71w-pecl-memcached php71w-pecl-redis php71w-opcache
$ php -v

PHP 7.1.33 (cli) (built: Oct 26 2019 10:16:23) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies

# 安装 pecl
$ curl -o go-pear.php https://pear.php.net/go-pear.phar
$ php go-pear.php
$ pecl -V

PEAR Version: 1.10.10
PHP Version: 7.1.33
Zend Engine Version: 3.1.0
Running on: Linux VM_0_28_centos 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64
```

检查扩展

```shell
$ php -m

[PHP Modules]
bz2
calendar
Core
ctype
curl
date
dba
dom
exif
fileinfo
filter
ftp
gd
gettext
gmp
hash
iconv
igbinary
json
libxml
mbstring
mcrypt
memcached
msgpack
mysqli
mysqlnd
openssl
pcntl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
posix
readline
redis
Reflection
session
shmop
SimpleXML
sockets
SPL
sqlite3
standard
sysvmsg
sysvsem
sysvshm
tokenizer
wddx
xml
xmlreader
xmlwriter
xsl
Zend OPcache
zip
zlib

[Zend Modules]
Zend OPcache
```

大概会缺少以下几个扩展和配置，先修改 php.ini

```shell
$ pecl install msgpack

## 添加 msgpack 扩展
$ vim /etc/php.d/msgpack.ini
; Enable msgpack extension module
extension=msgpack.so

$ systemctl restart php-fpm
```

比较 php.ini

```shell
$ egrep -v "^$|;" /etc/php.ini

[PHP]
engine = On
short_open_tag = Off
precision = 14
output_buffering = 4096
zlib.output_compression = Off
implicit_flush = Off
unserialize_callback_func =
serialize_precision = 17
disable_functions =
disable_classes =
zend.enable_gc = On
expose_php = On
max_execution_time = 30
max_input_time = 60
memory_limit = 128M
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
display_errors = Off
display_startup_errors = Off
log_errors = On
log_errors_max_len = 1024
ignore_repeated_errors = Off
ignore_repeated_source = Off
report_memleaks = On
track_errors = Off
html_errors = On
variables_order = "GPCS"
request_order = "GP"
register_argc_argv = Off
auto_globals_jit = On
post_max_size = 8M
auto_prepend_file =
auto_append_file =
default_mimetype = "text/html"
default_charset = "UTF-8"
doc_root =
user_dir =
enable_dl = Off
file_uploads = On
upload_max_filesize = 2M
max_file_uploads = 20
allow_url_fopen = On
allow_url_include = Off
default_socket_timeout = 60
[CLI Server]
cli_server.color = On
[Date]
date.timezone = "Asia/Shanghai"
[filter]
[iconv]
[intl]
[sqlite]
[sqlite3]
[Pcre]
[Pdo]
[Pdo_mysql]
pdo_mysql.cache_size = 2000
pdo_mysql.default_socket=
[Phar]
[mail function]
sendmail_path = /usr/sbin/sendmail -t -i
mail.add_x_header = On
[SQL]
sql.safe_mode = Off
[ODBC]
odbc.allow_persistent = On
odbc.check_persistent = On
odbc.max_persistent = -1
odbc.max_links = -1
odbc.defaultlrl = 4096
odbc.defaultbinmode = 1
[Interbase]
ibase.allow_persistent = 1
ibase.max_persistent = -1
ibase.max_links = -1
ibase.timestampformat = "%Y-%m-%d %H:%M:%S"
ibase.dateformat = "%Y-%m-%d"
ibase.timeformat = "%H:%M:%S"
[MySQLi]
mysqli.max_persistent = -1
mysqli.allow_persistent = On
mysqli.max_links = -1
mysqli.cache_size = 2000
mysqli.default_port = 3306
mysqli.default_socket =
mysqli.default_host =
mysqli.default_user =
mysqli.default_pw =
mysqli.reconnect = Off
[mysqlnd]
mysqlnd.collect_statistics = On
mysqlnd.collect_memory_statistics = Off
[OCI8]
[PostgreSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
[bcmath]
bcmath.scale = 0
[browscap]
[Session]
session.save_handler = files
session.use_strict_mode = 0
session.use_cookies = 1
session.use_only_cookies = 1
session.name = PHPSESSID
session.auto_start = 0
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_domain =
session.cookie_httponly =
session.serialize_handler = php
session.gc_probability = 1
session.gc_divisor = 1000
session.gc_maxlifetime = 1440
session.referer_check =
session.cache_limiter = nocache
session.cache_expire = 180
session.use_trans_sid = 0
session.hash_function = 0
session.hash_bits_per_character = 5
url_rewriter.tags = "a=href,area=href,frame=src,input=src,form=fakeentry"
[Assertion]
zend.assertions = -1
[mbstring]
[gd]
[exif]
[Tidy]
tidy.clean_output = Off
[soap]
soap.wsdl_cache_enabled=1
soap.wsdl_cache_dir="/tmp"
soap.wsdl_cache_ttl=86400
soap.wsdl_cache_limit = 5
[sysvshm]
[ldap]
ldap.max_links = -1
[mcrypt]
[dba]
[curl]
[openssl]

## 重启
$ systemctl restart php-fpm
```

检查配置

```shell
$ egrep -v "^$|;" /etc/php-fpm.conf

pid = /var/run/php-fpm/php-fpm.pid
error_log = /var/log/php-fpm/error.log
daemonize = yes
include=/etc/php-fpm.d/*.conf

$ grep 'core id' /proc/cpuinfo | sort -u | wc -l
$ grep 'processor' /proc/cpuinfo | sort -u | wc -l
$ dmidecode -t memory | grep Size: | grep -v "No Module Installed"

# 开发服 2 核 2 G，开启 dynamic，默认启动 5 个 fpm，最高 35 个
# 正式服 4 核 8 G，开启 static，直接启动 200 个 fpm （每个 fpm 最高占用约 30M）即4核情况下，Nginx 的每个 work 平均 1 秒最高可以调用 50 个 fpm（PHP接口执行 20 ms）
$ egrep -v "^$|;" /etc/php-fpm.d/www.conf

[www]
user = nobody
group = nobody
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
listen.backlog = 4096

pm = dynamic
pm.max_children = 200
pm.max_requests = 600
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache

## 重启
$ systemctl restart php-fpm

# Apache Benchmark 测试 QPS
$ yum -y install httpd-tools
$ ab -c 100 -n 10000 "http://127.0.0.1/monitor"
```

### Nginx

通过 rpm 安装

```shell
## 安装
$ rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
$ yum install -y nginx

$ systemctl enable nginx # 加入开机启动
$ systemctl start nginx # 启动服务
```

检查配置

```shell
$ vim /etc/nginx/nginx.conf

worker_processes auto; # 设置值和CPU核心数一致
worker_rlimit_nofile 51200;

user nobody;
pid        /var/run/nginx.pid;
error_log  /var/log/nginx/error.log warn;

events {
 		use epoll;
    worker_connections 51200;
}

http {
		include       /etc/nginx/mime.types;
    #default_type  application/octet-stream;
    default_type  text/html;

    log_format main  '$host | $server_addr | $http_x_forwarded_for | $remote_addr | $remote_user | $time_local | $request | $status | $body_bytes_sent '
    								 '| $http_referer | $http_user_agent | $upstream_addr | $upstream_status | $upstream_response_time | $request_time';

    log_format aly   '$remote_addr - $remote_user [$time_local] "$request" '
                     '$request_time $request_length '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" ' 
                     '"$http_x_forwarded_proto" "$host"';

    log_format json '{"@timestamp":"$time_iso8601",'
    								'"clientip":"$http_x_forwarded_for",'
                    '"url":"$uri",'
                    '"responsetime":$request_time,'
                    '"status":"$status",'
                    '"referer":"$http_referer",'
                    '"agent":"$http_user_agent"}';

#    add_header Access-Control-Allow-Origin *;
#    add_header Access-Control-Allow-Headers X-Requested-With;
#    add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

    client_max_body_size            64m;
    client_header_buffer_size       32k;
    #client_body_buffer_size         64m;
    
    map_hash_bucket_size            64;
    types_hash_bucket_size          64;
    variables_hash_bucket_size      128;
    server_names_hash_bucket_size   128;
    server_name_in_redirect         off;
    sendfile    on;
    tcp_nopush  on;
    tcp_nodelay on;
    server_tokens       off;
    keepalive_timeout   15;
    client_body_timeout 3600;
    client_header_timeout 3600;

		gzip on;
    gzip_min_length     1k;
    gzip_buffers        4 16k;
    gzip_http_version   1.0;
    gzip_comp_level     2;
    gzip_types      text/plain application/x-javascript text/css application/xml;
    gzip_vary           on;
    ssi on;
    ssi_silent_errors on;
    ssi_types text/shtml;
    charset  utf-8;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_buffer_size 128k;
    proxy_buffers 4 256k;
    proxy_busy_buffers_size 256k;
    proxy_temp_file_write_size 256k;
    proxy_connect_timeout 3600;
    
    include /etc/nginx/conf.d/*.conf;
}

$ vim /etc/nginx/conf.d/default.conf

server {
    listen      80;
    server_name localhost;
    root   /opt/server;
    index  index.php index.html index.htm;
    access_log off;
    error_log  /var/log/error.log warn;

    location / {
    	try_files $uri $uri/ /index.php$is_args$args;
    }
    
    location ~ \.php$ {
      try_files $uri = 404;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
      include        fastcgi_params;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

$ systemctl restart nginx # 启动服务
```

### MySQL 5.6

通过 rpm 安装

```shell
# 安装
$ wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
$ rpm -ivh mysql-community-release-el6-5.noarch.rpm
$ yum repolist enabled | grep "mysql.*-community.*"
# 正式服只需要安装客户端
$ yum -y install mysql-community-client
# 开发服则需要安装服务端 
$ yum -y install mysql-community-server 
$ systemctl enable mysqld # 加入开机启动
$ systemctl start mysqld # 启动服务或 service mysqld start

# 改密码
$ mysql -uroot
$ mysql -uroot -p
```

检查配置

```shell
$ egrep -v "^$|#" /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

$ systemctl start mysqld
```

修改密码

```mysql
# 切换数据库
USE mysql 
# 修改密码
UPDATE user SET password=PASSWORD("******") WHERE user='root';
# 更改作用域
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '******' WITH GRANT OPTION;
# 刷新权限
FLUSH PRIVILEGES;
```

导入数据表

```shell
$ cd /opt/server/mysql/
$ mysql -uroot -p
```

```mysql
mysql> create database db_test;
mysql> use db_test;
mysql> set names utf8;
mysql> source /opt/server/mysql/schema.sql;
mysql> quit;
```

### Redis

通过 yum 安装

```shell
$ yum -y install redis
$ redis-cli -v
$ redis-server /etc/redis.conf
```

检查配置

```shell
$ egrep -v "^$|#" /etc/redis.conf

bind 127.0.0.1
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile /var/log/redis/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /var/lib/redis
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

### Golang

通过安装包安装

```shell
$ wget https://studygolang.com/dl/golang/go1.10.1.linux-amd64.tar.gz
$ sudo tar -C /usr/local/ -zxf go1.10.1.linux-amd64.tar.gz

## 修改环境变量
$ vim /etc/profile

export GO_HOME=/usr/local/go
export GOPATH=/opt/go/
export PATH=$PATH:$GO_HOME/bin:$GOPATH/bin

## 生效
$ source /etc/profile
$ go env
```

### Node.js

安装指定版本 Node.js

```shell
$ yum -y install npm

$ wget https://nodejs.org/dist/v8.10.0/node-v8.10.0-linux-x64.tar.gz
$ tar -C /usr/local/ -zxf node-v8.10.0-linux-x64.tar.gz
$ cd /usr/local/
$ rm -rf /usr/bin/node
$ ln -s node-v8.10.0-linux-x64/ node
$ ln -s /usr/local/node/bin/node /usr/bin/node


## 修改环境变量
$ vim /etc/profile

export NODE_HOME=/usr/local/node
export PATH=$PATH:$NODE_HOME/bin

## 生效
$ source /etc/profile
$ node -v

v8.10.0
```

### Java

安装 java SE 环境

```bash
# https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
$ cd /opt/lib/downloads
$ tar -C /opt/lib -zxvf jdk-8u231-linux-x64.tar.gz
$ vim /etc/profile

export JAVA_HOME=/opt/lib/jdk1.8.0_231
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin

## 生效
$ source /etc/profile
$ java -version

java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```

## 其他工具

### phpMyAdmin

```shell
$ wget https://files.phpmyadmin.net/snapshots/phpMyAdmin-4.8+snapshot-all-languages.tar.gz
$ tar -xvf phpMyAdmin-4.8+snapshot-all-languages.tar.gz
$ mv phpMyAdmin-4.8+snapshot-all-languages phpmyadmin
```

更改配置

```shell
$ cd phpmyadmin
$ cp config.sample.inc.php config.inc.php
$ vi config.inc.php

$ chown -R nobody.nobody /var/lib/php/session
$ chmod -R 755 /var/lib/php/session
```

```php
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = '127.0.0.1';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;
```

### phpRedisAdmin

```shell
$ curl -s http://getcomposer.org/installer | php
$ php composer.phar create-project -s dev erik-dubbelboer/php-redis-admin path/to/install
$ git clone https://github.com/ErikDubbelboer/phpRedisAdmin.git
$ cd phpRedisAdmin
$ git clone https://github.com/nrk/predis.git vendor
```

