---
title: 使用shell脚本自动修改Xcode工程编译版本号
---


## 背景
公司的项目开发中，肯定是需要打包的，目前已经建立起了一套[gitlab](https://about.gitlab.com/)+[fastlane](https://fastlane.tools/)的持续集成方案。但是在项目开发周期的测试阶段以及上线之前，项目打包的频次会升高不少。所以在一个团队开发中，尤其是测试人员如何区分哪次的打包，是一个很常见的问题。

要区分不同的包也很简单，通过build号来区分。Xcode工程的build号可以随便填的，但是如果每次打包都要手动填写肯定也是一个麻烦的事情，所以我们的解决思路肯定是**让工程自动变更build号**。

## 解决方案
·方法1
让fastlane每次打包前，把build号处理。具体详情请戳本人之前的一篇文章
[Fastlane自动打包工具build号自增处理配置方法](http://www.jianshu.com/p/7b3f9cacc2ba)

·方法2
通过给Xcode添加shell脚本实现build号处理，具体方法如下：
1.在工程target中，选择Build Phases点击左上角加号，新建script脚本
![image.png](http://upload-images.jianshu.io/upload_images/1447375-1eb4e82f3885ec87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.在代码区加入以下脚本，让build号为编译的时间。

```
# Update build number with buldTime
buildNumber=$(date +%Y%m%d%H%M%S)
echo "Updating build number to $buildNumber"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}.dSYM/Contents/Info.plist"

```
3.把```Run script only when installing```勾选上，不然debug会报错。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-a96c351bb65039f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加入这段脚本之后，就可以实现build号自动处理，是不是**So Easy**?

## 方法对比
两种方法都能实现对工程build号的修改，究其本质是修改了项目的info.plist。不同之处在于方法1是用fastlane打包工具中的[increment_build_number](https://docs.fastlane.tools/actions/increment_build_number/#increment_build_number)方法来修改，而方法2是直接在Xcode工程里面加入了一段shell脚本，所以不需要借助任何工具，很简单快捷。

个人推荐方法2来实现build自动处理，因为不需要第三方的依赖。文中如有不足之处还望批评指正~

