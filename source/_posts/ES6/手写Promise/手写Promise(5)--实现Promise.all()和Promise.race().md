---
title: 手写Promise(5)--实现Promise.all()和Promise.race()
date: 2021/2/9 13:30:00
categories: 
- ES6
tags: 
- Promise
comments: true
top: false
cover: false
keywords: 手写Promise ES6 Promise
---

### 1.Promise.all()

Promise.all()方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。如下示例：

``` javascript
const p = Promise.all([p1, p2, p3]);
```
Promise.all()方法有以下特点：

- 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
- 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

> 注意：Promise.all()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。

实现如下：

``` javascript
class JunPromise {

	...
    //all方法
    static all = (pList) => {
        let result = [], indexLength = pList.length, index = 0
        return new JunPromise((resolve, reject) => {
            pList.forEach(p => {
                JunPromise.resolve(p).finally(() => {
                    if(p.status == 'rejected'){
                        reject(p.reason)
                    }
                    result.push(p)
                    index ++
                    if(index === indexLength){
                        resolve(result)
                    }
                })
            })
        })
    }
	...

}
```


### 2.Promise.race()

Promise.race()方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

``` javascript
const p = Promise.all([p1, p2, p3]);
```

上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

> 注意：Promise.race()方法的参数与Promise.all()方法一样，如果不是 Promise 实例，就会先调用Promise.resolve()方法，将参数转为 Promise 实例，再进一步处理。

实现如下：

``` javascript
class JunPromise {
	...
    // race方法
    static race = (pList) => {
        return new JunPromise((resolve, reject) => {
            pList.forEach(p => {
                JunPromise.resolve(p).finally(() => {
                    if(p.status == 'fulfilled'){
                        resolve(p.value)
                    }
                    else if(p.status == 'rejected'){
                        reject(p.reason)
                    }
                })
            })
        })
    }
	...
 }
```

### 3.总结
附上本文[源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)的GIT链接，欢迎指错和讨论