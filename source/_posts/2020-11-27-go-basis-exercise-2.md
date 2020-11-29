---
title: Go语言基础练习二
date: 2020-04-09 00:12:07
tags: 
- Go语言
- 运算符
- 基础练习
categories: 
- Go语言
- 基础练习
---


> ## Go语言基础练习二

以下是笔者学习Go语言基础的代码随笔，延续了练习一中的数据类型、变量常量申明，以及Go 语言的几种运算符。

### 变量申明

**第一种，指定变量类型，如果没有初始化，则变量默认为零值**。

```go
var v_name v_type
v_name = value
```

<!-- more -->

**第二种，根据值自行判定变量类型。**

```go
var v_name = value
```

**第三种，省略 var, 注意 \**:=\** 左侧如果没有声明新的变量，就产生编译错误，格式：**

```go
v_name := value
```

### 多变量声明

```go
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 // 和 python 很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 // 出现在 := 左侧的变量不应该是已经被声明过的，否则会导致编译错误


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)
```

### Go 语言运算符

运算符用于在程序运行时执行数学或逻辑运算。

Go 语言内置的运算符有：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符
- 其他运算符

## 练习代码

```go
package main

import "fmt"

func main() {

	// TODO -> 并发、map、切片、类型转换、循环、条件语句

	/* Go数据类型、变量常量申明 */
	//var a string = "Runoob"
	//var bool bool = true
	//var b string
	//b = "sapphire"
	//var c = "spr"
	fmt.Println("first Go: " + "hello world!")
	//fmt.Print(c)

	//const申明常量
	const a, b = 100, "gogogo"
	fmt.Println(b)

	//声明一个变量并初始化方式
	//方式1
	var test_1 = "RUNOOB"
	fmt.Println(test_1)

	// 没有初始化就为零值
	var test_2 int
	fmt.Println(test_2)

	// bool 零值为 false
	var test_3 bool
	fmt.Println(test_3)

	//方式2 = (自动判断类型)
	var test_4 = true
	fmt.Println(test_4)

	//方式3 := (必须申明新变量)
	test_5 := "申明变量"
	fmt.Println(test_5)

	//iota，特殊常量，const每申明一次iota+=1
	const (
		aa = iota   //0
		bb          //1
		cc          //2
		dd = "ha"   //独立值，iota += 1
		ee          //"ha"   iota += 1
		ff = 100    //iota +=1
		gg          //100  iota +=1
		hh = iota   //7,恢复计数
		ii          //8
	)
	fmt.Println(aa,bb,cc,dd,ee,ff,gg,hh,ii)
	// 输出：0 1 2 ha ha 100 100 7 8

	/* 运算符 */
	// 加+ 减- 乘* 除/ 余% 自增++ 自减--
	var operator int = 20
	operator++
	fmt.Printf("operator 的值为 %d\n", operator )
	operator--
	fmt.Printf("operator 的值为 %d\n", operator )

	/* 关系运算符 判断 */
	// 相等== 不等!= 大于> 小于< 大于等于>= 小于等于<=
	fmt.Println(21 == 22)
	fmt.Println(21 != 22)

	/* 逻辑运算符 */
	// 与(逻辑and)&& 或(逻辑or)|| 非(逻辑not)!
	and1, and2 := true, false
	if (and1 && and2) {
		fmt.Println("条件为true")
	}
	if (and1 || and2) {
		fmt.Println("条件为true")
	}

	/* 位运算符 */
	// 按位与& 按位或| 按位异或^ 左移<< 右移>>
	var example uint = 13 // 13二进制 1101
	fmt.Println(example >> 1) // 右移1位 -> 0110

	/* 赋值运算符
	运算符	描述	          实例
	=		简单的赋值运算符 C = A + B 将 A + B 表达式结果赋值给 C
	+=		相加后再赋值	  C += A 等于 C = C + A
	-=		相减后再赋值	  C -= A 等于 C = C - A
	*=		相乘后再赋值	  C *= A 等于 C = C * A
	/=		相除后再赋值	  C /= A 等于 C = C / A
	%=		求余后再赋值	  C %= A 等于 C = C % A
	<<=		左移后赋值	  C <<= 2 等于 C = C << 2
	>>=		右移后赋值	  C >>= 2 等于 C = C >> 2
	&=		按位与后赋值	  C &= 2 等于 C = C & 2
	^=		按位异或后赋值	  C ^= 2 等于 C = C ^ 2
	|=		按位或后赋值	  C |= 2 等于 C = C | 2
	*/

	// 其他运算符
	// &返回变量储存地址 *指针变量
	var_tmp := 99
	fmt.Println(&var_tmp) // 指针地址
	fmt.Println(*(&var_tmp)) // 指针的值

	fmt.Println("谁大一点:", max(100,99))

	swap_a ,swap_b := swap("lang", "go")
	fmt.Println("交换顺序", swap_a ,swap_b)

	// 数组
	var balance1 [10] float32
	// 赋值
	var balance2 = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
	fmt.Println(balance1, balance2)
	var salary float32 = balance2[3]
	fmt.Println(salary)

	array()

	var book1 Books

	// 结构体
	book1.title = "三国演义"
	book1.bookId = 1
	book1.author = "罗贯中"
	book1.subject = "历史"
	fmt.Println(book1)
	// 结构体指针
	var book_pointer *Books
	book_pointer = &book1
	fmt.Println(book_pointer)

}

/* 函数返回两个数的最大值 */
func max(num1, num2 int) int {
	/* 声明局部变量 */
	var result int

	if (num1 > num2) {
		result = num1
	} else {
		result = num2
	}
	return result
}

// 返回值有多个
func swap(x, y string) (string, string) {
	return y, x
}

func array() {
	var n [10]int /* n 是一个长度为 10 的数组 */
	var i,j int

	/* 为数组 n 初始化元素 */
	for i = 0; i < 10; i++ {
		n[i] = i + 100 /* 设置元素为 i + 100 */
	}

	/* 输出每个数组元素的值 */
	for j = 0; j < 10; j++ {
		fmt.Printf("Element[%d] = %d\n", j, n[j] )
	}
}


/* 结构体 */
type Books struct {
	title   string
	author  string
	subject string
	bookId  int
}
```

