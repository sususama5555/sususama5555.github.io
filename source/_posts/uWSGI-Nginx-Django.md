---
title: 使用 uWSGI + Nginx 部署 Django 项目 
date: 2020-05-20 19:47:02
tags: uWSGI + Nginx + Django
categories: Django
---
> 摘要

在开发<kbd>Django</kbd>框架时，我们通常使用<kbd>python manage.py runserver</kbd>来运行本地服务,但是这只适用开发环境中使用，此时启动的Django项目通过127.0.0.1:8000的本地端口进行转发，也只能在本地环境下进行访问。那么我们基于Django的SaaS是怎样发布在公网、局域网环境下，并且通过域名访问站点的呢？通常的方案是：我们就需要用到<kbd>uWSGI</kbd>配合<kbd>Nginx(Apache)</kbd>进行代理转发。以下是笔者在实践此方案的过程中整理的一些要点。
<!--more-->
> 正文

## 一、uWSGI简介

uWSGI是一个快速的、纯C语言开发的、自维护的、对开发者友好的WSGI服务器，旨在提供专业的Python web应用发布和开发。可使用C/C++/Objective-C来为uWSGI编写插件。我们在根目录下的requirements.py中可以找到uWSGI的依赖，Windows环境本地安装依赖的时候一般都会注释掉它，因为uWSGI是运行在Linux上的服务器。可能大家对WSGI、uwsgi、uWSGI这几个概念很头疼，简单地说，WSGI是一个规范协议，定义了Web服务器如何与Python应用程序进行交互；uwsgi和WSGI一样是通信协议，是uWSGI服务器的单独形式，用于自定义传输类型；而uWSGI是重头戏，他是一个web服务器,实现了WSGI协议、uwsgi协议。以下是WSGI，uwsgi，uWSGI的实现过程图解。


![uWSGI](uWSGI-Nginx-Django/uwsgi.png)

## 二、Nginx简介

相比uWSGI，Nginx的知名度高了许多，它是一个开源的、支持高性能、高并发的代理服务软件，Nginx不但是一个优秀的web服务软件,还可以作为反想代理和负载均衡,以及缓存服务或使用。
实际上，一个uWSGI的web服务器，再加上Django这样的web框架，就已经可以实现网站的功能了。那为什么还需要Nginx呢？经过笔者查阅资料，总结有以下几点：

### 2.1、安全问题：
程序不能直接被浏览器访问到，而是通过Nginx只开放某个接口，uWSGI本身是内网接口，这样运维人员在Nginx上加上安全性的限制，可以达到保护程序的作用；

### 2.2、载均衡问题：
一个uWSGI很可能不够用，即使开了多个work也是不行，毕竟一台机器的cpu和内存都是有限的，有了Nginx做代理，一个Nginx可以代理多台uWSGI完成负载均衡；

### 2.3、静态文件问题：
用Django或是uWSGI这种东西来负责静态文件的处理是很浪费的行为，而且他们本身对文件的处理也不如Nginx好，所以整个静态文件的处理都直接由nginx完成。


## 三、uWSGI + Nginx 部署 Django 网站的实践
接下来笔者将使用uWSGI+Nginx搭建Django网站，此方案的架构图如下：
![uWSGI](uWSGI-Nginx-Django/uwsgi+nginx+django.png)

###	3.1、安装配置uWSGI：

#### 3.1.1安装uWSGI：
```
pip install uwsgi
```
也可以指定版本安装；

#### 3.1.2	uWSGI配置：
实际上，我们可以uwsgi命令配合参数来启动Django项目，但是我们可以把这些参数放在uwsgi9090.ini配置文件中，记录配置并且实现后台运行uWSGI：
```
[uwsgi]
socket = 127.0.0.1:9090
master = true         //主进程
vhost = true          //多站模式
no-site = true        //多站模式时不设置入口模块和文件
workers = 2           //子进程数
reload-mercy = 10     
vacuum = true         //退出、重启时清理文件
max-requests = 1000   
limit-as = 512
buffer-size = 30000
pidfile = /var/run/uwsgi9090.pid    //pid文件，用于下面的脚本启动、停止该进程
daemonize = /website/uwsgi9090.log
```
如上配置，接下来运行uwsgi uwsgi9090.ini即可uWSGI后台启动Django；

#### 3.1.3	Django项目根目录的settings.py中配置收集静态资源：
配置完毕后运行python manage.py collectstatic
```python
# 静态资源访问的起始url
STATIC_URL = '/static/'
# 指定静态资源所在的目录
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
# 设置收集静态资源的路径(部署时使用)
STATIC_ROOT = os.path.join(BASE_DIR, 'collect_static/')
```

### 3.2	安装配置Nginx：
#### 3.2.1、安装编译工具及库文件：
```shell
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```
#### 3.2.2、编译安装Nginx，版本可自定义：
```shell
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.6.2.tar.gz
tar zxvf nginx-1.6.2.tar.gz
cd nginx-1.6.2
./configure --prefix=/usr/local/nginx
make
make install
```

#### 3.2.3、找到Nginx的安装目录，打开conf/nginx.conf文件，修改server配置： 
```
server {
        listen 443 default_server ssl;  //https必须用ssl和443端口，http可以用80
        server_name  xxx.xxx.com;     //个人的已经注册过的域名
        location / {  
		include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9090;              //必须和uwsgi中的设置一致
            uwsgi_param UWSGI_SCRIPT demosite.wsgi;  //入口文件，即wsgi.py相对于项目根目录的位置，“.”相当于一层目录
            uwsgi_param UWSGI_CHDIR /demosite;       //项目根目录
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }
```

#### 3.2.4、使用systemctl restart nginx重启Nginx服务器。

## 四、项目启动及维护
使用`ss -lntpd | grep nginx（uwsgi）`和`ps -ef | grep nginx（uwsgi）`来确认nginx、uwsgi是否启动成功、转发代理的端口是否正确。
如果为启动失败，请确认配置是否正确，使用systemctl restart nginx重启nginx，使用killall -9 uwsgi杀死uwsgi服务，使用uwsgi uwsgi9090.ini重启uwsgi和Django项目；如果一切正常，我们可以在浏览器中输入nginx.conf中配置的server_name域名，来访问已启动的Django web项目，这样就实现了uWSGI+Nginx部署Django web项目了。




