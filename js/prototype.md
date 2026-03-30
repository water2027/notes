
原型分为显式原型(`prototype`)和隐式原型(`__proto__`). 

## `prototype`

有且仅有函数拥有prototype属性. 它是一个**包含所有实例共享的属性和方法**的对象. 

## `__proto__`

所有对象都拥有`__proto__`, 指向构造函数的prototype. 

eg:

```javasciprt
function Person() {
	this.name = '1'
}

const p1 = new Person()
console.log(p1.__proto__ === Person.prototype) // true
```

> 目前更推荐使用Object.getPrototypeOf来访问隐式原型. 

## 原型链

这是js用来查找对象属性的一种机制

当访问对象某个方法或属性时:

1. 在对象自身上找属性. 找到了, 直接使用
2. 没有找到, 找对象的隐式原型. 如果隐式原型上有, 直接用. 
3. 没有找到, 找隐式原型的隐式原型. 重复这个过程. 

由`__proto__`串联起来的链条, 就是原型链. 