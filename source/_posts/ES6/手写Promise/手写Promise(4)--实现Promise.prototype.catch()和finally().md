---
title: 手写Promise(4)--实现Promise.prototype.catch()和finally()
date: 2021/2/8 17:48:00
categories: 
- ES6
tags: 
- Promise
comments: true
top: false
cover: false
keywords: 手写Promise ES6 Promise
---

### 1.Promise.prototype.catch()

Promise.prototype.catch()方法是.then(null, rejection)或.then(undefined, rejection)的别名，用于指定发生错误时的回调函数。
因此，catch函数只是then函数的语法糖而已，所以实现起来也非常的简单，只有一行代码：

``` javascript
class JunPromise {

	...
    //catch函数
    catch = (rejectCallback) => {
        return this.then(undefined, rejectCallback)
    }
	...

}
```


### 2.Promise.prototype.finally()

Promise.prototype.finally()方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。
语法有点像try-catch-finally，实际上finally方法是then方法的特例，因此返回的也是一个Promise对象，并且finally方法里面的内容与Promise的对象无关，实现如下：

``` javascript
class JunPromise {
	...
    //finally
    finally = (callback) => {
        return this.then(
            value => callback(value),
            reason => callback(reason)
        )
    }
	...
 }
```

考虑到callback回调函数为Promise对象的情况，修改如下：

``` javascript
class JunPromise {
	...
    //finally
    finally = (callback) => {
        return this.then(
            value => JunPromise.resolve(callback()).then(() => value),
            reason => JunPromise.resolve(callback()).then(() => { throw reason })
        )
    }
	...
 }
```

### 3.总结
附上本文[源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)的GIT链接，欢迎指错和讨论