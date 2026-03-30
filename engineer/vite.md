vite诞生在ESM被推广的时代. 

在不启用rolldown的情况下, 开发使用esbuild, 构建使用rollup. 启用的话, 开发和构建都使用rolldown. 

## 开发

webpack做什么都会先打包, 但是vite不一样. vite只会在浏览器需要某个文件时才会发过去. 

比如main.js

```javascript
import App from './App.vue'
```

如果是webpack, 最后会打包成一个bundle.js发过去. 但是vite只会把原原本本的main.js发过去. (如果有插件, 可能有额外的内容)

之后浏览器解析这一条语句, 向vite发起请求时, vite才会将App.vue翻译成js代码发过去(通过vue插件). 

不过这样同样会面临最开始的问题: 如果引用的js很多, 就需要开启多条连接, 特别是第三方库. 

因此, vite会把代码分为两类: 依赖(较稳定)和源码(较不稳定), 

> 依赖通常指node_modules内的东西. 

对于依赖, vite会进行预构建, 行为类似webpack的打包, 最终转换成一个文件. 

这样数百条请求就变成了一条请求. 

> 在预构建阶段, vite会完成commonjs/umd到ESM的转换. 

预构建通常只需要一次, 直到锁文件改变, 或者vite.config.js相关字段, NODE_ENV, 不定文件夹修改时间的改变, 才有可能再次构建. 

> 预构建产物会放在node_modules/.vite中, 也可以自己决定. 

同时, 预构建产物vite会让浏览器进行强缓存, 一次请求就不再请求, 直到再次预构建. 

### 热重载

vite会与浏览器建立ws连接. 当vite有更新时, 通知浏览器. 

浏览器拿到通知后, 下载新的js, 执行代码并找出对应的节点: 如果节点自己解决了事件, 无事发生; 如果没有, 进行冒泡, 重复这一过程. 

冒泡到父节点时, 浏览器会进行新资源的下载. 

```javascript
import { a } from './a.ts'
console.log(a)
if (import.meta.hot) {
	import.meta.hot.accept('./a.ts', (mod) => {
		console.log('mod', mod?.a)
	})
}
```

这里mod解析出来的就是新资源的导出. 原本的实际上还是保留旧引用. 

如果一直没有节点解决, 那只能刷新页面了. 

## 构建

vite同样需要一个依赖图, 然后将静态导入的打包到一个js文件里(如果没有手动配置分包), 动态导入的分成一个个的js文件. 基于vite打包的动态导入不需要使用JSONP, 只需要在浏览器需要的时候提供对应的js文件即可. 

vite打包出来的js文件几乎没有运行时, 每个模块都会尽可能地打平. webpack则需要放在一个函数里, 变成一个个的module.exports

vite会自动进行modulepreload的优化, 在某些情况下通过js动态地放一些到head中. 

对于访问网站就一定要下载的东西, 会提前放在head里, 比如vue库, css资源等. 

## 开发与构建一致性

由于开发使用esbuild, 构建使用rollup, 但是大部分插件其实都是rollup的插件. 

为了让esbuild也能使用vite插件, 尽可能实现开发与构建一致, vite提供了PluginContainer. 

在这个容器中, 会模拟rollup的行为, 提供rollup的钩子. 

代码先经过PluginContainer, 然后再交给esbuild处理. 


