## vue 报错相关

### 【npm install】node-sass 安装报错解决办法
#### 全局安装淘宝NPM镜像
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
#### 重新install
`cnpm install`

### npm run serve 报错 ENOSPC:System limit for number of file watchers reached
1. 添加以下内容到 /etc/sysctl.conf 文件末尾
> fs.inotify.max_user_watches = 524288
2. 执行以下命令并重启ide
> sudo sysctl -p --system



### nodejs

#### node.js: cannot find module 'request'

> ```js
> npm init --yes
> npm install request --save
> ```
