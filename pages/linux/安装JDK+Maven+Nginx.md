## 安装JDK
#### 下载openJDK
```bash
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel

# 查看安装路径
dirname $(readlink $(readlink $(which java)))

/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64/jre/bin
```
#### 配置环境变量
`vi /etc/profile`

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```
`source /etc/profile`

## 安装Maven
#### 下载Maven
```bash
wget  http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
# 解压并移动文件夹
tar vxf apache-maven-3.6.1-bin.tar.gz
mv apache-maven-3.6.1 /usr/local/maven3.6
```
#### 配置环境变量
`vi /etc/profile`
```bash
export MAVEN_HOME=/usr/local/maven3.6
export PATH=${PATH}:${MAVEN_HOME}/bin
```
`source /etc/profile`

## 安装Nginx
#### 安装必要插件
```bash
yum -y install gcc gcc-c++ automake autoconf libtool make openssl openssl-devel perl pcre-devel zlib-devel tcl libaio
```
> 选定安装目录，本文选择  /usr/local/src
#### 安装PCRE库
```bash
cd /usr/local/src
wget https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz
tar -zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure
make && make install
```
#### 安装zlib库
```bash
cd /usr/local/src
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make && make install
```
#### 安装openssl
```bash
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
tar -zxvf openssl-1.0.1t.tar.gz
cd openssl-1.0.1t
./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib no-shared zlib-dynamic
make && make install
```
> 配置环境变量
>
> ```bash
> export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib64
> echo "export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib64" >> ~/.bashrc
> ```
>
> 验证openssl版本
>
> `openssl version`

#### 安装Nginx

```bash
cd /usr/local/src
wget http://nginx.org/download/nginx-1.13.8.tar.gz
tar -zxvf nginx-1.13.8.tar.gz
cd nginx-1.13.8
./configure
make && make install
```
#### nginx启动、关闭、重启

##### 验证nginx配置文件是否正确

```
/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
```

##### 启动
```bash
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
````
##### 重启

```
/usr/local/nginx/sbin/nginx -s reload
```

##### 关闭
```bash
# 从容停止
ps -ef|grep nginx
kill -QUIT PID

# 强制关闭
pkill -9 nginx
```

#### 设置开机自动启动

1. 进入到/lib/systemd/system/
2. 创建nginx.service文件，并编辑
3. 加入开机自启动

```bash
cd /lib/systemd/system/

vim nginx.service

systemctl enable nginx
```
> nginx.service
```bash
[Unit]
Description=nginx service
After=network.target 
   
[Service] 
Type=forking 
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true 
   
[Install] 
WantedBy=multi-user.target
```

#### Nginx配置
> nginx.conf

```bash
user root;
#nginx进程数，建议设置为等于CPU总核心数
worker_processes  4;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;

events {
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    client_max_body_size 20m;
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    # gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

    server {
        listen       80;
        server_name  web.zgjwlt.com;
        root /app/soft/docker-config/nginx/data/itboys-vue/;
        location / {
            root /app/soft/docker-config/nginx/data/itboys-vue/;
            try_files $uri $uri/ /index.html last;
            index index.html;
        }
    }

    server {
    		#配置80/443共存
    		#listen 80 default backlog=2048;
        listen      443 ssl;
        server_name api.goodluck.tuanxo.com;
        #ssl on;
        ssl_certificate  /usr/local/nginx/conf/vhosts/cert/5727849_api/5727849_api.goodluck.tuanxo.com.pem;
        ssl_certificate_key  /usr/local/nginx/conf/vhosts/cert/5727849_api/5727849_api.goodluck.tuanxo.com.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location ~* \.(txt)$ {
                root /project/goodluck/html;
        }
        location / {
            proxy_pass http://127.0.0.1:9100;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_redirect off;
            proxy_set_header  Host  $host;
            proxy_set_header  X-Real-IP  $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        }

        location ^~/vuejs-admin {
            alias /project/goodluck/html/admin/;
            #index index.html;
            try_files $uri $uri/ @rewrites; 
        }           
        location @rewrites {
            rewrite ^/(vuejs-admin)/(.+)$ /$1/index.html last;
        }
    }
}
```

> 1）location的匹配指令：
>
> ~      #波浪线表示执行一个正则匹配，区分大小写
> ~*    #表示执行一个正则匹配，不区分大小写
> ^~    #^~表示普通字符匹配，不是正则匹配。如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
> =      #进行普通字符精确匹配
> @     #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
>
> 2）location 匹配的优先级(与location在配置文件中的顺序无关)
>
> = 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
> 普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
> ^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。
> 最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。



#### Nginx如果未开启SSL模块，配置Https时提示错误

```
nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:37
```

1. 切换到源码,查看nginx原有的模块

   ```bash
   cd /usr/local/src/nginx-1.11.3
   
   /usr/local/nginx/sbin/nginx -V
   ```

   > 在configure arguments:后面显示的原有的configure参数如下：
   >
   > `--prefix=/usr/local/nginx --with-http_stub_status_module`

2. 运行新的命令即可，等配置完

   ```
   ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
   
   make
   ```

