---
title: OC常用方法总结--不定期更新
---


### 1.统一收起键盘

``` 
[[[UIApplication sharedApplication] keyWindow] endEditing:YES];
```
### 2.数组去重
```
NSArray *newArr = [oldArr valueForKeyPath:@“@distinctUnionOfObjects.self"];
@distinctUnionOfObjects.self  表示比较对象
@distinctUnionOfObjects.[key] 表示比较对象的任意keyboarded
```
### 3.修改textField的placeholder的字体颜色、大小
```
[textField setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
[textField setValue:[UIFont boldSystemFontOfSize:16] forKeyPath:@"_placeholderLabel.font"];
```
### 4.给一个view截图
```
UIGraphicsBeginImageContextWithOptions(view.bounds.size, YES, 0.0);
[view.layer renderInContext:UIGraphicsGetCurrentContext()];
UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
```
### 5.UIView背景颜色渐变
```
UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 100)];
[self.view addSubview:view];
CAGradientLayer *gradient = [CAGradientLayer layer];
gradient.frame = view.bounds;
gradient.colors = [NSArray arrayWithObjects:(id)[[UIColor blackColor] CGColor], (id)[[UIColor whiteColor] CGColor], nil];
[view.layer insertSublayer:gradient atIndex:0];
```
### 6.为UIView某个角添加圆角
```
// 左上角和右下角添加圆角
UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:view.bounds byRoundingCorners:(UIRectCornerTopLeft | UIRectCornerBottomRight) cornerRadii:CGSizeMake(20, 20)];
CAShapeLayer *maskLayer = [CAShapeLayer layer];
maskLayer.frame = view.bounds;
maskLayer.path = maskPath.CGPath;
view.layer.mask = maskLayer;
```

### 7.拿到当前正在显示的控制器
```
- (UIViewController *)getVisibleViewControllerFrom:(UIViewController*)vc {
    if ([vc isKindOfClass:[UINavigationController class]]) {
        return [self getVisibleViewControllerFrom:[((UINavigationController*) vc) visibleViewController]];
    }else if ([vc isKindOfClass:[UITabBarController class]]){
        return [self getVisibleViewControllerFrom:[((UITabBarController*) vc) selectedViewController]];
    } else {
        if (vc.presentedViewController) {
            return [self getVisibleViewControllerFrom:vc.presentedViewController];
        } else {
            return vc;
        }
    }
}
```
### 8.tableViewCell分割线顶到头

```
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
    [cell setSeparatorInset:UIEdgeInsetsZero];
    [cell setLayoutMargins:UIEdgeInsetsZero];
    cell.preservesSuperviewLayoutMargins = NO;
}
- (void)viewDidLayoutSubviews {
    [self.tableView setSeparatorInset:UIEdgeInsetsZero];
    [self.tableView setLayoutMargins:UIEdgeInsetsZero];
}
```

### 9.image添加倒影效果

```
- (void)setImageShadow {
    CGRect frame = self.frame;
    frame.origin.y += (frame.size.height + 1);

    UIImageView *reflectionImageView = [[UIImageView alloc] initWithFrame:frame];
    self.clipsToBounds = TRUE;
    reflectionImageView.contentMode = self.contentMode;
    [reflectionImageView setImage:self.image];
    reflectionImageView.transform = CGAffineTransformMakeScale(1.0, -1.0);

    CALayer *reflectionLayer = [reflectionImageView layer];
    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.bounds = reflectionLayer.bounds;
    gradientLayer.position = CGPointMake(reflectionLayer.bounds.size.width / 2, reflectionLayer.bounds.size.height * 0.5);
    gradientLayer.colors = [NSArray arrayWithObjects:
                                                        (id)[[UIColor clearColor] CGColor],
                                                        (id)[[UIColor colorWithRed:1.0 green:1.0 blue:1.0 alpha:0.3] CGColor], nil];
    gradientLayer.startPoint = CGPointMake(0.5,0.5);
    gradientLayer.endPoint = CGPointMake(0.5,1.0);
    reflectionLayer.mask = gradientLayer;
    [self.superview addSubview:reflectionImageView];
}
```

### 10.添加图片水印

