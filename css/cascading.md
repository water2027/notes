## 层叠

当样式冲突时, 浏览器会依次比较来源, 优先级, 源码顺序. 

### 来源

1. Transition(过渡中的样式): 正在执行CSS过渡的属性
2. !important浏览器默认样式
3. !important用户自定义样式
4. !important作者样式
5. Animation: 正在执行动画的css属性
6. 普通作者样式: 我们平常写的代码
7. 浏览器默认样式

优先级依次从高到低. 

### 优先级

通过一个四元组来判断, 依次比较:

1. 是否是内联, 是就是1, 不是就是0
2. id选择器的数量
3. 属性, 类, 伪类的数量
4. 元素, 伪元素的数量

比如

```css
#a .a .b h1 {
  font-size: larger;   
}

h1 {
  font-size: large;
}
```

那么四元组计算出来就是第一个里面的是0, 1, 2, 1, 第二个的是0, 0, 0, 1

依次比较, 会发现id选择器的数量是第一个大, 那么对于#a .a .b h1, font-size是larger. 

### 源码顺序

源码中, 靠后的覆盖靠前的. 

```css
h1 {
    font-size: larger;
}

h1 {
    font-size: large;
}
```

下面的靠后, 所以h1的font-size是large. 
