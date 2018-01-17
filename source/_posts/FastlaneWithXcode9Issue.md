---
title: 当Fastlane遇到Xcode9打包出来不一定是ipa而是坑
---

## Fastlane简介
[Fastlane](https://fastlane.tools/)  是用Ruby语言编写的一套自动化工具集和框架，每一个工具实际都对应一个Ruby脚本，用来执行某一个特定的任务。Fastlane的强大之处，就是可以将不同的工具（[action](https://docs.fastlane.tools/actions/)）有机而灵活的结合在一起，从而形成一个完整的自动化流程，大大提高了日常的开发测试效率，推荐大家使用。
如果你尚未使用这个工具，可以点击一下几篇文章学习如何使用：
[【CI】持续集成-引导篇](http://www.jianshu.com/p/6e6503b47453)
[【CI】持续集成-第一篇 fastlane](http://www.jianshu.com/p/2d88a88467a5)

## 背景&现象
公司项目日常开发中，已经使用了fastlane，配合gitlab一起实现自动打包以及分发，使用期间感觉很顺畅，没有出过什么问题。但是，自从Xcode升级到Xcode9正式版之后打包一直出问题，打不了包。

##### 环境配置

fastlane版本： 2.59.0
```
| Key                         | Value                                       |
| --------------------------- | ------------------------------------------- |
| OS                          | 10.13                                       |
| Ruby                        | 2.3.3                                       |
| Bundler?                    | false                                       |
| Git                         | git version 2.13.5 (Apple Git-94)           |
| Installation Source         | ~/.rvm/gems/ruby-2.3.3/bin/fastlane         |
| Host                        | Mac OS X 10.13 (17A365)                     |
| Ruby Lib Dir                | ~/.rvm/rubies/ruby-2.3.3/lib                |
| OpenSSL Version             | OpenSSL 1.0.2k  26 Jan 2017                 |
| Is contained                | false                                       |
| Is homebrew                 | false                                       |
| Is installed via Fabric.app | false                                       |
| Xcode Path                  | /Applications/Xcode.app/Contents/Developer/ |
| Xcode Version               | 9.0                                         |

*generated on:* **2017-09-30**
```
工程项目中 provisioning profile（以下简称pp文件)，灵活运用了match这个action外加手动配置。

![工程配置.png](http://upload-images.jianshu.io/upload_images/1447375-1697cd95bad7cd6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 错误描述：
![error.png](http://upload-images.jianshu.io/upload_images/1447375-d1045b328b601048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看到了框框的那句话，刚开始以为是推送证书或者是provisioning profile的问题，然后手动在此执行了match，把证书什么的里里外外更新了一遍，发现还是不起作用。没办法，只能翻看github 看看issue，看看有木有类似的处理方案。

##### 原因
功夫不负有心人，喵神大佬道破真相：
[Xcode9将不会允许你访问钥匙串里的内容，除非设置allowProvisioningUpdates。](https://github.com/fastlane/fastlane/issues/9589)

![社会我喵神.png](http://upload-images.jianshu.io/upload_images/1447375-79ee4612a68db052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所以说，在执行xcode -exportArchive的时候，因为权限访问钥匙串，所以无法读取到项目工程里的pp文件，进而打包失败，并且报错说缺少pp文件。

所以解决方法，在你的fastfile里gym action加入 ```allowProvisioningUpdates```

```
gym(
	scheme: scheme,
	export_method: "ad-hoc",
	silent:true,
	output_directory:outputDirectory,
	output_name:ipaName,
	archive_path:outputDirectory,
	export_xcargs: "-allowProvisioningUpdates",
)
```
保存，再执行一次打包命令，请注意，这个时候xcode会弹窗让你确认，点```一直允许```就是了。特别注意的是，如果和我们公司一样的环境，那么那台远程打包的服务器，在更新完fastfile第一次跑的时候，要远程连接上去手动点击确认框，不然打包脚本就会一直卡在那里。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-5dd59d3cb0fb6fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到此为止，Xcode9读取钥匙串权限的问题就结束了。
## 但是，你以为这样就结束了吗？就能打包成功了吗？

紧接着又是另外一个错误：
![不匹配.png](http://upload-images.jianshu.io/upload_images/1447375-3aeed0119a88534a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
WTF，明明打包方式是ad-hoc，但是匹配的pp文件确是AppStore的？😓😓
## 这必然是个坑
在此打开了[官方文档](https://docs.fastlane.tools/codesigning/xcode-project/)看到了这么一句话
![image.png](http://upload-images.jianshu.io/upload_images/1447375-bf3da1d9b383afb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
翻译一下，如果你们没用match这个action，来管理pp文件，我们推荐你在fastfile里面，自己定义一个map，指定bundleID 和 pp文件，以保证能够正常构建app。但是我在文章开头就po图了，我项目工程里的pp文件都是用match来管理的。所以刚开始，这段话被我忽略了。后续又是一阵在github issue上漫游。https://github.com/fastlane/fastlane/issues/10315 找到一个类似的问题。
##### 既然bundleID，或者pp文件不对，那我能不能尝试着主动写死正确的bundleID和pp文件名，放进fastfile？
于是把gym加了个参数，不同环境下指定好：
```
        gym(
			scheme: scheme,
			export_method: "ad-hoc",
			silent:true,
			output_directory:outputDirectory,
			output_name:ipaName,
			archive_path:outputDirectory,
			export_xcargs: "-allowProvisioningUpdates",
			export_options: {
				provisioningProfiles: {
					app_identifier => "match AdHoc #{app_identifier}"
				}
			}
		)
```
保存，重新run打包脚本，终于可以成功打包了😁😄。
![oh yeah.png](http://upload-images.jianshu.io/upload_images/1447375-12a45d828532d107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 简单说说Xcode8和9打包
笔者由于fastlane问题没解决，所以这期间打包工作都是手动完成的，``` Product - Archive ```然后等待Xcode自动打，这些操作相信大家都会的，不多说了。选择测试包之后，弹出框框问你是否瘦身，一般别管就是。然后重点来了，相比Xcode8多了这么个页面：
![image.png](http://upload-images.jianshu.io/upload_images/1447375-a1ad5f89db8449ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择pp文件，从网络获取，Xcode必须要登录开发者账号，从账号里面拉取相关pp文件，还要自己选择是哪个。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-e0b6232d81dae8eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
平时的测试包选择“ad-hoc”就是，然后接着生产ipa。在Xcode8里，最终生成的一个ipa文件夹，里面就一个ipa，但是xcode9除了ipa之外，还有一些其他的东西，如下图。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-1d881fb388b187c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
除了ipa之外，有个东西引起了我的注意```ExportOptions.plist ``` 预览了一下这个plist，里面就记录了刚刚上几步的一些选择项。比如瘦身选项，还有个很关键的东西``` provisioningProfile```这个字段是个dictionary。
![image.png](http://upload-images.jianshu.io/upload_images/1447375-df5bc2f32e68d04a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ### 难道没有一种似曾相识的赶脚？
再次看看这张图：
![error.png](http://upload-images.jianshu.io/upload_images/1447375-d1045b328b601048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
红框框的部分的最后，说的plist就是它。这个就是Xcode在导出ipa的时候一些参数。另外，手动Xcode打包和在终端里面敲 xcodebuild -archivePath，xcodebuild -exportArchive -exportOptionsPlist 本质是一样的，都是打包，参数就是同一个。
再回过头来，看看我们最终的解决方案，我们在gym中添加了一个字段，就是在写参数而已
``` 
export_options: {
				provisioningProfiles: {
					app_identifier => "match AdHoc #{app_identifier}"
				}
			} 
```
### 可能原因
到这一步，在回头看看以上出现的bundleID和PP文件不匹配错误，在用match管理pp文件的前提下，有可能是因为Xcode9引起的问题，打包流程或者参数读写哪里有点变化，需要fastlane团队做进一步适配更新。上文中issue的链接状态到截止笔者写作完成之时，任是在open状态。所以这个原因有待关注，当然也希望能有大佬在指出原因，感激不尽~

至此为止，踩过的坑全部都填平了。不光是fastlane，用其他的一些工具或者第三方的时候还是要多多关注github上的issue，这里是块大大宝藏，等待被你挖掘！