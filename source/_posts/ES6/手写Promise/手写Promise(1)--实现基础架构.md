---
title: 手写Promise(1)--实现基础架构
categories: 
- ES6
tags: 
- Promise
comments: true
top: true
cover: false
keywords: 手写Promise ES6 Promise
---

### 1.Promise介绍

 Promise是一种解决异步编程回调函数的方案。最早由社区提供和实现，在ES2015中将其写入标准语法中，提供原生的对象。  
 Promise对象更像一个容器，将需要进行的操作（通常是异步操作）放入其中，其总结有以下几个特点：  
- Promise有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败），且不受外界影响
- Promise对象的状态单向的，只能从 pending => fulfilled 活着从 pending => rejected，且一旦变化就不可再改变
 
 > 关于Promise的详细介绍可以查看阮一峰的[ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/promise)

### 2.实现基础架构
Promise对象的构造函数接受一个函数，且包含两个参数，同时Promise的原型上还有then函数
- resolve函数：将Promise对象的状态从pending变为fulfilled，在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
- reject函数：将Promise对象的状态从pending变为rejected，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。
- then函数：为 Promise 实例添加状态改变时的回调函数。then方法的第一个参数是resolved状态的回调函数，第二个参数是rejected状态的回调函数，它们都是可选的。
 
 了解以上，那么我们手写的Promise对象的基本结构就可以写出来了

``` javascript
class JunPromise {
    //构造函数
    constructor(execute){
        execute(this.resolve, this.reject)
    }

    //状态 (pending-等待，fulfilled-完成，rejected-失败)
    status = 'pending'

    //状态pending => fulfilled
    resolve = (value) => {

    }

    //状态pending => rejected
    reject = (value) => {

    }

    //then方法
    then = (resolveCallback, rejectCallback) => {

    }
 }
```

### 3.实现resolve(),reject()函数

resolve()和reject()函数都会修改status的状态，同时其返回的结果会作为参数传递出去，在这里，我们用两个变量记录返回的结果

``` javascript
class JunPromise {
    //构造函数
    constructor(execute){
        execute(this.resolve, this.reject)
    }

    //状态 (pending-等待，fulfilled-完成，rejected-失败)
    status = 'pending'
    //回调成功的值
    successMessage = undefined
    //回调失败的值
    failMessage = undefined

    //状态pending => fulfilled
    resolve = (value) => {
		//状态已经一旦改变便不可逆
        if(this.status != 'pending'){
            return
        }
        this.status = 'fulfilled'
        this.successMessage = value
    }

    //状态pending => rejected
    reject = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'rejected'
        this.failMessage = value
    }

    //then方法
    then = (resolveCallback, rejectCallback) => {

    }
 }
```

### 4.then函数的初步实现

then方法接受两个参数，的第一个参数是resolved状态的回调函数，第二个参数是rejected状态的回调函数，我们也用两个变量进行记录，并且then函数会根据当前状态进行不同的处理

``` javascript
class JunPromise {
    //构造函数
    constructor(execute){
        execute(this.resolve, this.reject)
    }

    //状态 (pending-等待，fulfilled-完成，rejected-失败)
    status = 'pending'
    //回调成功的值
    successMessage = undefined
    //回调失败的值
    failMessage = undefined

    //状态pending => fulfilled
    resolve = (value) => {
        //状态已经一旦改变便不可逆
        if(this.status != 'pending'){
            return
        }
        this.status = 'fulfilled'
        this.successMessage = value
    }

    //状态pending => rejected
    reject = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'rejected'
        this.failMessage = value
    }

    //then方法
    then = (resolveCallback, rejectCallback) => {
        if(this.status == 'fulfilled') {
            resolveCallback(this.successMessage)
        } else if(this.status == 'rejected') {
            rejectCallback(this.failMessage)
        }
    }
 }
```

接下来用个demo来测试一下
```javascript
let p0_1 = new JunPromise(function(resolve, reject){
	resolve('我是一个基础架构')
})
p0_1.then(function(value){
	console.log('p0_1:', value)
})

///p0_1: 我是一个基础架构
```
### 5.总结
以上，一个基本的Promise架构就实现了,最后附上本文[源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)的GIT链接，欢迎指错和讨论