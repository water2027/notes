一般情况下, 指向调用者. 

## 默认绑定

对于直接调用的, 如`say()`, 可以看作是这样:

```javascript
say()

say.call(window) // 严格模式下是undefined
```

## 隐式绑定

```javascript
const user = {
	a: '1',
	say() {
		console.log(this.a)
	}
}

user.say() // 1

const a = user.say
a() // undefined, 相当于a.call(window)或者a.call(undefined)
```

## 显式绑定

call/apply/bind

## new绑定

```javascript
function Person() {
	// this指向新实例
	this.name = ''
}
const p1 = new Person()
```

## 箭头函数

箭头函数没有this. 

```javascript
this.w = 2
function a() {
	this
	this.w = 1
	const say = () => {
		console.log(this.w) // 打印1. 相当于闭包函数
	}
}
```

