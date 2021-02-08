---
title: 手写Promise(2)--实现Promise.prototype.then()
date: 2021/1/31 18:17:00
categories: 
- ES6
tags: 
- Promise
comments: true
top: false
cover: false
keywords: 手写Promise ES6 Promise
---

Promise 实例具有then方法，它的作用是为 Promise 实例添加状态改变时的回调函数。then有两个参数，第一个参数是resolved状态的回调函数，第二个参数是rejected状态的回调函数，并且它们都是可选的

### 1.then的异步调用

在第一节中，then已经可以直接调用，但是如果传入的是一个异步操作，则无法满足需求，如下事例：

``` javascript
let p = new JunPromise(function(resolve, reject){
	setTimeout(() => {
		resolve(111)
	}, 1000);
})

p.then(function(value){
	console.log('value :>> ', value);
}, function(error){
	console.log('error :>> ', error);
})
```

上面的例子最后没有任何输出，因为resolve在一个异步操作中(定时器)，所以在调用then方法的方式，当Promise状态位pending。因此我们需要修改then方法，使其满足异步操作。
修改的方法就是：如果当前状态是pending，则使用两个变量记录then的两个回调参数，在resolve或者reject时调用。

``` javascript
class JunPromise {
    //构造函数
    constructor(execute){
        execute(this.resolve, this.reject)
    }

    //状态 (pending-等待，fulfilled-完成，rejected-失败)
    status = 'pending'
    //回调成功的值
    value = undefined
    //回调失败的值
    reason = undefined
    //成功回调函数
    resolveCallback = undefined
    //失败回调函数
    rejectCallback = undefined

    //状态pending => fulfilled
    resolve = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'fulfilled'
        this.value = value

        //成功回调已存在 则回调并返回成功值
        this.resolveCallback && this.resolveCallback(value)
    }

    //状态pending => rejected
    reject = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'rejected'
        this.reason = value

        //失败回调已存在 则回调并返回失败值
        this.rejectCallback && this.rejectCallback(value)
    }

    //then方法
    then = (resolveCallback, rejectCallback) => {
        if(this.status == 'fulfilled') {
            resolveCallback(this.value)
        } else if(this.status == 'rejected') {
            rejectCallback(this.reason)
        } else {
            //等待状态，记录回调函数
          	this.resolveCallback = resolveCallback
          	this.rejectCallback = rejectCallback
        }
    }
 }
```

### 2.多个then调用

then函数可以多次重复调用，前面讲过 Promise的状态一旦确认，就不可以变化。因此多次调用then函数得到的结果也是一样。
修改记录then回调参数为数组，当调用then方法状态为pending时，则压入记录回调数组中。

``` javascript
class JunPromise {
    //构造函数
    constructor(execute){
        execute(this.resolve, this.reject)
    }

    //状态 (pending-等待，fulfilled-完成，rejected-失败)
    status = 'pending'
    //回调成功的值
    value = undefined
    //回调失败的值
    reason = undefined
    //成功回调函数
    resolveCallback = []
    //失败回调函数
    rejectCallback = []

    //状态pending => fulfilled
    resolve = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'fulfilled'
        this.value = value

        //成功回调已存在 则回调并返回成功值
        //this.resolveCallback && this.resolveCallback(value)

        //循环调用成功回调函数
        while(this.resolveCallback.length > 0){
            let callback = this.resolveCallback.shift()
            callback && callback(value)
        }
    }

    //状态pending => rejected
    reject = (value) => {
        if(this.status != 'pending'){
            return
        }
        this.status = 'rejected'
        this.reason = value

        //失败回调已存在 则回调并返回失败值
        //this.rejectCallback && this.rejectCallback(value)

        //循环调用成功回调函数
        while(this.resolveCallback.length > 0){
            let callback = this.resolveCallback.shift()
            callback && callback(value)
        }
    }

    //then方法
    then = (resolveCallback, rejectCallback) => {
        if(this.status == 'fulfilled') {
            resolveCallback(this.value)
        } else if(this.status == 'rejected') {
            rejectCallback(this.reason)
        } else {
			//等待状态，记录回调函数
          	//this.resolveCallback = resolveCallback
          	//this.rejectCallback = rejectCallback
		
            // 等待状态，记录回调函数，等待事件回调
            this.resolveCallback.push(resolveCallback)
            this.rejectCallback.push(rejectCallback)
        }
    }
 }
```

