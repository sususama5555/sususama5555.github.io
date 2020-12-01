---
title: 初识 - Zabbix
date: 2020-07-02 00:27:07
tags: 
- Zabbix
- 监控
- 告警
- 初识
- 开源
categories: 
- 监控
- Zabbix
---

# Zabbix 介绍

- Zabbix 由 Alexei Vladishev 创建，目前由其成立的公司—— Zabbix SIA 积极的持续开发更新维护， 并为用户提供技术支持服务。
- Zabbix 是一个企业级分布式开源监控解决方案。
- Zabbix 软件能够监控众多网络参数和服务器的健康度、完整性。Zabbix 使用灵活的告警机制，允许用户为几乎任何事件配置基于邮件的告警。这样用户可以快速响应服务器问题。Zabbix 基于存储的数据提供出色的报表和数据可视化功能。这些功能使得 Zabbix 成为容量规划的理想选择。
- Zabbix 支持主动轮询（polling）和被动捕获（trapping）。Zabbix所有的报表、统计数据和配置参数都可以通过基于 Web 的前端页面进行访问。基于 Web 的前端页面确保您可以在任何地方访问您监控的网络状态和服务器健康状况。适当的配置后，Zabbix 可以在监控 IT 基础设施方面发挥重要作用。无论是对于有少量服务器的小型组织，还是拥有大量服务器的大企业而言，同样适用。
- Zabbix 是免费的。Zabbix 是根据 GPL 通用公共许可证的第二版编写和发布的。这意味着产品源代码是免费发布的，可供公共使用。

<!-- more -->

# Zabbix 功能

Zabbix 是一个高度成熟完善的网络监控解决方案，一个的软件包中包含了多种功能。

