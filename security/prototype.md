可能常见于url参数解析. 

eg

```javascript
function getUsers(request) {
  const result = {};
  const userNames = new URL(request.url).searchParams.getAll("names");
  const fields = new URL(request.url).searchParams.getAll("fields");
  for (const name of userNames) {
    const userInfo = database.lookup(name);
    result[name] ??= {};
    for (const field of fields) {
      // Pollution source
      result[name][field] = userInfo[field];
    }
  }
  return result;
}
```

> 来自mdn

如果传入了`names=__proto__&fields=age`, 那么result会变成

```javascript
result.__proto__.age = undefined // 或者用户__proto__的年龄
// 相当于这样
Object.prototype.age = xxx
```

只有这一处可能没什么问题. 一般会结合其它代码攻击. 

***

```javascript
// Just an object with a property called `__proto__`
const options = JSON.parse('{"__proto__": {"test": "value"}}');
const withDefaults = Object.assign({ mode: "cors" }, options);
// In the process of merging `options`, we indirectly executed
// withDefaults.__proto__ = { test: "value" }, causing `withDefaults` to have
// a different prototype
console.log(withDefaults.test); // "value"
```

恶意第三方库

```javascript
function test() {
	const obj = {
		a: 1,
		b: 2,
		c: 3,
	}
	return {
		get(key) {
			return obj[key]
		}
	}
}

Object.defineProperty(Object.prototype, 'jack', {
	get() {
		return this
	}
})

const res = test()
const obj = res.get("jack")
console.log(obj)
obj.c = 100
console.log(obj.c === res.get('c')) // true
```