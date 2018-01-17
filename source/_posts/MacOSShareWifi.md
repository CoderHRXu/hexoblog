---
title: 关于Mac共享WiFi网络，看这里就够了(2.4G，5G ， ipv6)
---

### 前言：

WiFi这个词大家都不陌生，前几年移动互联网的飞速发展，如今家家户户都有无线路由器，都有自家的WiFi。但是总有一些场景，我们需要WiFi网络来给我们的移动设备使用，但是现场有没有相关的硬件设备，这就催生了一些WiFi使用的需求。在windows系统上，已经有不少成熟的产品，可以快速创建专属的WiFi网络。例如小米随身WiFi，本身就是一个无线网卡，作为AP分享。笔记本电脑一般都自带无线网卡，一些软件也能利用笔记本的无线网卡做热点分享。那么，在Mac平台上我们如何创建自己的专属WiFi呢？


### So Easy
苹果电脑在已经连接了有线网络的情况下（前提条件），无需下载安装其他WiFi共享软件即可实现WiFi共享，但如果连接的是无线网络时则不能共享WiFi，即连着WiFi时不能共享WiFi。
1.点击左上角的黑苹果🍎，选择【系统偏好设置】

![step1.png](http://upload-images.jianshu.io/upload_images/1447375-cb62ad9287cef03f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.单击【共享】

![step2.png](http://upload-images.jianshu.io/upload_images/1447375-4cc0258b11d3e344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.检测一下你电脑的无线网卡有没有开启，如果没开启，点击系统右上角的无线网icon，打开

![step3.png](http://upload-images.jianshu.io/upload_images/1447375-5eeb6cd66e296020.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.上面的框框，意思是你要把哪个网络共享，我的电脑是自带RJ45网口的，网络环境直接插网线就可以上网，所以我这里选择的是【以太网】，如果是新款的MacBook不带网口的，而是接了USB网卡，这里的来源就要选择【USB网卡设备】。但是还有一种情况，不管你电脑带不带网卡，只要是你要在电脑进行PPPoE上网的（俗称拨号上网，家庭网络这样的认证方式最多），这里的来源就要选择【PPPoE网络】，这里请根据实际情况。

![image.png](http://upload-images.jianshu.io/upload_images/1447375-b0d8159d71ebb35a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面的框框，意思就是把网络通过什么端口共享出去，很明显这里选择WiFi，就是通过无线的方式分享出去。

5.然后点击【WiFi选项】，弹出一个页面，填写你要创建的WiFi名称，以及安全性，自己创建的WiFi当然想自己用啦，选择【WPA2个人级】的就好，填写WiFi密码 点击好就OK了。

![image.png](http://upload-images.jianshu.io/upload_images/1447375-d6dfa21f09370eb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6.也许第5步你们发现了一个问题，关于频段，我没有具体说明。点开频段之后，弹出了一个选择界面，有很多频段，我们该如何选择？而且11和36之间，怎么会多了一根分割线？

![image.png](http://upload-images.jianshu.io/upload_images/1447375-12a2f1fc97ace1ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里就要补充一些关于WiFi的一些基本知识了。Wi-Fi是一种允许电子设备连接到一个无线局域网（WLAN）的技术,通常使用2.4G UHF或5G SHF ISM 射频频段。 频段是固定，那么所以的设备都在这个频段使用，就会容易造成干扰，就好比马路宽度是一定的，大家都在一条道上来回走，就会碰撞。所以又划分出了信道，就好比把马路划分了很多车道，不同速度等级的各行其道互不干扰。不同国家划分标准不一致，以中国为例，运行在2.4G的WiFi网络，信道划分如下图。

![image.png](http://upload-images.jianshu.io/upload_images/1447375-3d70320d3a2d08a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    1.IEEE 802.11b/g标准工作在2.4G频段，频率范围为2.400—2.4835GHz，共83.5M带宽
    2.划分为14个子信道
    3.每个子信道宽度为22MHz
    4.相邻信道的中心频点间隔5MHz 
    5.相邻的多个信道存在频率重叠(如1信道与2、3、4、5信道有频率重叠)
    6.整个频段内只有3个（1、6、11）互不干扰信道

如果创建2.4G的WiFi的话，选择的频道的时候，在1、6、11里面选择一种就行了。到这里就可以理解那根分割线是啥意思了，分割线以上的是2.4G的信道，分割线一下就是5G的信道，那么5G的信道如何选择？

很简单，随便选！（😜嘿嘿嘿）因为现阶段5G用的还是不多，不会有啥干扰。
如果你的电脑，点开这个选项，发现只有1到11的信道选择，说明你的电脑的无线网卡，硬件上不支持5GHz的WiFi，那就木有办法咯。

7.回到我们的话题来，选择好你要的频段，设置好之后，点击勾选【互联网共享】，此时会弹出一个框框，点击【启动】就好啦！

![image.png](http://upload-images.jianshu.io/upload_images/1447375-57bd55c1eb869ee3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功之后，你会发现，WiFi的小图标变成了这个样子。

![image.png](http://upload-images.jianshu.io/upload_images/1447375-7cc166273d8a3b16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
到这里就大功告成啦。

## Wait 说好的IPV6呢？
先要知道一个概念，目前我们绝大多数使用的网络，都是IPV4的，IP地址有四段0~255的数字组成。 由于[IPv4](http://baike.baidu.com/item/IPv4)最大的问题在于网络地址资源有限，严重制约了互联网的应用和发展。IPV6应运而生，IPv6的使用，不仅能解决网络地址资源数量的问题，而且也解决了多种接入设备连入互联网的障碍。IPv6的地址长度为128b，是IPv4地址长度的4倍。于是IPv4点分十进制格式不再适用，采用十六进制表示。简单讲，ipv6能分配的地址非常多，绝对够用。
那么我们如何创建一个ipv6的网络呢？

### 还是So Easy
1.点击左上角的黑苹果，选择【系统偏好设置】，敲黑板！！按住'Option'按键的同时点选“共享”图标，注意：不要放开'Option'按键，一直按住不放哦；这样操作之后，你会发现下面多了一个选项【创建NAT64网络】，勾选它就OK啦。

![image.png](http://upload-images.jianshu.io/upload_images/1447375-140c0ef7198e6bc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 那么问题来了，NAT64是啥？
这里我找了一张图

![image.png](http://upload-images.jianshu.io/upload_images/1447375-506743972838ede3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 NAT64是一种有状态的网络地址与协议转换技术，一般只支持通过IPv6网络侧用户发起连接访问IPv4侧网络资源。但NAT64也支持通过手工配置静态映射关系，实现IPv4网络主动发起连接访问IPv6网络。NAT64可实现TCP、UDP、ICMP协议下的IPv6与IPv4网络地址和协议转换。
 DNS64则主要是配合NAT64工作，主要是将DNS查询信息中的A记录（IPv4地址）合成到AAAA记录（IPv6地址）中，返回合成的AAAA记录用户给IPv6侧用户。
 NAT64一般与DNS64协同工作，而不需要在IPv6客户端或IPv4服务器端做任何修改。

一句话，可以简单理解为中转机构。
当然了，ipv6目前对大众使用没啥用户需求，但是对于行业相关技术人员，有需要使用ipv6的环境做一些相关的测试。


### 最后
到此为止，Mac共享WiFi网络的方法其实很简单，一学就会，掌握这个技巧，可以很高效的运用到你的工作或者生活中，快去试试吧。






