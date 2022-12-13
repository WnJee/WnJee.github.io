## 安装 ES ##
### 下载ES6.2.2镜像
`docker pull docker.elastic.co/elasticsearch/elasticsearch:6.2.2`
### 使用镜像创建容器，启动elasticsearch服务
> 分两种方式，开发者模式和生产模式，开发者不需要配置太多，直接一行命令搞定，生产模式需要更多的配置

#### 开发者模式
##### 创建网络
> 如果需要安装kibana等其他，需要创建一个网络，名字任意取，让他们在同一个网络，使得es和kibana通信

```
docker network create esnet
```
##### 创建并启动elasticsearch容器
> -e "xpack.security.enabled=false" 禁用密码验证

> 6.2.2不需要禁用 5.5.0需要

```
docker run --name es550  -p 9200:9200 -p 9300:9300  --network esnet -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.2.2
```
##### 配置跨域
1. 进入容器 `docker exec -it es622 /bin/bash`
2. 修改配置文件 `vi config/elasticsearch.yml`
```
#集群名称
cluster.name: es-lemon
#本节点名称
node.name: master
#是否master节点
node.master: true
#是否存储数据
node.data: true
#跨域设置
http.cors.enabled: true
http.cors.allow-origin: "*"
#http端口
http.port: 9200
#java端口
transport.tcp.port: 9300
#可以访问es集群的ip  0.0.0.0表示不绑定
network.bind_host: 0.0.0.0
#es集群相互通信的ip  0.0.0.0默认本地网络搜索
#network.publish_host: 0.0.0.0

#6.x配置
discovery.zen.minimum_master_nodes: 1
xpack.license.self_generated.type: basic
```
3. 重启容器
`docker restart es622`
4. 查看运行情况 `http://ip:9200/`
5. 默认密码
```
elastic	changeme	
kibana	changeme	
logstash_system	changeme
```

## docker部署es-head
#### 拉取镜像
```
docker pull mobz/elasticsearch-head:5
```
#### 运行容器
```
docker run -d --name es_head5 -p 9100:9100 mobz/elasticsearch-head:5
```
#### 查看运行情况
`http://ip:9100/`

> 异常处理： {
"error": "Content-Type header [application/x-www-form-urlencoded] is not supported",
"status": 406
}
```
#进入容器内部编辑
vim _site/vendor.js
#如果没有vim 需要更新再安装
apt-get update
apt-get install -y vim

1. 6886行 /contentType: "application/x-www-form-urlencoded 
    改成 contentType: "application/json;charset=UTF-8" 
2. 7574行 var inspectData = s.contentType === "application/x-www-form-urlencoded" && 
    改成 var inspectData = s.contentType === "application/json;charset=UTF-8" &&
```

## springboot整合es
#### 添加pom依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
#### 修改配置
```
spring.data.elasticsearch.cluster-name=es-lemon
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
spring.data.elasticsearch.repositories.enabled=true
```
#### coding
```
@Document(indexName = "eslog")
public class MoEsLog {
    private String message;
 
    public String getMessage() {
        return message;
    }
 
    public void setMessage(String message) {
        this.message = message;
    }
 
    private String dateTime;
 
    public String getDateTime() {
        return dateTime;
    }
 
    public void setDateTime(String dateTime) {
        this.dateTime = dateTime;
    }
 
    @Id
    private String _id;
}
```
```
@Repository
public interface IEsRepository extends ElasticsearchRepository<MoEsLog, String> {
}
```
```
@Service
public class EsLogServiceImpl {
    @Autowired
    private IEsRepository esRepository;
 
    public void addEsLog(String message) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ");
        MoEsLog esLog = new MoEsLog();
        esLog.setMessage(message);
        esLog.setDateTime(simpleDateFormat.format(new Date()));
        MoEsLog save = esRepository.save(esLog);
    }
}
```
```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = App.class)
public class Test {
 
    @Autowired
    private EsLogServiceImpl esLogService;
 
    @Test
    public void esLogService() {
        esLogService.addEsLog("aaa");
    }
}
```