```
// 画水印
- (void) setImage:(UIImage *)image withWaterMark:(UIImage *)mark inRect:(CGRect)rect {
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 4.0){
        UIGraphicsBeginImageContextWithOptions(self.frame.size, NO, 0.0);
    }
    //原图
    [image drawInRect:self.bounds];
    //水印图
    [mark drawInRect:rect];
    UIImage *newPic = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    self.image = newPic;
}
```
### 11.删除NSUserDefaults所有记录
```
//方法一
NSString *appDomain = [[NSBundle mainBundle] bundleIdentifier];
[[NSUserDefaults standardUserDefaults] removePersistentDomainForName:appDomain];  
//方法二  
- (void)resetDefaults {   
  NSUserDefaults * defs = [NSUserDefaults standardUserDefaults];
     NSDictionary * dict = [defs dictionaryRepresentation];
     for (id key in dict) {
          [defs removeObjectForKey:key];
     }
      [defs synchronize];
 }
// 方法三
[[NSUserDefaults standardUserDefaults] setPersistentDomain:[NSDictionary dictionary] forName:[[NSBundle mainBundle] bundleIdentifier]];
```

### 12.跳进app权限设置

```
if (UIApplicationOpenSettingsURLString != NULL) {
    NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
    [[UIApplication sharedApplication] openURL:url];
}
```

### 13.获取app缓存大小

```
- (CGFloat)getCachSize {

    NSUInteger imageCacheSize = [[SDImageCache sharedImageCache] getSize];
    // 获取自定义缓存大小
    // 用枚举器遍历 一个文件夹的内容
    // 1.获取 文件夹枚举器
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    NSDirectoryEnumerator *enumerator = [[NSFileManager defaultManager] enumeratorAtPath:myCachePath];
    __block NSUInteger count = 0;
    // 2.遍历
    for (NSString *fileName in enumerator) {
        NSString *path = [myCachePath stringByAppendingPathComponent:fileName];
        NSDictionary *fileDict = [[NSFileManager defaultManager] attributesOfItemAtPath:path error:nil];
        //自定义所有缓存大小
        count += fileDict.fileSize;
    }
    // 得到是字节  转化为M
    CGFloat totalSize = ((CGFloat)imageCacheSize+count)/1024/1024;
    return totalSize;
}
```

### 14.清理app缓存
```
- (void)cleanAppCache {
    // 删除两部分
    // 1.删除 sd 图片缓存
    // 先清除内存中的图片缓存
    [[SDImageCache sharedImageCache] clearMemory];
    // 清除磁盘的缓存
    [[SDImageCache sharedImageCache] clearDisk];
    // 2.删除自己缓存
    NSString *myCachePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches"];
    [[NSFileManager defaultManager] removeItemAtPath:myCachePath error:nil];
}
```

### 15.几个常用权限判断
```
- (void)checkAppAuthorization{
    if ([CLLocationManager authorizationStatus] ==kCLAuthorizationStatusDenied{
    
        NSLog(@"没有定位权限");
    }
    AVAuthorizationStatus statusVideo = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (statusVideo == AVAuthorizationStatusDenied) {
        NSLog(@"没有摄像头权限");
    }
    AVAuthorizationStatus statusAudio = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeAudio];
    if (statusAudio == AVAuthorizationStatusDenied) {
        NSLog(@"没有录音权限");
    }
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
        if (status == PHAuthorizationStatusDenied) {
            NSLog(@"没有相册权限");
        }
    }];
}

```

### 16.文字生成二维码
```
/**
 生成二维码

 @param codeString 二维码客串
 @return 二维码图片
 */
+ (UIImage *)qrCodeWithString:(NSString *)codeString{
    
    // 1.创建过滤器
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    // 2.恢复默认
    [filter setDefaults];
    // 3.给过滤器添加数据(正则表达式/账号和密码)
    NSString *dataString = codeString;
    NSData *data = [dataString dataUsingEncoding:NSUTF8StringEncoding];
    [filter setValue:data forKeyPath:@"inputMessage"];
    // 4.获取输出的二维码
    CIImage *outputImage = [filter outputImage];
    // 5.将CIImage转换成UIImage，并放大显示
    return [self createNonInterpolatedUIImageFormCIImage:outputImage withSize:400];
}

/**
 根据CIImage生成指定大小的UIImage
 
 @param image CIImage
 @param size 图片宽度
 */
+ (UIImage *)createNonInterpolatedUIImageFormCIImage:(CIImage *)image withSize:(CGFloat)size {
    
    CGRect extent = CGRectIntegral(image.extent);
    CGFloat scale = MIN(size/CGRectGetWidth(extent), size/CGRectGetHeight(extent));
    // 1.创建bitmap;
    size_t width = CGRectGetWidth(extent) * scale;
    size_t height = CGRectGetHeight(extent) * scale;
    CGColorSpaceRef cs = CGColorSpaceCreateDeviceGray();
    CGContextRef bitmapRef = CGBitmapContextCreate(nil, width, height, 8, 0, cs, (CGBitmapInfo)kCGImageAlphaNone);
    CIContext *context = [CIContext contextWithOptions:nil];
    CGImageRef bitmapImage = [context createCGImage:image fromRect:extent];
    CGContextSetInterpolationQuality(bitmapRef, kCGInterpolationNone);
    CGContextScaleCTM(bitmapRef, scale, scale);
    CGContextDrawImage(bitmapRef, extent, bitmapImage);
    // 2.保存bitmap到图片
    CGImageRef scaledImage = CGBitmapContextCreateImage(bitmapRef);
    CGContextRelease(bitmapRef);
    CGImageRelease(bitmapImage);
    return [UIImage imageWithCGImage:scaledImage];
}
```

