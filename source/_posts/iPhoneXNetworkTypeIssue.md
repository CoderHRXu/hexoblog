---
title: iPhone X获取设备网络类型错误分析
---

## 背景
 在公司项目开发中，需要在网络请求中包含当前设备的网络状态参数，发请求的时候手机网络的类型有2G,3G,4G,WIFI等等，获取参数的方法如下：
```
获取网络状态
- (NSString *)getNetStatus {
    
    NSArray *children = [[[[UIApplication sharedApplication] valueForKeyPath:@"statusBar"] valueForKeyPath:@"foregroundView"] subviews];
    NSString *state = nil;
    int netType = 0;
    for (id child in children) {
        
        if ([child isKindOfClass:NSClassFromString(@"UIStatusBarDataNetworkItemView")]) {
            netType = [[child valueForKeyPath:@"dataNetworkType"] intValue];
            
            switch (netType) {
                    case 0:
                    state = @"无网络";
                    break;
                    case 1:
                    state =  @"2G";
                    break;
                    case 2:
                    state =  @"3G";
                    break;
                    case 3:
                    state =   @"4G";
                    break;
                    case 5:
                    state =  @"WIFI";
                    break;
                default:
                    state =  @"未知网络";
                    break;
            }
        }
    }
    return state;
}

```
从代码可以看出，基本原理是读取手机的状态栏，然后遍历状态栏的子控件，查看看一下苹果私有属性```UIStatusBarDataNetworkItemView```，根据属性的值来判断当前手机处于的什么样的网络状态。

## iPhone X错误&原因分析
当项目代码选择iPhone X模拟器运行的时候，会奔溃在上面的函数。
由于iPhone X独特的全面屏+刘海的设计(我觉得很丑)，手机顶部的状态栏完全是全新的设计，如下面对比截图：

![一般iPhone的状态栏.png](http://upload-images.jianshu.io/upload_images/1447375-9da96bd0c3d97433.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![iPhone X的状态栏](http://upload-images.jianshu.io/upload_images/1447375-d2b0ceb80288b953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

于是，想自己来hack一下这个iPhone X的状态栏
```
//obj 即为被遍历对象
unsigned int outCount = 0;
Ivar *ivars = class_copyIvarList([obj class], &outCount); 
for (int i = 0; i < outCount; i++) {
     Ivar ivar = ivars[i];
     printf("======== |%s\n", ivar_getName(ivar));
}

之后获取到以下Path，发现类似dataNetworkType
的_networkTypeView

NSArray *items = [[[[UIApplication sharedApplication] valueForKeyPath:@"statusBar"] valueForKeyPath:@"statusBar"] items];id obj = [[items valueForKeyPath:@"_UIStatusBarCellularExpandedItem"] valueForKeyPath:@"_networkTypeView"];

```

![image.png](http://upload-images.jianshu.io/upload_images/1447375-bd3e0ecbcd9b86a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但纵览整个属性列表，没有发现可用的NetworkType，只能弃用。没办法，谁让苹果爸爸用这样的特(sha)立(bi)独(nao)行(can)的设计，所以要另辟蹊径。

## 解决方法

通过翻阅文档发现可以通过CTTelephonyNetworkInfo获取网络运营商相关信息，这不就是3G，4G技术名称嘛！！！
```
导入 #import <CoreTelephony/CTTelephonyNetworkInfo.h>
点进去查看：
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyGPRS          __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyEdge          __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyWCDMA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSDPA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSUPA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMA1x        __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORev0  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevA  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevB  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyeHRPD         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyLTE           __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);

```
之后再次发现苹果的一个开源类[Reachability](https://developer.apple.com/library/content/samplecode/Reachability/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007324)中已对此做了相应封装，拿来即用。

```
typedef enum : NSInteger {
    NotReachable = 0,
    ReachableViaWiFi,
    kReachableVia2G,
    kReachableVia3G,
    kReachableVia4G
} NetworkStatus;

PS:最新版Reachability已简化枚举
typedef enum : NSInteger {
	NotReachable = 0,
	ReachableViaWiFi,
	ReachableViaWWAN
} NetworkStatus;

```

经过改写后的方法如下

```
+ (int)getNetStatus {
    
    NSString *stateString = nil;
    int netStatusNumber = 0;
    switch ([[Reachability reachabilityForInternetConnection] currentReachabilityStatus]) {
            
        case NotReachable: {
            
            stateString = @"无网络";
            netStatusNumber = 0;
        }
            break;
        case kReachableVia2G: {
            
            stateString = @"2G";
            netStatusNumber = 2;
        }
            break;
        case kReachableVia3G: {
         
            stateString = @"3G";
            netStatusNumber = 3;
        }
            break;
        case kReachableVia4G: {
            
            stateString = @"4G";
            netStatusNumber = 4;
        }
            break;
        case ReachableViaWiFi: {
            
            stateString = @"WIFI";
            netStatusNumber = 1;
        }
            break;
        default: {
            
            stateString = @"不可识别的网络";
            netStatusNumber = -1;
        }
            break;
    }
    return netStatusNumber;
}
```
这样处理完之后，不管是不是iPhone X都可以获取到正确的设备网络类型了，相比之前通过状态栏来判断，利用苹果自己的类更加的准确。
# 尽管这款刘海iPhone X在此刻尚未发售，能早点填一个坑就早点填。
