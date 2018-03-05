---
title: Swift中使用Contains的正确姿势
---



## 1.正确姿势
当我们在用Swift进行编码的时候，经常会用到contains(element:)方法来判断Array是否包含了某个元素。举个栗子：
```
enum Animal {
  case dog
  case cat
}
let animals: [Animal] = [.dog, .dog]
let hasCat = animals.contains(.cat)       // fasle
```
看起来灰常的简单对不？然鹅，我们把Animal的枚举定义修改成带参数的枚举，比如这样：
```
enum Animal {
  case dog(String)
  case cat(String)
}
let animals: [Animal] = [.dog("汪汪"), .dog("阿黄")]
let hasCat = animals.contains(.cat("喵喵"))      // compile error  
```
编译报错了，为毛？
原因是能使用contains(element:)的元素是需要遵循Equatable协议的，但是Animal是没有遵循的，所以会报错。

那肿么办？幸好苹果还提供了另外一个方法来判断
```
public func contains(where predicate: (Element) throws -> Bool)
rethrows -> Bool
```
这个方法提供了一个闭包参数—— predicate，当数组中的元素没有遵循Equatable协议时，我们可以用这个参数来定义检查元素的规则。

所以之前代码中的错误代码改成这样：
```
enum Animal {
    case dog(String)
    case cat(String)
}

let animals : [Animal] = [.dog("汪汪"), .dog("阿黄")]
let hasCat = animals.contains { (animal) -> Bool in
    
    if case .cat = animal {
        return true
    }
    
    return false
}

// fasle
```
这么改hasCat就能正确计算出是false了。

## 2.进阶用法
另外，我们可以自定义predicate，来定义检查筛选的条件，比如：
检查数组中是否有元素大于100。
```
let expenses = [21.37, 55.21, 9.32, 10.18, 388.77, 11.41]
let hasBigPurchase = expenses.contains { $0 > 100 }
// 'hasBigPurchase' == true
```
再给个栗子：检查一个数组的元素，它可是否以除以7。
```
let array = [2, 5, 6, 7, 19, 40]

array.contains { (element) -> Bool in
    element % 7 == 0
}
或者
array.contains { $0 % 7 == 0 } 
```
这里的$0就是指数组的元素；

>原文链接：
[Contains in Swift](https://medium.com/@iostechset/contains-function-in-swift-5e0276638eb5) -- by 故胤道长（英文博客，需要科学上网工具）
[官方文档](https://developer.apple.com/documentation/swift/array/2297359-contains)