### 17.手机号码验证
```
/**
 *  手机号码验证
 *
 *  @param mobileNum 手机号
 *
 *  @return 是否
 */
+ (BOOL)isMobileNumber:(NSString *)mobileNum {
    
    if (mobileNum.length != 11){
        return NO;
    }
    /**
     * 手机号码:
     * 13[0-9], 14[5,7], 15[0, 1, 2, 3, 5, 6, 7, 8, 9], 17[0, 1, 6, 7, 8], 18[0-9]
     * 移动号段: 134,135,136,137,138,139,147,150,151,152,157,158,159,170,178,182,183,184,187,188
     * 联通号段: 130,131,132,145,152,155,156,170,171,176,185,186
     * 电信号段: 133,134,153,170,173,177,180,181,189
     */
    NSString *MOBILE = @"^1(3[0-9]|4[57]|5[0-35-9]|7[013678]|8[0-9])\\d{8}$";
    /**
     * 中国移动：China Mobile
     * 134,135,136,137,138,139,147,150,151,152,157,158,159,170,178,182,183,184,187,188
     */
    NSString *CM = @"^1(3[4-9]|4[7]|5[0-27-9]|7[08]|8[2-478])\\d{8}$";
    /**
     * 中国联通：China Unicom
     * 130,131,132,145,152,155,156,170,171,176,185,186
     */
    NSString *CU = @"^1(3[0-2]|4[5]|5[256]|7[016]|8[56])\\d{8}$";
    /**
     * 中国电信：China Telecom
     * 133,134,153,170,173,177,180,181,189
     */
    NSString *CT = @"^1(3[34]|53|7[037]|8[019])\\d{8}$";
    
    
    NSPredicate *regextestmobile = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", MOBILE];
    NSPredicate *regextestcm = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CM];
    NSPredicate *regextestcu = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CU];
    NSPredicate *regextestct = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CT];
    
    if (([regextestmobile evaluateWithObject:mobileNum] == YES)
        || ([regextestcm evaluateWithObject:mobileNum] == YES)
        || ([regextestct evaluateWithObject:mobileNum] == YES)
        || ([regextestcu evaluateWithObject:mobileNum] == YES)){
        
        return YES;
    }
    else{
        return NO;
    }
}

```

### 18.邮箱验证
```
/**
 邮箱验证

 @param email 邮箱地址
 @return 是否
 */
+ (BOOL)isValidateEmail:(NSString *)email {
    
    NSString *emailRegex = @"[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}";
    NSPredicate *emailTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", emailRegex];
    return [emailTest evaluateWithObject:email];
}

```


### 19.根据年和月返回当月天数
```
+ (NSInteger)getDaysWithYear:(NSInteger)year
                       month:(NSInteger)month{
                       
    switch (month) {
        case 1:
            return 31;
            break;
        case 2:
            if (year%400==0 || (year%100!=0 && year%4 == 0)) {
                return 29;
            }else{
                return 28;
            }
            break;
        case 3:
            return 31;
            break;
        case 4:
            return 30;
            break;
        case 5:
            return 31;
            break;
        case 6:
            return 30;
            break;
        case 7:
            return 31;
            break;
        case 8:
            return 31;
            break;
        case 9:
            return 30;
            break;
        case 10:
            return 31;
            break;
        case 11:
            return 30;
            break;
        case 12:
            return 31;
            break;
        default:
            return 0;
            break;
    }
}

```

