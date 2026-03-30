## 微信小店一面
## 八股

- 输入URL并回车发生了什么

- js的原型链是什么

- 使用new的时候js在做什么事情

- 讲一下事件循环

- 如果一个微任务又放一个微任务到微队列, 会不会卡死

```js
console.log(1)

Promise.resolve().then(() => {
    console.log(2)
    Promise.resolve().then(() => {
        console.log(3)
    })
})

setTimeout(() => {
    console.log(4)
})

console.log(5)
```

打印什么

- js的作用域有哪些

- var是什么作用域

- 什么样创建一个块级作用域

- 什么东西可以声明块级作用域变量

```js
var x = 1
{
    console.log(x)
    let x = 2
}
```

打印什么

- 讲一下web的性能优化

- null和undefined有什么区别

- BFC是什么

- css的优先级是怎么样的

- 伪元素的优先级怎么样

- 讲一下js的this是什么

- 函数的this指向可以是什么, 怎么改变

- 一个函数bind得到的新函数, 再次bind得到第二个新函数, 那么原来函数内的this指向第一个还是第二个

## 算法

中缀运算式转后缀运算式

如`1+2-1*2`变`12+12*-`


## PCG技术栈-应用效能技术 客户端开发
### 八股
- 线程和进程有什么区别
- 什么时候用进程什么时候用线程
- 浏览器的一个tab是进程还是线程
- 死锁是什么
- 怎么解决死锁问题
- tcp为什么需要三次握手, 不是两次或四次
- tcp和udp有什么区别
- 举一个使用tcp的例子和使用udp的例子
- 讲一下tls握手

### 算法
```
// 输出最长单一字符子串

// 编程实现查找最长子字符串，其中子字符串须满足：由连续相同字符组成

// 输入：字符串

// 输出：最长单一字符子串

// 示例：输入str=“abbbbcccddddddddeee”，输出就是“dddddddd”

// 样例1:

// [输入]

// abbbbcccddddddddeee

// [输出]

// dddddddd

// [说明]

  

// 样例2:

// [输入]

// abbbbcccddddddddeeeccccccccccccc

// [输出]

// ccccccccccccc

// [说明]
```