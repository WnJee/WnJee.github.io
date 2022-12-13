## 安装docker ##
##### docker要求CentOS系统的内核版本高于3.10,首先验证你的CentOS版本 #####
`uname -r`
##### 使用root权限登录Centos,确保yum包更新到最新 #####
`yum -y update`
##### 卸载旧版本(如果安装过旧版本的话) #####
`yum remove docker docker-common docker-selinux docker-engine`
##### 安装需要的软件包 #####
`yum -y install yum-utils device-mapper-persistent-data lvm2`
##### 设置yum源 #####
`yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
##### 安装最新稳定版docker #####
`yum -y --nobest install docker-ce`
##### 启动并加入开机启动 #####
`systemctl start docker`

`systemctl enable docker`
##### 验证安装是否成功 #####
`docker version`
##### 卸载docker #####
`yum -y remove docker-engine`

docker常用的命令有:
> * docker search 查找docker-hub上的镜像
> * docker pull 拉取镜像到本地
> * docker images 查看镜像列表
> * docker rmi 删除镜像
> * docker ps -a 查看容器列表
> * docker rm 删除容器
> * docker run 运行容器
> * docker exec 进入容器内部 </br>

具体详细用法请自行百度

## 安装docker-compose ##
##### 下载docker-compose #####
```bash
sudo wget -c -t 0 https://github.com/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -O /usr/local/bin/docker-compose
```
```bash
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
##### 添加可执行权限 #####
`chmod +x /usr/local/bin/docker-compose`
##### 测试安装结果 #####
`docker-compose --version`

## 镜像加速 ##
```bash
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

systemctl restart docker
```
##### 其他加速地址
`vi /etc/docker/daemon.json`
```bash
{
"registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
```
`service docker restart`


## Docker tab自动补全 ##
##### 安装插件 #####
`yum install -y bash-completion`
##### 配置插件生效 #####
`source /usr/share/bash-completion/bash_completion`


## 容器备份和恢复
### 备份
