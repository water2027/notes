第三方服务需要在oauth中心登记, 获取client_id等, 并记录redirect_uri以及自己的应用类型. 

> 如果是public client(如App, SPA应用), 那么不需要client_secret
> 
> 如果是confidential client(带有后端的应用), 那么需要client_secret

第三方服务在自己的登录页面放置通过xx登录. 

> 在这个过程中, 第三方服务应该准备这些东西: state, code_verifier, code_challenge, code_challenge_method
> 
> state是第三方服务自己用的, oauth服务本身不会校验state, 只会原原本本地发回去. 
>
> code_verifier是一个随机字符串, code_challenge_method是加密方法, 通常是SHA256
> 
> code_challenge = Base64URL(SHA256(code_verifier))

用户点击后, 跳转到授权页面 GET /authorize?redirect_uri=xxx&scope=xxx&client_id=xxx&code_challenge=xxx&code_challenge_method=xxx&state=xxx&response_type=code. 
> 如果没有登录, 先进行登录
> 
> response_type=token被废弃了, 不安全. 一般是code. 如果实现了OIDC, 可以是id_token
> 
> 服务端生成一个随机字符串作为auth ticket, 并建立起映射关系. 在授权表单中嵌入这个ticket. 一方面作为csrf_token, 另一方面避免表单侧嵌入过多数据. 

用户点击授权. 

> 服务端根据ticket找出对应的数据, 校验数据, 最终生成一个code, 并建立一
> 个code跟user_id, code_challenge, code_challenge_method, redirect_uri的映射, 最后重定向至redirect_uri?code=xxx&state=xxx

第三方服务校验state, 将code, code_verifier换取token

> 服务端使用code_verifier计算出code_challenge, 并进行比对. 比对通过, 返回token. 
> 
> 如果是confidential client, 还需要校验client_secret

- code_challenge的作用
如果有恶意应用拦截了服务器返回的code, 那么它可以直接使用code换取token. 但是加上挑战码, 由于恶意应用无法知道code_verifier, 因此无法用code换取token. 