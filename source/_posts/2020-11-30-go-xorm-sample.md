---
title: Go语言之xorm使用实例
date: 2020-11-30 00:52:27
tags: 
- Go语言
- xorm
- ORM
- 数据库
- 实战练习
categories: 
- Go语言
- xorm
---

> # 前言

本文为 ***xorm - Go 语言 ORM*** 之后，对 ***xorm*** 的练习代码。

学习Go语言之初，在 ***Go语言之顺序编程*** 这篇文章中，记录了条件、循环、选择、跳转等语句的练习情况。而最近又学到了 ***xorm - Go 语言 ORM*** 的内容，我就想把这两部分内容结合起来，实现一个简单的需求场景。

需求具体为：使用shell作为交互界面，sqlite作为数据库，使用xorm实现类似于人员信息或者银行账户的增删改查，里面也会涉及到顺序编程的内容。

你可以在`GitHub`上找到本次代码：https://github.com/sususama5555/go-xorm

<!-- more -->

# 实现详情

以下就是一个简单的人员信息录入的系统：

## main.go

main.go为项目的主入口，负责shell界面操作者的交互，以及使用主体逻辑的实现。

其中使用`fmt.Println`和`fmt.Scanf`作为shell交互的输出和输入，监听操作者键入的操作选项（数字1~6），使用`switch`区分不同的选项，然后调用`models.go`中`xorm`与`sqlite`数据库交互的公共函数，实现了该需求的主要逻辑。

### 代码一览

***<u>以下为`main.go`主函数的代码：</u>***

```go
package main

import "fmt"

const info  = `请输入操作选项
1、创建新用户
2、查询指定用户
3、列出全部用户
4、更新指定用户
5、删除指定用户
6、退出`

func main() {
	fmt.Println("欢迎使用信息录入系统:")
Exit:
	for {
		fmt.Println(info)
		var input int
		fmt.Scanf("%d \n", &input)
		switch input {
		case 1:
			var name, country string
			fmt.Print("请输入姓名: ")
			fmt.Scanf("%s\n", &name)
			fmt.Print("请输入所在国家: ")
			fmt.Scanf("%s\n", &country)
			if err := newAccount(name, country); err != nil {
				fmt.Println("创建失败:", err)
			} else {
				fmt.Println("创建成功")
			}
		case 2:
			fmt.Println("请输入要查询的账号 <id>:")
			var id int64
			fmt.Scanf("%d\n", &id)
			data, err := getAccount(id)
			if err != nil {
				fmt.Println("Fail to get account:", err)
			} else {
				fmt.Printf("%#v\n", data)
			}
		case 3:
			fmt.Println("以下是所有账号信息:")
			allData, err := getAccountsAscId()
			if err != nil {
				fmt.Println("Fail to get accounts:", err)
			} else {
				for i, a := range allData {
					fmt.Printf("%d: %#v\n", i+1, a)
				}
			}
		case 4:
			fmt.Println("请输入要更新的账号 <id>:")
			var id int64
			fmt.Scanf("%d\n", &id)
			var name,country string
			fmt.Print("请输入更新的姓名:")
			fmt.Scanf("%s\n", &name)
			fmt.Print("请输入更新的国家:")
			fmt.Scanf("%s", &country)
			data, err := updateAccount(id, name, country)
			if  err != nil{
				fmt.Println("更新失败:", err)
			} else {
				fmt.Printf("更新成功 %#v\n", data)
			}
		case 5:
			fmt.Println("请输入要删除的账号 <id>:")
			var id int64
			fmt.Scanf("%d\n", &id)
			if err := deleteAccount(id); err != nil {
				fmt.Println("删除失败:", err)
			} else {
				fmt.Printf("删除成功 %d", &id)
			}
		case 6:
			fmt.Println("感谢您的使用")
			break Exit
		}
	}
}

```

## models.go

models.go主要为使用xorm对该项目的数据库进行增删改查，主要是常用函数的封装，我们可以在main.go里面对这些公共方法进行调用。

### 安装和引入xorm

#### 安装

```shell
go get xorm.io/xorm
```

#### 引入

在使用xorm的文件开头，import以下几个包，主要为`go-xorm`与`go-sqlite3`

```go
package main

import (
	"errors"
	"log"

	"github.com/go-xorm/xorm"
	_ "github.com/mattn/go-sqlite3"
)
```



### 创建 Engine 引擎

可以看到，我们按照xorm的操作手册，使用`var x *xorm.Engine`首先创建了单个ORM引擎，然后使用`init`函数对基于`sqlite`的数据库初始化，为了方便，指定了同级目录下的`bank.db`作为数据表，最后使用`x.Sync(new(Account))`实现了同步结构体与数据表。

```go
// ORM 引擎
var x *xorm.Engine

func init() {
	// 创建 ORM 引擎与数据库
	var err error
	x, err = xorm.NewEngine("sqlite3", "./bank.db")
	if err != nil {
		log.Fatalf("Fail to create engine: %v\n", err)
	}

	// 同步结构体与数据表
	if err = x.Sync(new(Account)); err != nil {
		log.Fatalf("Fail to sync database: %v\n", err)
	}
}
```

### 定义表结构体

然后是定义表结构体，我们对Column 表的属性进行了定义，创建了一个名为 Account 的结构体，实现了对数据库表的映射。

