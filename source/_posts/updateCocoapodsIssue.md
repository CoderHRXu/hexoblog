
---
title: 更新cocoapods 遇到的坑
---

### 现象：
最近在项目，由于打包的时候报警，与其他同事电脑保持cocoapods版本号一致（想要更新到最新的1.3.1，目前1.2.0），于是在终端开始执行一下命令：
```
  sudo gem install cocoapods
  pod --version
```
![更新.png](http://upload-images.jianshu.io/upload_images/1447375-95764f2528cfd0ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
过程看着都很正常，但是在敲一下命令看版本，发现还是原先1.2.0版本。瞬间傻了眼，不起作用~
在终端里面继续敲
```
which pod 
```
查看一下当前pod的路径，竟然发现刚刚安装pod的路径和pod运行的路径不一样：
![路径1.png](http://upload-images.jianshu.io/upload_images/1447375-fdf83dec54075b87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![路径2.png](http://upload-images.jianshu.io/upload_images/1447375-3b4ffb6619f2c13a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两个文件夹下面都有pod（上图是被我删掉pod之后的截图），所以理所当然的想把刚刚更新的pod复制一份到which pod指定的目录下，结果发现也是不起作用。
后续我一次又一次的指定pod的安装目录，安装，卸载pod都不管用，安装完版本号始终为1.2.0（┑(￣Д ￣)┍）。
### 解决方案：
无奈，只好另寻他法，既然我无法指定，那能否直接全部删除呢，全部重来？
后面尝试着在终端敲移除命令
```
  sudo gem uninstall cocoapods
```
这个时候居然发现我电脑里面有好几个版本的cocoapods，看到第6个选项全部版本，果断选了6。
![全部移除.png](http://upload-images.jianshu.io/upload_images/1447375-9099cc7fe9e7507c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
完毕之后这个时候我电脑的就没有任何cocoapods版本了（很棒棒）。
然后重新执行安装命令，终于可以了。pod安装的目录，和现执行的pod路径为同一个了。
![重新安装.png](http://upload-images.jianshu.io/upload_images/1447375-fa7c51850b1b41ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 原因&总结：

为何会出现不同的路径？cocoapods是用的ruby语言写的一个工具。MacOS当中本身就集成了Ruby，所以路径不一样的原因，是因为电脑里面有一个自带的Ruby路径，还有一个就是通过Rvm管理的Ruby。可以在在终端里面输入```which ruby```来查看使用的是什么ruby。  

那么如何切换ruby呢？
```
rvm use system # 使用系统 ruby 
rvm use 2.3  # 使用 rvm ruby
```
在切换 ruby 版本之后，gem 也会跟着切换，我们就可以安装两个版本的 CocoaPods 了。


