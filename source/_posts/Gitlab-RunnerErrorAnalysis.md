---
title: gitlab-runner之build failed with exit status 1问题分析
---

## 前言：
工作需要，原先的一台专门负责打包的机器要被调走，所以另起炉灶，把需要打包的项目迁移到新机器，本篇文章讲述了迁移过程遇到的一些问题以及解决的方法。


## 1.gitlab-runner
首先还是简单说一下什么是gitlab-runner，官网是这么描述的：
>GitLab Runner is the open source project that is used to run your jobs and send the results back to GitLab. It is used in conjunction with [GitLab CI](https://about.gitlab.com/gitlab-ci), the open-source continuous integration service included with GitLab that coordinates the jobs.

也就是说gitlab-runner是配合gitlab-ci进行使用的。一般地，gitlab里面的每一个工程都会定义一个属于这个工程的软件集成脚本，用来自动化地完成一些软件集成工作。
当这个工程的仓库代码发生变动时，比如有人push了代码或者分支合并，gitlab就会将这个变动通知gitlab-ci。这时gitlab-ci会找出与这个工程相关联的runner，并通知这些runner把代码更新到本地并执行预定义好的执行脚本。

##  2.gitlab-runner安装和注册

[安装](https://docs.gitlab.com/runner/install/index.html)
[注册](https://docs.gitlab.com/runner/register/index.html)
这块废话不多说，直接看官方文档，根据系统一次操作一下即可

## 3.问题描述
新机器到手后，把机器上所有的软件环境统统更新了个遍，升级到macOS High Sierra，ruby 升级到2.4等等。然后开始安装gitlab-runner紧接着注册到gitlab-ci，一切的一切都看似很平静很正常。

然后开始试跑脚本,然后莫名其妙的出错了，log如下：
```
Running with gitlab-runner 10.3.0 (5cf5e19a)
  on xxx的MacBook pro (0dee7c95)
Using Shell executor...
Running onxxxdeMacBook-Pro.local...
Cloning repository...
Cloning into '/Users/xxx/builds/0dee7c95/0/iOS-Team/xxx-test'...
Checking out 0766deb4 as develop...
Skipping Git submodules setup
ERROR: Job failed: exit status 1
```
clone之后立马出问题，一脸懵逼...这谁看得懂出的啥错？凉了凉了~
## 4.问题分析

刚开始以为是clone出问题，打开build的目录文件，里面所有文件目录都是完整的，直接打开项目工程都能顺畅的跑起来。
尝试用这些log输出，Google了一下，能搜到好几个Stack Overflow的结果，但是很遗憾，都未能解决问题。

**那么问题到底出在哪？**

首先的换一个思考问题的角度：**gitlab-ci上的log看不出，搜不出什么端倪，那么gitlab-runner呢？毕竟跑脚本是gitlab-runner的事情，它自己最清楚了。** 

于是在打包机器终端里面输入 ```gitlab-runner --debug run``` 这样runner的每一步操作都能看的到。
```
Checking for jobs... nothing                        runner=0dee7c95
Feeding runners to channel                          builds=0
Checking for jobs... received                       job=402 repo_url=http://gitlab.xxxx.cn/iOS-Team/xxx-test.git runner=0dee7c95
Failed to requeue the runner:                       builds=1 runner=0dee7c95
Running with gitlab-runner 10.3.0 (5cf5e19a)
  on xxx的MacBook pro (0dee7c95)  job=402 project=47 runner=0dee7c95
Shell configuration: environment: []
dockercommand:
- sh
- -c
- "if [ -x /usr/local/bin/bash ]; then\n\texec /usr/local/bin/bash --login\nelif [
  -x /usr/bin/bash ]; then\n\texec /usr/bin/bash --login\nelif [ -x /bin/bash ]; then\n\texec
  /bin/bash --login\nelif [ -x /usr/local/bin/sh ]; then\n\texec /usr/local/bin/sh
  --login\nelif [ -x /usr/bin/sh ]; then\n\texec /usr/bin/sh --login\nelif [ -x /bin/sh
  ]; then\n\texec /bin/sh --login\nelse\n\techo shell not found\n\texit 1\nfi\n\n"
command: bash
arguments:
- --login
passfile: false
extension: ""
  job=402 project=47 runner=0dee7c95
Using Shell executor...                             job=402 project=47 runner=0dee7c95
Waiting for signals...                              job=402 project=47 runner=0dee7c95
WARNING: Job failed: exit status 1                  job=402 project=47 runner=0dee7c95
Submitting job to coordinator... ok                 job=402 runner=0dee7c95
Feeding runners to channel                          builds=0
Checking for jobs... nothing                        runner=0dee7c95
```
以上的log是为了复现问题，把相关配置改成问题解决之前，而操作出来的，可能和问题解决之前的log有些出入。根据这些log，再次Google，果然找到了相关信息：
https://gitlab.com/gitlab-org/gitlab-runner/issues/114
真相大白，原来问题出在**RVM**上。

RVM相信开发者都不陌生，它是一个便捷的多版本ruby环境的管理和切换工具。有一个注意点，MacOS本身是集成ruby环境的，不同版本的系统默认集成的ruby版本也不一样。所以日常开发中，我们会根据需要对ruby环境统一管理。这个时候就会用到rvm了。通过rvm可以升级管理ruby。通过 ```which ruby``` 命令可以查看目前使用的是什么ruby。比如 ```/Users/xxx/.rvm/rubies/ruby-2.4.1/bin/ruby``` 输出的路径包含了 ```.rvm``` 就说明目前使用的是rvm下的ruby。

到此为止确定了目前使用的是rvm的ruby，在深入定位一下，问题是出在rvm重新定义了 ```cd``` 命令
```
$ type cd
cd is a function
cd () 
{ 
    __zsh_like_cd cd "$@"
}
$ unset cd
$ type cd
cd is a shell builtin
```
所以，clone下来一直出错的原因是cd命令被重定义了，导致路径切换的不对，无法执行命令，进而报错。这就解释了为什么gitlab-runner上输出的log ```exec /bin/sh --login\nelse\n\techo shell not found\n\texit 1\nfi\n\n```

## 5.问题解决
知道了原因name如何解决呢？刚刚的问题里面也给出了方案
在 ```.bashrc``` 或 ```.bash_profile``` 加上 ```unset cd``` 。
操作如下：
```
1、打开terminal(终端)
2、cd ~ ( 进入当前用户的home目录)
3、open .bash_profile (打开.bash_profile文件，如果文件不存在就  创建文件：touch .bash_profile  编辑文件：open -e bash_profile)
4、直接更改弹出的.bash_profile文件内容
5、command + s 保存文件，然后关闭
6、在terminal(终端)中输入 source .bash_profile (使用刚才更新之后的内容)
```
或者直接finder里面找，用文本编辑器修改一下。

**等等 ```.bashrc``` ```.bash_profile```又是什么鬼？ unset呢？**
百度了一下这些得到如下信息：
[profile\bashrc\bash_profile之间的区别和联系:](http://download.csdn.net/download/ygl_ygl/4354972)
>/etc/profile        每个用户，首次登录时被执行；
>/etc/bashrc       每个运行bash shell的用户都执行此文件，当bsh被打开时，该文件被读取；
>~/.bash_profile 专用于本用户的shell信息，仅被执行一次；
>~/.bashrc          文件包含本用户的bsh信息，登录及每次打开shell时被读取。

unset指令：删除指定的shell变量或函数
【语    法】unset [-fv][变量或函数名称]
【功能介绍】该指令作用主要是删除指定的shell变量和函数。

因为gitlab-runner注册的时候[runner executor](https://docs.gitlab.com/runner/executors/README.html)用的shell，所以shell脚本的语法是适用的。使用```unset cd```就意味着把rvm重新定义的cd方法删除，这样路径切换的时候就正常了。gitlab-runner都能正常跑起来了。

## 6.总结
1.思考问题角度要全面，一套完整的业务流程里出现问题，可以根据不同角度，不同端想错误点靠近，逐个的排查；
2.多关注debug信息，活用debug输出的log搜索信息，尽量谷歌搜索，百度太垃圾；
3.shell脚本是个神奇的东西，工作之余可以多学习学习。