---
title: iOS小数点格式化：最多保留两位小数，小数点后末尾的0不要

---

## 需求 
如题的需求，再详细解释一下就是：
1.如果有两位小数不为0则保留两位小数,eg: "0.23"
2.如果有一位小数不为0则保留一位小数，否则显示整数, eg: "0.2" "0"
也就是说，不能出现"0.20","0.00"这种情况。

平时在项目开发中，格式化小数乘字符串是时候，经常会保留2位小数处理，不足则用0补全，也就是在格式化方法中用 %.2f 作占位符，来占位小数。这样得出来的字符串肯定是类似"X.XX"的字符串，且如果不足两位小数，都会用0来补全。

然鹅现在的需求变了，要求保留两位小数，且能不显示0就不显示。

## 解决方法
OC：
```
/**
 过滤器/ 将.2f格式化的字符串，去除末尾0

 @param numberStr .2f格式化后的字符串
 @return 去除末尾0之后的
 */
- (NSString *)removeSuffix:(NSString *)numberStr{
    if (numberStr.length > 1) {
        
        if ([numberStr componentsSeparatedByString:@"."].count == 2) {
            NSString *last = [numberStr componentsSeparatedByString:@"."].lastObject;
            if ([last isEqualToString:@"00"]) {
                numberStr = [numberStr substringToIndex:numberStr.length - (last.length + 1)];
                return numberStr;
            }else{
                if ([[last substringFromIndex:last.length -1] isEqualToString:@"0"]) {
                    numberStr = [numberStr substringToIndex:numberStr.length - 1];
                    return numberStr;
                }
            }
        }
        return numberStr;
    }else{
        return nil;
    }
}

```
Swift
```
    /// 过滤器 将.2f格式化的字符串，去除末尾0
    ///
    /// - Parameter numberString: .2f格式化后的字符串
    /// - Returns: 去除末尾0之后的
    func removeSuffix(numberString : String) -> String {
        
        if numberString.count > 1 {
            let strs = numberString.components(separatedBy: ".")
            let last = strs.last!
            if strs.count == 2 {
                if last == "00" {
                    
                    let indexEndOfText      = numberString.index(numberString.endIndex, offsetBy:-3)
                    return String(numberString[..<indexEndOfText])
                    
                }else{
                    
                    let indexStartOfText    = numberString.index(numberString.endIndex, offsetBy:-1)
                    let str                 = numberString[indexStartOfText...]
                    let indexEndOfText      = numberString.index(numberString.endIndex, offsetBy:-1)
                    if str == "0" {
                        return String(numberString[..<indexEndOfText])
                    }
                    
                }
                
            }
            
            return numberString
            
        }else{
            return ""
        }
    }
```
思路：1.先将小数按照.2f格式化处理成“0.00”;
2.然后再根据小数点"."，切割字符串比较,判断最后的是不是“0”，是的话，直接切除掉；

同样思路的也适用于格式化.3f的情况，判断方法中再加入“000”的判断，然后判断切割后的字符串。