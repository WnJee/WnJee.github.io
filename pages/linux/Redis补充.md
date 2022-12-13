### Linux安装单机版

1. 下载：http://download.redis.io/releases/redis-4.0.2.tar.gz
2. 解压：tar -zxvf redis
3. 安装gcc依赖：yum install gcc -y
4. 进入redis目录编译：make PREFIX=/usr/local/redis MALLOC=libc install
5. 复制配置文件并修改：cp redis.conf /usr/local/redis/
6. 运行：./bin/redis-server redis.conf



### redis执行lua脚本

#### 编写一个限流脚本`limit.lua`
```bash
local key = KEYS[1]

--传入参数 
-- 写法一 （java object...数组传参）
-- local limit = tonumber(ARGV[1])
-- local expire_time = ARGV[2]
-- 写法二 （java map传参）
local map_json =  cjson.decode(ARGV[1])
local limit = map_json.limit
local expire_time = map_json.expire

-- 日志打印 需要和redis.conf配置的级别一致
-- 级别设置 （redis.LOG_DEBUG、redis.LOG_VERBOSE、redis.LOG_NOTICE、redis.LOG_WARNING）
redis.log(redis.LOG_NOTICE,limit)

local is_exists = redis.call("EXISTS", key)
if is_exists == 1 then 
    if redis.call("INCR", key) > limit then
        return 0
    else
        return 1
    end
else
    redis.call("SET", key, 1)
    redis.call("EXPIRE", key, expire_time)
    return 1
end
```
#### redis-cli加载脚本
`redis-cli -a 12345678 script load "$(cat limit.lua)"`
> 会返回一个sha码 `ec4d15893aab8b889f1d8f00b2d1f714daddf17f`
#### 执行脚本
```bash
127.0.0.1:6379> EVALSHA ec4d15893aab8b889f1d8f00b2d1f714daddf17f 1 ip-limit 5 10
```
> 命令：`EVALSHA shal numkeys key [key...] arg [arg...]`
> + **shal:** `script load`脚本得到的sha值
> + **numkeys:** 脚本中使用到redis-key的个数，也就是`KEYS[]`数组的长度
> + **[key...]:** 传入脚本的`KEYS[]`数组
> + **[arg...]:** 传入脚本的参数`ARGV[]`数组

### springboot执行脚本代码
```java
@Resource
private RedisTemplate<String, Object> redisTemplate;

private DefaultRedisScript<Long> getRedisScript;

@PostConstruct
public void init(){
    getRedisScript = new DefaultRedisScript<>();
    getRedisScript.setResultType(Long.class);
    getRedisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("luascripts/limit.lua")));
}

public Long scriptLua() {
    /**
     * List设置lua的KEYS
     */
    List<String> keyList = new ArrayList();
    keyList.add("vote-limit");
    /**
     * 传入两个参数 表示在10秒内只允许访问5次
     * 调用脚本并执行
     */
    Long result = redisTemplate.execute(getRedisScript,keyList, 5, 10);
    System.out.println(result);
    return result;
}
```
> 脚本返回值泛型`DefaultRedisScript<Long>`只支持以下几种类型：否则会报错`java.lang.IllegalStateException: null`
```java
public static ReturnType fromJavaType(@Nullable Class<?> javaType) {
    if (javaType == null) {
        return ReturnType.STATUS;
    }
    if (javaType.isAssignableFrom(List.class)) {
        return ReturnType.MULTI;
    }
    if (javaType.isAssignableFrom(Boolean.class)) {
        return ReturnType.BOOLEAN;
    }
    if (javaType.isAssignableFrom(Long.class)) {
        return ReturnType.INTEGER;
    }
    return ReturnType.VALUE;
}
```

#### 脚本管理
+ **script kill :** 强制停止正在运行且阻塞的脚本
+ **script exists shal :** 查询是否存在某个脚本
+ **script flush :** 清空脚本

