---
date: 2022-01-11 09:46:36
tags:
  - 服务器
  - Flask部署
title: Flask + gunicorn + nginx 部署腾讯云服务器
---

# Flask + gunicorn + nginx 部署腾讯云服务器

#### 1. 通过 git 或者 ftp 工具将项目放入服务器

#### 2. 安装 python 环境（注意：安装完虚拟环境后的所有操作均需要在虚拟环境中进行）

```shell
sudo apt-get install python3

sudo apt-get install python3-pip  # 安装pip
```

腾讯云是自带 python2 环境的，如果使用的是 python2 则无需安装

2.1 创建虚拟环境

```shell
sudo apt-get install python-virtualenv
sudo easy_install virtualenvwrapper # 安装虚拟环境

# 1. 创建目录用来存放虚拟环境
 mkdir $HOME/.virtualenvs
# 2. 在~/.bashrc中最后添加行：
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3  # 你的python路径，这样创建虚拟环境默认为python3
export VIRTUALENVWRAPPER_VIRTUALENV=/home/python/.local/bin/virtualenv # 你的virtualenv路径，通过命令 which virtualenv 可以查看路径
source /home/python/.local/bin/virtualenvwrapper.sh
#3. 重启:
source ~/.bashrc
```

​ 2.2 虚拟环境的使用

​ 创建 `mkvirtualenv -p [python的路径] [虚拟环境名称]` # 如果在`~/.bashrc`文件中指定了 python 的路径，这里则不需要添加

​ 切换 `workon [虚拟环境名称]`

​ 退出虚拟环境 `deactivate`

​ 删除。 `rmvirtualenv [虚拟环境名称]`

2.3 进入你创建的虚拟环境

#### 3. 安装 gunicorn

```shell
sudo apt-get install gunicorn  # 需要在虚拟环境中进行安装
pip install  #安装模块
```

3.1 在项目目录创建 gunicorn.conf 的文件写入以下内容

```shell
#绑定服务器套接字 使用云服务器注意时内网地址
bind='127.0.0.1:5000'
#等待服务的客户的数量
backlog=2048
#用于处理工作进程的数量
workers=2
#处理请求的工作线程数，使用指定数量的线程运行每个worker
threads=2
#守护Gunicorn进程，即后台运行
daemon=True
# 日誌輸出文件
errorlog='/home/python/error.log'
errorlog = '-' # 记录到标准输出
loglevel = 'debug' # 记录等级
```

3.2 启动`gunicorn`

```shell
# 通过配置文件启动
gunicorn -c gunicorn.conf [项目启动文件名]:app. # flask项目的app,如果更改，也要修改成对应的名字
# 命令行启动
gunicorn --workers=2 --bind=127.0.0.1:8000 --log-level=debug --access-logfile /home/ubuntu/error.logo demo:app
# level=debug 输出日志的等级为debug
# 将日志文件输入到指定文件  errorlog = '[file]'
# 这种方式仅适合测试时使用
```

​ 3.3 查看是否启动

```shell
ps ajx | grep gunicorn
# 如果出现三个进程代表启动成功
```

​ 3.4 通过地址访问

#### 4.安装 nginx

```shell
sudo apt-get install nginx
```

4.1 启动 nginx

```shell
sudo service nginx start
```

4.2 查看是否成功启动

```shell
直接访问127.0.0.1或者服务器地址，如果出现nginx的欢迎界面则代表启动成功
```

4.3 修改配置

主要添加的是`server`和`location`两个配置

```shell
http {
    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    ##
    # Virtual Host Configs
    ##
    # --------主要添加时这一段
    server{
        listen 80; # 监听的端口
    server_name 127.0.0.1:5000;
        location /static {  # url的转发规则
                root 你的路径;
                }
        location / {
                # 转发到gunicorn服务器 这个地址是外网地址
                proxy_pass http://127.0.0.1:5000;
                # 请求转发到多个gunicorn服务器
                # proxy_pass http://flask;
                # 设置请求头，并将头信息传递给服务器端
                proxy_set_header Host $host;
                # 设置请求头，传递原始请求ip给gunicorn 服务器
                proxy_set_header X-Real-IP $remote_addr;

                }
}
# --------------end
    # 下面这两个需要注释，否则nginx的配置文件无法生效
    #include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;
}
```
