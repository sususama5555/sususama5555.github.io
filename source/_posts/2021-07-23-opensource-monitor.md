---
title: 开源监控系统横向对比
date: 2021-07-23 00:32:07
tags: 
- 监控
- Prometheus
- Zabbix
- 技术选型
categories: 
- 监控系统
---

## 前言

最近对开源监控系统比较感兴趣，之前的博客对`Prometheus `、`Zabbix`（去年）都做过研究，也稍微提及了`Grafana`，甚至`Telegraf`+`InfluxDB`。最终在云服务器上逐个进行了部署和体验。

但是明显感受到知识是不成体系的，所以决定结合个人体验和文档，对多个监控解决方案进行横向对比，这样才能更好地理解它们的优劣和擅长的领域。

以下将详细对比Prometheus和Zabbix，它们分别作为监控领域的新宠和老将，是具有代表性的。

<!-- more -->

## Prometheus和Zabbix

> Prometheus是云原生的宠儿，是由SoundCloud开发的开源监控报警系统和时序列数据库。从字面上理解，Prometheus由两个部分组成，一个是监控报警系统，另一个是自带的时序数据库（TSDB）

>Zabbix是由Alexei Vladishev开源的分布式监控系统，是一个企业级的分布式开源监控方案。能够监控各种网络参数以及服务器健康性和完整性的软件。同样拥有监控、告警等功能。

二者从大体功能上来说差距不大，但实现细节差别很大：

### Prometheus：

- Golang开发，诞生晚，是容器监控最好的解决方案，对Kubernetes支持较好
- 本地存储基于TSDB（时序数据库）
- 主要由Prometheus Server、多种Exporter、告警组件alartmanager、推送网关pushgateway组成
- 通过HTTP周期Pull各类Exporter的指标，自由度高，满足格式即可接入。或者使用pushgateway实现推送监控数据。
- 支持PromQL提供多维度数据模型和灵活的查询，通过监控指标关联多个tag的方式，将监控数据进行任意维度的组合以及聚合。

### Zabbix

- c + php开发，出现于上个世纪，2012年开源
- 核心组件主要是Agent和Server
  - Agent采集数据并通过主动或者被动的方式采集数据发送到Server/Proxy，另外，Agent还支持执行自定义脚本。Server主要负责接收Agent发送的监控信息，并进行汇总存储，触发告警等
- 使用关系型数据库存储时序数据，支持MySQL、PostgreSQL、Oracle，监控大规模集群时捉襟见肘。
- 安装部署较为繁琐，容易踩坑（详情见Zabbix安装与部署一文）。
  - 需要安装对应的安装包，如php、数据库、服务器等依赖介质。手动创建数据库、导入表结构，以及zabbix、php等配置，需要配置Nginx或者Apache实现代理转发，最终启动和管理多个服务
- 通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，可以运行在多平台上

### 综合对比

从**开发语言**上看，为了应对高并发和快速迭代的需求，监控系统的慢慢从C语言转移到Go。不得不说，Go凭借简洁的语法和优雅的并发，准确定位中间件开发需求，在当前开源中间件产品中被广泛应用。

从**系统成熟度**上看，Zabbix是老牌的监控系统，系统功能比较稳定，成熟度较高。而Prometheus是最近几年才诞生的，功能还在不断迭代更新，站在了巨人的肩膀之上，在架构设计上借鉴了很多老牌监控系统的经验。

从**数据存储**方面来看，Zabbix采用关系数据库保存，这极大限制了Zabbix采集的性能，而Prometheus自研一套高性能的时序数据库，在V3版本可以达到每秒千万级别的数据存储，通过对接第三方时序数据库扩展历史数据的存储。

从**配置复杂度**上看，Prometheus只有一个核心server组件，一条命令便可以启动，相比而言，Zabbix其他系统配置相对麻烦。

从**社区活跃度**上看，自然是Prometheus更具有优势，背靠CNCF，github上的综合指标也是更胜一筹；

从**容器支持**角度看，由于Zabbix出现得比较早，当时容器还没有诞生，自然对容器的支持也比较差。而Prometheus的动态发现机制，不仅支持swarm原生集群，还支持Kubernetes容器集群的监控，是目前容器监控最好解决方案。

Zabbix在传统监控系统中，尤其是在服务器相关监控方面，占据绝对优势。甚至环境变动不会很频繁的情况下，Zabbix 也会比 Prometheus 好使；但如果是云环境，Zabbix就水土不服了，需要做各种定制，且c + php开发效率不算高。Prometheus开始成为主导及容器监控方面的标配，并且在未来可见的时间内被广泛应用。

如果是刚刚要上监控系统的话，Prometheus 准没错。



-------------

参考资料：

[Overview | Prometheus](https://prometheus.io/docs/introduction/overview/)

[Zabbix：企业级开源监控解决方案](https://www.zabbix.com/cn)

[prometheus和zabbix的对比](https://www.cnblogs.com/xiaoyuxixi/p/12235979.html)

