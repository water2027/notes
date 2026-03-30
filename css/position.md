- static: 初始值
- relative: 相对正常的自己定位

```css
.box {
    position: relative;
    top: 10px;
}
```

这就会相对正常参与文档流的自己向下移动10px. 

依旧参与正常文档流. 

- absolute: 相对包含块或初始块定位

不参与正常文档流

- fixed: 相对视口定位

不参与正常文档流

- sticky: static和fixed的混合. 

在正常布局流滚动, 直到出现在设定的最近滚动祖先的位置, 就跟fixed的效果一样了. 

## 定位产生的层叠上下文

使用定位(非static)会产生层叠的能力, 可以覆盖或被覆盖. 

我们可以使用z-index来声明在z轴的顺序. 