---
id: es6-promise
title: 简要介绍 ES6 Promise
create: 1626297250000
categories:
  - 技术分享
tags:
  - JavaScript
  - ES6
description: ES6 中新增了一个非常重要的特性，即本篇文章介绍的主角，异步编程的终极解决方案 —— Promise。
---

ES6 中新增了一个非常重要的特性，即本篇文章介绍的主角，异步编程的终极解决方案 —— Promise。

# 同步和异步
## 基础概念
在介绍Promise之前，我们先来说说什么是同步，什么是异步，以及两者间的区别。

同步、异步关注的是消息通信机制。在一般编程中，

**同步** (Synchronous) 是指在程序执行流程中，方法一步一步依次执行，后面的方法需要等前面的方法执行结束之后才能执行。

**异步** (Asynchronous) 是指在程序执行流程中，方法有其独立的时间线，不会阻塞主线程中其他方法的执行，而是执行结束后通过回调来通知主线程执行结果。

**举个通俗一点的例子：** 你去某机关单位办事，先去提交申请，然后一直傻站在那里等结果出来，拿到结果再回家，这种一步一步依次进行的方法就是**同步**；如果你提交了申请就去做些别的事情，例如看看电影逛逛街，等处理结果出来了打电话通知你（回调）再回去拿，这种方式就是**异步**。

## 异步的使用情景
在 JS 中，最常见的需要用到异步的情景就是网络请求了。
我们把 JS 的执行过程想象成一条时间轴，所有的 JS 代码都是同步执行的。如果我们要进行一次网络请求，例如获取用户的数据，那么就需要等待网络请求结束之后才能执行其他的 JS 代码。这意味着用户在网络请求的过程中将无事可做，并且如果用户的网络环境较差，用户可能一直干等下去，这显然不是一个好的使用体验。所以我们一般使用异步的方法来处理网络请求。

还有一种常见的使用异步的情景是定时任务。我们常见的`setTimeout`, `setInterval`都是异步函数，它们接受一个回调函数作为参数，在定时结束后自动调用该函数。

# 传统异步方法的缺点
如果是一个简单的异步操作，我们一般使用传统的异步方法就可以了。但当网络请求极为复杂时，使用传统的异步方法就会产生一个很大的麻烦，即我们常说的**回调地狱**。

以传统的 jQuery Ajax 网络请求方法，如果要实现一个 “D请求依赖C请求，C请求依赖B请求，B请求依赖A请求” 的操作，需要这样写：
```js
$.ajax('example.com/apiA', function (dataA) {
  console.log(dataA.msg)
  $.ajax(dataA.apiB, function (dataB) {
    console.log(dataB.msg)
    $.ajax(dataB.apiC, function (dataC) {
      console.log(dataC.msg)
      $.ajax(dataC.apiD, function (dataD) {
        console.log(dataD.msg)
      })
    })
  })
})
```
这样的代码虽然能正常运作，但是非常不利于观看和维护。试想一下，如果把上面每个请求中的`console.log`语句换成数百行的处理代码，后期维护起来就是一个噩梦。

所以我们需要以一种更为优雅的方式进行这种异步操作，于是 `Promise` 出现了。

# Promise 的基本使用

