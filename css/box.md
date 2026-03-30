页面基本都是由盒子构成的,一般分为行盒和块盒, 主要由display决定. 

- `display: block;` 块盒
- `display: inline` 行盒

不存在块级元素或者行级元素, 一些元素天生就是行盒或者块盒只是因为相关规范定义这些元素的display初始值是block或者inline而已. 

对于文字相关的标签, 一般初始值是行盒. 

## 盒子的显示类型

### 外部显示类型

外部显示类型决定盒子与其它盒子的关系. 

块盒和行盒都是盒子的外部显示类型. 

一个block外部显示类型的盒子会这样:

- 盒子会产生换行(独占一行)
- width和height属性生效
- padding, margin, border会推开其它元素
- 如果没有指定width, 那么会尽可能填充当前宽度(一般情况下, 盒子会跟容器一样宽, 占据可用空间的100%)

一个inline外部显示类型的盒子会这样

- 盒子不会产生换行
- width和height属性不生效
- 垂直方向的padding, margin, border会被应用, 但是不会推开其它inline盒子
- 水平方向的padding, margin, border会被应用, 且会推开其它inline盒子

> 有一个原则: 一个块盒里面要么全是块盒, 要么全是行盒. 如果块盒里有一个块盒, 那么必须所有都是块盒. 
>
> 像我们这样写: `<div><span>123</span><div>456</div></div>`, 这个span会生成一个匿名块盒来包裹, 这个456则会生成一个匿名行盒进行包裹. 

### 内部显示类型

内部显示类型决定盒子内元素的布局方式. 

同样通过调整`display`来修改显示类型

- `display: flex;`
- `display: grid`
- `display: inline-flex`
- ...

如果没有指定内部显示类型, 那么内部元素会以标准流的方式布局. 

## CSS盒模型

CSS盒模型定义了盒子的组成部分: margin, border, padding, content. 

CSS盒模型整体适用于块盒, 行盒只有盒模型定义的部分行为. 

![来自MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model/box-model.png)

> 图来自MDN

对于标准盒模型中, width和height决定的是content盒的宽高. 

```css
.box {
  display: block;
  width: 350px;
  height: 150px;
  margin: 10px;
  padding: 25px;
  border: 5px solid black;
}
```

比如这个box, content盒的宽是350px, 高150px. 最终实际占用的宽高是

$$
实际宽度 = 350 + 25 * 2 + 5 * 2 = 410px
$$

$$
实际高度 = 150 + 25 * 2 + 5 * 2 = 210px
$$

> margin不计入盒子的实际大小. 它只影响盒子外的空间, 但盒子的面积止步于边框, 不会延伸到外边框. 

除了标准盒, 还有替代盒可能是开发中更加常用的. 

```css
.box {
    box-sizing: border-box;
}
```

那之前的实际宽高就变成了350px和150px

$$
content盒的宽 = 350 - 5 * 2 - 25 * 2 = 290px
$$

$$
content盒的高 = 150 - 5 * 2 - 25 * 2 = 90px
$$

对于border-box, width和height的声明就是实际大小. 

## 外边距折叠问题

一般情况下, 如果两个盒子对同一个空间都有margin, 那么就会取最大的. 

```css
.box1 {
    height: 100px;
    margin-bottom: 10px;
}
.box2 {
    height: 100px;
    margin-top: 20px;
}
```

```html
<div class="box1">1</div>
<div class="box2">2</div>
```

这样子的话, box1和box2实际只会相隔20px. margin-bottom就跟没有设置一样. 

有一种解决方法很简单, 自己把margin-top改成30px就可以了. 

除了两个正数, 还有可能两个负数, 一正一负. 

如果是一正一负, 那么把两个值相加. 比如30px和-10px, 得到20px. 那么两个相隔20px. 

如果是10px和-20px, 那么得到-10px, 第二个会覆盖第一个盒子10px. 

如果两个负数, 会发生折叠. 折叠部分的值是最小值的绝对值. 

比如-30px和-50px, 那么第二个覆盖第一个50px. 

## inline-block

`display: inline-block;`会产生一个特殊的盒子. 它会得到一个具有部分块盒性质的行盒. 

- width和height会生效
- padding, margin, border会推开其它元素(包括垂直方向)