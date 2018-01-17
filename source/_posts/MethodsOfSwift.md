---
title: Swift常用方法总结----不定期更新
---

###1.本地读取json文件

```
	func readJsonFileByFileName(fileName : String) -> Any? {
        
        let path    = Bundle.main.path(forResource: "\(fileName).json", ofType: nil)
        let data    = NSData(contentsOfFile: path!)
        let jsonStr = try? JSONSerialization.jsonObject(with: data! as Data, options:.allowFragments)
        return jsonStr
    }
```
###2.实现手机号银行卡号输入加空格
eg：188 8888 8888

```

func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        
        let oldStr = textField.text
        var newStr = (oldStr as NSString?)?.replacingCharacters(in: range, with: string)
        if textField.placeholder == "请输入银行预留手机号" {
            
            newStr = newStr?.replacingOccurrences(of: " ", with: "")
            
            let expression = "^([1]{1}([34578]{1}([0-9]{0,9}+)?)?)?$"
            let regex = try?NSRegularExpression.init(pattern: expression, options: .caseInsensitive)
            let numOfMatches = regex?.matches(in: newStr!, options: NSRegularExpression.MatchingOptions(rawValue: 0), range: NSMakeRange(0, (newStr?.characters.count)!))
            
            if numOfMatches?.count == 0 {
                
                return false
            }
            
            _ = self.formatPhoneNumText(textField: textField, range: range, string: string)
            
            return false
        }
        else if textField.placeholder == "请输入银行卡号"{
        
            
            // 只能输入数字
            let characterSet = NSCharacterSet.init(charactersIn: "0123456789")
            
            let tempString = string.replacingOccurrences(of: " ", with: "")
            if (tempString.rangeOfCharacter(from: characterSet.inverted) != nil) {
                
                return false
            }
            
            var text = ((oldStr as NSString?)?.replacingCharacters(in: range, with: string))!
            text = text.replacingOccurrences(of: " ", with: "")
            
            var newString = ""
            
            while (text.characters.count) > 0 {
                
                let subString = (text as NSString).substring(to: min(text.characters.count, 4))
                newString = newString.appending(subString)
                if subString.characters.count == 4 {
                    
                    newString = newString.appending(" ")
                }
                text = (text as NSString).substring(from: min(text.characters.count, 4))
            }
            newString = newString.trimmingCharacters(in: characterSet.inverted)
            
            if newString.characters.count >= 24 {
                
                return false
            }
            
            if range.location != textField.text?.characters.count {
                
                textField.text = newString
                
                return false
            }
            
            textField.text = newString
            
         
            return false
            
            
            
        }else {
            // 只能输入数字
            let characterSet = NSCharacterSet.init(charactersIn: "0123456789")
            
            let tempString = string.replacingOccurrences(of: " ", with: "")
            if (tempString.rangeOfCharacter(from: characterSet.inverted) != nil) {
                
                return false
            }
            if (newStr?.characters.count)! > 20 {
                
                return false
            }
        }
        
        return true
    }

private func formatPhoneNumText (textField:UITextField, range:NSRange, string:String) -> Bool {
        
        var text = textField.text!
        
        // 只能输入数字
        let characterSet = NSCharacterSet.init(charactersIn: "0123456789")
        
        let tempString = string.replacingOccurrences(of: " ", with: "")
        if (tempString.rangeOfCharacter(from: characterSet.inverted) != nil) {
            
            return false
        }
        
        text = ((text as NSString?)?.replacingCharacters(in: range, with: string))!
        text = text.replacingOccurrences(of: " ", with: "")
        
        text.insert(" ", at: (text.startIndex))
        var newString = ""
        
        while (text.characters.count) > 0 {
            
            let subString = (text as NSString).substring(to: min(text.characters.count, 4))
            newString = newString.appending(subString)
            if subString.characters.count == 4 {
                
                newString = newString.appending(" ")
            }
            text = (text as NSString).substring(from: min(text.characters.count, 4))
        }
        newString = newString.trimmingCharacters(in: characterSet.inverted)
        
        if newString.characters.count >= 14 {
            
            return false
        }
        
        
        if range.location != textField.text?.characters.count {
            
            textField.text = newString
            
            if textField == self.phoneTextField {
                
            }
            
            return false
        }
        
        textField.text = newString
        
        if textField == self.phoneTextField {
            
        }
        return false
    }

```
###3.给UITextField的placeholder设置字体大小
```
使用：textField.verticalLeftPlaceholder(phFontSize: 15)

源码：
extension UITextField {
    
    func verticalLeftPlaceholder(phFontSize size: CGFloat = 15, text: String = "")  {
        
        var placeholderStr = text
        
        if text.isEmpty {
            
            placeholderStr = self.placeholder ?? ""
        }
        
        let pStyle = NSMutableParagraphStyle()
        pStyle.minimumLineHeight    = (self.font?.lineHeight)! - ((self.font?.lineHeight)! - UIFont.systemFont(ofSize: size).lineHeight)/2.0
        let aString = NSAttributedString(string: placeholderStr, attributes: [NSFontAttributeName: UIFont.systemFont(ofSize: size), NSParagraphStyleAttributeName: pStyle])
        
        self.attributedPlaceholder    = aString
    }
    
}
```

