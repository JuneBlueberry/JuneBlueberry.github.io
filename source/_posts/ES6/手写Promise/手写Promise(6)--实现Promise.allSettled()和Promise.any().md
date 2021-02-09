---
title: 手写Promise(6)--实现Promise.allSettled()和Promise.any()
date: 2021/2/9 14:14:00
categories: 
- ES6
tags: 
- Promise
comments: true
top: false
cover: false
keywords: 手写Promise ES6 Promise
---

### 1.Promise.allSettled()

Promise.allSettled()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。特点如下：

- 只有等到所有参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。
- 包装实例一旦结束，状态总是fulfilled，不会变成rejected
- 该方法返回一个数组，依次包含每个实例的Promise对象

> 注意：Promise.allSettled()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。


> 该方法由 ES2020 引入。

实现如下：

``` javascript
class JunPromise {

	...
    //allSettled方法
    static allSettled = (pList) => {
        let result = [], indexLength = pList.length
        return new JunPromise((resolve, reject) => {
            pList.forEach(p => {
                JunPromise.resolve(p).finally(() => {
                    result.push(p)
                    if(result.length === indexLength){
                        resolve(result)
                    }
                })
            })
        })
    }
	...

}
```


### 2.Promise.any()

Promise.any()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回，特点如下：

- 只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态，并返回首个fulfilled状态对象的成功值
- 如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。并返回包含所有rejected状态实例对象的数组

Promise.any()跟Promise.race()方法很像，只有一点不同，就是不会因为某个 Promise 变成rejected状态而结束。

> 注意：Promise.any()方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。

>该方法由 ES2021 引入。 

实现如下：

``` javascript
class JunPromise {
	...
    //any方法
    static any = (pList) => {
        let isSuccess = false, indexLength = pList.length, index = 0
        let successValue = undefined, failReasonList = []
        return new JunPromise((resolve, reject) => {
            pList.forEach(p => {
                JunPromise.resolve(p).finally(() => {
                    if(!isSuccess){
                        if(p.status == 'fulfilled'){
                            isSuccess = true
                            successValue = p.value
                        } else if (p.status == 'rejected'){
                            failReasonList.push(p.reason)
                        }
    
                        index++
                        if(isSuccess){
                            resolve(successValue)
                        }
                        if(index === indexLength){
                            throw failReasonList
                        }
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