
---
title: 网易云信IM已读回执开发总结
---



## 1.网易云信
网易云信算是国内比较老牌的IM即时通讯服务商了，公司项目里集成的网易云信SDK，关于这个SDK就不多说了，官网上的介绍比较详细。

项目里接入了SDK，然后UI界面是使用的网易提供的一套UI框架，叫`NIMKit.h` 在此基础上进行开发。

## 2.IM已读回执
IM已读回执，整个的过程简单说来就是：
```
1.聊天消息的发送方，发出的一条带开通回执功能的消息（可以是文字，图片等等，语音一般都自带回执）；
2.聊天接收方查看了（前提是收到了该消息）了这条消息，要告诉发送方我我已经查看了，此时发一条该消息已读的指令给发送方；
3.聊天发送方，收到接收方，发来的消息已读回执。这个时候，就要更新本地数据，在UI界面上把那条聊天信息，显示为已读；
```
群聊也是如此，在群聊中发出一条消息，通过回执功能可以看出谁已读或者未读。
如果你用过钉钉，就很好理解这个流程。

## 3.前期准备
既然是迭代开发，没玩过这个SDK肯定是要查看开发文档的。
[NIMKit文档](https://github.com/netease-im/NIM_iOS_UIKit/blob/master/Documents/nim_custom_ui.md)
[单聊文档](https://dev.yunxin.163.com/docs/product/IM%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF/SDK%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/iOS%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/%E6%B6%88%E6%81%AF%E6%94%B6%E5%8F%91?#%E5%B7%B2%E8%AF%BB%E5%9B%9E%E6%89%A7)
[群聊文档](https://dev.yunxin.163.com/docs/product/IM%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF/SDK%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/iOS%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/%E7%BE%A4%E7%BB%84%E5%8A%9F%E8%83%BD?#%E7%BE%A4%E7%BB%84%E5%B7%B2%E8%AF%BB%E5%9B%9E%E6%89%A7)


**敲黑板，划重点，要考！！**
**网易云信SDK，已读回执功能，需要单独开通服务，而且是付费服务**
也就是说，代码写的再好，不花钱开通也没吊用。
![卧槽.png](https://upload-images.jianshu.io/upload_images/1447375-5d168f1a9ad04095.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

讲到这里，其实最最首要的任务是：**充钱，充钱，充钱！**

## 4.拿起键盘就是干
![就是干](https://upload-images.jianshu.io/upload_images/1447375-7e42bf91528876c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/613/format/webp)

### 4.1 聊天Cell定制
如下图，NIMKit里面已经封装好了一套聊天Cell。根据业务需要自行调整。
![cell结构](https://upload-images.jianshu.io/upload_images/1447375-70ef1b7523d72146.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
比如，它的readLabel是放在最左侧，我们需求是放在底部位置。所以自行重定义布局，很简单代码就不贴了。

当然NIMKitConfig里面你要自定义好readLabel的字体大小颜色，内容等等。
NIMMessageModel是每一条聊天消息的model实体，readlabel的隐藏显示是由shouldShowReadLabel这个属性来驱动。

NIMSessionTableAdapter这个类用来负责聊天页面的tableview相关，因为需求是将已读放到了消息底部，那必然要对cell的高度进行调整。
```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
CGFloat cellHeight = 0;
id modelInArray = [[self.interactor items] objectAtIndex:indexPath.row];
if ([modelInArray isKindOfClass:[NIMMessageModel class]]) {
NIMMessageModel *model          = (NIMMessageModel *)modelInArray;
CGSize size                     = [model contentSize:tableView.nim_width];
UIEdgeInsets contentViewInsets  = model.contentViewInsets;
UIEdgeInsets bubbleViewInsets   = model.bubbleViewInsets;
CGFloat unreadLabelHeight       = [model.message.from isEqualToString:[NIMSDK sharedSDK].loginManager.currentAccount] ? 20 : 0;// 高度适配
cellHeight                      = size.height + contentViewInsets.top + contentViewInsets.bottom + bubbleViewInsets.top + bubbleViewInsets.bottom + unreadLabelHeight;
}
....
}
```


### 4.2 P2P聊天
P2P就是点对点聊天，就是单聊。

在会话界面中调用发送已读回执的接口并传入最后一条消息，即表示这之前的消息都已读，对端将收到此回执。
发送已读回执
```
@protocol NIMChatManager <NSObject>
- (void)sendMessageReceipt:(NIMMessageReceipt *)receipt
completion:(NIMSendMessageReceiptBlock)completion;
@end
```
接受已读回执
```
@protocol NIMChatManagerDelegate <NSObject>
- (void)onRecvMessageReceipts:(NSArray<NIMMessageReceipt *> *)receipts;
@end
```
文档里说了这些，所有只要在生命周期方法里，把最后一条消息捞出来，调一下sendMessageReceipt方法即可。
如果发送了回执，对方的onRecvMessageReceipts就会被调用。网易牛逼的地方就是，收到回执的时候，SDK内部已经把数据都处理好了，上层只需要把UI刷新一下即可。 
**什么？你问我怎么刷新？？**
在onRecvMessageReceipts里`[self.tableView reloadData];`
爽不爽？

### 4.3 群聊发送已读回执
群聊和单聊方法基本差不多，但是要改一个属性，

发送需要标记已读回执的群组消息标记
```
/**
*  其他群成员收到此消息是否需要发送已读回执
*  @discussion 默认为NO，设置成 YES 之后所有群回执相关操作才会生效
*/
@property (nonatomic,assign) BOOL teamReceiptEnabled;
```
一句话搞定   
`[NIMSDKConfig sharedConfig].teamReceiptEnabled = YES;`


消息接收方收到消息，并回复回执 在会话界面中调用发送已读回执的接口并传入最后一条消息，即表示这之前的消息都已读，对端将收到此回执。
```
@protocol NIMChatManager <NSObject>
- (void)sendMessageReceipt:(NIMMessageReceipt *)receipt
completion:(NIMSendMessageReceiptBlock)completion;
@end
```
到这里就遇到坑了，属性都配置好了，发回执就是不行。后来咨询了网易的技术支持，他丢给了我一个链接[https://faq.yunxin.163.com/kb/main/#/item/KB0335](https://faq.yunxin.163.com/kb/main/#/item/KB0335)。
**原来网易还有知识宝库这玩意**
![image.png](https://upload-images.jianshu.io/upload_images/1447375-09bac5e00a497e92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原来不只是要将NIMSDKConfig的teamReceiptEnabled要改成YES，NIMMessage中的setting中的teamReceiptEnabled也要改成YES，这个文档里居然没写。要不是技术支持发给我谁知道有这玩意？？？
```
#pragma mark - NIMChatManagerDelegate
- (void)willSendMessage:(NIMMessage *)message {

id<NIMSessionInteractor> interactor = self.interactor;
if (message.session.sessionType == NIMSessionTypeTeam) {
message.setting.teamReceiptEnabled = YES; // 设置里打开才行。
}
....
}
```
这样之后才能正确发送已读回执。

发送已读回执之后，对方的那条消息，就能看到
`model.message.teamReceiptInfo.unreadCount`就能再UI上显示多少人未读。

### 4.4 群聊查看消息已读未读列表
SDK里面已提供了查询消息已读id列表接口
查询详情接口
包括已读人数的 id 和 未读人数的 id 列表。需要注意查询详情对象不会跟着回执人数变化而变化，如果要获取最新的详情，必须再次调用此接口。
```
@protocol NIMChatManagerDelegate <NSObject>

/**
*  查询群组消息回执详情
*
*  @param NIMMessage    要查询的消息
*  @discussion          详情包括已读人数的 id 列表和未读人数的 id 列表
*                       查询详情对象不会跟着回执人数变化而变化，如果要获取最新的详情，必须再次调用此接口
*
*/
- (void)queryMessageReceiptDetail:(NIMMessage *)message
completion:(NIMQueryReceiptDetailBlock)completion;

@end
```
参数正确之后，会返回这条消息的已读id列表和未读id列表，列表里面保存的是userid。

这里也遇到一个坑：
**在聊天界面，当前新发送的消息，能够查询到已读未读列表。但是打开页面之前的历史消息，无论如何都查不到。**
经过debug发现，当退出聊天窗口，所有的消息变成历史数据之后，才进入聊天窗口，所有的历史消息`message.setting.teamReceiptEnabled`统统变成了NO，解决方法也很简单，每次`queryMessageReceiptDetail`查询之前，再把这个属性设置成YES就能正确查询列表了。

**注意点：网易云信最多支持100人群聊的已读消息回执** 
超过100也就不支持了，如果不能满足需求，要么有本事自己写一套IM，要么就换一家SDK吧。

## 5.结束语
虽然是第一次接触，总体觉得网易这整个一套IM的SDK也不是很难，嵌套的比较多，需要认真梳理各个类各个协议，所承载的功能。其次，开发难免会遇到一些坑和弯路，写这篇文章也算是为了帮后来人吧。

如有错误，还请各位看官批评指正~

