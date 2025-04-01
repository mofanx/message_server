- 编译
``` 
# 克隆仓库
git clone https://github.com/gotify/server

# 安装依赖
pkg install -y golang git make sqlite nodejs
npm install -g yarn

# 设置Go环境变量
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 进入项目
cd server

# 下载go开发工具
make download-tools

# 构建UI
cd ui
yarn install

# 设置Node.js环境变量以解决加密模块问题
export NODE_OPTIONS=--openssl-legacy-provider

# 构建
yarn build

# 返回 server
cd ..

# 设置版本信息
export LD_FLAGS="-w -s -X main.Mode=prod"

# 直接构建当前架构的二进制文件
echo "构建Gotify服务器..."
go build -ldflags="$LD_FLAGS" -o gotify-server

# 创建gotify数据目录
mkdir -p $HOME/go/gotify/data

# 创建log日志目录
mkdir -p $HOME/go/gotify/logs

# 移动构建后的go程序到指定文件夹
mv gotify-server $HOME/go/gotify/


```
- 创建配置文件`config.yml`
`vi $HOME/go/gotify/config.yml`
``` 
server:
  keepaliveperiodseconds: 0
  listenaddr: ""
  port: 8326

  ssl:
    enabled: false
    redirecttohttps: true
    listenaddr: ""
    port: 8443
    certfile:
    certkey:
    letsencrypt:
      enabled: false
      accepttos: false
      cache: data/certs
      hosts:

  stream:
    pingperiodseconds: 45
    allowedorigins:

database:
  dialect: sqlite3
  connection: data/gotify.db

defaultuser:
  name: admin
  pass: admin
passstrength: 10
uploadedimagesdir: data/images
pluginsdir: data/plugins
registration: false
```

- 创建 `ecosystem.config.js`
``` 
module.exports = {
  apps: [
    {
      name: 'gotify',
      script: '/data/data/com.termux/files/home/go/gotify/gotify-server',
      interpreter: 'none',
      args: '-c /data/data/com.termux/files/home/go/gotify/config.yml',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      watch: false,
      max_memory_restart: '500M',
      restart_delay: 3000,
      max_restarts: 10,
      kill_timeout: 3000,
      error_file: '/data/data/com.termux/files/home/go/gotify/logs/error.log',
      out_file: '/data/data/com.termux/files/home/go/gotify/logs/output.log',
      merge_logs: true,
      log_date_format: 'YYYY-MM-DD HH:mm:ss'
    }
  ]
};
```

- 使用`pm2`来运行
``` 
# 确认pm2已安装
npm install -g pm2

pm2 start ecosystem.config.js
```
