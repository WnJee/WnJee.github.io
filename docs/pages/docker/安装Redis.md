> 在服务器创建配置文件目录如下,并cd到 **docker-config**目录下<br/>
```
docker-config
└───redis
   |
   └───conf
   |
   └───data
```
## 安装单机版Redis
#### 下载redis镜像
`docker pull redis:6.2`

#### 添加redis配置文件
`vi redis/conf/redis.conf`

```
# Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
# 启用守护进程后，Redis会把pid写到一个pidfile中，在/var/run/redis.pid
# daemonize yes
#设置进程锁文件
pidfile /var/run/redis.pid
#端口
port 6379
#密码
requirepass qaz123wsx
# 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 60000
# 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
# debug (很多信息, 对开发／测试比较有用)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel verbose
# 日志记录方式，默认为标准输出，如果配置为redis为守护进程方式运行，而这里又配置为标准输出，则日志将会发送给/dev/null
logfile /data/logs
# 设置数据库的数量，默认数据库为0，可以使用select <dbid>命令在连接上指定数据库id
# dbid是从0到‘databases’-1的数目
databases 16
##指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
#save <seconds> <changes>
#Redis默认配置文件中提供了三个条件：
save 900 1
save 300 10
save 60 10000
#指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，
#可以关闭该#选项，但会导致数据库文件变的巨大
rdbcompression yes
#指定本地数据库文件名
dbfilename dump.rdb
#指定本地数据库路径
dir ./
#指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能
#会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有
#的数据会在一段时间内只存在于内存中
appendonly yes
#指定更新日志条件，共有3个可选值：
#no：表示等操作系统进行数据缓存同步到磁盘（快）
#always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
#everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec
#禁用部分redis命令
#rename-command FLUSHALL ""
#rename-command CONFIG   ""
#rename-command EVAL     ""
#禁止外网访问 Redis
#bind 127.0.0.1
#protected-mode no

#
stop-writes-on-bgsave-error no
```
#### 启动redis容器
```
docker run --name redis -p 6379:6379 \
-v $PWD/redis/conf/redis.conf:/etc/redis/redis.conf \
-v $PWD/redis/data:/data -d redis:6.2 redis-server /etc/redis/redis.conf
```
> 如果配置文件密码不生效，输入 **docker exec -it docker-redis redis-cli** 进入redis客户端执行以下命令<br/>
**CONFIG SET requirepass "qaz!@#wsx"**

##### 启动redis时常见几个Warning及解决方法
+ `WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.`
+ `WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.`
```
vi /etc/sysctl.conf
//添加并保存
vm.overcommit_memory = 1
net.core.somaxconn= 1024 
//执行
sysctl -p 
```
+ `WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.`
```
vi /etc/rc.local
//添加并保存
echo never > /sys/kernel/mm/transparent_hugepage/enabled

//添加执行权限
chmod +x /etc/rc.d/rc.local
//重启
reboot
```

## 配置一主二从
> 分别再启动两个redis服务: <br/>
docker-redis80 -> 6380 | docker-redis81 -> 6381 <br/>
需要关闭防火墙 `systemctl disable firewalld` | `chkconfig iptables off` 并重启docker `systemctl start docker`
### 使用宿主机IP
> `docker run`需要添加如下代码使用宿主机IP <br/>
使用宿主机IP时,不需要指定`-p`
```
--network host

docker run --network host --name docker-redis \
-v $PWD/redis/conf/redis.conf:/etc/redis/redis.conf \
-v $PWD/redis/data:/data -d redis:3.2 redis-server /etc/redis/redis.conf
```

### 自定义IP
#### 创建一个新的bridge网络
```
docker network create --driver bridge --subnet=172.18.12.0/16 --gateway=172.18.1.1 mynet
```
> 配置主从时,`docker run`需要添加如下代码指定容器固定IP
```
--network=mynet --ip 172.18.12.1

docker run --network mynet --ip 172.18.12.1 --name docker-redis -p 6379:6379 \
-v $PWD/redis/conf/redis.conf:/etc/redis/redis.conf \
-v $PWD/redis/data:/data -d redis:3.2 redis-server /etc/redis/redis.conf
```

### 使用容器默认IP
#### 查看主机 `docker-redis -> 6379` 容器IP
```
docker inspect --format '{{.NetworkSettings.IPAddress}}' docker-redis

// 172.17.0.2
```
### 分别配置 docker-redis80 和 docker-redis81
```
vim redis80/conf/redis.conf

#配置master机器地址
slaveof 92.168.239.15 6379
#配置master访问密码
masterauth 12345678
#配置只读
slave-read-only yes
#配置传输延迟默认为no，跨机房/节省带宽 配置为yes
repl-disable-tcp-nodelay no
```
> 需要给docker-redis添加配置`masterauth 12345678`
#### 重启 docker-redis80 和 docker-redis81 完成 redis 主从配置

## redis哨兵机制
####在data文件夹下面添加配置`sentinel.conf`
```
# Sentinel节点的端口
port 26379
dir /data/
logfile "26379.log"

# 当前Sentinel节点监控 172.18.12.1:6379 这个主节点
# 2代表判断主节点失败至少需要2个Sentinel节点节点同意
# mymaster是主节点的别名
sentinel monitor mymaster 92.168.239.15 6379 2
sentinel auth-pass mymaster 12345678

# 每个Sentinel节点都要定期PING命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过30000毫秒且没有回复，则判定不可达
sentinel down-after-milliseconds mymaster 30000

# 当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，原来的从节点会向新的主节点发起复制操作，限制每次向新的主节点发起复制操作的从节点个数为1
sentinel parallel-syncs mymaster 1

# 故障转移超时时间为180000毫秒
sentinel failover-timeout mymaster 30000
```
#### 分别启动三个哨兵
> redis-sentinel -> 26379 | redis-sentinel80 -> 26380 | redis-sentinel81 -> 26381 
```
docker run --name redis-sentinel -p 26379:26379 -v $PWD/redis/data:/data -d redis:3.2 redis-sentinel /data/sentinel.conf
```

## redis集群