> 一般情况下，Promise在实现了ES6标准的浏览器中都可用。但如果你需要支持一些比较旧的浏览器（例如IE），你可以引入一个polyfill的库，例如 [es6-promise](https://github.com/stefanpenner/es6-promise)，然后`Window.Promise`会自动可用。

我们使用`setTimeout`定义一个延时1秒的定时任务，并用它来充当本文中所有异步操作的示例：
```js
setTimeout(() => {
  console.log("一秒过去")
}, 1000)
```

## 创建 Promise 对象

我们需要使用`new`关键字来创建一个`Promise`对象。`Promise`的构造函数接受一个函数作为参数，而作为参数的这个函数本身具有两个参数：`resolve`和`reject`。因此我们需要这样来创建`Promise`对象：
```js
new Promise((resolve, reject) => {
  // 函数的内部写上需要封装的异步操作
  setTimeout(() => {
    console.log("一秒过去")
  }, 1000)
})
```
假设我们需要在延时一秒后输出消息A，然后延时两秒输出消息B，再延时三秒后输出消息C，可能会这样写：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log("msgA")
    setTimeout(() => {
      console.log("msgB")
      setTimeout(() => {
        console.log("msgC")
      }, 3000)
    }, 2000)
  }, 1000)
})
```
这样写还是有多层嵌套，代码一点也没有清晰整洁，回调地狱也还是避免不了。`Promise`显然不是这样用的，于是我们的`resolve`出场了。

## Promise 的链式调用
我们在创建`Promise`对象时往其构造函数中传入了一个函数，`Promise`的构造函数会自动向这个函数中传入两个参数：`resolve`和`reject`。实际上，这两个参数也是两个函数，可以在`Promise`对象中被调用。

> 这也是`Promise`难以被理解的地方，向`Promise`中传入的参数是函数，该参数函数中的两个参数也是函数，区别在于第一层参数函数是由我们自己传入的，第二层的两个参数函数是由`Promise`自动传入的。

### resolve

我们可以在异步操作完成后，需要进行下一步操作时调用`resolve`函数。例如：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    // 异步操作完成，调用 resolve 函数
    resolve()
  }, 1000)
})
```
此时我们就可以使用`Promise`对象的`then`方法来进行链式调用。`then`方法接受一个函数作为参数，里面是`Promise`对象中的异步操作完成后进行的回调处理操作。像单个计时器就可以这样来写：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()       // 这样在结束异步操作后就会调用resolve函数
  }, 1000)          //     也就是下面then方法中的内容
}).then(() => {
  console.log('一秒过去')
})
```
如果我们想要链式调用异步操作，例如上面的嵌套计时器，我们可以在`then`方法中的回调函数中`return`一个新的`Promise`对象，并对其再次调用`then`方法：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()
  }, 1000)
}).then(() => {
  console.log('msgA')
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    })
  })
}).then(() => {    // 下一个then可以直接接在上一个then的后面
  console.log('msgB')
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    })
  })
}).then(() => {
  console.log('msgC')
})
```
### resolve 传值
如果异步操作得到了一个结果需要处理（例如网络请求返回的数据），直接将处理代码和异步操作写在一起是不够优雅的，也违背了使用`Promise`的目的。`Promise`的目的是将异步操作与处理代码分开，以使代码结构清晰。
我们可以把需要处理的内容作为参数传入`resolve`函数，并在`then`方法的回调函数中也定义一个参数，`Promise`会自动将内容传进去：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    let data = { msg: 'hello' }  // 假设是需要处理的数据
    resolve(data)         // 传入需要处理的数据
  }, 1000)
}).then((data) => {        // 由Promise自动传入
  console.log(data.msg)
})
```

### reject
说完了第一个参数函数`resolve`，我们再来说说第二个参数函数`reject`。
`reject`用于捕获异步操作中产生的错误。向`reject`函数中传入的值会被传进`catch`方法的回调函数中：
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    let success = false      // 模拟异步操作失败
    let errorMsg = 'Error!'    // 假设是产生的错误
    if (success) {      // 判断操作是否成功
      resolve(data)     // 若成功则向then方法传入结果，进行下一步操作
    } else {
      reject(errorMsg)  // 若失败则向catch方法传入失败信息
    }
  }, 1000)
}).then((data) => {
  console.log(data.msg)
}).catch((error) => {
  console.log(error)
})
```

> `then`方法也接受第二个参数，同样为回调函数，处理的是`reject`函数传入的错误。推荐使用`catch`方法而不是这种方法，原因是成功和错误放在一起处理会不够清晰。

# Promise 的三种状态
我们给异步操作包装的`Promise`对象拥有三种状态，分别为`pending`(等待状态/进行中)，`fulfilled`(满足状态/已完成)和`rejected`(拒绝状态/已失败)。
- **pending** - 进行中，异步操作正在进行，例如网络请求未完成，定时任务没到时间等。
- **fulfilled** - 已完成，异步操作执行结束，调用`resolve`函数之后的状态
- **rejected** - 已失败，异步操作执行失败，调用`reject`函数之后的状态

# Promise 的原型方法
## Promise.finally()
`finally`方法用于指定一个`Promise`对象最终调用的回调函数，且该回调不受状态的影响。也就是说，无论该`Promise`对象的状态是`fulfilled`还是`rejected`，都会调用该回调。
```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    if (/* success */ true) resolve('msg')
    else reject('error')
  })
}).then((msg) => {
  console.log(msg)
}).catch((err) => {
  console.log(err)
}).finally(() => {
  console.log('无论成功还是失败都会调用')
})
```
该方法于 ES2018 引入标准。

## Promise.all()
`Promise.all`方法可以将多个`Promise`对象包装成一个`Promise`对象。
该方法接受一个数组作为参数，数组内的所有元素都必须为`Promise`对象。
```js
// 假设 p1, p2, p3 都为 Promise对象
const p = Promise.all([p1, p2, p3])
```
只有`p1`，`p2`，`p3`的状态都为`fulfilled`时，`p`的状态才为`fulfilled`；`p1`，`p2`，`p3`中只要有一个的状态是`rejected`，`p`的状态就为`rejected`。