3. 然后将刚刚编译好的nginx覆盖掉原有的nginx,并重新检查

   ```
   cp ./objs/nginx /usr/local/nginx/sbin/
   
   /usr/local/nginx/sbin/nginx -V　
   ```

#### Nginx 配置Http和Https共存

```
server {
    listen 80 default backlog=2048;
    listen 443 ssl;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

		## ssl性能调优
    ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    server_name xxx.abc.com;
    root /var/www/html;
    ssl_certificate /usr/local/Tengine/sslcrt/ wosign.com.crt;
    ssl_certificate_key /usr/local/Tengine/sslcrt/ wosign.com .Key;
    location / {
    		proxy_pass   http://localhost:9100;
        proxy_redirect off;
        proxy_set_header  Host  $host;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    }
}
```



#### 安装Redis6.0

> Redis 6.0.7 对gcc的版本有要求，需要gcc5.3以上。检查当前gcc版本 :  gcc -v
>
> 低于5.3这个版本的，执行下面的命令升级一下
>
> ```
> yum -y install centos-release-scl
>  
> yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
> 
> # 建立会话使用的gcc
> scl enable devtoolset-9 bash
> # scl命令启用只是临时的，新开的会话默认还是原gcc版本。如果要长期使用gcc 9.1的话执行下面的命令即可
> echo -e "\nsource /opt/rh/devtoolset-9/enable" >>/etc/profile
> ```

```
cd /usr/local/src
wget http://download.redis.io/releases/redis-6.0.11.tar.gz
tar -zxvf redis-6.0.11.tar.gz
cd redis-6.0.11

make distclean
make
# make test
make install PREFIX=/usr/local/redis

cp redis.conf ../../redis/

# 修改的redis.conf文件参数
vim /usr/local/redis/redis.conf

#注释 bind 127.0.01          允许外部连接
#修改 protected-mode 为 no   关闭保护模式
#修改 daemonize 为 yes       允许后台运行
#设置密码 requirepass qaz123wsx
##stop-writes-on-bgsave-error no
#设置日志路径 logfile 'redis.log'

# 启动
/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf

# 客户端连接不上redis可能跟linux防火墙有关，将其关闭
systemctl stop firewalld 
systemctl disable firewalld
```

> 去除redis启动日志的warning
>
> ```
> vi /etc/sysctl.conf
> //添加并保存
> vm.overcommit_memory = 1
> net.core.somaxconn= 1024 
> //执行
> sysctl -p 
> ```

##### 配置以systemctl方式启动

```
vim /lib/systemd/system/redis.service

[Unit]
Description=Redis
After=network.target
[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
[Install]
WantedBy=multi-user.target

# 加入开机自启
systemctl enable redis
# 重载服务
systemctl daemon-reload
# 启动
systemctl start redis
# 查看redis状态
systemctl status redis
# 重启
systemctl restart redis
```



#### 安装Mysql8

```
# 安装MySQL源
cd /usr/local/src
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
rpm -Uvh mysql80-community-release-el7-3.noarch.rpm
# 检查是否安装成功
yum repolist enabled | grep "mysql.*-community.*"

# 安装mysql
sudo yum install mysql-community-server

# 修改mysql密码策略和其他配置
vim /etc/my.cnf

# 启动 
sudo systemctl start mysqld
# 查看状态
sudo systemctl status mysqld
# 加入开机启动
systemctl enable mysqld

# 查看mysql root用户的密码
sudo grep 'temporary password' /var/log/mysqld.log
# 登录mysql并修改root密码和权限
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'qaz123wsx';
# 允许远程连接
CREATE USER 'root'@'%' IDENTIFIED BY 'qaz123wsx';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
# 设置加密方式，是SQLyog可以链接
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'qaz123wsx';

#5.6/5.7
#创建用户并授权
create user zhangsan identified by 'zhangsan';
grant all privileges on zhangsanDb.* to zhangsan@'%' identified by 'zhangsan';
flush  privileges;
#修改密码
update mysql.user set password = password('zhangsannew') where user = 'zhangsan' and host = '%';
flush privileges;

# 卸载
yum -y remove mysql*
# 安装包下载地址
wget http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-8.0.26.tar.gz

```

##### mysql通用配置 /etc/my.cnf

```
[mysqld]
port=3306

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

server_id = 3306
character_set_server = UTF8MB4
skip_name_resolve = 1
default_time_zone = "+8:00"

lock_wait_timeout = 3600
open_files_limit = 65535
back_log = 1024
max_connections = 512
max_connect_errors = 1000000
table_open_cache = 1024
table_definition_cache = 1024
thread_stack = 512K
sort_buffer_size = 16M
join_buffer_size = 16M
read_buffer_size = 8M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 64M
thread_cache_size = 768
interactive_timeout = 600
wait_timeout = 600
tmp_table_size = 96M
max_heap_table_size = 96M

validate_password.check_user_name = 0
validate_password.policy = 0
validate_password.mixed_case_count = 0
validate_password.number_count = 0
validate_password.special_char_count = 0
validate_password.length = 0

[mysql]
default-character-set=utf8mb4
[client]
port=3306
default-character-set=utf8mb4
[mysqldump]
quick
```



