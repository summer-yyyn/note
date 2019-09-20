闭包
把一个东西放在里面，外面拿不到，只给外边拿到想给他用的东西

### Promise干嘛用
##### 一个异步解决方案
> 例如 多个setTimeout()嵌套造成回调地狱

```
setTimeout(function() {
    // do something..
    setTimeout(function() {
        // do something..
    }, 1000)
}, 1000)
```
###### 可以用Promise解决

```
new Promise(function(resolve, reject){
    setTimeout(function() {
        // do something..
        resolve(res)
    }, 1000)
}).then(res => {
    setTimeout(function() {
        // do something..
        return res
    }, 1000)
}).then(res => {
    setTimeout(function() {
        // do something..
        return res
    }, 1000)
}).catch(err => {
    console.log(err)
})
```



### Promise怎么用？

```
new Promise(function(resolve, reject){
    resolve(res) // 进入.then()
    reject(res) // 进入.catch()
}).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})
```

### Promise特点
1. Promise状态(PromiseStatus)一旦改变，不可逆，不可再更改
```
new Promise(function(resolve, reject){
    resolve(res) // 只能进入.then()
    reject(res) // 不会进入.catch 
}).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})
```
2. Promise的then方法参数期望收到一个函数，如果传入非函数，则发生值穿透
```
new Promise(function(resolve, reject){
    resolve('123')
}).then('res').then(res => {
    console.log(res) // 打印出‘123’ 值穿透到下一个then里面
})
```

```
new Promise(function(resolve, reject){
    resolve('123')
}).then('res').catch(res => {
    console.log(res) // 值不会穿透进入catch
})
```
3. Promise回调是同步的，then的回调是异步的

```
new Promise(function(resolve, reject){
    console.log(1)
    resolve('123')
}).then(res => {
    console.log(2)
    console.log(res)
    return(`${res}4`)
}).then(res => {
    console.log(3)
    console.log(res)
})
console.log(4)
```
==结果==

```
1
4
2
123
3
1234
```
4. Promise链式调用.then() 如果有return的话 返回的是一个promise对象，如果不return 则下一个then接不到参数；如果==抛出异常==则返回一个reject状态的Promise，进入catch.上一个then的返回值是下一个then接收到的参数。

```
new Promise(function(resolve, reject){
    resolve('123')
}).then(res => {
    console.log(res)
    throw new Error('this is an error')
}).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err) // 进入catch  在此打印 this is an error
})
```
==必须抛出异常throw error，如果return的话就进入下一个then了==

```
new Promise(function(resolve, reject){
    resolve('123')
}).then(res => {
    console.log(res)
    return new Error('this is an error')
}).then(res => {
    console.log(res) // 进入then  在此打印 this is an error
}).catch(err => {
    console.log(err)
})
```
5. then的回调里return一个Promise会进入等待状态，直到return的Promise改变

```
new Promise(function(resolve, reject){
    resolve('123')
}).then(res => {
    return new Promise(function(resolve, reject) {
        setTimeout(function(){
            resolve('456') // 3秒后打印456
            // reject('789') // 3秒后打印789 
        }, 3000)
    }) // return中的promise状态发生改变后才会继续执行
}).then(res => {
    console.log(res) // 3秒后打印456
}).catch(err => {
    console.log(err) // 3秒后打印789 
})
```
==return中的promise状态发生改变后才会继续执行==
###### return中肯定是返回一个Promise的，但是返回任何其他的值，都是一个成功的回调，只有new Promise的时候会等待

```
new Promise(function(resolve, reject){
    resolve('123')
}).then(res => {
    return setTimeout(function(){
        console.log(888) // 3秒后打印888
    }, 3000)
}).then(res => {
    console.log(res) // 不会等待，直接打印一串数字，
}).catch(err => {
    console.log // 不会进入这里
})
```
### Javascript 异步机制
##### Javascript执行顺序
1. JavaScript先扫描一遍代码
2. 主线程，宏任务队列，微任务队列
3. 主线程执行完一遍，先查询微任务队列，如果有任务，执行完毕
4. 再查询宏任务队列，如果宏任务队列有任务，将宏任务中的一个任务拿到主线程，将其执行，执行完再询问微任务。。。以此类推

> 举例 
> Promise属于微任务 
> setTimeout属于宏任务

```
setTimeout(() => {
    console.log('set1')   
});

let p1 = new Promise((resolve, reject) => {
    console.log('promise1')
})

setTimeout(() => {
    console.log('set2')
})

p1.then(() => {
    console.log('then1')
})

console.log(2)
```
###### 执行结果

```
promise1
2
then1
set1
set2
```
1. set1入宏任务队列
2. p1 (new Promise) 同步,直接执行  // promise1
3. set2入宏任务队列
4. p1.then 异步 入微任务队列
5. console 2 直接执行 // 2
6. 执行微任务队列 p1.then // then1
7. 宏任务set1取出至主线程执行 // set1
8. 微任务空，宏任务set2取出至主线程执行 // set2
9. 结束