###4.插入排序
⒈ 从第一个元素开始，该元素可以认为已经被排序
⒉ 取出下一个元素，在已经排序的元素序列中从后向前扫描
⒊ 如果该元素（已排序）大于新元素，将该元素移到下一位置
⒋ 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
⒌ 将新元素插入到下一位置中
⒍ 重复步骤2~5

```
public func insertionSort(arr: inout Array<Int>) {

    for i in 0..<arr.count - 1 {
        if arr[i + 1] < arr[i] {
            let temp = arr[i + 1]
            for j in (1...(i + 1)).reversed() {
                if arr[j - 1] > temp {
                    (arr[j - 1],arr[j]) = (arr[j],arr[j - 1])
                }
            }
        }
    }
}
```

###5.冒泡排序

⒈比较相邻的元素。如果第一个比第二个大，就交换他们两个。
⒉ 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
⒊ 针对所有的元素重复以上的步骤，除了最后一个。
⒋ 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。
```
public func bubbleSort(arr: inout Array<Int>) {
    for i in 0..<(arr.count - 1) {
        for j in 0..<(arr.count - i - 1) {
            if arr[j] > arr[j + 1] {
                (arr[j],arr[j + 1]) = (arr[j + 1],arr[j])
            }
        }
    }
}
```

###6.快速排序（冒泡排序的改进版）

⒈设置两个变量i、j，排序开始的时候：i=0，j=n-1。
⒉ 以第一个数组元素作为关键数据，赋值给key，即key=A[0]。
⒊ 从j开始向前搜索，即由后开始向前搜索(j--)，找到第一个小于key的值A[j]，将A[j]和A[i]互换。
⒋ 从i开始向后搜索，即由前开始向后搜索(i++)，找到第一个大于key的A[i]，将A[i]和A[j]互换。
⒌重复第3、4步，直到i=j
```
 public func quickSort(arr: inout Array<Int>, leftIndex:  Int, rightIndex:  Int) {

    if leftIndex < rightIndex {
        var left = leftIndex
        var right = rightIndex
        let midIndex = getMidIndex(arr: &mArray, leftIndex: &left, rightIndex: &right)
        quickSort(arr: &mArray, leftIndex: leftIndex, rightIndex: midIndex - 1)
        quickSort(arr: &mArray, leftIndex: midIndex + 1, rightIndex: rightIndex)
    }
}

private func getMidIndex(arr: inout Array<Int>, leftIndex: inout Int, rightIndex: inout Int) -> (Int) {
    let temp = arr[leftIndex]
    while leftIndex < rightIndex {

        while leftIndex < rightIndex && temp <= arr[rightIndex] {
            rightIndex -= 1
        }
        if leftIndex < rightIndex {
            arr[leftIndex] = arr[rightIndex]
        }
        while leftIndex < rightIndex && arr[leftIndex] <= temp {
            leftIndex += 1
        }
        if leftIndex < rightIndex {
            arr[rightIndex] = arr[leftIndex]
        }
    }
    arr[leftIndex] = temp
    return leftIndex
}
```
###7.归并排序（速度仅次于快速排序）

⒈将序列每相邻两个数字进行归并操作（merge)，形成floor(n/2)个序列，排序后每个序列包含两个元素。
⒉ 将上述序列再次归并，形成floor(n/4)个序列，每个序列包含四个元素。
⒊ 重复步骤2，直到所有元素排序完毕。
```
 public func mergeSort(arr: inout Array<Int>) {
    var tempArr: Array<Array<Int>> = []
    for item in arr {
       var subArr: Array<Int> = []
       subArr.append(item)
       tempArr.append(subArr)
    }
    while tempArr.count != 1 {
        var i = 0
        while i < tempArr.count - 1 {
            tempArr[i] = mergeTowArr(arr1: tempArr[i], arr2: tempArr[i+1])
            tempArr.remove(at : i+1)
            i += 1
        }
    }
 }

private func mergeTowArr(arr1: Array<Int>, arr2: Array<Int>) -> Array<Int> {
    var mergeArr: Array<Int> = []
    var firstIndex = 0
    var secondIndex = 0
    while firstIndex < arr1.count && secondIndex < arr2.count {
        if arr1[firstIndex]  < arr2[secondIndex] {
            merge.append(arr1[firstIndex])
            firstIndex += 1
        }else{
            merge.append(arr2[secondIndex])
            secondIndex += 1
        }
    }
    while firstIndex < arr1.count {
         merge.append(arr1[firstIndex])
         firstIndex += 1
    }
    while secondIndex < arr2.count {
         merge.append(arr2[secondIndex])
         secondIndex += 1
    }

    return mergeArr
}
```

