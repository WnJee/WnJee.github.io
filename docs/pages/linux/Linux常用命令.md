#### 查看端口号占用
```bash
netstat -antp|grep 端口号
```
#### linux磁盘清理

```bash
1.查看磁盘使用情况  df -lh
2.查看哪个文件夹占用空间最大  du -h --max-depth=1
3.将文件以从大到小顺序展现  ls -lS
4.删除文件  rm -rf file
5.确认删除文件是否被占用  /usr/sbin/lsof|grep deleted
6.杀掉占用进程  kill -9 pid
```

#### nohup后台运行

```bash
nohup cmd >/dev/null 2>&1 &
```
#### 拷贝文件到目标服务器

```bash
scp -r file root@IP:/directory  
```
#### ssh相关
```bash
## ssh密钥的生成
ssh-keygen -t rsa -C "username"

##工具连不上，服务器自带的能连得上
vim /etc/hosts.allow
#在文件中添加
sshd: ALL
#重启
systemctl restart sshd
```

#### Linux关闭防火墙命令
```bash
##查看防火状态

systemctl status firewalld

service  iptables status

##暂时关闭防火墙

systemctl stop firewalld

service  iptables stop

##永久关闭防火墙

systemctl disable firewalld

chkconfig iptables off

##重启防火墙

systemctl enable firewalld

service iptables restart  

##永久关闭后重启

##暂时还没有试过

chkconfig iptables on

################### 开放端口 ####################
1、开启防火墙 
    systemctl start firewalld

2、开放指定端口
      firewall-cmd --zone=public --add-port=1935/tcp --permanent
 命令含义：
--zone #作用域
--add-port=1935/tcp  #添加端口，格式为：端口/通讯协议
--permanent  #永久生效，没有此参数重启后失效

3、重启防火墙
      firewall-cmd --reload

查看所有打开的端口： 
firewall-cmd --zone=public --list-ports

4、查看端口号
netstat -ntlp   //查看当前所有tcp端口·

netstat -ntulp |grep 1935   //查看所有1935端口使用情况·
```

## 解压缩
```bash
tar
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。

-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出
下面的参数-f是必须的
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
# tar -cf all.tar *.jpg
这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
# tar -rf all.tar *.gif
这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。

# tar -uf all.tar logo.gif
这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。

# tar -tf all.tar
这条命令是列出all.tar包中所有文件，-t是列出文件的意思

# tar -xf all.tar
这条命令是解出all.tar包中所有文件，-t是解开的意思
压缩
tar -cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg 
tar -czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar -cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar -cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
解压
tar -xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar -xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
总结
1、*.tar 用 tar -xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar -xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar -xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar -xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压
```

### 相关脚本

#### java应用脚本

```bash
#!/bin/sh

# jar的路径放这里  路径最好小点 否则 解压jar的时候 调用系统zip命令解压的 会因为路径过长导致一些包不被解压
jarfile=/project/youpik/member-center/member-server.jar
# 控制台out文件的路径  这里不输出
consolePath=/dev/null
# gc日志文件
gcFile=/var/log/gc/youpik/gc-member-center.log
# 进程ID 的路径
pidFile=/pid/youpik/member-center.pid
# 系统端口
serverPort=10300
# 遇到某个字符 控制台 tail 就 break
breakStr=successStartdongshanBackend
#最大堆内存
maxHeapMemory=256m
#kill pid 不成功的时候 去grep 是的 查找关键字 比如site系统 查找 site
grepKeyWord=member-server
#heap内存溢出的时候 自动dump 到哪个路径下
heapDumpPath=/data/dump/heap/youpik/member-center.bin

rm -rf $consolePath

kill -15 `cat $pidFile`
sleep 3
ps -ef | grep $grepKeyWord | grep $serverPort | grep -v grep | awk '{print $2}'  | sed -e "s/^/kill -9 /g" | sh -

nohup java -Dserver.port=$serverPort -server -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=$heapDumpPath -Xms$maxHeapMemory -Xmx$maxHeapMemory -XX:MaxMetaspaceSize=256m -Xss512k -XX:+DisableExplicitGC -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=2 -XX:+PrintGCTimeStamps -XX:PretenureSizeThreshold=3145728 -XX:+PrintGCDetails -Xloggc:$gcFile -jar $jarfile > $consolePath 2>&1 &
echo $! > $pidFile
echo "----------- starting ----------"
#sleep 2
#tail -f $consolePath | sed '/$breakStr/Q'

#########健康检查#########
function health_check()
{

    if [ "$serverPort" = "" ];
    then
       echo -e "\033[0;31m 未获取到检测的端口号,健康检查跳过.... \033[0m"
       return
    fi
    HEALTH_CHECK_URL=http://127.0.0.1:$serverPort/health/check # 应用健康检查URL
    exptime=0
    APP_START_TIMEOUT=120
    WORK_HOME=/project/youpik/${grepKeyWord}
    echo "健康检查 开始 ${HEALTH_CHECK_URL}"
    while true
    do
        status_code=`/usr/bin/curl -L -o /dev/null --connect-timeout 5 -s -w %{http_code}  ${HEALTH_CHECK_URL}`
        if [ x$status_code != x200 ];then
           if [ x$status_code = x404 ];then
                echo "****************Actuator is not opened,please check...."
                echo -e "\033[0;31m ******************健康检查跳过.... \033[0m"
                return
            fi
            sleep 1
            ((exptime++))
            echo -n -e "\rWait app to pass health check: $exptime..."
            echo
        else
            break
        fi
        if [ $exptime -gt ${APP_START_TIMEOUT} ]; then
            echo
            echo 'app start failed'
            cat  /project/youpik/member-center/log.out
            sleep 1
            exit 1
        fi
    done
    echo "健康检查 完成 ${HEALTH_CHECK_URL} HTTP_STATUS OK "
}

health_check
echo "----------- finished ----------"
```



