## 安装 FastDFS ##
#### 下载FastDFS文件系统的docker镜像 ####
`docker pull delron/fastdfs`
#### 进入到你准备安装FastDFS的目录 ####
`cd /app/docker-config/fastdfs/`
#### 构建tracker容器（跟踪服务器，起到调度的作用） ####
```
docker run -d --network=host --name tracker -v $PWD/tracker:/var/fdfs delron/fastdfs tracker
```
#### 构建storage容器（存储服务器，提供容量和备份服务） ####
```
docker run -d --network=host --name storage -e TRACKER_SERVER=服务器IP:22122 -v $PWD/storage:/var/fdfs -e GROUP_NAME=group1 delron/fastdfs storage
```
#### 进入storage容器，配置http访问的端口和nginx ####
`docker exec -it storage bash`
##### 配置http访问的端口 #####
`vi /etc/fdfs/storage.conf`
```
# since V4.05
connection_pool_max_idle_time = 3600

# use the ip address of this storage server if domain_name is empty,
# else this domain name will ocur in the url redirected by the tracker server
http.domain_name = 

# the port of the web server on this storage server
http.server_port = 8888
```
##### 配置nginx #####
`vi /usr/local/nginx/conf/nginx.conf`
```
server {
    listen  8888;
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root html;
    }
}
```
#### FastDFS搭建完成，现在手动将文件放到服务器然后通过网络访问 ####
1. 将一张照片放到安装目录下的storage文件夹下（$PWD/storage/）
2. 进入storage容器 `docker exec -it storage bash`
3. 进入/var/fdfs目录运行下面命令: `/usr/bin/fdfs_upload_file /etc/fdfs/client.conf cumt.jpg` <br/>
cumt.jpg为刚才上传的文件，此时将该图片已上传至文件系统，并在执行该语句后返回图片存储的uri：
`group1/M00/00/00/CgCJ611vG16ACG9yAAAhLlLCmfQ191.jpg`
4. 通过url访问`http://ip:8888/group1/M00/00/00/CgCJ611vG16ACG9yAAAhLlLCmfQ191.jpg`即可查看到图片

## SpringBoot2.X 整合 FastDFS ##
#### 添加pom依赖 ####
```
<!--fstdfs client-->
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
    <version>1.26.5</version>
</dependency>
```
#### 添加application.properties配置 ####
```
#fastdfs相关配置
fdfs.so-timeout=1501
fdfs.connect-timeout=601
fdfs.thumb-image.height=200
fdfs.thumb-image.width=200
#tracker-list可以配置多个storage地址
fdfs.tracker-list[0]=47.101.31.165:22122
```
#### 导入FdfsClient配置 ####
```
@Configuration
@Import(FdfsClientConfig.class)
// 解决jmx重复注册bean的问题
@EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
public class FdfsConfigImport {
}
```
#### 测试上传功能 ####
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ItboysWebApplicationTests {

    @Autowired
    private FastFileStorageClient storageClient;
    @Autowired
    private ThumbImageConfig thumbImageConfig;

    @Test
    public void testUpload() throws FileNotFoundException {
        File file = new File("C:\\test\\4.jpg");
        // 上传并且生成缩略图
        System.out.println("-------------上传图片并且生成缩略图---------------");
        StorePath storePath = this.storageClient.uploadImageAndCrtThumbImage(
                new FileInputStream(file), file.length(), "jpg", null);
        // 带分组的路径
        System.out.println(storePath.getFullPath());
        // 不带分组的路径
        System.out.println(storePath.getPath());
        // 获取缩略图路径
        String path = thumbImageConfig.getThumbImagePath(storePath.getPath());
        System.out.println(path);
        System.out.println("-------------只上传图片---------------");
        // 只上传图片
        StorePath uploadFile = this.storageClient.uploadFile(
                new FileInputStream(file), file.length(), "jpg", null);
        // 带分组的路径
        System.out.println(uploadFile.getFullPath());
    }
}
```