### redis常见配置
```bash
# redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程：

daemonize no

# 当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定：

pidfile /var/run/redis.pid

# 指定redis监听端口，默认端口号为6379，作者在自己的一篇博文中解析了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利女歌手Alessia Merz的名字：

port 6379

# 设置tcp的backlog，backlog是一个连接队列，backlog队列总和=未完成三次握手队列+已完成三次握手队列。在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn 的值，所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的

# 效果：

tcp-backlog 511

# 绑定的主机地址：

bind 127.0.0.1

# 当客户端闲置多长时间后关闭连接，如果指定为0，表示永不关闭：

timeout 300

# 设置检测客户端网络中断时间间隔，单位为秒，如果设置为0，则不检测，建议设置为60：

tcp-keepalive 0

# 指定日志记录级别，redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose：

loglevel verbose

# 日志记录方式，默认为标准输出，如果配置redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null：

logfile stdout

# 设置数据库数量，默认值为16，默认当前数据库为0，可以使用select<dbid>命令在连接上指定数据库id：

databases 16

# 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合：

# save <seconds><changes>
save 300 10：表示300秒内有10个更改就将数据同步到数据文件

# 指定存储至本地数据库时是否压缩数据，默认为yes，redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变得巨大：

rdbcompssion yes

# 指定本地数据库文件名，默认值为dump.rdb：

dbfilename dump.rdb

# 指定本地数据库存放目录：

dir ./

# 设置当本机为slave服务时，设置master服务的IP地址及端口，在redis启动时，它会自动从master进行数据同步：

# slaveof <masterip><masterport>

# 当master服务设置了密码保护时，slave服务连接master的密码：

# masterauth <master-password>

# 设置redis连接密码，如果配置了连接密码，客户端在连接redis时需要通过auth <password>命令提供密码，默认关闭：

requirepass foobared

# 设置同一时间最大客户端连接数，默认无限制，redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数，如果设置maxclients 0，表示不作限制。当客 户端连接数到达限制时，redis会关闭新的连接并向客户端返回 max number of clients reached错误消息：

maxclients 128

# 指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会先尝试清除已到期或即将到期的key，当次方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制， 会把key存放内存，value会存放在swap区：

# maxmemory <bytes>

# 设置缓存过期策略，有6种选择：（LRU算法最近最少使用）

# volatile-lru：使用LRU算法移除key，只对设置了过期时间的key；

# allkeys-lru：使用LRU算法移除key，作用对象所有key；

# volatile-random：在过期集合key中随机移除key，只对设置了过期时间的key;

# allkeys-random：随机移除key，作用对象为所有key；

# volarile-ttl：移除哪些ttl值最小即最近要过期的key；

# noeviction：永不过期，针对写操作，会返回错误信息。

maxmemory-policy noeviction

# 指定是否在每次更新操作后进行日志记录，redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内数据丢失。因为redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内置存在于内存中。默认为no：

appendonly no

# 指定更新日志文件名，默认为appendonly.aof：

appendfilename appendonly.aof

# 指定更新日志条件，共有3个可选值：

# no：表示等操作系统进行数据缓存同步到磁盘（快）；

# always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）；

# everysec：表示每秒同步一次（折中，默认值）

appendfsync everysec

# 指定是否启用虚拟内存机制，默认值为no，简单介绍一下，VM机制将数据分页存放，由redis将访问量较小的页即冷数据 swap到磁盘上，访问多的页面由磁盘自动换出到内存中：

vm-enabled no

# 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个redis实例共享：

vm-swap-file /tmp/redis.swap

# 将所有大于vm-max-memory的数据存入虚拟内存，无论vm-max-memory设置多小，所有索引数据都是内存存储的（redis的索引数据就是keys），也就是说，当vm-max-memory设置为0的时候，其实是所有value都存在于磁盘。默认值为 0：

vm-max-memory 0

# redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是根据存储的数据大小来设定的，作者建议如果储存很多小对象，page大小最好设置为32或者64bytes；如果存储很多大对象，则可以使用更大的page，如果不确定，就使用默认值：

vm-page-size 32

# 设置swap文件中page数量，由于页表（一种表示页面空闲或使用的bitmap）是放在内存中的，在磁盘上每8个pages将消耗1byte的内存：

vm-pages 134217728

# 设置访问swap文件的线程数，最好不要超过机器的核数，如果设置为0，那么所有对swap文件的操作都是串行的，可能会造成长时间的延迟。默认值为4：

vm-max-threads 4

# 设置在客户端应答时，是否把较小的包含并为一个包发送，默认为开启：

glueoutputbuf yes

# 指定在超过一定数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法：

hash-max-zipmap-entries 64

hash-max-zipmap-value 512

# 指定是否激活重置hash，默认开启：

activerehashing yes

# 指定包含其他配置文件，可以在同一主机上多个redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件：

include /path/to/local.conf
```

