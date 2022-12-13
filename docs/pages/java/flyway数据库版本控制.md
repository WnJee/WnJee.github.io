### 引入maven依赖
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```
### 配置`application.properties`
```bash
#jdbc数据源
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&useUnicode=true&useSSL=false&serverTimezone=UTC
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=123456

#对已经存在表结构数据的已有数据库使用flyway,需要设置此参数,同时脚本的版本号一定要比1大才能执行
#如：V1.0__Base_version.sql不会被执行，但V1.1__Base_version.sql就会被执行
spring.flyway.baseline-on-migrate=true
#指定版本控制文件存放目录
spring.flyway.locations[0]=classpath:db/migration
```
> `mysql-connector-java:8.0+`配置
```bash
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&useUnicode=true&useSSL=false&serverTimezone=UTC
```
> `mysql-connector-java:5.x`配置
```bash
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&amp;characterEncoding=UTF-8
```
### 编写sql脚本
> `classpath:db/migration`文件夹下新建sql脚本, 脚本名称必须符合`V*.*__xxx.sql`,如：`V1.0__init.sql`,同时升级脚本需要升级版本号
``` sql
use test;

CREATE TABLE `test_sql` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `age` int(2) DEFAULT NULL,
  `status` int(1) DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `info` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `test`.`test_sql`(`id`, `age`, `status`, `name`, `info`) VALUES (1, 11111, 1, '无视为为', '1');
INSERT INTO `test`.`test_sql`(`id`, `age`, `status`, `name`, `info`) VALUES (2, 22222, 1, '阿萨姆奶茶', '2');
```
> 运行springboot项目后会生成一个`flyway_schema_history`表，存放的是执行的sql版本记录，每次运行都会检查文件夹下的脚本是否有新脚本，如果有则执行

### flyway的其他相关配置
```bash
flyway.baseline-description对执行迁移时基准版本的描述.
flyway.baseline-on-migrate当迁移时发现目标schema非空，而且带有没有元数据的表时，是否自动执行基准迁移，默认false.
flyway.baseline-version开始执行基准迁移时对现有的schema的版本打标签，默认值为1.
flyway.check-location检查迁移脚本的位置是否存在，默认false.
flyway.clean-on-validation-error当发现校验错误时是否自动调用clean，默认false.
flyway.enabled是否开启flywary，默认true.
flyway.encoding设置迁移时的编码，默认UTF-8.
flyway.ignore-failed-future-migration当读取元数据表时是否忽略错误的迁移，默认false.
flyway.init-sqls当初始化好连接时要执行的SQL.
flyway.locations迁移脚本的位置，默认db/migration.
flyway.out-of-order是否允许无序的迁移，默认false.
flyway.password目标数据库的密码.
flyway.placeholder-prefix设置每个placeholder的前缀，默认${.
flyway.placeholder-replacementplaceholders是否要被替换，默认true.
flyway.placeholder-suffix设置每个placeholder的后缀，默认}.
flyway.placeholders.[placeholder name]设置placeholder的value
flyway.schemas设定需要flywary迁移的schema，大小写敏感，默认为连接默认的schema.
flyway.sql-migration-prefix迁移文件的前缀，默认为V.
flyway.sql-migration-separator迁移脚本的文件名分隔符，默认__
flyway.sql-migration-suffix迁移脚本的后缀，默认为.sql
flyway.tableflyway使用的元数据表名，默认为schema_version
flyway.target迁移时使用的目标版本，默认为latest version
flyway.url迁移时使用的JDBC URL，如果没有指定的话，将使用配置的主数据源
flyway.user迁移数据库的用户名
flyway.validate-on-migrate迁移时是否校验，默认为true
```