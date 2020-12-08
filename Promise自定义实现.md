#自定义代码
__这里是我自定义的Promise,如果想看原版,可以跳过最后有符合PromiseA+规范的源码__
~~~js
class 承诺 {
    constructor(处理器函数) { //1. 处理器函数是在_new 承诺(处理器函数)_的时候当做参数传入到构造函数里的,构造函数里会执行这个处理器函数.
        let self = this //2.注册函数执行时可能没有前缀,如果注册函数里用this可能就是window,这里把this赋给变量,形成闭包.
        this.状态 = '等待...' //3.这个属性有三个值,默认是等待,只有调用注册成功或注册失败才会改变这个属性的值,**只能改变一次**
        this.成功值 = undefined //4.这个成功值是由使用者调用注册成功函数的时候传进函数里的,然后我们在注册成功函数给这个属性赋值,简单点说就是形参赋值给属性
        this.失败原因 = undefined //5.同上
        //说明:这里需要了解'那时()'函数,可以看完'那时()'再来看,这两在异步的情况才会用到,使用者在调用'那时()'的时候,承诺仍然是等待状态(也就是说注册成功和注册失败两个函数没有被调用)
        this.成功回调函数组 = [] //6.承诺变为成功时要调用的函数,多次调用then传函数,我们就把'那时()'传进来的函数push到这个数组里,
        this.失败回调函数组 = [] //7.同上,承诺变为失败时要调用的函数

        let 注册成功 = function (成功值) { //7.'注册成功()'函数当实参传进了处理器函数,这样使用者在编写处理器函数得时候可以根据情况调用.
            if (self.状态 === '等待...') { //8.这个判断的作用:承诺的状态只能由等待->成功或等待->失败,而且只能改变一次.我们默认初始态是等待,如果不是等待说明之前已经调用过'注册成功或注册失败'
                self.成功值 = 成功值 //9.将使用者调用'注册成功(成功值)'函数传进来的值由对象属性保存,这个值在调用'那时()'传进来的函数时会用到
                self.状态 = '成功' //10.调用注册成功,改变承诺状态为成功
                for (let 函数 of self.成功回调函数组) { //11.调用'那时()'传进来的函数
                    函数(成功值)
                }
                // this.成功回调函数组.forEach(函数=>函数())
            }

        }
        let 注册失败 = function (失败原因) { //12. 同上
            if (self.状态 === '等待...') {
                self.失败原因 = 失败原因
                self.状态 = '失败'
                for (let 函数 of self.失败回调函数组) {
                    函数(失败原因)
                }
                // this.成功回调函数组.forEach(函数=>函数())
            }

        }


        try {
            处理器函数(注册成功, 注册失败) //13.执行处理器函数,把我们定义好的两个函数传进去,这样使用者就可以用这两个函数来改变承诺的状态和值
        } catch (错误) {
            注册失败(错误) //14.处理器函数是使用者编写的,有可能报错,出错了我们就调用注册失败()来改变这个承诺的状态和值
        }
    }

    那时(成功的回调, 失败的回调) { //15.对象的'那时()'方法,由使用者传进来两个函数参数,规定当前承诺对象为成功时调用第一个,失败调用第二个
        let self = this
        成功的回调 = typeof 成功的回调 === 'function' ? 成功的回调 : 成功值 => 成功值
        失败的回调 = typeof 失败的回调 === 'function' ? 失败的回调 : 失败原因 => {
            throw 失败原因
        } //16.这两个参数是可选参数,当他们不是函数时我们给他默认值

        let 新的承诺 = new 承诺((注册成功, 注册失败) => { //17.返回一个新的承诺,这样就可以链式调用'那时()'了,这里要注意新承诺的状态和值由'那时()'传进来的函数执行情况决定
            //18.判断承诺的状态,决定立即执行回调还是将回调函数push到回调函数组里等使用者调用注册成功()在注册成功()里执行
            if (self.状态 === '等待...') { //19.如果是等待,显然承诺的成功值或失败值还是undefined,所以我们把他push到回调函数组里,让他在注册成功()或注册失败()函数里调用
                self.成功回调函数组.push(() => { //20.这个函数要异步,因为我们会在里面用到新承诺,然而新承诺现在还没被赋值,要彻底理解这里应该需要js执行机制知识,目前我还没有....
                    setTimeout(() => {
                        try {
                            let 回调的返回值 = 成功的回调(self.成功值) //21.调用使用者传进来的函数得到返回值,如果使用者没有写返回语句默认是返回undefined
                            善后处理(新的承诺, 回调的返回值, 注册成功, 注册失败) //22.根据返回值决定我们这个新承诺的值和状态 这里传进去的是我们新承诺的注册成功和注册失败函数,我们在里面调用他们来改变我们新承诺的状态
                        } catch (错误) {
                            注册失败(错误) //23.如果执行使用者的函数出错就把我们新承诺的状态和值改变
                        }

                    })
                })

                self.失败回调函数组.push(() => { //24.同上
                    setTimeout(() => {
                        try {
                            let 回调的返回值 = 失败的回调(self.失败原因)
                            善后处理(新的承诺, 回调的返回值, 注册成功, 注册失败)
                        } catch (错误) {
                            注册失败(错误)
                        }
                    })
                })

            }
            if (self.状态 === '成功') { //25.如果承诺的状态是成功的说明注册成功()函数已经被调用了,承诺的状态和值都被使用者改变了,
                //我们可以取到对应的值来传进回调里,让使用者用.
                setTimeout(() => {
                    try { //逻辑同20-23
                        let 回调的返回值 = 成功的回调(self.成功值)
                        善后处理(新的承诺, 回调的返回值, 注册成功, 注册失败)
                    } catch (错误) {
                        注册失败(错误)
                    }

                })
            }
            if (self.状态 === '失败') { //同上
                setTimeout(() => {
                    try {
                        let 回调的返回值 = 失败的回调(self.失败原因)
                        善后处理(新的承诺, 回调的返回值, 注册成功, 注册失败)
                    } catch (错误) {
                        注册失败(错误)
                    }
                })
            }
        })


        return 新的承诺

    }

}
~~~
## 工具函数,这个函数才是真核心

