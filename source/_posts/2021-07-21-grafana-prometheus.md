---
title: Grafana数据可视化
date: 2021-07-21 00:41:38
tags: 
- Grafana
- 云原生
- 数据可视化
- Golang
categories: 
- Golang
- Grafana
---

## Grafana简介

> Grafana allows you to query, visualize, alert on and understand your metrics no matter where they are stored. Create, explore, and share dashboards with your team and foster a data driven culture.

从上文得知，Prometheus部署配置完成后，官方提供了三种可视化方案：

- EXPRESSION BROWSER（Prometheus自带的`/graph`界面）
- **GRAFANA（绝配）**
- CONSOLE TEMPLATES（使用 Go 模板语言创建自定义控制台，有学习和开发曲线）

使用**GRAFANA**实现可视化是低成本、高回报的体现，并且可以使用社区中的开源DASHBOARD（仪表盘），简直是开箱即用了。

以下是Grafana的安装部署、数据源接入、仪表盘等内容。

<!-- more -->

## 部署

### 下载

我遵循一个原则，优先使用docker安装服务，尤其是契合GO语言生态的开源软件，docker都是最优的选择。

使用docker search从 Docker Hub 网站来搜索grafana镜像

```shell
docker search grafana
```

然后docker pull，下载这个镜像，不指定版本默认lastest的tag

```shell
docker pull grafana/grafana
```

这样就能在`docker images`中看到它

### 启动

运行容器可以和Prometheus Server一样，使用`-v`参数绑定及挂载数据卷，新建空文件夹grafana-storage，用来存储数据。

```
mkdir /opt/grafana-storage
```

设置权限（比较粗暴），科学的做法是给grafana加入用户组，给文件夹赋予组权限

```
chmod 777 -R /opt/grafana-storage
```

启动grafana

```shell
docker run -d \
  -p 3000:3000 \
  --name=grafana \
  -v /opt/grafana-storage:/var/lib/grafana \
  grafana/grafana
```

等待几秒钟，查看端口3000状态，已经正常启动

```shell
[root@iZwz9cdthtgi8exxs9m71vZ /opt/prometheus/prometheus.yml]#netstat -anpt | grep docker
tcp6       0      0 :::9911                 :::*                    LISTEN      22125/docker-proxy  
tcp6       0      0 :::3000                 :::*                    LISTEN      2362/docker-proxy   
tcp6       0      0 :::10080                :::*                    LISTEN      28202/docker-proxy  
tcp6       0      0 :::9090                 :::*                    LISTEN      22860/docker-proxy  
tcp6       0      0 :::10022                :::*                    LISTEN      28213/docker-proxy  
```

访问http://47.112.240.167:3000/即可登录grafana，默认密码是`admin/admin`：

{% asset_img grafana-login.png %}

## 可视化

### 接入Prometheus数据源

此时的HOME是没有空白没有数据的，首先我们查看grafana数据源：

{% asset_img grafana-data-source.png %}

除了Prometheus、influxdb，还支持许多开源服务如Elasticsearch、Loki（类似于Prometheus的日志采集软件），也能直接接入MySQL、MongoDB等数据库进行占时，不过需要配置可视化规则比较麻烦，以后再研究吧。

创建一个Prometheus的数据源，配置如下：

{% asset_img prometheus-data-source.png %}

配置完之后还需要关联dashboards才能展示数据。

### 配置Dashboards

可以在另一个tab中直接import默认的dashboard：

{% asset_img prometheus-dashboard.png %}

展示三个其中较好的仪表盘：

{% asset_img prometheus-dashboard-stats.png %}

可以发现，有的指标我们没听过也不需要关注，可用性比较差，我们可以在此仪表盘上EDIT，编辑我们想要的。

### Dashboards社区

这里提供一种更好用的方法，在grafana中是支持且推崇社区的。

我们可以在https://grafana.com/grafana/dashboards中挑选符合业务监控指标和风格的仪表盘，如同手机逛应用商店一样简单，

{% asset_img grafana-dashboards-community.png %}

选取一款合适的仪表盘，然后在复制下ID`13105`就可以了

{% asset_img grafana-dashboards1.png %}

仪表盘的配置是JSON存储的，可移植性非常高，不能访问社区的需要使用JSON导入。

如图，在`Dashboards`菜单点击`Import`导入，然后输入仪表盘超市的ID即可，配置如下：

{% asset_img grafana-dashboards-settings.png %}

生成后，我们稍作微调，就完成了一块很棒的仪表盘。

{% asset_img grafana-dashboards-show.png %}

可以在`Configuration / Preferences`中，将其设置为主目录的仪表盘。

## 后续

简单而科学的Grafana链路已经完成啦，Grafana功能非常强大，除了上述最常用的仪表盘、数据源，还有可视化，他也有告警、数据清洗、分组权限等功能，基本满足了数据可视化的大多数场景。

在云原生生态中，除了`Prometheus` + ` Grafana`，还有`Telegraf` +  `InfluxDB`+ `Grafana`的强力组合，有兴趣的话可以慢慢了解。



------

参考资料：

[Grafana: The open observability platform | Grafana Labs](https://grafana.com/)

[Grafana | Prometheus](https://prometheus.io/docs/visualization/grafana/)

[Grafana Dashboards - discover and share dashboards for Grafana. | Grafana Labs](https://grafana.com/grafana/dashboards)

[grafana/grafana: The open and composable observability and data visualization platform. Visualize metrics, logs, and traces from multiple sources like Prometheus, Loki, Elasticsearch, InfluxDB, Postgres and many more. (github.com)](https://github.com/grafana/grafana)

