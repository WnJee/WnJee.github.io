> 在服务器创建配置文件目录如下,并cd到 **docker-config**目录下<br/>
```
docker-config
└───mongodb
    |
    └───conf
    |
    └───data
    |
    └───backup
```
## 安装MongoDB
#### 下载mongodb镜像
`docker pull mongo:4`
#### 运行mongodb容器
```
docker run --name docker-mongo -p 27017:27017 \
-v $PWD/mongodb/data:/data/db \
-v $PWD/mongodb/conf:/data/configdb \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=admin1234 -d mongo:4
```
#### 以admin身份进入容器创建用户
`docker exec -it docker-mongo mongo`
```
// 创建一个admin管理账号
// db.createUser({ user: 'admin', pwd: 'admin1234', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

# 创建普通用户、密码和数据库
db.auth("admin","admin1234");
use app
db.createUser({ user: 'swen', pwd: 'swen123456', roles: [ { role: "readWrite", db: "app" } ] });
# 登录app数据库
use app
db.auth("swen","swen123456");
db.test.save({name:"zhangsan"});
```

## 部署MongoDB副本集
#### 启动三个节点
```
docker run --name mongo0 -p 37017:27017 -d mongo --replSet "rs"
docker run --name mongo1 -p 47017:27017 -d mongo --replSet "rs"
docker run --name mongo2 -p 57017:27017 -d mongo --replSet "rs"
```
#### 连接任意一个节点，进行副本集配置
`docker exec -it mongo0 bash `
> 连接三个节点中的任意一个,注意ip地址为宿主机ip,我当前的为47.101.31.165

`mongo --host 47.101.31.165 --port 37017`
> 此时已连接到m0节点，进行副本集配置

```
var config={
     _id:"rs",
     members:[
         {_id:0,host:"47.101.31.165:37017"},
         {_id:1,host:"47.101.31.165:47017"},
         {_id:2,host:"47.101.31.165:57017"}
]};
rs.initiate(config)
```
> 响应应该类似下面，注意此时命令提示符已经发生变化，由原来的 > 变成了 rs:SECONDARY>

```
{
    "ok" : 1,
    "operationTime" : Timestamp(1522810920, 1),
    "$clusterTime" : {
        "clusterTime" : Timestamp(1522810920, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
}
```
#### 查看副本集配置信息
`rs.conf()`
#### 查看副本集状态
`rs.status()`



> JPA支持的Repository如下：

![image](http://106.13.189.134:8888/group1/M00/00/00/wKgABF1wuZqAanKrAAOdreJtFz8803.png)



