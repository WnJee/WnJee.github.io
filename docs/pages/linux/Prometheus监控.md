## 安装node_exporter
#### 下载
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.0/node_exporter-0.18.0.linux-amd64.tar.gz
```
#### 解压
`tar -xvf node_exporter-0.18.0.linux-amd64.tar.gz`
#### 运行
`nohup ./node_exporter &`

## 安装mysql_exporter
#### 下载
```bash
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.10.0/mysqld_exporter-0.10.0.linux-amd64.tar.gz
```
#### 解压
`tar -xvf mysqld_exporter-0.10.0.linux-amd64.tar.gz`
#### 添加mysql连接配置
`vi .my.cnf`
```bash
[client]
user=root
password=123456
```
#### 运行
`nohup ./mysqld_exporter -config.my-cnf=".my.cnf" &`

## 安装redis_exporter
#### 下载
```bash
wget https://github.com/oliver006/redis_exporter/releases/download/v0.13/redis_exporter-v0.13.linux-amd64.tar.gz
```
#### 解压
`tar -xvf redis_exporter-v0.13.linux-amd64.tar.gz`
#### 运行
```bash
## 无密码
nohup ./redis_exporter redis//ip:6379 &
## 有密码
nohup ./redis_exporter -redis.addr ip:6379  -redis.password 123456 &
```

## prometheus的搭建
#### 拉取prometheus镜像
`docker pull prom/prometheus`
#### 编辑配置文件
`vi prometheus.yml`
```bash
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['ip:9090']
        labels:
          instance: prometheus

  - job_name: linux
    static_configs:
      - targets: ['ip:9100']
        labels:
          instance: sys1
          
  - job_name: redis
    static_configs:
      - targets: ['ip:9121']
        labels:
          instance: redis1

  - job_name: mysql
    static_configs:
      - targets: ['ip:9104']
        labels:
          instance: db1
```
#### 启动prometheus服务
```bash
docker run -d -p 9090:9090 --net=host \
-v $PWD/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
--name prometheus prom/prometheus
```
## grafana的搭建
#### 拉取grafana镜像
`docker pull grafana/grafana:5.0.0`
#### 启动grafana服务
```bash
docker run -d -p 3000:3000 --net=host \
-v $PWD/grafana/data:/var/lib/grafana \
-e "GF_SECURITY_ADMIN_PASSWORD=123456" \
--name grafana grafana/grafana:5.0.0
```

#### 访问
> 地址：http://ip:3000/ <br/>
账号密码：admin/123456

## 配置grafana
#### 在Data Sources选项中添加数据源
```bash
Name   : Prometheus
Type   : Prometheus
URL    : http://localhost:9090
Access : proxy
```
#### import Dashbord (ID)
ID|描述
-|-|
9276|主机基础监控(cpu，内存，磁盘，网络)
8919|Node Exporter 0.16+ 监控展示看板
7362|MySQL Overview
2751|Prometheus Redis