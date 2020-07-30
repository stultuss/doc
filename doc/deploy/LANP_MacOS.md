# MacOS 下部署 PHP 及其开发环境

### PHP & Apache

默认已经安装 php ，但需要进行相关配置

```shell
$ sudo cp /etc/php-fpm.conf.default /etc/php-fpm.conf
$ egrep -v "^$|;" /etc/php-fpm.conf

pid = /usr/local/var/run/php-fpm/php-fpm.pid
error_log = /usr/local/var/log/php-fpm/error.log
daemonize = yes
include=/private/etc/php-fpm.d/*.conf

$ egrep -v "^$|;" /etc/php-fpm.d/www.conf

[www]
user = root
group = wheel
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
listen.backlog = 4096

pm = dynamic
pm.max_children = 200
pm.max_requests = 600
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

slowlog = /usr/local/var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /usr/local/var/log/php-fpm/www-error.log

## 重启
$ killall php-fpm
$ sudo php-fpm
```

### Nginx

通过 brew 安装

```shell
## 安装
$ brew search nginx
$ brew install nginx

# 默认路径
/usr/local/Cellar/nginx/<version>
/usr/local/etc/nginx/nginx.conf
/usr/local/var/www

$ brew services start nginx # 加入开机启动并启动服务
```

检查配置

```shell
$ vim /usr/local/etc/nginx/

worker_processes  1;
error_log  /usr/local/var/log/nginx/error.log warn;
pid        /usr/local/var/run/nginx.pid;

events {
    worker_connections  256;
}

http {
    include       		mime.types;
    default_type  		application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    sendfile        	on;
    include servers/*;
}

$ vim /usr/local/etc/nginx/servers/default.conf

server {
    listen              80;
    server_name         localhost;
    root                /Users/mars/Project/php/game;
    index               index.php server.php index.html index.htm;    
    error_log           /usr/local/var/log/nginx/default-error.log warn;
    
    location / {
        try_files       $uri $uri/ /index.php$is_args$args /server.php$is_args$args;
    }
    
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}

$ brew services restart nginx # 启动服务
```

### MySQL 5.6

通过 brew 安装

```shell
# 安装
$ brew clearup
$ brew install mysql
$ brew services start mysql # mysql.server start

# 默认路径
/etc/my.cnf
/etc/mysql/my.cnf
/usr/local/etc/my.cnf
~/.my.cnf 
$ cp /usr/local/etc/my.cnf.default /etc/my.cnf

# 改密码
$ mysql -uroot
$ mysql -uroot -p
```

检查配置

```shell
$ egrep -v "^$|#" /etc/my.cnf

[client]
socket=/usr/local/var/mysql/mysql.sock

[mysqld]
bind-address = 127.0.0.1
mysqlx-bind-address = 127.0.0.1

datadir=/usr/local/var/mysql
socket=/usr/local/var/mysql/mysql.sock
symbolic-links=0
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
[mysqld_safe]
log-error=/usr/local/var/log/mysqld.log
pid-file=/usr/local/var/run/mysqld.pid

$ brew services restart mysql
```

修改密码

```mysql
# 切换数据库
USE mysql 

# 修改密码
UPDATE user SET password=PASSWORD("123456") WHERE user='root';
# 更改作用域
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
# MYSQL 8.0 以上
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'qx#Xim!0Euhwc';

# 刷新权限
FLUSH PRIVILEGES;
```

### Redis

通过 brew 安装

```shell
$ brew install redis
$ redis-cli -v
$ redis-server /usr/local/etc/redis.conf
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
