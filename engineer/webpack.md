webpack诞生在没有ESM或者ESM没有被广泛推广的时代. web前端开发者只能通过script标签将模块挂载到window全局. 

> 缺点: 
> - 缺少代码提示
> - 过多的script链接导致浏览器需要加载太多的模块

如果可以像写node一样用require就好了. webpack就是为了解决这个问题. 

webpack runtime自己实现了一个类require函数. 

## 构建

构建时, 对于一个入口(entry):

1. 遇到require/import, 加到依赖图, 然后进入被导入的js文件, 重复流程, 得到一个依赖图
> 这里还涉及加载器, 如css/scss代码. 由loader解析, 最后也是得到一份js代码
> 
> 对于bundle, 一切都是js. 
> 
> 构建这个依赖图主要是为了TreeShaking以及代码去重, 最后打包成一份大文件. 

完成依赖图后, webpack构建出一份bundle.js出来.

## runtime

### 静态导入

这个bundle.js中, webpack需要额外的运行时, 并提供了类require函数: `__webpack_require__`

```javascript
// The module cache
var __webpack_module_cache__ = {};

// The require function
function __webpack_require__(moduleId) {
	// Check if module is in cache
	var cachedModule = __webpack_module_cache__[moduleId];
	if (cachedModule !== undefined) {
		return cachedModule.exports;
	}
	// Check if module exists (development only)
	if (__webpack_modules__[moduleId] === undefined) {
		var e = new Error("Cannot find module '" + moduleId + "'");
		e.code = 'MODULE_NOT_FOUND';
		throw e;
	}
	// Create a new module (and put it into the cache)
	var module = __webpack_module_cache__[moduleId] = {
		// no module.id needed
		// no module.loaded needed
		exports: {}
	};

	__webpack_modules__[moduleId](module, module.exports, __webpack_require__);

	// Return the exports of the module
	return module.exports;
}
```



### 动态导入

如果是动态导入, webpack也有解决方案. 

在打包时, 会把相关代码拆分出去成为chunk, webpack runtime内部还会维护installedChunks来标记这些chunk的状态. 

如果没有加载, 那么runtime会创建一个script标签进行下载并执行对应的js代码. 

> 也就是JSONP

```javascript
(self["webpackChunkwebpack"] = self["webpackChunkwebpack"] || []).push(...)
```

这里的push被runtime劫持了. 

```javascript
// install a JSONP callback for chunk loading
var webpackJsonpCallback = (parentChunkLoadingFunction, data) => {
	var [chunkIds, moreModules, runtime] = data;
	// add "moreModules" to the modules object,
	// then flag all "chunkIds" as loaded and fire callback
	var moduleId, chunkId, i = 0;
	if(chunkIds.some((id) => (installedChunks[id] !== 0))) {
		for(moduleId in moreModules) {
			if(__webpack_require__.o(moreModules, moduleId)) {
				__webpack_require__.m[moduleId] = moreModules[moduleId];
			}
		}
		if(runtime) var result = runtime(__webpack_require__);
	}
	if(parentChunkLoadingFunction) parentChunkLoadingFunction(data);
	for(;i < chunkIds.length; i++) {
		chunkId = chunkIds[i];
		if(__webpack_require__.o(installedChunks, chunkId) && installedChunks[chunkId]) {
			installedChunks[chunkId][0]();
		}
		installedChunks[chunkId] = 0;
	}

}

var chunkLoadingGlobal = self["webpackChunkwebpack"] = self["webpackChunkwebpack"] || [];
chunkLoadingGlobal.forEach(webpackJsonpCallback.bind(null, 0));
chunkLoadingGlobal.push = webpackJsonpCallback.bind(null, chunkLoadingGlobal.push.bind(chunkLoadingGlobal));
```

然后将导出结果放进缓存表里, 并修改相关chunk的状态. 

最后就是执行Promise的回调了. 

## 热重载

开启HMR, 浏览器会与开发服务器通过ws通信. 

当某处代码修改时, webpack会标记受影响的节点为失效, 然后重新构建对应的代码, 最终通过ws将被更新模块的chunkId通知给浏览器. 

浏览器根据chunkId, 更新manifest, 下载并执行js代码, 更新缓存表对应的缓存. 

在更新缓存后, 还需要通知依赖被更新模块的模块, 让他们使用新的模块. 

> 从被更新的模块开始, 向上冒泡查找依赖它的依赖

runtime提供了HMR的接口(module.hot), 让模块自己处理热更新. 

如果模块没有处理热更新(使用module.hot.accept), 那么事件会冒泡. 如果冒泡到入口都没有解决, 那么整页刷新. 

## 优化

为了避免构造依赖图, webpack使用内存缓存和持久化缓存的解决方案. 

- 开发环境下, 在内存中进行依赖图的构建
- 将构建结果存到硬盘中, 下次启动直接复用, 跳过大部分解析过程