## 安装 gitlab ##
> 编写一个gitlab脚本docker-compose.yml
```
version: '3'
services:
 gitlab:
  image: gitlab/gitlab-ce:12.0.3-ce.0
  container_name: gitlab
  ports:
  - "9022:9022"
  - "9080:80"
  volumes:
  - "/app/data/gitlab/cfg:/etc/gitlab"
  - "/app/data/gitlab/logs:/var/log/gitlab"
  - "/app/data/gitlab/data:/var/opt/gitlab"
  restart: always
```
> 后台启动服务 docker-compose up -d

#### 配置gitlab ####
> gitlab会监听22端口(ssh连接)，80端口(http)及443端口(https)，我们gitlab前面加上一个haproxy做反向代理，haproxy监听443端口代理到9443端口，docker不开80端口全部都走9443端口(映射至433端口)

`docker container exec -it gitlab bash`
> 使用vi编辑gitlab的配置文件，具体配置请百度，一般默认即可

`vi /etc/gitlab/gitlab.rb`
```
################################################################################
################################################################################
##                Configuration Settings for GitLab CE and EE                 ##
################################################################################
################################################################################

##! Docs: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/gitlab.yml.md
################################################################################
# gitlab_rails['gitlab_ssh_host'] = 'ssh.host_example.com'
# gitlab_rails['time_zone'] = 'UTC'

### Email Settings
# gitlab_rails['gitlab_email_enabled'] = true
# gitlab_rails['gitlab_email_from'] = 'example@example.com'
# gitlab_rails['gitlab_email_display_name'] = 'Example'
# gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
# gitlab_rails['gitlab_email_subject_suffix'] = ''

### GitLab user privileges
# gitlab_rails['gitlab_default_can_create_group'] = true
# gitlab_rails['gitlab_username_changing_enabled'] = true

### Default Theme
# gitlab_rails['gitlab_default_theme'] = 2

```
> 配置完成后输入一下命令重新配置gitlab

`gitlab-ctl reconfigure`

#### 使用gitlab
https://blog.csdn.net/justlpf/article/details/80681853