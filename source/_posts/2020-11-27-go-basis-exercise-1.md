---
title: Go语言基础练习一
date: 2020-04-03 00:04:00
tags: 
- Go语言
- 数据类型
- 基础练习
categories: 
- Go语言
- 基础练习
---

> ## Go语言基础练习一

以下是笔者学习Go语言基础的代码随笔，主要为数据类型与变量申明。

### 数据类型

在 Go 编程语言中，数据类型用于声明函数和变量。

Go 语言按类别有以下几种数据类型：

| 序号 | 类型和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **布尔型** 布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true。 |
| 2    | **数字类型** 整型 int 和浮点型 float32、float64，Go 语言支持整型和浮点型数字，并且支持复数，其中位的运算采用补码。 |
| 3    | **字符串类型:** 字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本。 |
| 4    | **派生类型:** 包括：(a) 指针类型（Pointer）(b) 数组类型(c) 结构化类型(struct)(d) Channel 类型(e) 函数类型(f) 切片类型(g) 接口类型（interface）(h) Map 类型 |

<!-- more -->

## 练习代码

```go
package main

import (
	"fmt"
    "unsafe"
)

const (
	aa = iota
	bb = iota
	cc = iota
)

func  test()  {
	var test1 int16 = 99
	var str1, str2 = "hhh", "eee"
	var a = true
	b := "sapphire"
	c, d, e := "office", "word", "ppt"
	fmt.Println(test1)
	fmt.Println(str1)
	fmt.Println(str2)
	fmt.Println(a)
	pointer := &b
	fmt.Println(&b)
	fmt.Println(c + d + e)
	fmt.Println(pointer, "gogogo")
	const spr = "sapphire"
	fmt.Println("my name is:", spr)
	fmt.Println(unsafe.Sizeof(spr))
	fmt.Println(unsafe.Sizeof(test1))
	fmt.Println(bb)
	if bb != 0 {
		fmt.Println("bb is not 0")
	}
}

func main() {
	test2()
}

func test2()  {
	var a uint = 60      /* 60 = 0011 1100 */
	var b uint = 13      /* 13 = 0000 1101 */
	var c uint = 0

	c = a & b       /* 12 = 0000 1100 */
	fmt.Printf("第一行 - c 的值为 %d\n", c )

	c = a | b       /* 61 = 0011 1101 */
	fmt.Printf("第二行 - c 的值为 %d\n", c )

	c = a ^ b       /* 49 = 0011 0001 */
	fmt.Printf("第三行 - c 的值为 %d\n", c )

	c = a << 2     /* 240 = 1111 0000 */
	fmt.Printf("第四行 - c 的值为 %d\n", c )

	c = a >> 2     /* 15 = 0000 1111 */
	fmt.Printf("第五行 - c 的值为 %d\n", c )
	test3()
}

func test3()  {
	var a int = 4
	str1 := "sapphire"
	var b int32
	var c float32
	var ptr *int

	/* 运算符实例 */
	fmt.Printf("第 1 行 - a 变量类型为 = %T\n", a )
	fmt.Printf("第 2 行 - b 变量类型为 = %T\n", b )
	fmt.Printf("第 3 行 - c 变量类型为 = %T\n", c )

	/*  & 和 * 运算符实例 */
	ptr = &a     /* 'ptr' 包含了 'a' 变量的地址 */
	fmt.Printf("a 的值为  %d\n", a)
	fmt.Printf("*ptr 为 %d\n", *ptr)
	fmt.Printf("name 为 %s\n", str1)
}

```

