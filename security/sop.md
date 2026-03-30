> 源: 协议+端口(如果指定了)+主机
> 
> file:// 一般不认为是同源的, 但是看浏览器具体实现

## 跨源操作

- 跨源写
- 跨源资源嵌入
- 跨源读

![[/images/Pasted image 20260317180521.png]]

![[images/Pasted image 20260317180718.png]]

## 允许跨源(CORS)

### 请求的分类

- 简单请求
	- 使用GET/HEAD/POST
	- 只允许手动设置Accept, Accept-Language, Content-Language, Content-Type, Range
	- Content-Type是text/plain, multipart/form-data, application/x-www-form-urlencoded之一
	- 如果使用xhr, 不监听upload的任何事件
	- 请求没有使用ReadableStream
- 复杂请求: 不是简单请求

对于简单请求, 发送请求前不预检; 复杂请求先发送预检请求. 

### 相关的HTTP头

- 客户端侧
	- Origin
	- Access-Control-Request-Method
	- Access-Control-Request-Headers
- 服务端侧
	- Access-Control-Allow-Origin
	- Vary
	- Access-Control-Expose-Headers
	- Access-Control-Max-Age
	- Access-Control-Allow-Credentials
	- Access-Control-Allow-Methods
	- Access-Control-Allow-Headers

### 预检请求

浏览器对于复杂请求先进行预检请求, 告知服务端源, 将要使用的方法, 将要使用的非标准头. 