js本身是一个单线程语言, 通过事件循环来实现异步编程. 

js runtime会维护一组用来执行js代码的代理(Agent). 每个代理都由事件循环驱动. 代理由一组执行上下文的集合, 执行上下文栈, 主线程, 一个或多个任务队列, 一个微队列和一组可能创建用于执行worker的额外线程集合构成. 

> 除了主线程可能多个代理共用, 其他的组成部分对于代理是唯一的. 
> 
> 代理通常指window, worker, worklet.
> 
> 任务队列实际上是一个set而不是queue, 事件循环处理模型会取出第一个可运行的任务, 而不是取出第一个任务. 

## 浏览器下的事件循环

浏览器的事件循环会维护一些变量:
- 当前正在运行的任务(可以是null, 初始值为null)
- 微任务队列
- 微任务检查点(boolean, 初始值为false)
- last render opportunity time(最近一次渲染的时间, 初始值为0)
- last idle period start time(最近一个空闲期的开始时间, 初始值为0)

一次事件循环是这样的:
1. 从任务队列中取出一个任务并执行
2. 清空微任务队列
3. 执行一个渲染检查点
	1. 检查相关变量(如页面是否可见, 离上次渲染时间是否接近等)
		- 如果需要渲染, 那么渲染
		- 如果不需要, 那么跳过
```javascript
function updateRendering() {
	if (!needRender() || !isVSyncTime()) {
		return;
	}
	
	runRAF();
	reCalculateStyles();
	performLayout();
	paint();
	// ...
}
```

4. 执行一个空闲检查点
	1. 如果离下一帧还有时间, 或者判定处于空闲时间, 打卡记录last idle period start time
	2. 启动requestIdleCallback
> 剩余时间约等于`last idle period start time` + 16.67ms - 当前时间

```javascript
requestIdleCallback((ddl) => {
	while (ddl.timeRemaining()) {
		// do sth
	}
})
```

## 浏览器的任务队列

浏览器内部维护了一些任务源以及对应的任务队列
- DOM任务源
- 用户交互任务源
- 网络任务源
- 导航与横移任务源
- 渲染任务源(vSync信号)
- 延时任务源
不同的任务源放进不同的任务队列中, 浏览器按自己的策略, 从里面取出任务. 
> 一般情况下, 用户交互任务源优先级最高. 

这些任务源的主要作用是为了唤醒主线程. 主线程运行完毕后, 如果没有任务要执行那么就会挂起. 之后有任务再唤醒. 

## 浏览器的主线程阻塞

由于js是单线程的, 如果有重型任务在主线程一直执行就会导致阻塞. 可以使用web worker来缓解. 

### 一些卡死的情况

```javascript
// 卡死
function w() {
	while (true) {}
}

// 卡死, 但是不会导致栈溢出
function p() {
	queueMicrotask(() => {
		p()
	})
}

// 卡死, 但不会阻止交互和渲染, 也不会栈溢出
function s() {
	setTimeout(s, 0)
}
```

> btw, 这些只能卡死主线程, 卡不死合成线程. 