---

```
setTimeout(() => {
    console.log('set1')
    new Promise((resolve, reject) => {
        console.log('promise2')
        resolve(2)
    }).then(res => {
        console.log('then2')
    })
});

let p1 = new Promise((resolve, reject) => {
    console.log('promise1')
    resolve(2)
})

setTimeout(() => {
    console.log('set2')
})

p1.then((res) => {
    console.log('then1')
})

console.log(2)
```
###### 执行结果

```
promise1
2
then1
set1
promimse2
then2
set2
```
1. 扫描
2. set1入宏任务队列
3. p1同步直接执行   // promise1
4. set2入宏任务队列
5. p1.then异步入微任务队列
6. console 2 直接执行  // 2
7. 微任务队列p1.then执行  // then1
8. 宏任务set1取出至主线程执行，promise2同步直接执行，promise2.then异步 入微任务 // set1 , promise2
9. 主线程执行完毕，找微任务，执行promise2.then // then2
10. 宏任务set2取出至主线程执行 // set2
### 自己实现一个promise

```
// 回调函数里执行resolve - then方法把回调加入resolveArr-执行整个resolveArr，并且改变状态返回新的promise

var isFunction = function(fn) {
    if (typeof fn === 'function') {
        return true
    } else {
        return false
    }
}

Function.prototype.bind = function(context) {
    var self = this
    return function() {
        // call 明确知道参数有多少， apply 参数不定的情况下使用
        self.apply(context, arguments)
    }
}

// 通过then 注册一个任务回调， resolve触发，执行这个then的注册函数
function mypromise(handle) {
    this.status = 'PENDING' // 状态
    this.val = undefined // 值，resolve 的参数
    this.resolveArr = [] // 回调队列，then里面定义的方法加入到此
    this.rejectArr = [] // reject回调队列
    this.resolve = function(val) {
        // 执行resolveArr， 改变mypromise状态
        if (this.status !== 'PENDING') return  // 状态不可逆由此控制
        this.status = 'RESOLVE'
        this.val = val
        let cb;
        setTimeout(() => { // resolveArr中所有全部执行  then在这里变成异步（重要）
            while (cb = this.resolveArr.shift()) { // 把resolveArr第一个赋值给cb 有的话就是ture
                if (isFunction(cb)) {
                    cb(val)
                }
            }
        })
    }
    this.reject = function(err) {
        if (this.status !== 'PENDING') return  // 状态不可逆由此控制
        this.status = 'REJECT'
        this.val = err
    }
    // then里面抛出错误的话，如果不在catch里进行处理，就拿不到这个错误
    try{
        // this.resolve，相当于把this.resolve方法取出来，传给handle，handle调用的时候this指向window 所以bind this
        handle(this.resolve.bind(this), this.reject)
    } catch(err) {
        throw err
    }
}
// then  方法是promise对象调用，所以要放在prototype里
mypromise.prototype.then = function(suc, err) {
    const val = this.val
    const status = this.status
    return new mypromise((sucnext, errnext) => {
        // 享元模式 优化 先找出不一样的拿出来定义，再把一样的取出来，把不一样的配进去
        let _fn = undefined
        let _handle = undefined
        let run = function() {
            try{
                console.log(123)
                if (!isFunction(_handle)) {  // then传进的不是function时直接给出去
                    _fn(val)
                } else {
                    let res = _handle(val)
                    console.log(sucnext)
                    sucnext(res)
                }
            } catch(err) {
                errnext(err) // then 抛出错误catch到， 所以reject
            }
        }
        // let success = function() {
        //     try{
        //         if (!isFunction(suc)) {
        //             resolve(suc)
        //         } else {
        //             let res = suc(val)
        //             resolve(res)
        //         }
        //     } catch(err) {
        //         reject(err) // then 抛出错误catch到， 所以reject
        //     }
        // }
        // let fail = function() {
        //     try{
        //         if (!isFunction(suc)) {
        //             reject(suc)
        //         } else {
        //             let res = err(val)
        //             resolve(res)
        //         }
        //     } catch(err) {
        //         reject(err) // then 抛出错误catch到， 所以reject
        //     }
        // }
        switch (status) {
            case 'PENDING':
                this.resolveArr.push(suc)
                this.rejectArr.push(suc)
                break
            case 'RESOLVE':
                _fn = resolve
                _handle = suc
                run()
                // success()
                break
            case 'REJECT':
                _fn = reject
                _handle = err
                run()
                // fail()
                break
        }
    })
}


new mypromise(function(resolve, reject) {
    setTimeout(function() {
        resolve(4)
    }, 3000)
    console.log(33)
}).then(res => {
    console.log(res)
})
```

