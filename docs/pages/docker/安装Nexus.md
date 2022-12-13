### 编辑docker-compose.yml

```
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /app/soft/docker-file/nexus/data:/nexus-data
```
> 添加权限：chmod 777 /app/soft/docker-file/nexus/data

> 启动：docker-compose up -d

### 配置认证信息
> 在 Maven `settings.xml`中添加 Nexus 认证信息(servers节点下)
```
<server>
  <id>nexus-releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>
<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```
### 配置自动化部署
> 在 `pom.xml`中添加如下代码
```
<distributionManagement>  
  <repository>  
    <id>nexus-releases</id>  
    <name>Nexus Release Repository</name>  
    <url>http://106.13.189.134:8081/repository/maven-releases/</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus-snapshots</id>  
    <name>Nexus Snapshot Repository</name>  
    <url>http://106.13.189.134:8081/repository/maven-snapshots/</url>  
  </snapshotRepository>  
</distributionManagement> 
```
> 注意事项：
> + ID 名称必须要与 settings.xml 中 Servers 配置的 ID 名称保持一致。
> + 项目版本号中有 SNAPSHOT 标识的，会发布到 Nexus Snapshots Repository, 否则发布到 Nexus Release Repository，并根据 ID 去匹配授权账号。

### 配置代理仓库
> 在 `pom.xml`中添加如下代码
```
<repositories>
    <repository>
        <id>nexus</id>
        <name>Nexus Repository</name>
        <url>http://106.13.189.134:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>Nexus Plugin Repository</name>
        <url>http://106.13.189.134:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```
### 部署到仓库
`mvn deploy`
#### 上传第三方 JAR 包
> Nexus 3.0 不支持页面上传，可使用 maven 命令：
```
# 如第三方JAR包：aliyun-sdk-oss-2.2.3.jar ^
mvn deploy:deploy-file  ^
  -DgroupId=com.aliyun.oss  ^
  -DartifactId=aliyun-sdk-oss ^
  -Dversion=2.2.3 ^
  -Dpackaging=jar ^
  -Dfile=D:\aliyun-sdk-oss-2.2.3.jar ^
  -Durl=http://106.13.189.134:8081/repository/maven-releases/ ^
  -DrepositoryId=nexus-releases 
```
> 注意事项：
> + 建议在上传第三方 JAR 包时，创建单独的第三方 JAR 包管理仓库，便于管理有维护。（maven-3rd）
> + `-DrepositoryId=nexus-releases`对应的是 settings.xml中 Servers 配置的 ID 名称。（授权）

> PS：
> + `windows` 在 cmd 脚本中的连接符是 `^`
> + `Linux` 在 sh 脚本中实现的同样功能的连接符是 `\`