~~~js

//26.这个是核心,涉及到递归,这里我写的和PromiseA+规范的实现不同,他判断的是thenable,兼容性好,我只是为了理解Promise原理所以就简化了.

function 善后处理(新的承诺, 回调的返回值, 注册成功, 注册失败) {

    // let p2 = p.那时((成功值)=>{
    //     return p2
    // })

    if (回调的返回值 instanceof 承诺) { //27.判断使用者写的回调函数的返回值是不是一个承诺,如果不是承诺就直接调用我们新承诺的注册成功()函数改变我们新承诺的状态和值
        //如果返回值是一个承诺我们就得到这个承诺的值,把这个值给我们新承诺的注册函数
        if (新的承诺 === 回调的返回值) {
            //28.这里是为了解决这种使用情况
            // let p2 = p.那时((成功值)=>{
            //     return p2
            // })
            注册失败(new TypeError('循环引用'))
            return
        }
        try {
            回调的返回值.那时((成功值) => {
                善后处理(新的承诺, 成功值, 注册成功, 注册失败) //29.如果使用者返回的承诺的值还是一个承诺,继续'那时()'直到不是承诺
                //注意:这里传进去的注册成功,注册失败是我们新承诺的注册函数,递归进去,当不是承诺时就改变我们新承诺的状态和值了,然后递归一层层返回
                //这里是进递归,其他情况就是出递归
            }, (失败原因) => {
                注册失败(失败原因)
            })
        } catch (错误) {
            注册失败(错误)
        }
    } else {
        注册成功(回调的返回值)
    }


}
~~~

## 自定义完成,我们拉出来遛一遛
~~~js
//测试一

p = new 承诺((注册成功, 注册失败) => {
    setTimeout(()=>{
        注册失败('abc')
    },1000)

})

p.那时((成功的值) => {
    console.log(成功的值)

},(失败原因)=>{
    console.log(失败原因)
})
//输出abc
~~~
~~~js
//测试二:返回值是承诺

p = new 承诺((注册成功, 注册失败) => {
    setTimeout(() => {
        注册成功('我是最外面的')
    }, 1000)

})

p.那时((成功值) => {
        console.log(成功值)
        let p11 = new 承诺((注册成功, 注册失败) => {
            let p22 = new 承诺((注册成功, 注册失败) => {
                注册成功('最里层')
            })
            注册成功(p22)
        })
        return p11
    }, 1)
    .那时((成功值) => {
        console.log(成功值)
    }, (失败原因) => {
        console.log(失败原因)
    })
//输出:
//我是最外层
//我是最里层
~~~
~~~js
//测试三:失败穿透
p = new 承诺((注册成功, 注册失败) => {
    throw '(╥╯^╰╥)'
    setTimeout(() => {
        注册成功('♪(´▽｀)')

    }, 1000)

})

