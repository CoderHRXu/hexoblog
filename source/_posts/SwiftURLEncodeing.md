---
title: Swift中URL处理中的注意点
---
日常的开发当中，网络请求是不可或缺的。而在网络访问请求中，经常会遇到有中文空格字符的情况，直接用这些字符串去访问是无法正常访问，需要我们做进一步的处理。

## 一般处理
```
let urlString = "http://10.0.3.86/中文/main.html#/help"
```
比如以上的url，想使用webview进行访问或者是原生发起http请求，都需要进行转码处理。
有人会问，这有什么难的？拿起键盘就是干
![就是干.png](http://upload-images.jianshu.io/upload_images/1447375-7e42bf91528876c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
OC：
NSString* encodedString = [urlString stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

Swift：
let encodedString = urlString.addingPercentEscapes(using: .utf8)

```
一敲代码，emmmmm~~ Xcode发警告了，该方法已经过期，用下面的方法替代，于是紧接着:

```
OC:
NSString* encodedString = [urtString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];

Swift:
let encodedString = urlString.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)
```
OK,转码处理一下，再访问转码之后的url。
**what？还是不能正常访问？**
我们来看看转码之后的url是什么：
![image.png](http://upload-images.jianshu.io/upload_images/1447375-fee78cebefc1331c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**原来是因为最后的#被转码成了%23**
前端开发的小伙伴说这个#不能动，只能我们不转码。不转码那么夹杂中文字符怎么办呢？—— **修改参数**。
**所以我们的需求变化成：除了url里面的#不动，其他该转码的都转码**

``` 
NSMutableCharacterSet *set  = [[NSCharacterSet URLQueryAllowedCharacterSet] mutableCopy];
[set addCharactersInString:@"#"];
NSString *encodedString     = [urlSring stringByAddingPercentEncodingWithAllowedCharacters:set];
```
如上手动修改转码参数，OK 可以了。
Swift如法炮制
```
let charSet = CharacterSet.urlQueryAllowed as! NSMutableCharacterSet
charSet.addCharacters(in: "#")
let encodingString = urlStr.addingPercentEncoding(withAllowedCharacters: charSet as CharacterSet)
```
虽然语言不一样，但是思路一样。**emmmm...你会惊人的发现，根本不管用！！！**
![wc.png](http://upload-images.jianshu.io/upload_images/1447375-d729f1de206f6ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 原因分析
问题是出在```let charSet = CharacterSet.urlQueryAllowed as! NSMutableCharacterSet ```这一行，在swift语言中，Foundation框架中的很多class都重新用struct重写了，比如NSString和String，NSUrl和URL，如果要使用类似于OC一些特性，有时候需要as来强转成对应的NS开头的类。强转的过程中，CharacterSet应该转成NSCharacterSet,而不应该是NSMutableCharacterSet，也就是说**子类指针指向了父类对象**，父类里面没有子类的方法，所以执行```charSet.addCharacters(in: "#")```的时候，无法正确添加。

## 正确的写法
顺着原因一路分析，应该这么写：
```
方法一：
let charSet = CharacterSet.urlQueryAllowed as NSCharacterSet
let mutSet = charSet.mutableCopy() as! NSMutableCharacterSet
mutSet.addCharacters(in: "#")
let encodingURL = urlStr.addingPercentEncoding(withAllowedCharacters: mutSet as CharacterSet)
```
当然还有其他写法：
```
方法二：
let charSet = NSMutableCharacterSet()
charSet.formUnion(with: CharacterSet.urlQueryAllowed)
charSet.addCharacters(in: "#")
let encodingURL = urlStr.addingPercentEncoding(withAllowedCharacters: charSet as CharacterSet)
```
方法一和二本质是一样的，其实沿用的OC的思想，先构造一个可变对象，再加入自定义的字符。**如果要像OC这么搞，那么苹果设计swift的意义何在？换句话说，swift用结构体写重写这个类一定考虑到这个问题，那就应该有相应的处理方法。**

查阅官方文档吧，少年！
![image.png](http://upload-images.jianshu.io/upload_images/1447375-3c58407983a74f5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
果不其然，找到一个方法，用来插入字符。
所以还有第三种写法：
```
方法三：
var charSet = CharacterSet.urlQueryAllowed
charSet.insert(charactersIn: "#")
let encodingURL = urlStr.addingPercentEncoding(withAllowedCharacters: charSet )
```
我们来看一下最终结果
![结果.png](http://upload-images.jianshu.io/upload_images/1447375-8e4a9b88fe4a3f03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
OK，符合需求！
