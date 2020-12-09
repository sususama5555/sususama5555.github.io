---
title: RESTful接口设计风格
date: 2019-09-10 01:03:57
tags: 
- RESTful
- 接口风格
categories: 
- 学习笔记
---

> # 什么是REST

`REST`全称是Representational State Transfer，中文意思是表述（编者注：通常译为表征）性状态转移。

{% asset_img restful.gif %}

REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。 所以我们这里描述的REST也是通过HTTP实现的REST。

Representational State Transfer这三个单词到底意味着什么：

- 每一个URI代表一种资源；
- 客户端和服务器之间，传递这种资源的某种表现层；
- 客户端通过四个HTTP动词（`get`、`post`、`put`、`delete`），对服务器端资源进行操作，实现”表现层状态转化”。

一句话概括为：**URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作。**

- GET用来获取资源
- POST用来新建资源
- PUT用来更新资源
- DELETE用来删除资源。

只要 API 程序遵循了 REST 风格，那就可以称其为 RESTful API。目前在前后端分离的架构中，前后端基本都是通过RESTful API来进行交互。

Server 提供的 RESTful API 中，URL 中只使用**名词**来指定资源，原则上不使用**动词**。**资源**是 REST 架构或者说整个网络处理的核心。

现在以知识库为例，我们需要对文章进行查询、创建、更新和删除等操作，我们在编写程序的时候就要设计客户端浏览器与我们 Web 服务端交互的方式和路径。按照经验我们通常会设计成如下模式：

| **请求方法** | **URL**         | **含义**               |
| :----------- | :-------------- | :--------------------- |
| GET          | /article        | 查询文章（单个、所有） |
| POST         | /create_article | 创建文章               |
| POST         | /update_article | 更新文章               |
| POST         | /delete_article | 删除文章               |

如果按照RESTful API 风格设计如下：

| **请求方法** | **URL**       | **含义**        |
| :----------- | :------------ | :-------------- |
| GET          | /article/id/1 | 查询id为1的文章 |
| GET          | /article      | 查询所有文章    |
| POST         | /article      | 创建文章        |
| PUT          | /article/id/1 | 更新id为1的文章 |
| DELETE       | /article/id/1 | 删除id为1的文章 |

# RESTful的意义

“古代”网页是前端后端融在一起的，比如之前的PHP，JSP等。在之前的桌面时代问题不大，但是近年来移动互联网的发展，各种类型的 Client 层出不穷，RESTful可以通过一套统一的接口为 Web，iOS 和 Android 提供服务。另外对于广大平台来说，比如 Facebook platform，微博开放平台，微信公共平台等，它们不需要有显式的前端，只需要一套提供服务的接口，于是RESTful更是它们最好的选择。

{%  asset_img restfulAPI.jpg %}