### 3.then的链式调用

实际上调用then方法得到的返回是一个Promise对象，因此then方法是允许链式调用。这也是使用Promise一个很大的优势，使用链式调用可以解决多层异步嵌套的回调地狱。
修改then方法，用Promise对象进行封装，当前then方法的返回值当成下一个then方法的入参。

``` javascript
...
//then方法
then = (resolveCallback, rejectCallback) => {
    let _promise = new JunPromise((resolve, reject) => {
        if(this.status == 'fulfilled') {
			// 将回调的结果传给下一个then方法
            let resultPromise = resolveCallback(this.value)
            resovle(resultPromise)
        } else if(this.status == 'rejected') {
			// 将回调的结果传给下一个then方法
            let resultPromise = rejectCallback(this.reason)
            reject(resultPromise)
        } else {
            // 等待状态，记录回调函数，等待时间回调
            // this.resolveCallback.push(resolveCallback)
            // this.rejectCallback.push(rejectCallback)
			
			// 等待状态，记录回调函数，等待事件回调
            // then链式调用的时候，就需要把结果也返回给下一个then方法
            this.resolveCallback.push(() => {
                let resultPromise = resolveCallback(this.value)
                resolve(resultPromise)
            })
            this.rejectCallback.push(() => {
                let resultPromise = rejectCallback(this.value)
                reject(resultPromise)
            })
        }
    })
    return _promise
}
...
```

### 4.then返回的Promise不可以是自身

then方法返回的Promise对象，是不允许为自身。因此新增 judgmentPromise 方法进行判断then方法的返回值
- 如果是自己，则报错
- 如果是常数，则直接返回resolve(常数)
- 如果是Promise对象，则根据结果返回Promise.then()

因为判断当前Promise对象是在实例化当前对象里面执行，在 new JunPromise() 没有执行完是无法拿到当前对象，因此将 judgmentPromise 方法放入异步执行中。

``` javascript
...
//then方法
then = (resolveCallback, rejectCallback) => {
    let _promise = new JunPromise((resolve, reject) => {
        if(this.status == 'fulfilled') {
			// 放入异步中，否则，此处_promise拿不到
			// 将回调的结果传给下一个then方法
            setTimeout(() => {
                let resultPromise = resolveCallback(this.value)
                // return result
                judgmentPromise(_promise, resultPromise, resolve, reject)
            }, 0);
        } else if(this.status == 'rejected') {
			// 放入异步中，否则，此处_promise拿不到
			// 将回调的结果传给下一个then方法
            setTimeout(() => {
                let resultPromise = rejectCallback(this.reason)
                judgmentPromise(_promise, resultPromise, resolve, reject)
            }, 0);
        } else {
            // 等待状态，记录回调函数，等待时间回调
            // this.resolveCallback.push(resolveCallback)
            // this.rejectCallback.push(rejectCallback)

            // 等待状态，记录回调函数，等待事件回调
            // then链式调用的时候，就需要把结果也返回给下一个then方法
			this.resolveCallback.push(() => {
				setTimeout(() => {
					let resultPromise = resolveCallback(this.value)
					judgmentPromise(_promise, resultPromise, resolve, reject)
				}, 0);
			})
			this.rejectCallback.push(() => {
				setTimeout(() => {
					let resultPromise = rejectCallback(this.reason)
					judgmentPromise(_promise, resultPromise, resolve, reject)
				}, 0);
			})
        }
    })
    return _promise
}

 /**
  * 判断链式的返回值
  * 如果是自己，则报错
  * 如果是常数，则直接返回resolve(常数)
  * 如果是Promise对象，则根据结果返回Promise.then()
  */
 const judgmentPromise = (self, resultPromise, resolve, reject) => {
    if(self === resultPromise){
        reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
        console.error('Chaining cycle detected for promise #<Promise>')
    }
    if(resultPromise instanceof JunPromise){
        resultPromise.then(resolve, reject)
    }else{
        resolve(resultPromise)
    }
 }
...
```

