---
title: 使用 FastAPI 搭建前后端分离项目
date: 2021-05-06 00:50:40
tags: 
- Python 
- FastAPI
- vue.js
- 框架
categories: 
- Python 
- 框架
---

## 前言

​		由于工作中都是用Django + DRF来后端开发，Django虽然很生态很丰富、开箱即用也很美好，但是在某些场景中，并不需要这么多功能，后端只需提供json化的api即可，这样我就关注到了FastAPI，首先听闻它的性能可以比肩JAVA的框架，而且是异步非阻塞的，天生支持async、await，类似于flask的语法和风格也很吸引我，所以我就琢磨，能不能在之前的开发框架中，替换掉Django，使用FastAPI？很显然工作量是巨大的，需要支持已有的Login、Csrf、用户等功能，而且需要满足作为SaaS的docker部署流程。总之路很漫长，先从FastAPI的基础学起。

------

FastAPI 是一个用于构建 API 的现代、快速（高性能）的 web 框架，使用 Python 3.6+ 并基于标准的 Python 类型提示。

关键特性:

- **快速**：可与 **NodeJS** 和 **Go** 比肩的极高性能（归功于 Starlette 和 Pydantic）。[最快的 Python web 框架之一](https://fastapi.tiangolo.com/zh/#_11)。
- **高效编码**：提高功能开发速度约 200％ 至 300％。
- **更少 bug**：减少约 40％ 的人为（开发者）导致错误。
- **智能**：极佳的编辑器支持。处处皆可自动补全，减少调试时间。
- **简单**：设计的易于使用和学习，阅读文档的时间更短。
- **简短**：使代码重复最小化。通过不同的参数声明实现丰富功能。bug 更少。
- **健壮**：生产可用级别的代码。还有自动生成的交互式文档。
- **标准化**：基于（并完全兼容）API 的相关开放标准：[OpenAPI](https://github.com/OAI/OpenAPI-Specification) (以前被称为 Swagger) 和 [JSON Schema](https://json-schema.org/)。

先来实现一个sayhello的简单应用吧，以下使用FastAPI + SQLAlchemy(sqlite3) + html + css + vue.js + axios。

<!-- more -->

## FastAPI初始化

### 环境搭建

安装FastAPI的依赖包：`pip install fastapi`

并且安装`uvicorn`来作为服务器：`pip install uvicorn`

在main.py中初始化FastAPI应用：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

### 运行服务器

`uvicorn main:app --reload`

`uvicorn main:app` 命令含义如下:

- `main`：`main.py` 文件（一个 Python「模块」）。
- `app`：在 `main.py` 文件中通过 `app = FastAPI()` 创建的对象。
- `--reload`：让服务器在更新代码后重新启动。仅在开发时使用该选项。

### 接口运行

打开浏览器访问 [http://127.0.0.1:8000](http://127.0.0.1:8000/)。

你将看到如下的 JSON 响应：

```
{"message": "Hello World"}
```

### 自带 API 文档

跳转到 http://127.0.0.1:8000/docs。

将会看到自动生成的交互式 API 文档（由 [Swagger UI](https://github.com/swagger-api/swagger-ui) 提供），这是FastAPI天生的功能（专为api而生）：

{% asset_img index-01-swagger-ui-simple.png %}

### 另一个 API 文档

前往 http://127.0.0.1:8000/redoc。

你将会看到可选的自动生成文档 （由 [ReDoc](https://github.com/Rebilly/ReDoc) 提供)：

{% asset_img index-02-redoc-simple.png %}

## sayhello应用

接下来我们仿照flask的sayhello应用，搭建属于FastAPI版的sayhello。

### 数据库

使用**sqlite3** + **SQLAlchemy**的组合，我们需要定义数据库表结构和orm的配置：

**db.py**

```python
from sqlalchemy import create_engine

from sqlalchemy.orm import sessionmaker

engine = create_engine("sqlite:///database.db", connect_args={"check_same_thread": False})

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

session = SessionLocal()
```



**models.py**

```python
from datetime import datetime

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, String, DateTime, Integer

# 得到默认Base基类
Base = declarative_base()


class Message(Base):
    __tablename__ = "message"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(60), comment="昵称", nullable=False)
    body = Column(String(200), comment="内容", nullable=False)
    create_at = Column(DateTime, default=datetime.now, comment="创建时间")
```

### 日志模块

**logger.py**

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='fastapi.log'
)
logger = logging.getLogger(__name__)
```

### 主程序

**main.py**

```
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from sqlalchemy import func
from starlette.middleware.cors import CORSMiddleware

from schemas import *
from db import session
from logger import logger
import models

app = FastAPI(title="SayHello(留言板)",
              description="""
              翻自 《Flask Web开发实战_入门、进阶与原理解析（李辉著 ）》 中的实战项目SayHello
              原版Github: https://github.com/greyli/sayhello
              """
              )

# 设置跨域
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*", ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/index", name="欢迎首页")
async def index():
    return {"msg": "欢迎来到SayHello!"}


@app.post("/message", name="添加留言", response_model=Response200)
async def add_message(message: MessageCreate):
    message_obj = models.Message(
        name=message.name,
        body=message.body
    )
    session.add(message_obj)
    session.commit()
    session.refresh(message_obj)
    return Response200(data=message_obj)


@app.get("/message", name="分页获取留言列表", response_model=ResponseList200)
async def get_messages(limit: int = 5, page: int = 1):
    # 统计条数
    total = session.query(func.count(models.Message.id)).scalar()
    skip = (page - 1) * limit  # 计算当前页的起始数
    # 倒序显示
    data = session.query(models.Message).order_by(models.Message.create_at.desc()).offset(skip).limit(limit).all()
    return ResponseList200(total=total, data=data)
```

### 接口格式：

**schemas.py**

```python
from datetime import datetime

from fastapi import Body
from pydantic import BaseModel

from typing import List


class MessageBase(BaseModel):
    name: str = Body(..., min_length=2, max_length=8)
    body: str = Body(..., min_length=1, max_length=200)


class MessageCreate(MessageBase):
    pass


class Message(MessageBase):
    id: int = None
    create_at: datetime

    class Config:
        orm_mode = True


class Response200(BaseModel):
    code: int = 200
    msg: str = "操作成功"
    data: Message = None


class ResponseList200(Response200):
    total: int
    data: List[Message]


class Response400(Response200):
    code: int = 400
    msg: str = "无数据返回"

```

### 前端：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Say Hello!(FastAPI)</title>
  <link rel="icon" href="favicon.ico">
  <!-- 引入vue -->
  <script src="https://cdn.jsdelivr.net/npm/vue"></script>

  <!-- 引入 element ui -->
  <link href="https://cdn.bootcss.com/element-ui/2.4.5/theme-chalk/index.css" rel="stylesheet">
  <script src="https://cdn.bootcss.com/element-ui/2.4.5/index.js"></script>
  <!-- 引入axios -->
  <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.min.js"></script>



</head>

<body>

  <div id="app">
    <el-container>
      <el-header>
        <a href="/">
          <strong>Say Hello</strong>
        </a>
        <small>to the world</small>
      </el-header>
      <el-main>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-position="top" label-width="80px"
          class="demo-ruleForm">
          <el-form-item label="Name" prop="name">
            <el-input v-model.trim="ruleForm.name"></el-input>
          </el-form-item>
          <el-form-item label="Message" prop="desc">
            <el-input type="textarea" v-model.trim="ruleForm.desc"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" round plain @click.prevent="submitForm('ruleForm')">Submit</el-button>
            <el-button type="primary" round plain @click="resetForm('ruleForm')">Reset</el-button>
          </el-form-item>
        </el-form>
        <p class="total">{{messageTotal}} messages</p>
        <div class="list-groups" v-for="message in messages">
          <div class="list-group">
            <div class="item-header">
              {{message.name}}
              <small class="m-1">
                #{{message.id}}
              </small>
              <div>
                <small class="m-2">
                  {{message.create_at}}
                </small>
              </div>
            </div>
            <p>{{message.body}}</p>
          </div>
        </div>


        <!-- 分页 -->
        <el-pagination small layout="prev, pager, next" :current-page="page" :page-size="pageSize" :total="messageTotal"
          :hide-on-single-page="show" @current-change="handleCurrentChange1">
        </el-pagination>
      </el-main>

      <el-footer>
        <div>
          Copyright© 2021 /&nbsp
        </div>
        <div v-for="link in links">
          <a :href="link.url" target="_blank">{{link.name}}</a> /&nbsp
        </div>

      </el-footer>
    </el-container>
  </div>

  <script>
    // axios 配置
    instance = axios.create({
      baseURL: 'http://127.0.0.1:8000', // 本地接口服务
      timeout: 5000,
    });

    let app = new Vue({
      el: "#app",
      data: {
        messageTotal: 0,
        messages: [],
        show: true,
        result: false,
        // 请求数据的页码
        page: 1,
        pageSize: 5,
        links: [{
            name: "Github",
            url: "https://github.com/zy7y/sayhello.git"
          },
          {
            name: "Gitee",
            url: "https://gitee.com/zy7y"
          },
          {
            name: "Blog",
            url: "https://www.cnblogs.com/zy7y"
          },
          {
            name: "SayHello(Flask)",
            url: "https://github.com/greyli/sayhello"
          }
        ],
        ruleForm: {
          name: '',
          delivery: false,
          type: [],
          resource: '',
          desc: ''
        },
        rules: {
          name: [{
              required: true,
              message: '请输入姓名',
              trigger: 'change'
            },
            {
              min: 2,
              max: 5,
              message: '长度在 2 到 5 个字符',
              trigger: 'change'
            }
          ],
          desc: [{
              required: true,
              message: '请输入留言内容',
              trigger: 'change'
            },
            {
              min: 1,
              max: 150,
              message: '长度在 1 到 150 个字符',
              trigger: 'change'
            }
          ]
        },
      },
      methods: {
        getData() {
          instance
            .get('/message', {
              params: {
                page: this.page, // 页码
              }
            })
            .then(response => {
              this.messageTotal = response.data.total,
              this.messages = response.data.data,
              console.log("get")
            })
            .catch(err => { // 请求失败处理
              console.log(err);
            });
        },
        postData() {
          instance.post('/message', {
            name: this.ruleForm.name,
            body: this.ruleForm.desc
          }).then(response => {
            console.log("post请求")
            this.$message({
                message: '感谢您的留言~',
                type: 'success'
              })
            this.getData()
            this.resetForm("ruleForm")
          }).catch(error => {
            this.$message.error('请求错误!')
          });
        },
        submitForm(formName) {
          this.$refs[formName].validate((valid) => {
            // 验证成功
            if (valid) {
              // 添加接口
              this.postData()
            } else {
              return false
            }
          });
        },
        resetForm(formName) {
          this.$refs[formName].resetFields();
        },

        handleCurrentChange1: function (currentPage) { //页码切换
          this.page = currentPage
          this.getData()
        }
      },
      mounted() {
        this.getData(this.page)
      }
    })
  </script>

  <style>
    body {
      background-color: rgb(245, 245, 245) !important;
    }

    header {
      margin: 50px 0 40px 0;
    }

    footer {
      margin: 20px;
    }

    .el-header,
    .el-footer {
      /* background-color: #B3C0D1;
    color: #333; */
      text-align: center;
      line-height: 60px;
    }

    .el-aside {
      background-color: #D3DCE6;
      color: #333;
      text-align: center;
      line-height: 200px;
    }

    .el-main {
      color: #333;
      text-align: center;
      line-height: 160px;
    }

    body>.el-container {
      width: 80%;
      padding-right: 15px;
      padding-left: 15px;
      margin-right: auto;
      margin-left: auto;
    }

    .el-container:nth-child(5) .el-aside,
    .el-container:nth-child(6) .el-aside {
      line-height: 260px;
    }

    .el-container:nth-child(7) .el-aside {
      line-height: 320px;
    }

    .el-footer>div {
      display: inline
    }

    .el-header>div {
      display: inline
    }

    .el-form {
      width: 50%;
      padding-right: 15px;
      padding-left: 15px;
      margin-right: auto;
      margin-left: auto;
    }

    strong {
      font-size: 3.5rem;
      font-weight: 300;
      line-height: 1.2;
      color: #28a745 !important;
    }

    a {
      text-decoration: none !important;
    }

    small {
      font-size: 24px;
      font-weight: 400;
      color: #6c757d !important;
    }

    .el-textarea__inner {
      height: 85px;
    }

    /* 提交表单样式 */
    .el-form--label-top .el-form-item__label {
      float: left;
      padding: 0px;
    }

    .total {
      margin-bottom: .5rem;
      font-size: 1.25rem;
      font-family: inherit;
      font-weight: 500;
      line-height: 1.2;
      color: inherit;
      margin-left: 25%;
    }

    .list-groups {
      width: 50%;
      margin-right: auto;
      margin-left: auto;
    }

    /* 列表展示还有问题 圆角 */
    .list-group {
      position: relative;
      display: block;
      padding: .75rem 1.25rem;
      margin-bottom: -1px;
      background-color: #fff;
      border-top-left-radius: .25rem;
      border-top-right-radius: .25rem;
      border-radius: 1px solid rgba(0, 0, 0, .125);
      border: 1px solid rgba(0, 0, 0, .125);
    }

    .list-group>div {
      color: #28a745 !important;
      margin-bottom: .25rem !important;
      font-size: 1.25rem;
      text-align: left;
      line-height: 100%;
    }

    .list-group>div>div {
      display: inline;
      float: right;
    }

    .m-1 {
      color: #6c757d !important;
      font-size: 80%;
    }

    .m-2 {
      font-size: 30%;
      font-weight: 400;
    }

    p {
      margin-bottom: .25rem !important;
      text-align: left;
      line-height: 100%;
    }

    /* 分页 */
    .el-pagination {
      line-height: initial;
    }
  </style>

</body>

</html>
```

完工，但是现在我们使用了前后端分离的模式，需要分别启动前端和后端，后端没有入口可以进入前端的页面，只是单纯提供了json化的api接口，这有点违背继承在一个开发框架中的初衷，

## 扩展

我们需要的是一个集成的开发框架，使用vue.js的脚手架开发，然后打包成静态文件，后端提供web的入口进行模板渲染，类似于Django中的render。

通过查阅FastAPI的官方文档，支持扩展jinja2模板渲染，Django自带的也是这个，只不过mako用的比较多。

先来看看怎么改造才能实现吧。

### 安装

安装jinja2：`pip install jinja2`

如果还需要提供静态文件：`pip install aiofiles`

### 使用模板

```python
......
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()
......

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="static")


@app.get("/", response_class=HTMLResponse)
async def home(request: Request):
    return templates.TemplateResponse("message.html", {"request": request, "data": "sapphire"})
```

需要先定义模板文件（入口）的位置，然后home返回模板的响应，类似于render。

不过前端也需要改造，总所周知，jinja2的vue.js是有冲突的，二者的变量都用双大括号`{ { data } }`表示，所以需要改变一下vue.js的变量表示符，

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Say Hello!(FastAPI)</title>
  <link rel="icon" href="favicon.ico">
  <!-- 引入vue -->
  <script src="https://cdn.jsdelivr.net/npm/vue"></script>

  <!-- 引入 element ui -->
  <link href="https://cdn.bootcss.com/element-ui/2.4.5/theme-chalk/index.css" rel="stylesheet">
  <script src="https://cdn.bootcss.com/element-ui/2.4.5/index.js"></script>
  <!-- 引入axios -->
  <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.min.js"></script>



</head>

<body>

  <div id="app">
    <el-container>
      <el-header>
        <a href="/">
          <strong>Say Hello</strong>
        </a>
        <small>to the world -> {{ data }}</small>
      </el-header>
      <el-main>
     ......
      </el-main>
      <el-footer>
      ......
      </el-footer>
    </el-container>
  </div>

  <script>
	......
    let app = new Vue({
      el: "#app",
      delimiters: ['[[', ']]'],
      data: {
      },
      methods: {
      },
      mounted() {
      }
    })
  </script>

  <style>
  ......
  </style>

</body>

</html>
```

`delimiters: ['[[', ']]'],`在new Vue的申明中加一行这个就可以了，如上，Vue的变量使用`[[ data ]]`表示，jinja2模板的变量，使用{ { data } }表示，其他的接口还是使用axios来前后端分离地开展。

## 结果

{% asset_img result-01.png  %}

现在只是本地的，后续会部署到云服务器的docker上。

-------

参考资料：

[FastAPI官方文档](https://fastapi.tiangolo.com/)

[《Flask Web开发实战_入门、进阶与原理解析》 中的实战项目SayHello]([github.com](https://github.com/greyli/sayhello))

[FastAPI - sayhello demo](https://github.com/zy7y/sayhello)

