---
title: Moya，KingFisher中使用自签名证书发起HTTPS请求
---

### HTTPS握手

先说声https握手，发送 HTTPS 请求首先要进行 SSL/TLS 握手，握手过程大致如下：

1.客户端发起握手请求，携带随机数、支持算法列表等参数。
2.服务端收到请求，选择合适的算法，下发公钥证书和随机数。
3.客户端对服务端证书进行校验，并发送随机数信息，该信息使用公钥加密。
4.服务端通过私钥获取随机数信息。
5.双方根据以上交互的信息生成session ticket，用作该连接后续数据传输的加密密钥。
上述过程中，和“IP直连”有关的是第3步，客户端需要验证服务端下发的证书，验证过程有以下两个要点：

1.客户端用本地保存的根证书解开证书链，确认服务端下发的证书是由可信任的机构颁发的。
2.客户端需要检查证书的 domain 域和扩展域，看是否包含本次请求的 host。
如果上述两点都校验通过，就证明当前的服务端是可信任的，否则就是不可信任，应当中断当前连接。

当客户端使用“IP直连”解析域名时，请求URL中的host会被替换成解析出来的IP，所以在证书验证的第2步，会出现domain不匹配的情况，导致SSL/TLS握手不成功。


### 问题的产生：

      平时的swift项目开发过程中，我们需要在测试服进行开发以及测试，当网络环境配置成https的时候，经常需要加载网络请求，或者展示网络图片，测试服都是自建证书，默认情况下Moya，和KingFisher不会通过验证，导致网络(图片)加载不同。

### 解决方法：
关闭Https的证书认证
验证自签名证书，CN 是否合法
### Moya
```
方法一：
/// 关闭https认证
let serverTrustPolicies: [String: ServerTrustPolicy] = [
“172.16.88.106”: .disableEvaluation
]
let manager = Manager(
configuration: URLSessionConfiguration.default,
serverTrustPolicyManager: ServerTrustPolicyManager(policies: serverTrustPolicies)
)
let provider = MoyaProvider(manager: manager, plugins: [NetworkLoggerPlugin(verbose: true)])
方法一是一种变通实现的方法，它直接关闭了Https的Domain验证，虽然可以请求正常进行，但是如果在客户端和服务器之间增加代理，请求发送时代理替换证书，那么代理就可以轻易拿到请求的数据，出于安全考虑并不推荐这种做法。
方法二：
ServerTrustPolicy枚举中使用pinCertificates。
case pinCertificates(certificates: [SecCertificate], validateCertificateChain: Bool, validateHost: Bool)
传入自签名证书信息。
示例代码：
let path = Bundle.main .path(forResource: "selfSigned_pubCA.cer", ofType: nil)
let data = NSData(contentsOfFile: path!)
let certificates :[SecCertificate] = [data as! SecCertificate]
let policies : [String : ServerTrustPolicy] = ["172.16.88.230" : .pinCertificates(certificates: certificates, validateCertificateChain: true, validateHost: true)]
把这个 policies 作为参数传进manager 构造函数中即可

```
### KingFisher信任Host方法

A lightweight, pure-Swift library for downloading and caching images from the web.
[https://github.com/onevcat/Kingfisher](https://github.com/onevcat/Kingfisher)
```
//取出downloader单例
let downloader = KingfisherManager.shared.downloader
//信任ip为106的
Serverdownloader.trustedHosts = Set(["172.16.88.106"])
//使用KingFisher给ImageView赋网络图片
iconView.kf.setImage(with: iconUrl)
```