### 20.压缩图片成二进制数据
```
+ (NSData *)compressImage:(UIImage *)image {
    
    NSData *imageData = UIImageJPEGRepresentation(image, 1.0f);
    if (imageData.length <= 1000*1000) {
        
        return imageData;
    } else if (imageData.length > 1000*1000 && imageData.length < 3000 * 1000) {
        
        imageData = UIImageJPEGRepresentation(image, 0.8f);
        return imageData;
    } else if (imageData.length > 3000*1000 && imageData.length < 5000 * 1000) {
        
        imageData = UIImageJPEGRepresentation(image, 0.6f);
        return imageData;
    } else {
        
        imageData = UIImageJPEGRepresentation(image, 0.4f);
        return imageData;
    }
}
```

### 21.根据正则，过滤特殊字符
```
/**
 根据正则，过滤特殊字符

 @param regexString 正则表达式
 @return 过滤后的字符串
 */
- (NSString *)filterCharactorWithRegex:(NSString *)regexString {
    
    NSString *searchText = self;
    NSError *error = NULL;
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:regexString options:NSRegularExpressionCaseInsensitive error:&error];
    NSString *result = [regex stringByReplacingMatchesInString:searchText options:NSMatchingReportCompletion range:NSMakeRange(0, searchText.length) withTemplate:@""];
    return result;
}
```

### 22.身份证号码验证
```
/**
 身份证号码验证
 * 验证方法：
 * 1、将前面的身份证号码17位数分别乘以不同的系数。从第一位到第十七位的系数分别为：7－9－10－5－8－4－2－1－6－3－7－9－10－5－8－4－2
 * 2、将这17位数字和系数相乘的结果相加
 * 3、用加出来和除以11，看余数是多少？
 * 4、余数只可能有0－1－2－3－4－5－6－7－8－9－10这11个数字。其分别对应的最后一位身份证的号码为1－0－X －9－8－7－6－5－4－3－2
 * 5、通过上面得知如果余数是3，就会在身份证的第18位数字上出现的是9。如果对应的数字是10，身份证的最后一位号码就是罗马数字x
 @return 是/否
 */
- (BOOL)isIDCardNumber {
    
    NSInteger sum = 0;
    NSArray *numArr = @[@7, @9, @10, @5, @8, @4, @2, @1, @6, @3, @7, @9, @10, @5, @8, @4, @2]; // 系数
    NSArray *lastArr = @[@"1", @"0", @"X", @"9", @"8",@"7", @"6", @"5", @"4", @"3", @"2"]; // 最后一位身份证的号码
    
    if (self.length == 15) {
        NSString *lastCharactor =[self substringFromIndex:self.length -1];
        if ([lastCharactor isEqualToString:@"X"] || [lastCharactor isEqualToString:@"x"]) {
            NSString * subStr14 = [self substringToIndex:self.length -1];
            return [self isPureInt:subStr14];
        }else{
            return [self isPureInt:self];
        }
        
    }
    if (self.length == 18) {
        //将字符串转化为大写
        NSString *newString =[self uppercaseString];
        NSMutableArray *array = [NSMutableArray array];
        for (int i = 0; i < [self length]; i++) {
            [array addObject:[NSString stringWithFormat:@"%C", [newString characterAtIndex:i]]];
        }
        
        for (int i = 0; i < self.length - 1; i++) {
            sum += [array[i] integerValue] * [numArr[i] integerValue];
        }
        return [array[17] isEqualToString:lastArr[sum % 11]];
    }
    
    return NO;
}
```

### 23.判断字符串是否是纯数字
```
/**
 判断字符串是否是纯数字
 
 @param string 带判断字符串
 @return 是/否
 */
- (BOOL)isPureInt:(NSString *)string {
    
    NSScanner* scan = [NSScanner scannerWithString:string];
    int val;
    return [scan scanInt:&val] && [scan isAtEnd];
    
}
```

### 24.车牌校验
```
/**
 车牌校验

 @return 是/否
 */
- (BOOL)isCarLicenceNum{

    if (self.length == 7 || self.length == 8) {
        
        NSString *carRegex = @"^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-Z0-9]{4}[A-Z0-9挂学警港澳]{1}$";
        NSPredicate *carTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@",carRegex];
        return [carTest evaluateWithObject:self];
    }else{
        return NO;
    }
}
```

### 25.JSON格式的字符串转换成字典

