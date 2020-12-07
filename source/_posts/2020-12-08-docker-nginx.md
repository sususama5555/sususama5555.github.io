---
title: Docker 安装 Nginx
date: 2019-12-08 00:13:14
tags: 
- Docker
- Linux
- Nginx
- 容器
- 虚拟化
- 安装部署
categories: 
- Docker
- Nginx
---

> ## Docker 安装 Nginx

之前学习了安装和部署Docker，现在使用Docker安装最简单的实例——Nginx。

Nginx 是一个高性能的 HTTP 和反向代理 web 服务器，同时也提供了 IMAP/POP3/SMTP 服务 。

## 查看可用的 Nginx 版本

访问 Nginx 镜像库地址： https://hub.docker.com/_/nginx?tab=tags。

可以通过 Sort by 查看其他版本的 Nginx，默认是最新版本 **nginx:latest**。

我们还可以用 **docker search nginx** 命令来查看可用版本：

<!-- more -->

```linux
[root@iZwz9cdthtgi8exxs9m71vZ /data/docker]#docker search nginx
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                              Official build of Nginx.                        14126               [OK]                
jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   1920                                    [OK]
richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable of…   796                                     [OK]
linuxserver/nginx                  An Nginx container, brought to you by LinuxS…   131                                     
jc21/nginx-proxy-manager           Docker container for managing Nginx proxy ho…   118                                     
tiangolo/nginx-rtmp                Docker image with Nginx using the nginx-rtmp…   106                                     [OK]
bitnami/nginx                      Bitnami nginx Docker Image                      91                                      [OK]
alfg/nginx-rtmp                    NGINX, nginx-rtmp-module and FFmpeg from sou…   81                                      [OK]
jlesage/nginx-proxy-manager        Docker container for Nginx Proxy Manager        74                                      [OK]
nginxdemos/hello                   NGINX webserver that serves a simple page co…   63                                      [OK]
nginx/nginx-ingress                NGINX Ingress Controller for Kubernetes         46                                      
privatebin/nginx-fpm-alpine        PrivateBin running on an Nginx, php-fpm & Al…   43                                      [OK]
nginxinc/nginx-unprivileged        Unprivileged NGINX Dockerfiles                  25                                      
schmunk42/nginx-redirect           A very simple container to redirect HTTP tra…   19                                      [OK]
staticfloat/nginx-certbot          Opinionated setup for automatic TLS certs lo…   16                                      [OK]
centos/nginx-112-centos7           Platform for running nginx 1.12 or building …   15                                      
nginx/nginx-prometheus-exporter    NGINX Prometheus Exporter                       15                                      
centos/nginx-18-centos7            Platform for running nginx 1.8 or building n…   13                                      
raulr/nginx-wordpress              Nginx front-end for the official wordpress:f…   13                                      [OK]
mailu/nginx                        Mailu nginx frontend                            8                                       [OK]
sophos/nginx-vts-exporter          Simple server that scrapes Nginx vts stats a…   7                                       [OK]
bitnami/nginx-ingress-controller   Bitnami Docker Image for NGINX Ingress Contr…   7                                       [OK]
bitwarden/nginx                    The Bitwarden nginx web server acting as a r…   7                                       
wodby/nginx                        Generic nginx                                   1                                       [OK]
ansibleplaybookbundle/nginx-apb    An APB to deploy NGINX                          1                                       [OK]

```

## 取最新版的 Nginx 镜像

这里我们拉取官方的最新版本的镜像：

```shell
$ docker pull nginx:latest
```

> ### 解决docker pull 速度慢问题

#### 将docker镜像源修改为国内源

在 /etc/docker/daemon.json 文件中添加以下参数（没有该文件则新建）：

```ini
{
  "registry-mirrors": ["https://9cpn8tt6.mirror.aliyuncs.com"]
}
```

#### 服务重启

```shell
systemctl daemon-reload
systemctl restart docker
```

### 查看本地镜像

使用以下命令来查看是否已安装了 nginx：

```shell
$ docker images
```

```linux
[root@iZwz9cdthtgi8exxs9m71vZ /data/docker]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rabbitmq            latest              f50f482879b3        11 days ago         156MB
nginx               latest              bc9a0695f571        12 days ago         133MB
hello-world         latest              bf756fb1ae65        11 months ago       13.3kB

```

也可以指定镜像：`docker images nginx`

### 运行容器

安装完成后，我们可以使用以下命令来运行 nginx 容器：

```shell
$ docker run --name nginx-first -p 9000:80 -d nginx
```

参数说明：

- **--name nginx-test**：容器名称。
- **-p 8080:80**： 端口进行映射，将本地 8080 端口映射到容器内部的 80 端口。
- **-d nginx**： 设置容器在在后台一直运行。

### 安装成功

最后我们可以通过浏览器可以直接访问 8080 端口的 nginx 服务：

{% asset_img nginx-success.png %}

