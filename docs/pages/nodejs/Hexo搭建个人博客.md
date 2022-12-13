## 安装NodeJS和Git ##
##### 下载NodeJS #####
`wget https://npm.taobao.org/mirrors/node/v12.9.1/node-v12.9.1-linux-x64.tar.xz`
##### 解压 #####
```
xz -d node-v12.9.1-linux-x64.tar.xz
tar -xvf node-v12.9.1-linux-x64.tar
```
##### 设置软连接 #####
```
ln -s /app/node/node-v12.9.1-linux-x64/bin/node /usr/local/bin/node
ln -s /app/node/node-v12.9.1-linux-x64/bin/npm /usr/local/bin/npm
```
##### 查看NodeJS版本 #####
`node -v`
##### 安装Hexo #####
`npm install hexo-cli -g`
##### 安装Git #####
`yum -y install git`

## 搭建Hexo博客 ##
##### 把hexo命令添加到全局 #####
```
ln -s /app/node/node-v12.9.1-linux-x64/lib/node_modules/hexo-cli/bin/hexo /usr/local/bin/hexo
```
##### 部署hexo博客环境 #####
```
cd /app/hexo/
hexo init
```
##### 启动 #####
`hexo s &`

##### 博客主题 #####
> https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/quick-start.html

## 后台运行 ##
##### 安装pm2 #####
`npm  install -g pm2`
##### 把hexo命令添加到全局 #####
```
ln -s /app/node/node-v12.9.1-linux-x64/lib/node_modules/pm2/bin/pm2 /usr/local/bin/pm2
```
##### 在博客根目录下面创建一个hexo_run.js #####
```
//run
const { exec } = require('child_process')
exec('hexo server',(error, stdout, stderr) => {
        if(error){
                console.log('exec error: ${error}')
                return
        }
        console.log('stdout: ${stdout}');
        console.log('stderr: ${stderr}');
})
```
##### 运行脚本 #####
`pm2 start hexo_run.js --watch`
> 停止 pm2 stop hexo_run.js </br>
> 重启 pm2 restart hexo_run.js 或 pm2 reload hexo_run.js --watch