###8.选择排序

选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。 选择排序是不稳定的排序方法（比如序列[5， 5， 3]第一次就将第一个[5]与[3]交换，导致第一个5挪动到第二个5后面）
```
public func selectSort(arr: inout Array<Int>) {
    for i in 0..<arr.count {
        var index = i
        for j in (i + 1)..<arr.count {
            if arr[index] > arr[j] {
                index = j
            }
        }
        if index != i {
            (arr[i],arr[index]) = (arr[index],arr[i])
        }
    }
}
```
###9.堆排序（选择排序的一种）
通过headMakeFrom将数组arr变为大顶堆，再把大顶堆的第一值和最后一个值交换，数组长度减1。通过heapAdjast将交换后的数组重新排序成大顶堆直到数组长度为0。

```
public func heapSort(arr: inout Array<Int>) {//时间复杂度O(n*logn)
    var endIndex = arr.count - 1
    headMakeFrom(arr: &arr)

    while endIndex > 0 {
        (arr[0],arr[endIndex]) = (arr[endIndex],arr[0])
        heapAdjast(arr: &arr,starIndex: 1, arrLength: endIndex)
        endIndex -= 1
    }
}


private func heapMakeFrom(arr: inout Array<Int>) {
    var lastHead = arr.count/2
    while lastHead > 0 {
        heapAdjast(arr:  &arr, starIndex: lastHead, arrLenght: arr.count)
        lastHead -= 1
    }
}

private func heapAdjast(arr: Array<Int>, starIndex: Int, arrLength: Int) {
    var temp = arr[starIndex - 1]
    var parentIndex = starIndex
    var childIndex = 2 * parentIndex
    while childIndex <= arrLength {
        if childIndex < arrLength && arr[childIndex] > arr[childIndex - 1] {
            childIndex += 1
        }
        if arr[childIndex-1] > temp {
            arr[parentIndex - 1] = arr[childIndex - 1]
        }else{
            break 
        }
        parentIndex = childIndex
        childIndex = 2 * parentIndex 
    }
    arr[parentIndex - 1] = temp
}
```
###10. 生成指定尺寸的纯色图片
```
func imageWithColor(color: UIColor!, size: CGSize) -> UIImage{
    var size = size
    if CGSizeEqualToSize(size, CGSizeZero){
        size = CGSizeMake(1, 1)
    }
    let rect = CGRectMake(0, 0, size.width, size.height)
    UIGraphicsBeginImageContext(rect.size)
    let context = UIGraphicsGetCurrentContext()
    CGContextSetFillColorWithColor(context, color.CGColor)
    CGContextFillRect(context, rect)

    let image = UIGraphicsGetImageFromCurrentImageContext()
    UIGraphicsEndImageContext()
    return image
}
```

###11- 修改图片尺寸
```
func imageScaleToSize(image: UIImage, size: CGSize) -> UIImage{
    // 创建一个bitmap的context
    // 并把它设置成为当前正在使用的context
    // Determine whether the screen is retina
    if UIScreen.mainScreen().scale == 2.0{
        UIGraphicsBeginImageContextWithOptions(size, false, 2.0)
    }else{
        UIGraphicsBeginImageContext(size)
    }

    // 绘制改变大小的图片
    image.drawInRect(CGRectMake(0, 0, size.width, size.height))

    // 从当前context中创建一个改变大小后的图片
    let scaledImage = UIGraphicsGetImageFromCurrentImageContext()

    // 使当前的context出堆栈
    UIGraphicsEndImageContext()

    // 返回新的改变大小后的图片
    return scaledImage
}
```
###12. 压缩图片大小
```
func imageCompress(originalImage: UIImage) -> UIImage{
    guard let imageData = UIImageJPEGRepresentation(originalImage, 0.5) else{
        return originalImage
    }

    let compressImage = UIImage(data: imageData)!
    return compressImage
}
```