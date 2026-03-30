利用iframe, 将敏感操作按钮跟攻击者自己写的诱导按钮放一起, 用户点击按钮执行敏感操作. 

## 防范

- iframe: 设置相关的响应头, 禁止部分或全部页面被嵌入iframe
> Content-Security-Policy/X-Frame-Options
- cookie: 设置cookie的SameSite为Lax或者Strict, 这样ifrmae打开的网页没有登录态