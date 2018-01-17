---
title: 适配iOS 11和iPhoneX屏幕适配遇到的一些坑
---


随着iOS 11正式版,以及Xcode9正式版的发布，已有项目工程对于新版本系统和机型的适配就提上了日程。下面简单讲一讲遇到的一些坑，同时也向在解决问题中查阅的资料文章作者表示感谢！

### 安全区域

#### 坑NO.1 iOS 11下APP中tableView内容下移20pt

![image.png](http://upload-images.jianshu.io/upload_images/1447375-74617a67ad70b287.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个页面隐藏了导航栏（自定义导航栏），下面就是一个tableview，加上了MJRefresh第三方。可以明显看出上方状态栏的部分空白，整个tableview内容下滑了20pt。

原因：iOS 11中automaticallyAdjustsScrollViewInsets属性被废弃了，self.automaticallyAdjustsScrollViewInsets = NO 就等于没有设置（默认是YES），于是顶部就多了一定的contentInset。点开系统头文件就会有提示说明

![image.png](http://upload-images.jianshu.io/upload_images/1447375-af9395ec319c1c97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决方法：
```
if (@available(iOS 11.0, *)) {
    self.tableView.contentInsetAdjustmentBehavior = 	UIScrollViewContentInsetAdjustmentNever;
} else {
    self.automaticallyAdjustsScrollViewInsets = NO;
}
```
关于安全区域适配，简书上的这篇文章[iOS 11 安全区域适配](http://www.jianshu.com/p/efbc8619d56b)总结介绍得非常详细，请参考这篇文章。

### iPhoneX
#### 坑NO.2 LaunchImage问题

关于带刘海的iPhoneX，如果你的APP在iPhoneX上运行发现没有充满屏幕，上下有黑色区域，那么你应该也像我一样LaunchImage没有用storyboard而是用的Assets。Assets中解决办法添加一张启动图，尺寸为1125x2436，或者直接启用LaunchScreen.storyboard。
另外在这里提供一个判断iPhoneX机型的宏定义，前提是launchimage已经设置好。

```

#define IS_iPhoneX  UIScreen.mainScreen.currentMode.size.width == 1125 && UIScreen.mainScreen.currentMode.size.height == 2436

```

#### 坑NO.3 storyboard、xib上的安全区问题
本人在做解决第二个坑的时候，创建了storyboard作为启动图使用。按照以往的思路，搞一个imageView覆盖到整个控制器的view上去，拉好约束，一编译
编译发现系统报了个错。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-842058207ad29076.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 原因:
苹果在iOS7中引入的Top Layout Guide和Bottom Layout Guide,这些布局指南在iOS 11中被弃用，取而代之的是Safe Area Layout Guide.

因为项目要求系统最低适配iOS 8，所以要对这里进行处理。
##### 解决方法：
1、打开右侧的 Show the File inspetcor
2、去掉 Use Safe Area Layout Guides
![image.png](http://upload-images.jianshu.io/upload_images/1447375-bca78de3065ddcc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图，这样一来就和之前的界面一样。

目前遇到的就这些坑，欢迎大家指正补充~
最后，作为该行从业人员，不得不讲一句：“苹果爸爸说啥就是啥🙂🙂”



