---
layout: post
title: "云服务器部署Django项目踩坑记（Nginx+Django+uwsgi）"
date: 2019-09-13
description: "云服务器、Django部署"
tag: 云服务器、Django部署
---

### 准备工具

* 既然是在云服务器上部署，那肯定得有一个**云服务器**，阿里云、华为云之类的一般有学生优惠什么的，对于学习来说，还是相当划算的
* **FileZilla**：这是一款文件传输的软件，可以把项目代码啥的上传到你的云服务器上面，当然也可以把代码放在**GitHub**来，然后在云服务器上下载下来
* 还有如果是 *Windows* 系统的话，为了操控方便可以下个**Xshell**，不过我懒得下，而且觉得终端**ssh**连接其实也不错

### 开始

* 首先连接云服务器`ssh -p 22 root@公网ip`，然后输入你的密码就好了，连接上是这样的![](https://FXHao.github.io/images/posts/云服务器部署/图片1.png)

* 然后最好新建一个**用户名和组**，因为**root**是最高权限，以最高权限运行某些程序不太好，后面有个坑就是因为这个原因，新建的用户目录会在`/home`下，之后的文件啥的可以放用户目录里面

  ```
  groupadd fxh    # 创建一个用户组为fxh
  useradd -g fxh fxh  # 创建一个用户并指定其所属的组
  ```

### 软件的安装

* 首先先更新下环境，安装必要的依赖包之类的

  ```
  apt-get update
  sudo apt-get install python-dev
  sudo apt-get install python3-pip
  sudo apt-get install libxml*
  sudo apt-get install net-tools
  sudo apt-get install lsof
  ```

* 安装**MySQL**

  ```
  apt-get install mysql-server
  apt-get install mysql-client
  apt-get install libmysqlclient-dev
  ```

  * 数据库安装完后改个密码啥的（当然安装的时候可以设置，但我是一路回车下来的。。）

    ```
    mysql> set password for 用户名@localhost = password('新密码'); 
    ```

  * 有个**坑**就是，项目连接数据库时会连不上，得去云服务器更改安全组添加规则，把数据库的端口添加进去（还有的地方也需要开通端口才行，看你的程序要求吧，用到哪个开哪个）![](https://FXHao.github.io/images/posts/云服务器部署/图片3.png)

* 安装虚拟环境**virtualenv**，独立开开发环境（怎么安装自行百度吧）

  * 可以在本地的虚拟环境把里面的包打包起来，然后把打包起来的文件传给云服务器，在云服务器中建立虚拟环境安装包

    ```
    pip freeze > 1.txt  # 打包成文件
    pip install -r 1.txt  # 安装
    ```

  * 不过我用上面的方法有的会安装失败，我懒得看哪些没安装成功，而且有些模块是**pip**安装不了的，我就直接把本地虚拟环境中**python3**下的**site-packages**给复制一份传给云服务器，然后扔进`.virtualenvs/Django_py3/lib/python3.5/site-packages/`去替换掉就好，暴力方便！

* 安装**Nginx**

### 测试

* 在把你项目代码传到云服务器上后，最好先测试下代码是否可以运行

  ```
  python3 manage.py runserver 0.0.0.0:8000
  ```

  * 然后用你的本地浏览器来访问下（`公网ip:8000`，记得开端口），只要能访问成功就好了

### 用Nginx和uwsgi部署

* 上面虽然可以染别人访问到你的项目，但终端一关闭就某得了

* **Nginx.conf**配置

  ```
  server {
          listen 80;
          server_name  公网ip;  # 改成域名也可以
          access_log /home/fxh/code/Django/dailyfresh/nginx_access.log;
          error_log /home/fxh/code/Django/dailyfresh/nginx_error.log;
          charset utf-8;
          root /home/fxh/code/Django/dailyfresh;  # 项目目录
          
          location / {
                  #包含uwsgi的请求参数
                  include uwsgi_params;
                  #转交给uwsgi
                  uwsgi_pass 127.0.0.1:8080;  # 这个端口一定要与等下uwsgi配置的一样
                  #uwsgi_pass dailyfresh;
          }
          location /static {
                  # 指定存放静态文件的目录
                  alias /var/www/dailyfresh/static/;  # 今天文件存放目录(自己创建，然后收集)
          }
          location = / {
                  # 传递请求给静态文件服务器的nginx
                  proxy_pass http://公网ip;
          }
  ```

  * 配置完记得重启

* **uwsgi.ini** 配置

  ```
  [uwsgi]
  #uid = fxh
  #gid = fxh
  #使用nginx连接时使用
  socket=127.0.0.1:8080
  #socket=192.168.0.12:8080
  #直接做web服务器使用  python manage.py runserver ip:port
  #http=192.168.0.12:9080
  #项目目录
  cdir=/home/fxh/code/Django/dailyfresh
  #项目中wsgi.py文件的目录，相对于项目目录
  wsgi-file=dailyfresh/wsgi.py
  # 指定启动工作的进程数
  processes=4
  # 指定工作进程中的线程数
  threads=2
  master=True
  # 保存启动后主进程的pid
  pidfile=uwsgi.pid
  # 设置uwsgi后台运行
  daemonize=uwsgi.log
  # 设置虚拟环境的路径
  virtualenv=/root/.virtualenvs/Django_py3
  #uid=root
  ```

  * 启动：`uwsgi --ini uwsgi.ini`
    * 启动后记得**ps**查看下是否启动成功
    * 启动失败的坑：
      * 查看log日志里的错误，如果是 `running ....root !`啥的，就把前面两行打开

* 对了，项目的**settings.py**文件中要修改

  ```
  DEBUG = False
  ALLOWED_HOSTS = ['*']  # 意思就是允许任何请求，只有列表中的host才可以访问
  ```

  

### 验证是否部署成功

* 本地浏览器访问
  * 如果访问成功，那么恭喜恭喜，部署完成了
  * 如果访问失败502页面，那在试下访问静态资源，如果也不行，那去看看**Nginx**的配置，如果可以的话，可能就是**uwsgi**配置错误了，或者试试关闭**uwsgi**，用`uwsgi --ini uwsgi.ini --plugin python3`来启动，我就是这么解决的
  * 如果还不行，那.....就砸电脑吧。。开玩笑，还不行的话就查看下日志文件，细心检查下其余配置

### 完成----第一次云服务器部署项目，本文仅记录本人踩的一些坑，若有不正确的地方，望指点！！