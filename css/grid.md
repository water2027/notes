grid布局通常用来设计网格布局. 

## 定义一个网格

除了设置`display: grid;`, 通常还搭配其它属性. 

- grid-template-columns/grid-template-rows: 决定一列/一行排多少个, 每个有多宽/高

```css
.container {
    display: grid;
    grid-templates-columns: 200px 300px 400px;
}
```

这就表示一行排三列, 第一列200px, 第二列300px, 第三列400px. 

有一个新单位fr, 按比例分配可用空间. 

`grid-templates-columns: 1fr 2fr 3fr`, 表示把一行的可用空间平均分成六份, 第一列一份, 第二列两份, 第三列三份. 

- gap/grid-gap: 项之间的间隔
> gap和grid-gap效果一样, MDN推荐两个都写上去. 
- grid-auto-rows/grid-auto-columns

一般情况下我们只会使用grid-templates-columns和grid-templates-rows中的一个, 这个grid-auto就是帮助我们设置另一个的. 

`grid-auto-rows: 100px;`将每行的高度设置成100px, 如果不够浏览器就会拉伸. 可以使用auto, 设置成最合适的高度. 

***

以及两个函数

- repeat
我们有时候可能需要很多列/行, 手写太麻烦了就可以用这个函数. 

`grid-templates-columns: 1fr 1fr 1fr;`等价于`grid-templates-columns: repeat(3, 1fr)`

还可以使用auto-fill, 让浏览器自己决定多少列/行. 不过一般会搭配minmax来使用, 限制宽/高度. 

- minmax

设置值的最大和最小, 允许值在区间内变动. 

`grid-auto-rows: minmax(100px, auto)`确保每行至少100px, 高度自动可以防止溢出, 如果100px不够就会伸长到刚好完全展示内容. 

`grid-templates-columns: repeat(auto-fill, minmax(200px, 1fr));`表示每列至少200px, 否则就不排在这一行. 

使用这些东西就定义好了一个网格. 接下来就是放置元素到网格里了. 

## 放置元素

默认情况下, 子项会一个一个放进去. 

此外, 有基于线的元素放置和基于grid-template-areas的元素放置. 

我们的网格可以大致画成一个表的样子, 如果把分隔线和边框画出来的话. 

`grid-column: 2`表示第二列. 

`grid-column: 1 / 3`表示第一条行分割线开始, 到第三条行分隔线结束. 


基于grid-template-areas比较直观

```css
.container {
  display: grid;
  grid-template-columns: 1fr 3fr;
  grid-template-areas:
    "header header"
    "sidebar content"
    "footer footer";
  gap: 20px;
}

header {
  grid-area: header;
}

article {
  grid-area: content;
}

aside {
  grid-area: sidebar;
}

footer {
  grid-area: footer;
}
```