之后就是增删改查的操作，例如：

```go
// 人员信息
type Account struct {
	Id      int64
	Name    string `xorm:"unique"`
	Country string
	//Version int `
	//
	//
	//xorm:"version"` // 乐观锁
}
```

### 插入数据

`_, err := x.Insert(&Account{Name: name, Country: country})`

使用xorm，该语句可以实现插入一条数据。

```go
// 创建新的账户
func newAccount(name string, country string) error {
	// 对未存在记录进行插入
	_, err := x.Insert(&Account{Name: name, Country: country})
	return err
}
```



类似于`django orm`中的`create`，或者创建实例x后，再`x.save()`。

### 查询数据

#### 批量查询

`err = x.Find(&as)`

```go
// 按照 ID 正序排序返回所有账户
func getAccountsAscId() (as []Account, err error) {
	// 使用 Find 方法批量获取记录
	err = x.Find(&as)
	return as, err
}
```



#### 单条查询

`has, err := x.Id(id).Get(a)`

```go
// 获取账户信息
func getAccount(id int64) (*Account, error) {
	a := &Account{}
	// 直接操作 ID 的简便方法
	has, err := x.Id(id).Get(a)
	// 判断操作是否发生错误或对象是否存在
	if err != nil {
		return nil, err
	} else if !has {
		return nil, errors.New("账号不存在")
	}
	return a, nil
}
```



### 更新数据

更新数据前，获取到需要变更的记录的`Id`，然后对其他属性进行修改。

```go
// 更新账户信息
func updateAccount(id int64, name string, country string)  (*Account, error){
	_, err := getAccount(id)
	if err != nil {
		return nil, err
	}
	data := new(Account)
	data.Name = name
	data.Country = country
	// 对已有记录进行更新
	_, err = x.Id(id).Update(data)
	return data, err
}
```

### 删除数据

```go
// 删除账户
func deleteAccount(id int64) error {
	// 通过 Delete 方法删除记录
	_, err := x.Delete(&Account{Id: id})
	return err
}
```

### 代码一览

***<u>以下为`models.go`定义数据库交互的公共函数代码：</u>***

```go
package main

import (
	"errors"
	"log"

	"github.com/go-xorm/xorm"
	_ "github.com/mattn/go-sqlite3"
)

// 人员信息
type Account struct {
	Id      int64
	Name    string `xorm:"unique"`
	Country string
	//Version int `
	//
	//
	//xorm:"version"` // 乐观锁
}

// ORM 引擎
var x *xorm.Engine

func init() {
	// 创建 ORM 引擎与数据库
	var err error
	x, err = xorm.NewEngine("sqlite3", "./bank.db")
	if err != nil {
		log.Fatalf("Fail to create engine: %v\n", err)
	}

	// 同步结构体与数据表
	if err = x.Sync(new(Account)); err != nil {
		log.Fatalf("Fail to sync database: %v\n", err)
	}
}

// 创建新的账户
func newAccount(name string, country string) error {
	// 对未存在记录进行插入
	_, err := x.Insert(&Account{Name: name, Country: country})
	return err
}

// 更新已有的账户
func updateAccount(id int64, name string, country string)  (*Account, error){
	// TODO 注释方法不能用
	//data, err := getAccount(id)
	//if err != nil {
	//	return nil, err
	//}
	//data.Name = name
	//data.Country = country
	//// 对已有记录进行更新
	//_, err = x.Update(data)
	//return data, err
	_, err := getAccount(id)
	if err != nil {
		return nil, err
	}
	data := new(Account)
	data.Name = name
	data.Country = country
	// 对已有记录进行更新
	_, err = x.Id(id).Update(data)
	return data, err
}

// 获取账户信息
func getAccount(id int64) (*Account, error) {
	a := &Account{}
	// 直接操作 ID 的简便方法
	has, err := x.Id(id).Get(a)
	// 判断操作是否发生错误或对象是否存在
	if err != nil {
		return nil, err
	} else if !has {
		return nil, errors.New("账号不存在")
	}
	return a, nil
}

// 按照 ID 正序排序返回所有账户
func getAccountsAscId() (as []Account, err error) {
	// 使用 Find 方法批量获取记录
	err = x.Find(&as)
	return as, err
}

// 删除账户
func deleteAccount(id int64) error {
	// 通过 Delete 方法删除记录
	_, err := x.Delete(&Account{Id: id})
	return err
}
```



# 成果截图

完成以上代码后，使用go build编译成可执行的二进制文件，不出意外的话，我执行该exe文件，就会得到以下截图的结构：

## 开始界面

输入操作的选项

{% asset_img start.png %}

## 新增数据

输入表结构的字段，创建一条数据，第一次操作由于name的唯一性没有通过新增要求

{% asset_img insert.png %}

## 查询数据

查询所有和查询单条数据
{% asset_img select.png %}

## 修改数据

对指定Id的数据进行修改

{% asset_img update.png %}

## 删除数据

{% asset_img delete.png %}




******

***<u>参考链接：</u>***

可以参照本人另一篇文章 —— `xorm - Go 语言 ORM`，或者官方的操作手册：

[xorm 官方操作手册](https://gobook.io/read/gitea.com/xorm/manual-zh-CN/)