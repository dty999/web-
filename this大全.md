### this在什么时候创建?
在创建执行上下文的时候,执行上下文分三种(全局执行上下文,函数执行上下文,eval上下文(几乎不用先忽略)),所以说在用{}声明对象的时候是没有this的,全局一个this,函数执行时一个this.
现在既然知道this只有这两种创建方式,那么this的值是什么呢?
全局的this就是window,这个好说.
MDN上讲:在函数内部，this的值取决于函数被调用的方式。
我的理解是:函数被调用执行时离函数最近的前缀就是函数里this的指向,没有前缀直接调用[    **函数()**    ]其实前缀就是window,只是省略不写了.
还有另外几种情况需要讨论
1. call(),bind(),直接指定this的指向
2. new Fn()
3. 箭头函数的this
      箭头函数没有this,箭头函数的this指向取决于外层作用域中的this，外层作用域或函数的this指向谁，箭头函数中的this便指向谁
      不能直接修改箭头函数的this指向
      去修改被继承的普通函数的this指向，然后箭头函数的this指向也会跟着改变

下面时几个例子:
~~~js
var 对象 = {
    函数一:function(){
        function 函数二(){
            console.log(this)
        }
        return 函数二
    }
}
对象.函数一()()
//**对象.函数一()**这个整体相当于this所在的函数,而这个整体前面没有前缀所以this指向window
~~~
~~~js
var 对象 = {
    函数一:function(){
        function 函数二(){
            console.log(this)
        }
        return 函数二()
    }
}
对象.函数一()
//显然函数二执行时没有前缀,所以他里面的this指向window
~~~






this在函数调用时创建,一般的对象没有this,全局window可以理解为一个函数,他有一个全局this
==JavaScript 语言之所以有this的设计，跟内存里面的数据结构有关系。==
==函数里this的指向由函数的调用方式决定==

不服就干,几个例子来说明函数的调用方式

~~~js
var 变量 = 123
var 对象 = {
    变量: 456,
    函数: function () {
        var 变量 = 789
        console.log(this.变量)
    }
};
(对象.函数)(); 
//==>对象.函数()
(对象.函数 = 对象.函数)();
//var ret = (对象.函数 = 对象.函数);//执行括号里的赋值操作
//ret()
(对象.函数, 对象.函数)()
//var ret = (对象.函数, 对象.函数);返回最后一个
//ret()
~~~




~~~js
function a(xx) {
    this.x = xx;
    return this
};
var x = a(5);//此时x 是window
var y = a(6);//此时x 是6
console.log(x.x);//x是6 6.6 undefine
console.log(y.x);//window.x是6
~~~

~~~js
var 对象 = {
    函数一:function(){
        function 函数二(){
            console.log(this)
        }
        return 函数二()
    }
}
对象.函数一()//this是window,函数二虽然在函数一里面调用,但函数二前面没前缀,或者看做window.函数二(),所以函数二里的this就是window
~~~




~~~js
var num = 20;
var obj = {
    num:30,
    fn:(function (num) {
        this.num *= 3;
        num += 15;
        var num = 45;
        return function () {
            this.num *= 4;
            num += 20;
            console.log(num);
        }
    })(num)
}
var fn = obj.fn;
fn();//65
obj.fn();//85
console.log(window.num,obj.num);//240,120
~~~~



~~~js
var o = {
    f: function () {
        console.log(this);
    },
    2: function () {
        console.log(this);
    }
};
o.f(); //o
o[2]();//o !!!!!!!!!!这种和上一个等价
~~~





~~~js
var length = 10;

function fn() {
    console.log(this.length);
}
var obj = {
    length: 5,
    method: function (f) {
        f(); //10
        arguments[0](); //1  (arguments是一个伪数组对象,他的第一个参数是fn函数,他调用fn函数,所以this指的是arguments)
        arguments[0].call(this); //5(改变fn的指向,这里的this是obj)
    }
};
obj.method(fn);
~~~

## 箭头函数的this
箭头函数没有this,他的this是外层的this,外层的this一旦确定,这时候不管谁调用箭头函数,他的this都是原来确定的外层this,但是如果再改变外层的this,箭头函数的this跟着改变,

## call,apply,bind方法指定this
~~~js
函数.call(任意,函数参数,...)//call改变this会立即执行函数,第一个参数的类型不同改变的this指向也不同,如果是对象this就指向该对象,如果是字面量就转换成相应的对象this指向这个this,如果是null和undefined就是window
函数.apply(任意,[函数参数,...])
函数.bind(任意)//绑定后不会立即执行,啥时候再调用函数,函数里的this就是现在绑定的,==注意this只能改变一次==
~~~


![](/images/.gif)