```
/*
 * @brief 把格式化的JSON格式的字符串转换成字典
 * @param jsonString JSON格式的字符串
 * @return 返回字典
 */
+ (NSDictionary *)dictionaryWithJsonString:(NSString *)jsonString {
    
    if (jsonString == nil) {
        
        return nil;
        
    }
    
    NSData *jsonData = [jsonString dataUsingEncoding:NSUTF8StringEncoding];
    
    NSError *err;
    
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData
                                                        options:NSJSONReadingMutableContainers
                                                          error:&err];
    
    if(err) {
        
        NSLog(@"%@json解析失败：%@",jsonString,err);
        return nil;
        
    }
    return dic;
}
```

### 26.字典转json格式字符串
```
/**
 *  字典转json格式字符串：
 *
 *  @param NSString <#NSString description#>
 *
 *  @return <#return value description#>
 */

- (NSString*)dictionaryToJson {
    
    NSError *parseError = nil;
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:self options:NSJSONWritingPrettyPrinted error:&parseError];
    return [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
}

```



### 27.通过Value取Key
```
/**
 通过Value取Key

 @param value ValueString
 @return KeyString
 */
- (NSString *)keyForValue:(NSString *)value {
    
    __block NSString *keyStr = nil;
    [self enumerateKeysAndObjectsUsingBlock:^(NSString * key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        if ([value isEqualToString:obj]) {
            keyStr = key;
            *stop = YES;
        }
    }];
    return keyStr;
}
```
### 28.加载本地json文件

将本地json文件，导入工程，确定target membership勾选上

```
- (void)loadJson{

    NSError*error;
    //获取文件路径
    NSString *filePath          =[[NSBundle mainBundle] pathForResource:@"test" ofType:@"json"];
    //根据文件路径读取数据
    NSData *jdata               = [[NSData alloc]initWithContentsOfFile:filePath];
    //格式化成json数据
    NSDictionary *jsonObject    = [NSJSONSerialization JSONObjectWithData:jdata options:kNilOptions error:&error];

}
```

### 29.获取app中所有icon名字数组

```

	//获取app中所有icon名字数组
    NSDictionary *infoDict  = [[NSBundle mainBundle] infoDictionary];
    NSArray *iconsArr       = infoDict[@"CFBundleIcons"][@"CFBundlePrimaryIcon"][@"CFBundleIconFiles"];
    NSString *iconLastName  = [iconsArr lastObject];
    
```

### 30.计算两个时间
计算两个时间字符串的时间差 

```
/**
 计算时长
 */
- (NSString *)caculateDurationWithStartTime:(NSString *)startTime andEndTime:(NSString *)endTime{
    if (endTime == nil) {
        return @"未知";
    }
    NSDate *startDate 			= [NSDate date];
    NSDate *endDate 			= [NSDate date];
    
    NSDateFormatter *format 	= [[NSDateFormatter alloc]init];
    format.dateFormat 			= @"YYYY-MM-dd HH:mm:ss";
    format.timeZone				= [NSTimeZone systemTimeZone];
    startDate 					= [format dateFromString:startTime];
    endDate 						= [format dateFromString:endTime];
    NSTimeInterval duration 	= endDate.timeIntervalSince1970 - startDate.timeIntervalSince1970;
    int hour 						= (int)duration / 3600;
    int min 						= (int)duration % 3600 / 60;
    int sec 						= (int)duration % 60;
    
    return [NSString stringWithFormat:@"%d小时%d分%d秒",hour,min,sec];
}

```

### 31.计算两个NSDate经历了多少天
```
/**
 *  返回两个NSDate经历了多少天
 */
- (NSInteger)getDaysFrom:(NSDate *)startDate To:(NSDate *)endDate{
    
    NSCalendar *gregorian = [[NSCalendar alloc]initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    
    [gregorian setFirstWeekday:2];
    
    //去掉时分秒信息
    NSDate *fromDate;
    NSDate *toDate;
    [gregorian rangeOfUnit:NSCalendarUnitDay startDate:&fromDate interval:NULL forDate:startDate];
    [gregorian rangeOfUnit:NSCalendarUnitDay startDate:&toDate interval:NULL forDate:endDate];
    NSDateComponents *dayComponents = [gregorian components:NSCalendarUnitDay fromDate:fromDate toDate:toDate options:0];
    
    return dayComponents.day;
}

```

### 32.主动申请常用权限

记得在info.plist里面加入对应权限的描述。
```
    ///申请麦克风权限
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
    }];
    ///申请拍照权限
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeAudio completionHandler:^(BOOL granted) {
    }];
    ///申请相册权限
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
    }];
```



