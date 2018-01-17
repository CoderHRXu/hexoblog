---
title: Fastlane自动打包工具build号自增处理配置方法
---

如题所示，本文讲述build号具体的配置方法，也就是说在已经给工程配置好fastlane自动打包工具的前提下。
如果你尚未使用这个工具，可以点击一下几篇文章学习如何使用：
[【CI】持续集成-引导篇](http://www.jianshu.com/p/6e6503b47453)
[【CI】持续集成-第一篇 fastlane](http://www.jianshu.com/p/2d88a88467a5)
下面进入正题
## step1 修改工程配置
修改buildsettings里面的version配置，current project version 随便填一个。versionsystem 选择apple generic。
![t1.jpeg](http://upload-images.jianshu.io/upload_images/1447375-eadcb5ae43006646.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
修改info.plist的路径由绝对路径变为相对路径
![t2.jpg](http://upload-images.jianshu.io/upload_images/1447375-c76535321c288da1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## step2 配置fastfile

推荐用sublime text打开fastfile，编辑器右下角选择Ruby语言，方便编码。
定义专门的函数处理
```
def updateProjectBuildNumber
 
currentTime = Time.new.strftime("%Y%m%d")
build = get_build_number()
if build.include?"#{currentTime}."
# => 为当天版本 计算迭代版本号
lastStr = build[build.length-2..build.length-1]
lastNum = lastStr.to_i
lastNum = lastNum + 1
lastStr = lastNum.to_s
if lastNum < 10
lastStr = lastStr.insert(0,"0")
end
build = "#{currentTime}.#{lastStr}"
else
# => 非当天版本 build 号重置
build = "#{currentTime}.01"
end
puts("*************| 更新build #{build} |*************")
# => 更改项目 build 号
increment_build_number(
build_number: "#{build}"
)
end
定义好updateProjectBuildNumber函数后，在自定义的每个lane方法中，调用一下即可。
eg：
lane :uat do
 
updateProjectBuildNumber  // 这里调用
currentTime = Time.new.strftime("%Y-%m-%d-%H-%M")
ipaName = "UAT-#{currentTime}.ipa"
gym(
scheme: "XXX-UAT",
export_method:"ad-hoc",
archive_path:"./build/uat",
output_directory:"./build/uat",
output_name:ipaName
) # Build your app - more options available
#deliver(force: true)
pgyer(api_key: "xxxxx", user_key: "xxxxx")
# frameit
end
```
配置完了 就可以本地打包自增build号了，当然，可以根据自己项目需求自定义build号的规则，百度一下Ruby语法即可。