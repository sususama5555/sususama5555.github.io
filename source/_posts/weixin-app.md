---
title: 企业(政务)微信浏览器的前端开发兼容问题
date: 2020-08-20 22:50:54
tags: 微信轻应用
categories: 前端开发指南
---

**windows版本的企业（政务）微信浏览器基于Chromium 53版本，对ES7某些新特性不兼容**

![图片描述](https://www.tapd.cn/tfl/captures/2020-08/tapd_51310665_base64_1597896079_93.png)
<!--more-->

### 1、不支持Async/Await异步函数

官方的方法兼容文档：
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function 

![img](https://www.tapd.cn/tfl/captures/2020-08/tapd_51310665_base64_1597903440_31.png)

### 2、不支持Axios的promise.prototype.finally方法：

![img](https://www.tapd.cn/tfl/captures/2020-08/tapd_51310665_base64_1597904438_2.png)

解决办法：通过main.js引入依赖
```
npm i promise.prototype.finally
require('promise.prototype.finally').shim();
```

### 3、本地安装模拟企业（政务）微信内置Chromium 53版本：

#### 3.1、下载附件的安装包（Chromium 53版本）
[点此下载](https://file.tapd.cn/51310665/attachments/download/1151310665001000266/wiki)  

#### 3.2、右键解压成Chrome.7z，继续解压成Chrome-bin文件夹 

![img](https://www.tapd.cn/tfl/captures/2020-08/tapd_51310665_base64_1597904944_70.png)

#### 3.3、直接执行Chrome-bin / chrome.exe会跳转到本地最新版的chrome。

在此目录下编写脚本start.bat，点击脚本即可打开Chromium 53：

```
start "" "./chrome.exe"  " --user-data-dir=User Data"
```

![img](https://www.tapd.cn/tfl/captures/2020-08/tapd_51310665_base64_1597904997_55.png)