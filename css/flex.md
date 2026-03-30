flex通常用来设计横向或者纵向的布局. 

flex模型沿着两个轴来布局: 一个主轴(main axis), 一个交叉轴(cross axis). 这两个相互垂直. 

设置了`display: flex;`的父元素一般被称为flex容器, 容器中表现为弹性盒的叫做flex项. 

## 轴的方向

这个属性决定主轴的方向. 默认是row, 这使得主轴是水平的从左到右(至少我们一般接触到的都是从左到右, 部分国家地区可能是从右到左). 

如果改成column, 那么主轴是竖直的从上到下. 

flex项就沿着主轴依次排列. 

## 处理溢出

当flex项增多时, 且容器定宽/高就有可能发生溢出. 

可以使用flex-wrap, 可以设置no-wrap, wrap, wrap-reverse. 初始值是no-wrap, 设置成wrap就可以换行. 

启用换行后可能出现子项占据容器的宽/高, 可能是没有设置flex导致的. 

`flex: 200px`可以设置子项占据的最小空间, 浏览器发现达不到200px就会进行换行. 

## 动态尺寸

可用空间指的是主轴空间去除padding和子项的margin, 剩下的空间. 

- flex-grow: 声明占用可用空间的比例(初始值是0)

```css
.box1 {
    flex-grow: 1;
}
.box2 {
    flex-grow: 2;
}
```

假设只有这两个子项, 那么box1就会占据3份中的1份. box2占据3份中的2份. 

如果有两个box1和一个box2, 那么box1占据4份中的1份, box2占据4份中的2份. 

- flex-shrink: 如果容器空间不足, 缩小几倍以防止溢出(初始值是1)
- flex-basis: 最小值
```css
.box1 {
    flex-grow: 1;
    flex-basis: 200px;
}
.box2 {
    flex-grow: 2;
    flex-basis: 200px;
}
```

设置最小宽度, 达到最小宽度后再进行可用空间的分配. 

这里子项先分配200px的可用空间, 剩下的按比例分配. 

## 沿轴对齐

- align-items: 控制flex项在交叉轴的位置
    - 初始值是stretch, 所有flex项沿着交叉轴的方向填充容器. (如果没有设置宽/高度)
    - center: 保留原有高度, 沿交叉轴居中
    - flex-start/flex-end: 在交叉轴开始或结束处
> 可以用align-self来覆盖align-items的行为
- justify-content: 控制flex项在主轴的位置
    - flex-start/flex-end
    - center
    - space-around: 使所有flex沿主轴均匀分布
    - space-between: 跟space-around类似, 但是两端不会留下任何空间

## 排序

使用order属性, 初始值是0. 

order值大的排在后面. 