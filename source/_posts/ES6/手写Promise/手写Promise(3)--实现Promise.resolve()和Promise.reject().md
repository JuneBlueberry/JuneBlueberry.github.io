---
title: 手写Promise(3)--实现Promise.resolve()和Promise.reject()
date: 2021/2/4 23:13:00
categories: 
- ES6
tags: 
- Promise
comments: true
top: true
cover: false
keywords: 手写Promise ES6 Promise
---

Promise 实例具有then方法，它的作用是为 Promise 实例添加状态改变时的回调函数。then有两个参数，第一个参数是resolved状态的回调函数，第二个参数是rejected状态的回调函数，并且它们都是可选的

### 1.Promise.resolve()

Promise对象有resolve静态方法，可以将现有对象转为 Promise 对象，会根据传入的参数不同做不同的处理
- 参数是 Promise 实例，将不做任何修改、原封不动地返回这个实例。
- 参数是一个thenable(具有then方法的对象)对象，将返回一个Promise对象，并立即执行then函数
- 参数是一个原始值，或者是一个不具有then()方法的对象，将返回一个新的 Promise 对象，状态为fulfilled
- 没有参数，将直接返回一个fulfilled状态的 Promise 对象

``` javascript
class JunPromise {

	...
	//resolve方法
    static resolve = (callback) => {
        if(callback instanceof JunPromise){
            return callback
        } 
        else if (typeof callback == 'object' && callback.then){
            return new JunPromise(resolve => callback.then(resolve))
        } 
        else {
            return new JunPromise(resolve => resolve(callback))
        }
    }
	...

}
```

### 2.Promise.reject()

Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。

``` javascript
class JunPromise {
	...
	//reject方法
	static reject = (callback) => {
        return new JunPromise((resolve, reject) => reject(callback))
    }
	...
 }
```

### 3.总结
附上本文[源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)的GIT链接，欢迎指错和讨论