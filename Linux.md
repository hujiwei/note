# 查看CentOS版本信息
```shell
# linux内核版本
cat /proc/version
uname -a
uname -r

# 查看linux版本
lsb_release -a
cat /etc/redhat-release

# 查看系统是64位还是32位
getconf LONG_BIT
file /bin/ls
```

# 更新系统
```shell
yum update -y
```

# 安装git、curl
```shell
yum install curl git -y
```

# 通过nvm安装nodejs
```shell
# 安装nvm
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.5/install.sh | bash

# 验证是否安装成功
source ~/.bashrc
nvm --version

# 查看nodejs版本
nvm ls-remote

# 安装nodejs
nvm install v8
node -v
```

# 安装 MySQL
```shell
# 安装 mysql
yum install https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm -y
yum install mysql-community-server -y

# 启动 mysql
systemctl start mysqld
systemctl enable mysqld

# 查找 root 的初始密码
cat /var/log/mysqld.log | grep password

# 更改密码及配置安全策略
mysql_secure_installation

# 验证 mysql 是否安装成功
mysql -uroot -p
```

# 全局安装 ThinkJS 命令
```shell
npm install -g think-cli
thinkjs --version
```

# 运行NideShop
```shell
# 下载nideshop源码
mkdir /var/www
cd /var/www
git clone https://github.com/tumobi/nideshop

# 安装依赖
cd /var/www/nideshop
npm install

# 创建数据库并导入数据
mysql -uroot -p -e "create database nideshop character set utf8mb4"
mysql -uroot -p nideshop < /var/www/nideshop/nideshop.sql

# 修改 Nideshop 的数据库配置
vim /var/www/nideshop/src/common/config/adapter.js
```
修改后：
```js
/**
 * model adapter config
 * @type {Object}
 */
exports.model = {
  type: 'mysql',
  common: {
    logConnect: isDev,
    logSql: isDev,
    logger: msg => think.logger.info(msg)
  },
  mysql: {
    handle: mysql,
    database: 'nideshop',
    prefix: 'nideshop_',
    encoding: 'utf8mb4',
    host: '127.0.0.1',
    port: '3306',
    user: 'root',
    password: '你的密码',
    dateStrings: true
  }
};
```
编译项目
```shell
# 编译项目
npm run compile

# 以生产模式启动
node production.js

# 打开另一个终端验证是否启动成功
curl -I http://127.0.0.1:8360/
# 输出 HTTP/1.1 200 OK，则表示成功
```

# 使用 PM2 管理服务
```shell
# 安装配置 pm2
npm install -g pm2

# 修改项目根目录下的 pm2.json
vim pm2.json
```
修改后的内容如下 ：
```json
{
  "apps": [{
    "name": "nideshop",
    "script": "production.js",
    "cwd": "/var/www/nideshop",
    "exec_mode": "fork",
    "max_memory_restart": "256M",
    "autorestart": true,
    "node_args": [],
    "args": [],
    "env": {

    }
  }]
}
```
如果服务器配置较高，可适当调整 max_memory_restart 和instances的值

```shell
# 启动pm2
pm2 start pm2.json

# 再次验证是否可以访问
curl -I http://127.0.0.1:8360/
```

# 使用 nginx 做反向代理
```shell
# 使用 nginx 做反向代理
yum install nginx -y
systemctl start nginx.service
systemctl enable nginx.service

# 测试本地是否可以正常访问
curl -I localhost

# 修改nginx配置
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
vim /etc/nginx/nginx.conf
```
内容如下（只需更改 server 里面的内容）
```conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
  worker_connections 1024;
}

http {
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  access_log  /var/log/nginx/access.log  main;

  sendfile            on;
  tcp_nopush          on;
  tcp_nodelay         on;
  keepalive_timeout   65;
  types_hash_max_size 2048;

  include             /etc/nginx/mime.types;
  default_type        application/octet-stream;

  # Load modular configuration files from the /etc/nginx/conf.d directory.
  # See http://nginx.org/en/docs/ngx_core_module.html#include
  # for more information.
  include /etc/nginx/conf.d/*.conf;

  server {
    listen 80;
    server_name nideshop.com www.nideshop.com; # 改成你自己的域名
    root /var/www/nideshop/www;
    set $node_port 8360;

    index index.js index.html index.htm;
    if ( -f $request_filename/index.html ){
      rewrite (.*) $1/index.html break;
    }
    if ( !-f $request_filename ){
      rewrite (.*) /index.js;
    }
    location = /index.js {
      proxy_http_version 1.1;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy true;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_pass http://127.0.0.1:$node_port$request_uri;
      proxy_redirect off;
    }

    location ~ /static/ {
      etag         on;
      expires      max;
    }
  }
}
```
```shell
# 重新启动nginx并验证nginx是否还可以正常访问
nginx -t 
systemctl restart nginx.service
curl http://127.0.0.1/
```

# 配置https访问
```shell
# 安装certbot
yum install epel-release -y
yum install certbot-nginx -y
certbot --nginx

# 如果 certbot -nginx 这步出错，则执行
pip install --upgrade --force-reinstall 'requests==2.6.0' urllib3

# 重新执行
certbot --nginx

# 配置自动更新证书
certbot renew --dry-run

# 测试浏览器使用https形式访问是否成功
```

参考：[http://www.jianshu.com/p/5d5497697b0a](http://www.jianshu.com/p/5d5497697b0a)

# centOS yum命令
语法

```shell
yum(选项)(参数)
```

选项

```shell
-h：显示帮助信息；
-y：对所有的提问都回答“yes”；
-c：指定配置文件；
-q：安静模式；
-v：详细模式；
-d：设置调试等级（0-10）；
-e：设置错误等级（0-10）；
-R：设置yum处理一个命令的最大等待时间；
-C：完全从缓存中运行，而不去下载或者更新任何头文件。
```

参数

```shell
install：安装rpm软件包；
update：更新rpm软件包；
check-update：检查是否有可用的更新rpm软件包；
remove：删除指定的rpm软件包；
list：显示软件包的信息；
search：检查软件包的信息；
info：显示指定的rpm软件包的描述信息和概要信息；
clean：清理yum过期的缓存；
shell：进入yum的shell提示符；
resolvedep：显示rpm软件包的依赖关系；
localinstall：安装本地的rpm软件包；
localupdate：显示本地rpm软件包进行更新；
deplist：显示rpm软件包的所有依赖关系。
```

实例

- 自动搜索最快镜像插件：`yum install yum-fastestmirror`
- 安装yum图形窗口插件：`yum install yumex`
- 查看可能批量安装的列表：`yum grouplist`

> 安装
```shell
yum install #全部安装 
yum install package1 #安装指定的安装包package1
yum groupinsall group1 #安装程序组group1
```

> 更新和升级
```shell
yum update #全部更新
yum update package1 #更新指定程序包package1
yum check-update #检查可更新的程序
yum upgrade package1 #升级指定程序包package1
yum groupupdate group1 #升级程序组group1
```

> 查找和显示
```shell
yum info package1 #显示安装包信息package1
yum list #显示所有已经安装和可以安装的程序包
yum list package1 #显示指定程序包安装情况package1
yum groupinfo group1 #显示程序组group1信息
yum search string #根据关键字string查找安装包
```

> 删除程序
```shell
yum remove | erase package1 #删除程序包package1
yum groupremove group1 #删除程序组group1
yum deplist package1 #查看程序package1依赖情况
```

> 清除缓存
```shell
yum clean packages #清除缓存目录下的软件包
yum clean headers #清除缓存目录下的 headers
yum clean oldheaders #清除缓存目录下旧的 headers
```