### redis性能测试
+ 100个并发连接，10W个请求，检测redis服务器性能 <br/>
 `redis-benchmark -h 192.168.239.15 -p 6379 -c 100 -n 100000`
+ 测试存取大小为100字节的数据包性能 <br/>
 `redis-benchmark -h 192.168.239.15 -p 6379 -q -d 100`
+ 只测试`set`,`lpush`操作的性能 <br/>
 `redis-benchmark -h 192.168.239.15 -p 6379 -t set,lpush -n 100000 -q`
+ 只测试某些数据存取的性能 <br/>
 `redis-benchmark -h 192.168.239.15 -p 6379 -n 100000 -q script load "redis.call('set','foo','bar')"`


### 从mysql导出数据到redis
#### cms_clazz表数据
<table>
    <tr>
        <th>id</th>
        <th>name</th>
        <th>remark</th>
        <th>del_flag</th>
        <th>create_time</th>
        <th>update_time</th>
    </tr>
    <tr>
        <th>111111</th>
        <th>分类A</th>
        <th>a</th>
        <th>0</th>
        <th>2019-05-30 10:07:27</th>
        <th>2019-10-17 04:50:51</th>
    </tr>
    <tr>
        <th>222222</th>
        <th>分类B</th>
        <th>b</th>
        <th>0</th>
        <th>2019-05-30 10:07:27</th>
        <th>2019-10-17 04:50:51</th>
    </tr>
    <tr>
        <th>333333</th>
        <th>分类C</th>
        <th>c</th>
        <th>0</th>
        <th>2019-05-30 10:07:27</th>
        <th>2019-10-17 04:50:51</th>
    </tr>
</table>

#### 编写sql脚本`data.sql`
```sql
SELECT CONCAT(
'*14\r\n',
'$',LENGTH(redis_cmd),'\r\n',redis_cmd,'\r\n',
'$',LENGTH(redis_key),'\r\n',redis_key,'\r\n',
'$',LENGTH(hkey1),'\r\n',hkey1,'\r\n',
'$',LENGTH(hval1),'\r\n',hval1,'\r\n',
'$',LENGTH(hkey2),'\r\n',hkey2,'\r\n',
'$',LENGTH(hval2),'\r\n',hval2,'\r\n',
'$',LENGTH(hkey3),'\r\n',hkey3,'\r\n',
'$',LENGTH(hval3),'\r\n',hval3,'\r\n',
'$',LENGTH(hkey4),'\r\n',hkey4,'\r\n',
'$',LENGTH(hval4),'\r\n',hval4,'\r\n',
'$',LENGTH(hkey5),'\r\n',hkey5,'\r\n',
'$',LENGTH(hval5),'\r\n',hval5,'\r\n',
'$',LENGTH(hkey6),'\r\n',hkey6,'\r\n',
'$',LENGTH(hval6),'\r\n',hval6,'\r\n'
)
FROM
(
SELECT 
'HSET' AS redis_cmd,
CONCAT('cms:clazz:', id) AS redis_key,
'id' AS hkey1, id AS hval1,
'name' AS hkey2, name AS hval2,
'remark' AS hkey3, remark AS hval3,
'delFlag' AS hkey4, del_flag AS hval4,
'createTime' AS hkey5, create_time AS hval5,
'updateTime' AS hkey6, update_time AS hval6
FROM cms_clazz
) t
```
#### lunix服务器上执行命令
```bash
mysql -uroot -p123456 stress --default-character-set=utf8 --skip-column-names --raw < data.sql | redis-cli -h 192.168.239.15 -p 6379 -a 12345678 --pipe
```
