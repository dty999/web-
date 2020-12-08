## 原型和原型链



看图说话:

![](https://img2020.cnblogs.com/blog/2232092/202012/2232092-20201203152738310-1962576917.png)



1、对象内部具有[[Prototype]]属性，该属性不可直接访问，浏览器通过__proto__（两条‘_’）可以让用户读写该内部属性，最重要的是，该属性指向创建本对象的原型对象。
2、每个原型对象内部都有一个独有属性constructor，指向该原型对象的构造函数。
3、原型对象也是对象，因此它也有自己的__proto__属性，最终会指向Object.prototype

线（9）Obect.prototype对象和Function.prototype对象都是浏览器创建的，它们并非谁的实例。通过__proto__属性将二者联系起来，从而使得函数也是对象。



###  我的理解


首先浏览器给了两个对象 Functon.prototype和Object.prototype.以及两个函数Function()和Object()

因为Functon.prototype和Object.prototype这两家伙也是对象,既然是对象就有__proto__属性,但是他两个是浏览器给的,并不是构造函数构造出来的.

==他两个不是谁的实例,可以看出来所有的实例都是对象,但对象并不一定是实例,人都是妈生的,但是第一对男女可能是女娲创造的==

然后神(浏览器)指定他们的__proto__,让Object.prototype的__proto__为null,Function.prototype对象的__proto__属性指向Object.prototype对象.

再然后浏览器创建了个Function()函数,既然是函数那就有prototype,就让这个函数的prototype指向Functon.prototype,这样他构造出来的函数对象的__proto__就指向Function.prototype
根据万物皆对象的指导原则,这个Function()函数也是对象,浏览器就让这个Function()函数的__proto__也指向Functon.prototype,怎么理解呢?
因为Function()函数并不是哪个函数谁构造出来的,然后没有构造函数也就没有构造函数的prototype,这样也就无法设置他的__proto__属性
但是他毕竟也是一个函数对象就让他的__proto__指向了自己prototype
逻辑上就是自己生成自己,让自己的__proto__指向自己的prototype
==是蛋生鸡还是鸡生蛋?都不是,是浏览器这位上古大神创造的鸡和蛋.

接下来就是Object()这个函数了,他的__proto__指向Function.prototype,这好理解,因为这个函数是始祖函数Function创造的,他的__proto__就指向Function的prototype
又因为他是专门用来生成对象的,所以我让他的prototype指向Object.prototype,这样以后通过他构造的对象的__proto__就指向Object.prototype了.

原型对象并不是当前构造函数的实例,如果不指定,默认是Object()函数的实例也就是Object.prototype{==注意Object.prototype虽然也是对象但不是谁的实例,浏览器创造的==},如果指定了那就另当别论.

![](https://img2020.cnblogs.com/blog/2232092/202012/2232092-20201203152750787-238856468.png)



### 下面就是代码案例了

~~~js
function Fn() {
    this.x = 100;
    this.y = 200;
    this.getX = function () {
        console.log(this.x);
    }
}
Fn.prototype.getX = function () {
    console.log(this.x);
};
Fn.prototype.getY = function () {
    console.log(this.y);
};
let f1 = new Fn;
let f2 = new Fn;
console.log(f1.getX === f2.getX);//false
console.log(f1.getY === f2.getY);//true
console.log(f1.__proto__.getY === Fn.prototype.getY);//true
console.log(f1.__proto__.getX === f2.getX);//false
console.log(f1.getX === Fn.prototype.getX);//false
console.log(f1.constructor);//Fn
console.log(Fn.prototype.__proto__.constructor);//Object()
f1.getX();//100
f1.__proto__.getX();//undefine
f2.getY();//200
Fn.prototype.getY();//undefine
~~~



~~~js
var F = function () {}
Object.prototype.a = function () {
    console.log('a()')
}
Function.prototype.b = function () {
    console.log('b()')
}
var f = new F()
F.a()//a
F.b()//b
f.a()//a
f.b()//报错
~~~



~~~js
function Parent() {
    this.a = 1;
    this.b = [1, 2, this.a];
    this.c = {
        demo: 5
    };
    this.show = function () {
        console.log(this.a, this.b, this.c.demo);
    }
}
function Child() {
    this.a = 2;
    this.change = function () {
        this.b.push(this.a);
        this.a = this.b.length;
        this.c.demo = this.a++;
    }
}
Child.prototype = new Parent();
var parent = new Parent();
var child1 = new Child();
var child2 = new Child();
child1.a = 11;
child2.a = 12;
parent.show();//1,[1,2,1],5
child1.show();//11,[1,2,1],5
child2.show();//12,[1,2,1],5
child1.change();
child2.change();
parent.show();// 1,[1,2,1],5
child1.show();//5,[1,2,1,11,12],5
child2.show();//6,[1,2,1,11,12],5
~~~

最后这个案例下图分析:

![](https://img2020.cnblogs.com/blog/2232092/202012/2232092-20201203152803935-745096653.png)

![](https://img2020.cnblogs.com/blog/2232092/202012/2232092-20201203154805746-1928122971.gif)
