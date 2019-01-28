---
title: iOS小数四舍五入总结

---

iOS项目里，涉及到double类型是数据，经常会用到四舍五入这样的取值方式，网上查阅资料之后，经常会不起作用，最终把有效的方法，总结了一下。
OC:
```
/**
 四舍五入字符串

 @param round 小数位 eg: 2
 @param numberString 数字 eg 0.125
 @return 四舍五入之后的 eg: 0.13
 */
- (double )roundNumberStringWithRound:(NSInteger)round numberString:(NSString *)numberString{
    
    if (numberString == nil) {
        return 0;
    }
    NSDecimalNumberHandler *roundingBehavior    = [NSDecimalNumberHandler decimalNumberHandlerWithRoundingMode:NSRoundPlain scale:round raiseOnExactness:NO raiseOnOverflow:NO raiseOnUnderflow:NO raiseOnDivideByZero:NO];
    NSDecimalNumber *aDN                        = [[NSDecimalNumber alloc] initWithString:numberString];
    NSDecimalNumber *resultDN                   = [aDN decimalNumberByRoundingAccordingToBehavior:roundingBehavior];
    return resultDN.doubleValue;
}

```

Swift：
```
    /// 四舍五入2位小数小数
    ///
    /// - round: 小数位（默认2位小数）
    /// - Parameter numberString: 格式化之前 eg 0.125
    /// - Returns: 格式化之后 eg: 0.13
    func roundNumberString(round:Int = 2, numberString : String) -> Double {
        if numberString.isEmpty || round == 0 {
            return 0.0
        }
        
        let roudingBehavior     = NSDecimalNumberHandler(roundingMode: .plain, scale: Int16(round), raiseOnExactness: false, raiseOnOverflow: false, raiseOnUnderflow: false, raiseOnDivideByZero: false)
        let aDn                 = NSDecimalNumber(string: numberString)
        let resultDn            = aDn.rounding(accordingToBehavior: roudingBehavior)
        return resultDn.doubleValue
        
    }

```