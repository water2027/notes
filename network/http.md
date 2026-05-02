## http0.9

实现简单

eg:
```
GET /index.html
```

响应简单, 直接返回对应文件

## http1.0

feat: 版本号+header
bad: 非持续连接

eg:

```
GET /www.gif HTTP/1.0
User-Agent: xxx
```

```
200 OK
Date: xxx
...
```

采用非持续连接, 每一个HTTP请求都需要建立TCP连接. 

## http1.1

feat: host标头(虚拟主机)+持续连接
bad: 应用层队头阻塞(流水线/管道化)

### host标头

host标头让一个ip/一台服务器部署多个网站. 需要在请求头里标识访问哪个网站. 

流程: 请求到达负载均衡器(nginx或其他), nginx检查host标头, 根据host转发到其它服务器或者本地其他服务. 

### 持续连接

新增了Connection标头, 默认是keep-alive, 避免多次TCP握手. 

### 应用层队头阻塞

但是是串行处理请求: 后面的请求必须等待前面的请求处理完成. 

解决方案: 建立多个并行连接(一个域名一般至多6条连接)或者管道化(后面的请求不等待前面的请求完成就发出去)

由于HTTP本身无状态, 响应也没有标识自身是哪个请求响应的(没有规范要求实现响应标识请求, 开发者可以自己实现, 但一般情况下大家都不实现, 太麻烦了), 因此一个请求的响应必须在前面的请求响应之后, 也就是说请求有序发出, 响应也必须有序响应. 

eg:

```
fetch 1
fetch 2

// yes
return 1
return 2

->

fetch 1 return 1
fetch 2 return 2
```

```
fetch 1
fetch 2

return 2
return 1

->

fetch 1 return 2
fetch 2 return 1
```

实现太复杂, 出错概率大. 

> 管道化默认关闭

应用层队头阻塞根本原因: 响应根本不知道是哪个请求的响应, 只能根据顺序来凑对. 

## http2

feat: 二进制协议+头部压缩+StreamId(多路复用)
bad: TCP队头阻塞

### 头部压缩(HPACK算法)

双方各维护一张表. 理论上这张表双方都会进行相同的操作, 因此最终两张表都是一样的. 

表可以划分成静态表和动态表:

- 静态表是HTTP2规定的, 包含61个常见的HTTP头部. 如`Index 2`对应`:method: GET`, `Index 1` 对应`:authority`, `Index 4`对应`:path`
- 动态表一开始是空的, 双方交流过程中填充. 

eg:

第一次发送

```
GET /a
Host: www.watering.top
```
就会变成
```
Index 2
Index 4 /a
Index 1 www.watering.top
```
第二次发送
```
Index 2
Index 63
Index 62
```

第一次完成, 双方都会修改动态表. 这里`Index 62`对应`:authority: www.watering.top`, `Index 63`对应`:path: /a`

这里的原文还会使用哈夫曼编码进一步压缩. 

### StreamId

HTTP2将报文变成了帧. 分为首部帧和数据帧. 帧具有StreamId作为唯一标识符. 

有了唯一标识符后, 浏览器就有办法判断响应到底是哪个请求的了, 因此后面的请求不必等前面的请求完成再发送, 可以**并行**处理请求. 

### TCP队头阻塞

应用层虽然可以并行发出, 但是还是一份份交给传输层(TCP)的. 

当有一个请求对应的包需要重试(丢包或超时), 那么后面请求对应的包也不能交给应用层(TCP确保有序性). 

### HTTP3

feat: UDP+QUIC+无感切换网络+0RTT

### QUIC

HTTP3采用基于UDP的QUIC. QUIC实现了TCP的多种功能, 如可靠传输, 流量控制等, 还具有TLS的功能. 

### 无感切换网络

QUIC具有ConnectionId, 用来标识连接. 当用户切换网络(如切换到另一个wifi, 或者切流量), 通过这个ConnectionId可以实现无感切换. 

### 0RTT

当第二次与服务器建立连接时, 客户端可以把加密所需数据跟HTTP请求直接发过去. 但是浏览器通常只允许幂等的请求, 比如GET资源. 

> HTTP3的头部压缩采用了QPACK

## HTTP Cache

### 启发式缓存(默认情况下)

旧时代的产物(Cache-Control之前). 浏览器默认缓存上一次更新时间到现在的时间的1/10(在有Last-Modified的情况下)

eg:

上一次更新是十年前, 那么缓存一年. 

### 缓存头
> expires: 过期时间. 少见了

主要关于Cache-Control:
- max-age: 缓存有效期
- Last-Modified/If-Modified-Since
上次修改时间. 

第一次访问时, 服务端带上Last-Modified头, 浏览器存起来. 

进行协商缓存时, 浏览器会带上If-Modified-Since头, 服务端检查是否有效. 
- ETag/If-None-Match
资源标识符

第一次访问时, 服务端带上ETag, 浏览器存起来. 

进行协商缓存时, 服务端会带上If-None-Match头, 服务端检查是否有效. 

> ETag优先级更高. 

其它缓存策略:
- no-cache: 要求使用缓存前, 必须校验(强制协商缓存)
- no-store: 禁止使用缓存
- immutable: 用户刷新时, 不协商缓存, 直接用
- must-revalidate: 过期了必须重新验证, 不能直接使用

### 协商缓存与强缓存

当缓存无效(max-age过了)或者用户刷新了, 就会进行协商缓存. 如果服务端返回304, 那么使用缓存. 缓存无效, 返回200和资源(跟没有缓存一样). 

当缓存有效(max-age没过)或者用户刷新但是`immutable`, 浏览器直接使用缓存. 

## cookie

SameSite:
- Strict: 完全阻止第三方源请求携带cookie
- Lax: 允许顶级导航(如点击链接)携带cookie, 但是阻止其它跨站请求携带(默认为Lax)
- None: 允许跨站请求携带cookie(需要开启HTTPS)