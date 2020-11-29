---
title: Go语言之顺序编程
date: 2020-04-23 00:19:02
tags: 
- Go语言
- 顺序编程
- 基础练习
categories: 
- Go语言
- 顺序编程
---

> ## Go语言之顺序编程

以下是笔者学习Go语言基础的代码随笔，本次练习内容为<kbd>条件语句</kbd>与<kbd>循环语句</kbd>。

### Go 语言条件语句

Go 语言提供了以下几种条件判断语句：

| 语句                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [if 语句](https://www.runoob.com/go/go-if-statement.html)    | **if 语句** 由一个布尔表达式后紧跟一个或多个语句组成。       |
| [if...else 语句](https://www.runoob.com/go/go-if-else-statement.html) | **if 语句** 后可以使用可选的 **else 语句**, else 语句中的表达式在布尔表达式为 false 时执行。 |
| [if 嵌套语句](https://www.runoob.com/go/go-nested-if-statements.html) | 你可以在 **if** 或 **else if** 语句中嵌入一个或多个 **if** 或 **else if** 语句。 |
| [switch 语句](https://www.runoob.com/go/go-switch-statement.html) | **switch** 语句用于基于不同条件执行不同动作。                |
| [select 语句](https://www.runoob.com/go/go-select-statement.html) | **select** 语句类似于 **switch** 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。 |

*注意：Go 没有三目运算符，所以不支持* **?:** *形式的条件判断。*

<!-- more -->

### Go 语言循环语句

Go 语言提供了以下几种类型循环处理语句：

| 循环类型                                                   | 描述                                 |
| :--------------------------------------------------------- | :----------------------------------- |
| [for 循环](https://www.runoob.com/go/go-for-loop.html)     | 重复执行语句块                       |
| [循环嵌套](https://www.runoob.com/go/go-nested-loops.html) | 在 for 循环中嵌套一个或多个 for 循环 |

#### 循环控制语句

循环控制语句可以控制循环体内语句的执行过程。

GO 语言支持以下几种循环控制语句：

| 控制语句                                                     | 描述                                             |
| :----------------------------------------------------------- | :----------------------------------------------- |
| [break 语句](https://www.runoob.com/go/go-break-statement.html) | 经常用于中断当前 for 循环或跳出 switch 语句      |
| [continue 语句](https://www.runoob.com/go/go-continue-statement.html) | 跳过当前循环的剩余语句，然后继续进行下一轮循环。 |
| [goto 语句](https://www.runoob.com/go/go-goto-statement.html) | 将控制转移到被标记的语句。                       |

## 练习代码

```go
package main

import "fmt"

func main() {
	// ifFn()
	// forFn()
	// switchFn()
	loopFn()
}

func ifFn() {
	i := 2
	if i == 1 {
		fmt.Println("第一种方式")
	}
	if a, b := i, 7; a == 2 {
		fmt.Println(b)
		fmt.Println("第二种方式")
	} else if i == 3 {

	} else {

	}
}

func forFn() {
	// s := []int{1, 2, 4, 5, 7}
	m := map[string]int{"a": 1, "b": 2}
	for k, v := range m {
		fmt.Println(k, v)
	}
	i := 4
	if i == 1 {
		for {
			fmt.Println("第一种：无限循环，可通过break结束.")
			break
		}
	}
	if i == 2 {
		a := 1
		for a <= 2 {
			fmt.Printf("第二种：条件循环，类似while. (a=%v) \n", a)
			a++
		}
	}
	if i == 3 {
		for a := 1; a <= 2; a++ {
			fmt.Printf("第三种：与第二种相同，带初始化和自增. (a=%v) \n", a)
		}
	}
}

func switchFn() {
	i := 3
	switch i {
	case 1:
		a := 3
		switch a {
		case 1:
			fmt.Println("第一种方式, a==1")
		case 2:
			fmt.Println("第一种方式，a==2")
		default:
			fmt.Println("第一种方式，default")
		}
	case 2:
		a := 2
		switch {
		case a == 1:
			fmt.Println("第二种方式, a==1")
		case a == 2:
			fmt.Println("第二种方式, a==2")
		default:
			fmt.Println("第二种方式，default")
		}
	case 3:
		switch a := 3; {
		case a >= 1:
			fmt.Println("第三种方式, a>=1")
			fallthrough
		case a >= 2:
			fmt.Println("第三种方式, a>=2")
			fallthrough
		case a >= 2:
			fmt.Println("第三种方式, a>=2")
			fallthrough
		case a >= 2:
			fmt.Println("第三种方式, a>=2")
			fallthrough
		case a >= 2:
			fmt.Println("第三种方式, a>=2")
			fallthrough
		default:
			fmt.Println("第三种方式，default")
		}
	}

}

func loopFn() {
	i := 0
LABEL2:
	goto LABEL
LABEL:
	for i < 10 {
		for {
			for {
				i++
				if i == 3 {
					fmt.Println("跳过打印i=3")
					continue LABEL
				}
				if i == 5 {
					fmt.Println("goto一下")
					goto LABEL2
				}
				if i == 7 {
					fmt.Println("不打印剩下的")
					break LABEL
				}
				fmt.Printf("i=%d\n", i)
			}
		}
	}
}

```

