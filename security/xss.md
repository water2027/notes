跨站脚本攻击

一般出现在论坛类应用. 攻击者让用户执行恶意代码. 

完全信任用户的输入, 并嵌入html中. 

## 风险来源

- 恶意用户可能输入恶意代码, 执行敏感操作
- 嵌入html的数据不可信

## 常见类型

- 存储型
恶意代码被存储在服务端上, 持续有效
- 反射型
通过URL参数攻击服务端漏洞. 
```javascript
function search(input) {
	return `
		<h1>您搜索的是${input}</h1>
	`
}
```
- DOM型
通过URL参数攻击客户端漏洞. 
```vue
<template>
	<h1 v-html="input"></h1>
</template>
```

## 防范

- 输入时校验, 输出时转义/过滤