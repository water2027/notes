## 握手

quic的握手将tcp与tls1.3结合在一起了. 

1. 客户端发送`Initial`包, 包含连接ID(CID)以及附带自身Key Share的ClientHello. 
2. 服务端结合双方的Key Share, 计算出握手密钥, 发送ServerHello, 并将证书, 签名和Finished消息加密后发送. 
3. 客户端拿到Key Share, 计算出密钥, 解密数据并校验证书(Finished). 

### 再次连接

在握手即将结束时, 双方会使用握手密钥计算出一个预共享密钥PSK. 服务端会将PCK跟其他信息(如过期时间)打包成session ticket(类似jwt, 无状态)发给客户端. 

再次连接时, 客户端用PCK加密数据和Key Share, 并将密文和session ticket以及一起发过去. 服务端从session ticket中提取出PCK, 并解密数据. 然后结合双方Key Share计算出握手密钥, 将响应使用握手密钥进行加密, 并把自己的Key Share发过去. 客户端拿到后计算出新的握手密钥, 并使用握手密钥解密. 