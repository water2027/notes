
跨站请求仿造

假设正常网站存在一个表单, 攻击者可以伪造一个表单, 同时action和method都保持一致, 诱导用户填写. 或者干脆攻击者在页面加载时就模拟表单提交. 

如果cookie的SameSite设置不正确且用户登录过目标网站, 那么请求就会带上cookie. 

如果是简单请求, 那么请求会成功修改服务端状态. 

如果不是表单, 只要符合简单请求, 攻击者就可以轻易地模拟. 比如GET请求

```
<img src="xxx">
```

## 风险来源

- cookie
- 可跨源写
- 请求参数可预测

## 防范

- 设置安全的cookie, 如Strict. 
> Lax可能不可靠. 如果敏感操作是GET, 可以直接让用户点击敏感链接来执行操作, 同样具有cookie. 
- 使用复杂请求(如使用自定义头, 或者使用json来传递. 但是服务端也必须进行校验).
- 使用csrf_token, 最为有效. 
- 服务端检查sec-fetch-site头
	- same-origin, same-site, cross-site, or initiated directly by the user

一般情况下, csrf_token由服务端生成, 嵌入form表单中(一般会隐藏), 跟表单一起提交. 