**[数据采集](https://www.zabbix.com/documentation/4.0/manual/config/items)**

- 可用性和性能检查；
- 支持 SNMP（包括主动轮询和被动捕获）、IPMI、JMX、VMware 监控；
- 自定义检查；
- 按照自定义的时间间隔采集需要的数据；
- 通过 Server/Proxy 和 Agents 来执行数据采集。

**[灵活的阈值定义](https://www.zabbix.com/documentation/4.0/manual/config/triggers)**

- 您可以参考后端数据库定义非常灵活的告警阈值，即触发器

**[高度可配置化的告警](https://www.zabbix.com/documentation/4.0/manual/config/notifications)**

- 可以根据递增计划、接收者、媒介类型自定义发送告警通知；
- 使用宏变量可以使告警通知变得更加高效有用；
- 自动操作包含远程执行命令。

**[实时图形](https://www.zabbix.com/documentation/4.0/manual/config/visualisation/graphs/simple)**

- 使用内置图形功能可以将监控项实时绘制成图形。

**[Web 监控功能](https://www.zabbix.com/documentation/4.0/manual/web_monitoring)**

- Zabbix可以追踪模拟鼠标在 Web 网站上的点击操作，来检查 Web 网站的功能和响应时间。

**[丰富的可视化选项](https://www.zabbix.com/documentation/4.0/manual/config/visualisation)**

- 可以组合多个监控项到单个视图中，创建自定义图表；
- 网络拓扑图；
- 以仪表盘样式展示自定义聚合图形和幻灯片演示；
- 报表；
- 监控资源的更高层次展示视图（业务视图）。

**[历史数据存储](https://www.zabbix.com/documentation/4.0/manual/installation/requirements#database_size)**

- 存储在数据库中的数据；
- 历史配置；
- 内置数据管理机制（housekeeping）。

**[配置简单](https://www.zabbix.com/documentation/4.0/manual/config/hosts)**

- 将被监控设备添加为主机；
- 主机一旦添加到数据库中，就会采集数据用于监控；
- 将模板用于监控设备。

**使用[模板](https://www.zabbix.com/documentation/4.0/manual/config/templates)**

- 模板中分组检查；
- 模板可以关联模板，继承已关联模板的属性。

**[网络发现](https://www.zabbix.com/documentation/4.0/manual/discovery)**

- 自动发现网络设备；
- Zabbix Agent 发现设备后自动注册；
- 自动发现文件系统、网络接口和 SNMP OIDs 值。

**[快捷的 Web 界面](https://www.zabbix.com/documentation/4.0/manual/web_interface)**

- 基于 PHP 的 Web 前端；
- 可以从任何地方访问；
- 您可以定制自己的操作方式；
- 您可以通过审计日志来查看你的操作。

**[Zabbix API](https://www.zabbix.com/documentation/4.0/manual/api)**

- Zabbix API 为 Zabbix 提供可编程接口，用于批量操作、第三方软件集成和其他用途。

**[权限管理系统](https://www.zabbix.com/documentation/4.0/manual/config/users_and_usergroups)**

- 安全的用户身份验证；
- 指定的用户只能查看指定的权限范围内的视图。

**[功能强大且易于扩展的 Zabbix Agent](https://www.zabbix.com/documentation/4.0/manual/concepts/agent)**

- 部署于被监控对象上；
- 支持 Linux 和 Windows ；

**[二进制守护进程](https://www.zabbix.com/documentation/4.0/manual/concepts/server)**

- 为了更好的性能和更少的内存占用，采用 C 语言编写；
- 便于移植。

**[适应更复杂的环境](https://www.zabbix.com/documentation/4.0/manual/distributed_monitoring)**

- 使用 Zabbix Proxy 代理，可以轻松实现分布式远程监控。

# Zabbix 概述

## 架构

Zabbix 由几个主要的功能组件组成，其功能介绍如下所示。

### SERVER

[Zabbix server](https://www.zabbix.com/documentation/4.0/manual/concepts/server) 是 Zabbix软件的核心组件，agent 向其报告可用性、系统完整性信息和统计信息。server也是存储所有配置信息、统计信息和操作信息的核心存储库。

### 数据库

所有配置信息以及 Zabbix 采集到的数据都被存储在数据库中。

### WEB 界面

为了从任何地方和任何平台轻松访问 Zabbix ，我们提供了基于 web 的界面。该界面是 Zabbix server 的一部分，通常（但不一定）和 Zabbix server 运行在同一台物理机器上。

### PROXY

[Zabbix proxy](https://www.zabbix.com/documentation/4.0/manual/concepts/proxy) 可以代替 Zabbix server采集性能和可用性数据。Zabbix proxy在Zabbix的部署是可选部分；但是proxy的部署可以很好的分担单个Zabbix server的负载。

### AGENT

[Zabbix agents](https://www.zabbix.com/documentation/4.0/manual/concepts/agent) 部署在被监控目标上，用于主动监控本地资源和应用程序，并将收集的数据发送给 Zabbix server。

## 数据流

另外，回过头来整体的了解下 Zabbix 内部的数据流对Zabbix的使用也很重要。首先，为了创建一个采集数据的监控项，您就必须先创建主机。其次，在任务的另外一端，必须要有监控项才能创建触发器（trigger），必须要有触发器来创建动作（action）。因此，如果您想要收到类似“X个server上CPU负载过高”这样的告警，您必须首先为 *Server X* 创建一个主机条目，其次创建一个用于监控其 CPU的监控项，最后创建一个触发器，用来触发 CPU负载过高这个动作，并将其发送到您的邮箱里。虽然这些步骤看起来很繁琐，但是使用模板的话，实际操作非常简单。也正是由于这种设计，使得 Zabbix 的配置变得更加灵活易用。

# 软件简介

以下描述来自**开源中国**：

Zabbix 是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。

zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供柔软的通知机制以让系统管理员快速定位/解决存在的各种问题。

**zabbix**由2部分构成，**zabbix server**与可选组件zabbix agent。
**zabbix server**可以通过SNMP，**zabbix agent**，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux, Solaris, HP-UX, AIX, Free BSD, Open BSD, OS X等平台之上。
**zabbix agent**需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集。**zabbix agent**可以运行在Linux ,Solaris, HP-UX, AIX, Free BSD, Open BSD, OS X, Tru64/OSF1, Windows NT4.0, Windows 2000/2003/XP/Vista)等系统之上。

**zabbix server**可以单独监视远程服务器的服务状态；同时也可以与**zabbix agent**配合，可以轮询**zabbix agent**主动接收监视数据（trapping方式），同时还可被动接收**zabbix agent**发送的数据（trapping方式）。
另外**zabbix server**还支持SNMP (v1,v2)，可以与SNMP软件(例如：net-snmp)等配合使用。

## zabbix的主要特点

- 安装与配置简单，学习成本低
- 支持多语言（包括中文）
- 免费开源
- 自动发现服务器与网络设备
- 分布式监视以及WEB集中管理功能
- 可以无agent监视
- 用户安全认证和柔软的授权方式
- 通过WEB界面设置或查看监视结果
- email等通知功能
- 等等

## Zabbix主要功能

- CPU负荷
- 内存使用
- 磁盘使用
- 网络状况
- 端口监视
- 日志监视



******

[Zabbix Documentation 5.0 - Introduction](https://www.zabbix.com/documentation/5.0/manual/introduction/about)

[开源中国- Zabbix软件简介](https://www.oschina.net/p/zabbix?hmsr=aladdin1e1)