p.那时((成功的值) => {
    console.log(成功的值)
    
},(失败原因)=>{
    console.log('第一次失败:'+失败原因)
    throw '(╥╯^╰╥))'
}).那时(
    (成功的值) => {
        console.log(成功的值)

    }
).那时(null,(失败原因) => {
    console.log('失败穿透'+失败原因)
})
//输出
//第一次失败:(╥╯^╰╥)
//失败穿透(╥╯^╰╥))  
~~~
![](https://img2020.cnblogs.com/blog/2232092/202012/2232092-20201206170814750-874730263.gif)


## 到这里就结束了,下面是通过PromiseA+测试的源代码.
~~~js
function Promise(executor) {
    let self = this;
    self.value = undefined; // 成功的值
    self.reason = undefined; // 失败的值
    self.status = 'pending'; // 目前promise的状态pending
    self.onResolvedCallbacks = []; // 可能new Promise的时候会存在异步操作，把成功和失败的回调保存起来
    self.onRejectedCallbacks = [];

    function resolve(value) { // 把状态更改为成功
        if (self.status === 'pending') { // 只有在pending的状态才能转为成功态
            self.value = value;
            self.status = 'resolved';
            self.onResolvedCallbacks.forEach(fn => fn()); // 把new Promise时异步操作，存在的成功回调保存起来
        }
    }

    function reject(reason) { // 把状态更改为失败
        if (self.status === 'pending') { // 只有在pending的状态才能转为失败态
            self.reason = reason;
            self.status = 'rejected';
            self.onRejectedCallbacks.forEach(fn => fn()); // 把new Promise时异步操作，存在的失败回调保存起来
        }
    }
    try {
        // 在new Promise的时候，立即执行的函数，称为执行器
        executor(resolve, reject);
    } catch (e) { // 如果执行executor抛出错误，则会走失败reject
        reject(e);
    }
}
// then调用的时候，都是属于异步，是一个微任务
// 微任务会比宏任务先执行
// onFulfilled为成功的回调，onRejected为失败的回调
Promise.prototype.then = function (onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val;
    onRejected = typeof onRejected === 'function' ? onRejected : err => {
        throw err
    }
    let self = this;
    let promise2;
    // 上面讲了，promise和jquery的区别，promise不能单纯返回自身，
    // 而是每次都是返回一个新的promise，才可以实现链式调用，
    // 因为同一个promise的pending resolve reject只能更改一次
    promise2 = new Promise((resolve, reject) => {
        if (this.status === 'resolved') {
            // 为什么要加setTimeout？
            // 首先是promiseA+规范要求的
            // 其次是大家写的代码，有的是同步，有的是异步
            // 所以为了更加统一，就使用为setTimeout变为异步了，保持一致性
            setTimeout(() => {
                try { // 上面executor虽然使用try catch捕捉错误
                    // 但是在异步中，不一定能够捕捉，所以在这里
                    // 用try catch捕捉
                    let x = onFulfilled(self.value);
                    // 在then中，返回值可能是一个promise，所以
                    // 需要resolvePromise对返回值进行判断
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0)
        }
        if (self.status === 'rejected') {
            setTimeout(() => {
                try {
                    let x = onRejected(self.reason);
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0)
        }
        if (self.status === 'pending') {
            self.onResolvedCallbacks.push(() => {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(self.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                }, 0)
            });
            self.onRejectedCallbacks.push(() => {
                setTimeout(() => {
                    try {
                        let x = onRejected(self.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                }, 0)
            });
        }
    });
    return promise2
}


function resolvePromise(promise2, x, resolve, reject) {
    // 3.从2中我们可以得出，自己不能等于自己
    // 当promise2和x是同一个对象的时候，则走reject
    if (promise2 === x) {
        return reject(new TypeError('Chaining cycle detected for promise'))
    }
    // 4.因为then中的返回值可以为promise，当x为对象或者函数，才有可能返回的是promise
    let called
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
        // 8.从第7步，可以看出为什么会存在抛出异常的可能，所以使用try catch处理
        try {
            // 6.因为当x为promise的话，是存在then方法的
            // 但是我们取一个对象上的属性，也有可能出现异常，我们可以看一下第7步
            let then = x.then

            // 9.我们为什么在这里用call呢？解决了什么问题呢？可以看上面的第10步
            // x可能还是个promise，那么就让这个promise执行
            // 但是还是存在一个恶作剧的情况，就是{then:{}}
            // 此时需要新增一个判断then是否函数
            if (typeof then === 'function') {
                then.call(x, (y) => { // y是返回promise后的成功结果
                    // 一开始我们在这里写的是resolve(y)，但是考虑到一点
                    // 这个y，有可能还是一个promise，
                    // 也就是说resolve(new Promise(...))
                    // 所以涉及到递归，我们把resolve(y)改成以下

                    // 12.限制既调resolve，也调reject
                    if (called) return
                    called = true

                    resolvePromise(promise2, y, resolve, reject)
                    // 这样的话，代码会一直递归，取到最后一层promise

                    // 11.这里有一种情况，就是不能既调成功也调失败，只能挑一次，
                    // 但是我们前面不是处理过这个情况了吗？
                    // 理论上是这样的，但是我们前面也说了，resolvePromise这个函数
                    // 是所有promise通用的，也可以是别人写的promise，如果别人
                    // 的promise可能既会调resolve也会调reject，那么就会出问题了，所以我们接下来要
                    // 做一下限制，这个我们写在第12步

                }, (err) => { // err是返回promise后的失败结果
                    if (called) return
                    called = true
                    reject(err)
                })
            } else {
                if (called) return;
                called = true;
                resolve(x) // 如果then不是函数的话，那么则是普通对象，直接走resolve成功
            }
        } catch (e) { // 当出现异常则直接走reject失败
            if (called) return
            called = true
            reject(e)
        }
    } else { // 5.x为一个常量，则是走resolve成功
        resolve(x)
    }
}







module.exports = Promise;

Promise.defer = Promise.deferred = function () {
    let dfd = {};
    dfd.promise = new Promise((resolve, reject) => {
        dfd.resolve = resolve;
        dfd.reject = reject;
    });
    return dfd;
}
//执行命令promises-aplus-tests promise.js检测是否符合promiseA+规范,先安装包
~~~
