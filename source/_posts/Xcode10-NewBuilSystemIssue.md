---
title: 踩坑Xcode 10之New Build System
---


Xcode10 Version 10.0 (10A255)发布，笔者第一时间升级。很显然使用的过程中出现了不少问题，相信大家都有所耳闻，最典型的就是[libstdc++.6.0.9问题](http://www.cnblogs.com/chao8888/p/9674314.html)，网上已经有不少方法了，这里就不多说了。

## 遇到的问题
除了这个问题之外，笔者还遇到了build system的问题。在Xcode菜单栏选择File-- Workspace Setting就会出现如下的界面

![build system.png](https://upload-images.jianshu.io/upload_images/1447375-50b354ddfb81afbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出Xcode10是默认选中的最新的New Build System(Default)，在这个编译系统的环境下，我打包的CI脚本一直会报错

```
/bin/sh -c /Users/apple/Library/Developer/Xcode/DerivedData/XXX-bcebrkpahigmkngytwmzustybhev/Build/Intermediates.noindex/ArchiveIntermediates/XXX-SIT/IntermediateBuildFilesPath/XXX.build/Release-SIT-iphoneos/XXX.build/Script-187F7F6921537B0600C427DD.sh
[14:59:47]: ▸ Updating build number to 20180920145946
[14:59:47]: ▸ Set: Entry, ":CFBundleVersion", Does Not Exist
[14:59:47]: ▸ File Doesn't Exist, Will Create: /Users/apple/Library/Developer/Xcode/DerivedData/XXX-bcebrkpahigmkngytwmzustybhev/Build/Intermediates.noindex/ArchiveIntermediates/XXX-SIT/BuildProductsPath/Release-SIT-iphoneos/XXX.app.dSYM/Contents/Info.plist
[14:59:47]: ▸ Command PhaseScriptExecution failed with a nonzero exit code
```

问题是出在了更改项目工程buildnumber的shell脚本里：

```
# Update build number with buldTime
buildNumber=$(date +%Y%m%d%H%M%S)
echo "Updating build number to $buildNumber"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}.dSYM/Contents/Info.plist"
```

这段脚本会在最后会把xxx.app.dSYM中的info.plist中的CFBundleVersion值修改成当前时间，而报错说，这个info.plist不存在，创建的时候出问题报错。

**这会不会是苹果的bug??**

## 解决方法
把build system切换到
Legacy Build System，换言之就是切换成老的编译系统，就OK了。


