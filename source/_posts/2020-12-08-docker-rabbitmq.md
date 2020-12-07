---
title: Docker 安装 RabbitMQ
date: 2020-01-18 00:59:17
tags: 
- Docker
- Linux
- RabbitMQ
- 虚拟化
- 容器
- 安装部署
- 消息队列
categories: 
- Docker
- RabbitMQ
---

> ## Docker 安装 RabbitMQ

**RabbitMQ**是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）

**消息系统允许软件、应用相互连接和扩展。这些应用可以相互链接起来组成一个更大的应用，或者将用户设备和数据进行连接。消息系统通过将消息的发送和接收分离来实现应用程序的异步和解偶。**

或许你正在考虑进行数据投递，非阻塞操作或推送通知。或许你想要实现发布／订阅，异步处理，或者工作队列。所有这些都可以通过消息系统实现。

RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。

笔者在基于`Django`的`SaaS`开发中，使用`RabbitMQ`作为消息队列，所以最近学习了`Docker`，于是想部署基于容器的RabbitMQ。

<!-- more -->

## 查看可用的 RabbitMQ版本

访问 Nginx 镜像库地址： https://registry.hub.docker.com/_/rabbitmq/

可以通过 Sort by 查看其他版本的 Nginx，默认是最新版本 **rabbitmq:latest**。

**<u>ATTENTION:</u>**

如果`docker pull rabbitmq`拉下来的镜像不带`management`，启动rabbitmq后是无法打开管理界面的，所以我们要下载带`management`插件的rabbitmq.

我们还可以用 **docker search rabbitmq:management** 命令来查看可用版本：

```linux
[root@iZwz9cdthtgi8exxs9m71vZ /bin]#docker search rabbitmq:management
NAME                                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
macintoshplus/rabbitmq-management   Based on rabbitmq:management whit python and…   6                                       [OK]
xiaochunping/rabbitmq               xiaochunping/rabbitmq:management   2018-06-30   4                                       
transmitsms/rabbitmq-sharded        Fork of rabbitmq:management with sharded_exc…   0                                       
yunyan2140/rabbitmq                 docker pull rabbitmq:management                 0
```

## 取rabbitmq:management镜像

这里我们拉取官方的最新版本的镜像：

```shell
$ docker pull rabbitmq:management
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

## 查看本地镜像

使用以下命令来查看是否已安装了 rabbitmq：

```shell
$ docker images rabbitmq
```

```linux
[root@iZwz9cdthtgi8exxs9m71vZ /bin]#docker images rabbitmq
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rabbitmq            management          263c941f71ea        11 days ago         186MB
rabbitmq            latest              f50f482879b3        11 days ago         156MB
```

## 运行容器

安装完成后，我们可以使用以下命令来运行 nginx 容器：

### 方式一 / 默认账号密码为guest

```shell
docker run -d --hostname rabbit-first --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:management
```

### 方式二 / 设置用户名和密码

```shell
docker run -d --hostname my-rabbit --name rabbit -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password -p 15672:15672 -p 5672:5672 rabbitmq:management
```



### 参数说明

- **--name rabbit-first**：容器名称。
- **-p 15672:15672**： 端口进行映射，将本地 15672端口映射到容器内部的 15672 端口。
- **-d nginx**： 设置容器在在后台一直运行。

### RabbitMQ 端口号解析

- **4369 (epmd), 25672 (Erlang distribution)**

  Epmd 是 Erlang Port Mapper Daemon 的缩写，在 Erlang 集群中相当于 dns 的作用，绑定在4369端口上。

- **5672, 5671 (AMQP 0-9-1 without and with TLS)**

  AMQP 是 Advanced Message Queuing Protocol 的缩写，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，专为面向消息的中间件设计。基于此协议的客户端与消息中间件之间可以传递消息，并不受客户端/中间件不同产品、不同的开发语言等条件的限制。Erlang 中的实现有 RabbitMQ 等。

- **15672 (if management plugin is enabled)**

  通过 `http://serverip:15672` 访问 RabbitMQ 的 Web 管理界面，默认用户名密码都是 guest。（注意：RabbitMQ 3.0之前的版本默认端口是55672，下同）

- **61613, 61614 (if STOMP is enabled)**

  Stomp 是一个简单的消息文本协议，它的设计核心理念就是简单与可用性，[官方文档](http://stomp.github.com/stomp-specification-1.1.html)，实践一下 Stomp 协议需要：

  - 一个支持 stomp 消息协议的 messaging server (譬如activemq，rabbitmq）；

  - 一个终端（譬如linux shell);

  - 一些基本命令与操作（譬如nc，telnet)

- **1883, 8883 (if MQTT is enabled)**
    MQTT 只是 IBM 推出的一个消息协议，基于 TCP/IP 的。两个 App 端发送和接收消息需要中间人，这个中间人就是消息服务器（比如ActiveMQ/RabbitMQ），三者通信协议就是 MQTT

## 安装成功

由于我们是安装rabbitmq:management版本，是自带Web 管理界面的，所以通过浏览器可以直接访问 15672 端口的 RabbitMQ 服务：

***地址***：[RabbitMQ Management](http://47.112.240.167:15672/)

### ***登录页面***

{% asset_img rabbitmq-login.png %}

我们`docker run `是没有指定默认用户和密码，所以默认账号密码为

```ini
Username:guest
Password:guest
```

### ***主页面***

{% asset_img rabbitmq-success.png %}