### 5.then的参数可选

then方法的两个参数都是可选参数，如果没有传入，则当前的状态会传下下一个then方法。

``` javascript
...
//then方法
then = (resolveCallback, rejectCallback) => {
    //判断then方法是否有回调
    resolveCallback = resolveCallback ? resolveCallback : value => value
    rejectCallback = rejectCallback ? rejectCallback : reason => reason

    let _promise = new JunPromise((resolve, reject) => {
        if(this.status == 'fulfilled') {
			// 放入异步中，否则，此处_promise拿不到
			// 将回调的结果传给下一个then方法
            setTimeout(() => {
                let resultPromise = resolveCallback(this.value)
                // return result
                judgmentPromise(_promise, resultPromise, resolve, reject)
            }, 0);
        } else if(this.status == 'rejected') {
			// 放入异步中，否则，此处_promise拿不到
			// 将回调的结果传给下一个then方法
            setTimeout(() => {
                let resultPromise = rejectCallback(this.reason)
                judgmentPromise(_promise, resultPromise, resolve, reject)
            }, 0);
        } else {
            // 等待状态，记录回调函数，等待时间回调
            // this.resolveCallback.push(resolveCallback)
            // this.rejectCallback.push(rejectCallback)

            // 等待状态，记录回调函数，等待事件回调
            // then链式调用的时候，就需要把结果也返回给下一个then方法
			this.resolveCallback.push(() => {
				setTimeout(() => {
					let resultPromise = resolveCallback(this.value)
					judgmentPromise(_promise, resultPromise, resolve, reject)
				}, 0);
			})
			this.rejectCallback.push(() => {
				setTimeout(() => {
					let resultPromise = rejectCallback(this.reason)
					judgmentPromise(_promise, resultPromise, resolve, reject)
				}, 0);
			})
        }
    })
    return _promise
}
...
```

### 6.加入try,catch

最后，将then方法里面加上try-catch，一旦出错了就调用reject回调方法

``` javascript
//then方法
then = (resolveCallback, rejectCallback) => {
    //判断then方法是否有回调
    resolveCallback = resolveCallback ? resolveCallback : value => value
    rejectCallback = rejectCallback ? rejectCallback : reason => reason

        let _promise = new JunPromise((resolve, reject) => {
            if(this.status == 'fulfilled') {
                // 放入异步中，否则，此处_promise拿不到
                // 将回调的结果传给下一个then方法
                setTimeout(() => {
                    try {
                        let resultPromise = resolveCallback(this.value)
                        judgmentPromise(_promise, resultPromise, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                }, 0);
            } else if(this.status == 'rejected') {
                // 放入异步中，否则，此处_promise拿不到
                // 将回调的结果传给下一个then方法
                setTimeout(() => {
                    try {
                        let resultPromise = rejectCallback(this.reason)
                        judgmentPromise(_promise, resultPromise, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                }, 0);
            } else {
                // 等待状态，记录回调函数，等待时间回调
                // this.resolveCallback.push(resolveCallback)
                // this.rejectCallback.push(rejectCallback)
    
                // 等待状态，记录回调函数，等待事件回调
                // then链式调用的时候，就需要把结果也返回给下一个then方法
                this.resolveCallback.push(() => {
                    setTimeout(() => {
                        try {
                            let resultPromise = resolveCallback(this.value)
                            judgmentPromise(_promise, resultPromise, resolve, reject)
                        } catch (error) {
                            reject(error)
                        }
                    }, 0);
                })
                this.rejectCallback.push(() => {
                    setTimeout(() => {
                        try {
                            let resultPromise = rejectCallback(this.reason)
                            judgmentPromise(_promise, resultPromise, resolve, reject)
                        } catch (error) {
                            reject(error)
                        }
                    }, 0);
                })
            }
        })
        return _promise
}
...
```

### 7.总结
以上 Promise.prototype.then() 完成，附上本文[源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)的GIT链接，欢迎指错和讨论