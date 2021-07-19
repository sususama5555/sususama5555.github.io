---
title: Prometheus部署、集成与可视化
date: 2021-07-19 23:09:52
tags: 
- Prometheus
- 云原生
- 监控
- Golang
categories: 
- Golang
- Prometheus
---

## 前言

最近专注于Go语言学习，除开兴趣使然，也意识到他在云原生领域大放异彩，众多开源项目也是鼎鼎大名，例如**Docker**、**Kubernetes**、**InfluxDB**、**Grafana**等，这里从监控场景经常接触的**Prometheus**开始，掌握它的基础架构、使用场景和部署与集成。

Prometheus是由SoundCloud开发的开源监控报警系统和时序列数据库，2016年，由Google发起的Linux基金会旗下的原生云基金会（Cloud Native Computing Foundation）将Prometheus纳入其第二大开源项目。

在云原生时代，随着容器技术的迅速发展，Kubernetes生态已然成为大家追捧的容器集群管理系统，而Prometheus 作为 CNCF 生态圈中的重要一员，其活跃度仅次于 Kubernetes，现已广泛用于 Kubernetes 集群的监控系统中。

Prometheus 提供了一整套监控体系，包括数据的采集、数据存储、报警,、甚至是绘图(不太好用，官方也推荐使用 Grafana)。

以下暂且介绍Prometheus的部署、配置和可视化。

<!-- more -->

## 部署

有许多种方式安装Prometheus，包括使用预编译二进制文件、从源代码构建和Docker安装，这里只聊更科学的docker安装。

按正常步骤：

- `docker search prometheus` - 查找可用镜像；
- `docker pull prometheus` - 拉取镜像到本地；
- `docker run -p 9090:9090 prom/prometheus` - 运行一个Prometheus实例，暴露9090端口。

虽然很便捷，但是为了后续更好地使用，我们需要定义一些参数。

```shell
docker run \
	-d \ # 后台运行
	-p 9090:9090 \ # 绑定端口
	--name=prometheus \ # 为container命名
	-v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \ # 绑定及挂载数据卷
	prom/prometheus \ # 使用的镜像 
	--web.enable-lifecycle \ # 允许热加载配置
	--web.enable-admin-api \ # 开启对 admin api 的访问权限
	--config.file=/etc/prometheus/prometheus.yml \ # 使用额外的数据卷挂载配置文件
```

其中 **-v** 和  **--config.file** 很重要，使得我们可以把容器中Prometheus的配置挂载到宿主机的文件下，不用进入容器修改配置，

**--web.enable-lifecycle** 也很使用，开始使用时，我们需要经常修改配置文件，达到添加监控采集项的目的，需要重启容器才能使配置生效，这个参数允许我们POST请求 `http://47.112.240.167/-/reload`，达到热加载配置的效果，使用`curl -XPOST http://47.112.240.167/-/reload`进行reload。

访问http://47.112.240.167:9090即可进入页面。

{% asset_img prometheus-get-start.png %}

由于现在只安装了Prometheus Server，和zabbix类似，没有数据，需要借助agent，这里叫Exporter来采集数据。Exporter会处理好采集上来的数据，暴露接口给Prometheus Server定时拉取，是基于http协议的pull监控采集方式。所以我们需要安装队友的Exporter，并且写进Prometheus Server的配置里。

## 配置

这里介绍安装宿主机和容器的node_exporter，也就是最常见的服务器的监控采集Exporter。

### 普通安装node-exporter

直接拉取线上地址即可，开箱即用

```shell
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.0.linux-amd64.tar.gz
tar xvf node_exporter-0.18.0.linux-amd64.tar.gz
mv node_exporter-0.18.0.linux-amd64 /usr/local/bin/node_exporter
```

然后`node_exporter &`启动守护进程（后台任务）。

这里也可以创建用户及绑定systemd服务

```shell
groupadd prometheus
useradd -g prometheus -m -d /var/lib/prometheus -s /sbin/nologin prometheus
chown prometheus.prometheus -R /usr/local/prometheus
```

```shell
cat > /etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/
After=network.target
[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

这样就可以使用`systemctl start node_exporter`启动，也可以用status和restart等来管理这个服务。

### docker安装node-exporter

使用docker更加简单，

```shell
docker run -d -p 9911:9100 \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  --net="host" \
  prom/node-exporter
```

一条命令即可启动node-exporter实例，宿主机暴露的端口是9091，我们可以使用http://47.112.240.167:9911/metrics来进行查看Exporter收集到的数据，现在需要把Exporter接入到Prometheus Server中，才能实现展示。

### 配置prometheus.yml

打开配置文件prometheus.yml，在配置中添加两个node-exporter：

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s
 
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090'] # Prometheus Server节点
        labels:
          instance: prometheus
 
  - job_name: server
    static_configs:
      - targets: ['172.18.255.15:9100'] # 宿主机node-exporter
        labels:
          instance: server-172.18.255.15

      - targets: ['172.17.0.5:9111'] # 容器node-exporter
        labels:
          instance: docker-172.17.0.5
```

然后重载Prometheus Server配置：

`curl -XPOST http://ip:9090/-/reload`

这样就可以在http://47.112.240.167:9090/targets里找到两个Expoeter：

{% asset_img prometheus-targets.png %}

## 数据展示

如何使用Prometheus是一个大学问，可以根据采集数据进行监控展示，也可以进行指标告警。这里简单介绍下数据的展示，官方文档介绍了三种可视化方案：

- EXPRESSION BROWSER（Prometheus自带的/graph界面）
- **GRAFANA（绝配）**
- CONSOLE TEMPLATES（使用 Go 模板语言创建自定义控制台，有学习和开发曲线）

### EXPRESSION BROWSER

Prometheus 提供了一种名为 PromQL（Prometheus Query Language）的函数式查询语言，可以让用户实时选择和聚合时间序列数据。表达式的结果可以显示为图形，在 Prometheus 的表达式浏览器中查看为表格数据，也可以通过 HTTP API 由外部系统使用。

点开Graph界面，可以输入PromQL进行查询，这里执行up语句：

{% asset_img prometheus-up1.png %}

{% asset_img prometheus-up2.png %}

即可看到简单的图形数据，这里有Prometheus Server和两个node-exporter的采集。

但是Prometheus的图形功能比较复杂且不直观好用，业界最优的解决方案是使用**Grafana**展示。

## 接入Grafana

> To be continued...
>

-----------------

参考资料：

[Installation | Prometheus（官方文档）](https://prometheus.io/docs/prometheus/latest/installation/)

[prometheus/prometheus: The Prometheus monitoring system and time series database. (github.com)](https://github.com/prometheus/prometheus)

