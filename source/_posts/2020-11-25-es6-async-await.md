---
title: ES6之 async 与 await
date: 2020-11-25 23:53:32
tags: 
- JavaScript
- 前端
- ES6
- 异步函数
categories: 
- JavaScript
- 异步
---



## async

async 是 ES6 才有的与异步操作有关的关键字，和 Promise ， Generator 有很大关联的。

### 语法

```javascript
async function name([param[, param[, ... param]]]) { statements }
```

- name: 函数名称。
- param: 要传递给函数的参数的名称。
- statements: 函数体语句。

### 返回值

一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)，这个promise要么会通过一个由async函数返回的值被解决，要么会通过一个从async函数中抛出的（或其中没有被捕获到的）异常被拒绝。

<!-- more -->

### 描述

async函数可能包含0个或者多个[`await`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)表达式。await表达式会暂停整个async函数的执行进程并出让其控制权，只有当其等待的基于promise的异步操作被兑现或被拒绝之后才会恢复进程。promise的解决值会被当作该await表达式的返回值。使用`async` / `await`关键字就可以在异步代码中使用普通的`try` / `catch`代码块。

`await`关键字只在async函数内有效。如果你在async函数体之外使用它，就会抛出语法错误 [`SyntaxError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError) 。

`async`/`await`的目的为了简化使用基于promise的API时所需的语法。`async`/`await`的行为就好像搭配使用了生成器和promise。

async函数一定会返回一个promise对象。如果一个async函数的返回值看起来不是promise，那么它将会被隐式地包装在一个promise中。

例如，如下代码:

```js
async function foo() {
   return 1
}
```

等价于:

```js
function foo() {
   return Promise.resolve(1)
}
```

async函数的函数体可以被看作是由0个或者多个await表达式分割开来的。从第一行代码直到（并包括）第一个await表达式（如果有的话）都是同步运行的。这样的话，一个不含await表达式的async函数是会同步运行的。然而，如果函数体内有一个await表达式，async函数就一定会异步执行。

例如：

```js
async function foo() {
   await 1
}
```

等价于

```js
function foo() {
   return Promise.resolve(1).then(() => undefined)
}
```

在await表达式之后的代码可以被认为是存在在链式调用的then回调中。

### 示例

async 函数返回一个 Promise 对象，可以使用 then 方法添加回调函数。

```javascript
async function helloAsync(){
    return "helloAsync";
  }
  
console.log(helloAsync())  // Promise {<resolved>: "helloAsync"}
 
helloAsync().then(v=>{
   console.log(v);         // helloAsync
})
```

async 函数中可能会有 await 表达式，async 函数执行时，如果遇到 await 就会先暂停执行 ，等到触发的异步操作完成后，恢复 async 函数的执行并返回解析值。

await 关键字仅在 async function 中有效。如果在 async function 函数体外使用 await ，你只会得到一个语法错误。

```javascript
const task = () => {console.log("run task")}
const debounceTask  = debounce(task, 1000)
window.addEventListener('scroll', debounceTask)

function testAwait(){
   return new Promise((resolve) => {
       setTimeout(function(){
          console.log("testAwait");
          resolve();
       }, 1000);
   });
}
 
async function helloAsync(){
   await testAwait();
   console.log("helloAsync");
 }
helloAsync();
// testAwait
// helloAsync
```



### 特点

这个函数和generator函数有些类似，从例子中可以看得出来，async函数在function前面有个async作为标识，意思就是异步函数，里面有个await搭配使用，每到await的地方就是程序需要等待执行后面的程序，语义化很强，下面总结一下**async函数的特点**：

- 语义化强
- 里面的await只能在async函数中使用
- await后面的语句可以是promise对象、数字、字符串等
- async函数返回的是一个Promsie对象
- await语句后的Promise对象变成reject状态时，那么整个async函数会中断，后面的程序不会继续执行

## await

await 操作符用于等待一个 Promise 对象, 它只能在异步函数 async function 内部使用。

### 语法

```javascript
[return_value] = await expression;
```

- expression: 一个 Promise 对象或者任何要等待的值。

### 描述

await 表达式会暂停当前 [`async function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function) 的执行，等待 Promise 处理完成。若 Promise 正常处理(fulfilled)，其回调的resolve函数参数作为 await 表达式的值，继续执行 [`async function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)。

若 Promise 处理异常(rejected)，await 表达式会把 Promise 的异常原因抛出。

另外，如果 await 操作符后的表达式的值不是一个 Promise，则返回该值本身。

### 例子

返回 Promise 对象的处理结果。如果等待的不是 Promise 对象，则返回该值本身。

如果一个 Promise 被传递给一个 await 操作符，await 将等待 Promise 正常处理完成并返回其处理结果。

```javascript
function testAwait (x) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(x);
    }, 2000);
  });
}
 
async function helloAsync() {
  var x = await testAwait ("hello world");
  console.log(x); 
}
helloAsync ();
// hello world
```

正常情况下，await 命令后面是一个 Promise 对象，它也可以跟其他值，如字符串，布尔值，数值以及普通函数。

```javascript
function testAwait(){
   console.log("testAwait");
}
async function helloAsync(){
   await testAwait();
   console.log("helloAsync");
}
helloAsync();
// testAwait
// helloAsync
```



await针对所跟不同表达式的处理方式：

- Promise 对象：await 会暂停执行，等待 Promise 对象 resolve，然后恢复 async 函数的执行并返回解析值。
- 非 Promise 对象：直接返回对应的值。

## TODO

`AsyncFunction` 构造函数



******

[developer.mozilla.org(MDN) - async函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

[developer.mozilla.org(MDN) - await](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)

[菜鸟教程 - ES6 async 函数](https://www.runoob.com/w3cnote/es6-async.html)

