## Linux安装Nacos1.4

### 下载

> 下载地址：https://github.com/alibaba/nacos/releases

### 上传解压

> 解压命令：tar xvf nacos-server-1.4.0.tar.gz

### 配置

#### 数据库配置修改

> 修改conf 下 application.properties文件

```properties
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1
### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/youpik-nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password=qaz123wsx
```

#### 数据库初始化

> 使用conf 下nacos-mysql.sql 创建表和初始化数据

### 启动

#### 单机方式运行

> 切到bin目录下，执行 ./startup.sh -m standalone