---
title: 蓝鲸SaaS开发练习
date: 2020-06-15 04:14:18
tags: 
- Python
- Django
- Vue.js
- 蓝鲸
categories: 
- 蓝鲸
- 开发框架
---

# :wrench: 前言

基于蓝鲸开发框架2.0，适配简单运维场景，开发集成三种功能的SaaS	~~（为考试做准备）~~

GitHub仓库地址：https://github.com/sususama5555/bk-framework

蓝鲸开发框架2.0使用说明：https://docs.bk.tencent.com/blueapps/USAGE.html

**`后端`**：`Django` + `MySQL` + `RabbitMQ` + `Redis`

**`前端`**： `Vue.js` + `Django Template`

**`组件`**：`element-ui` + `Echart` + `axios`

<!-- more -->

# :art:功能

- **文件备份与记录**
- **主机性能监控**
- **模板与任务**

# :memo: 详情

## 文件备份与记录：

1.业务拓扑，主机选择

{% asset_img  backup-2.png%}

2.主机路径选择，文件搜索与 备份

{% asset_img  backup-1.png%}

## 主机性能监控：

1.主机选择，实时数据，性能监控

{% asset_img  monitor-1.png%}

## 模板与任务:

1.添加模板，表格导入

{% asset_img  task-2.png%}

2.基于模板创建任务

{% asset_img  task-4.png%}

3.任务流水线

{% asset_img  task-3.png%}

