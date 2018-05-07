---
title: Swift4如何扫描二维码了解一下
---


## 1.扫码简史
这些年移动互联网的普及，也让二维码技术成功的推广。在遥远的iOS7.0之前的年代，我们实现二维码扫描的功能，还需要借助两大开源组件ZXing和ZBar来实现。iOS7.0以后，苹果提供了AVFoundation框架，来实现二维码是扫码，而且效率更高。
与此同时，苹果的Swift开发语言，也经历了从1.0诞生到4.1，其中不乏一些新特性以及API的变化。

本文讲解了如何用Swift4，实现二维码扫描的功能

## 2.具体实现
### 2.1权限控制
实现二维码扫描，必然要打开手机摄像头，就需要获取权限。首先，在你的项目工程的info.plist中加入如下key-value，否则app调试的时候崩溃。
```
<key>NSCameraUsageDescription</key>
<string>CameraUsageDescription</string>
```
另外需要手动去检测当前APP的摄像头权限。如下代码：
```
func checkCameraAuth() -> Bool {
let status = AVCaptureDevice.authorizationStatus(for: .video)
return status == .authorized
}
```
不难看出，status是个枚举值，只有 .authorized才是已经获取摄像头权限，其余的都不行。
### 2.2 上代码
#### 2.2.1 初始化
导入AVFoundation框架之后，我们就可以初始化捕捉设备、创建捕捉会话、输入媒体类型、设置代理等
```
// 捕捉设备
guard let device = AVCaptureDevice.default(for: .video)  else {
return
}
do {
// 输入
inPut: AVCaptureDeviceInput = try AVCaptureDeviceInput.init(device: device)
} catch  {
print(error)
}

/// 输出
let outPut: AVCaptureMetadataOutput = {
let outPut = AVCaptureMetadataOutput.init()
outPut.connection(with: .metadata)
return outPut
}()

/// 会话 session
let session: AVCaptureSession = {
let session = AVCaptureSession.init()
if session.canSetSessionPreset(.high){
session.sessionPreset = .high
}
return session
}()

/// 预览层
let preLayer: AVCaptureVideoPreviewLayer = AVCaptureVideoPreviewLayer.init()
```
#### 2.2.2 设置代理
初始化之后，开始设置代理
```
// 设代理
outPut.setMetadataObjectsDelegate(self as AVCaptureMetadataOutputObjectsDelegate, queue: DispatchQueue.main)
// 指定预览层的捕捉会话
preLayer.session = session
```
#### 2.2.3 指定会话输入输出
然后把捕捉会话添加输入输出
```
// 捕捉会话加入input和output
if session.canAddInput(input) && session.canAddOutput(outPut) {
session.addInput(input)
session.addOutput(outPut)
// 设置元数据处理类型(注意, 一定要将设置元数据处理类型的代码添加到  会话添加输出之后)
outPut.metadataObjectTypes = [.ean13, .ean8, .upce, .code39, .code93, .code128, .code39Mod43, .qr]
}
```
设置元数据处理类型， 可见不仅有二维码，而且还有其他条码，就不一一介绍了。注意, 一定要将设置元数据处理类型的代码添加到会话添加输出之后。

#### 2.2.4 添加会话预览图层
接着开始在页面添加预览层, 这样才能看到摄像头捕捉到的画面。
```
// 添加预览图层
let flag = view.layer.sublayers?.contains(preLayer)
if flag == false || flag == nil {
self.preLayer.frame = view.bounds
view.layer.insertSublayer(preLayer, at: 0)
}
```
#### 2.2.5 开启会话
到此为止，这个session捕捉会话需要的参数都全了，然后开始愉快的开始这个会话
```
// 启动会话
session.startRunning()
```

#### 2.2.6 监听捕捉会话输出代理
开启捕捉会话，我们就可以在代理方法中查看会话捕捉到的东西。

```
func captureOutput(_ captureOutput: AVCaptureOutput!, didOutputMetadataObjects metadataObjects: [Any]!, from connection: AVCaptureConnection!)
```
识别到的就在这个方法里告诉你。
什么？什么？，方法不调用？
**敲黑板！！！API有变化了**
Swift4.0的代理方法在下面
```
func metadataOutput(_ output: AVCaptureMetadataOutput, didOutput metadataObjects: [AVMetadataObject], from connection: AVCaptureConnection){
var resultStrs = [String]()
for obj in metadataObjects {
guard let codeObj = obj as? AVMetadataMachineReadableCodeObject else {
return
} 
resultStrs.append(codeObj.stringValue ?? "")
}
}
```
在这个方法里面就能拿到扫码之后的结果了。
#### 2.2.7 思考
**写到这里，不经停下思考：**
**1.这里还只是基本的扫码功能，就已经这么多代码了，关于扫码页面的长啥样子的代码我还没写；**
**2.一般开发中页面少不了UI的网络的代码，难道我要再把这么一大坨AVFoundation代码都写到控制器吗？**
**3.如果我一个项目里面，不止一个地方用到扫码，难道我还要再把这么多代码再复制几遍**

![WTF.png](https://upload-images.jianshu.io/upload_images/1447375-b4a08ca6b7f7de8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.封装

**高内聚 低耦合**
就按照这个原则来封装。
1.首先把这些AVFoundation模块的代码，统统抽到一个工具类里，需要的时候，直接拿工具类调用，识别结果delegate返回。
2.可以根据经验，把一些定制的需求也放进去，比如说扫码的时候，中间透明的框框，加上周边的黑色蒙板。
3.扩展一些其他功能，比如扫码成功播放一段提示音等待

什么？你准备动手了？别着急，我已经弄好了,使劲戳👇👇
## [HRQRCodeScanTool](https://github.com/CoderHRXu/HRQRCodeScanTool)

最简单的，在控制器中，你只需要
```
// in ViewController
HRQRCodeScanTool.shared.delegate  = self
HRQRCodeScanTool.shared.beginScanInView(view: view)
```
然后扫码结果代理返回

```
// scan result will call in  delegate methods 
func scanQRCodeFaild(error: HRQRCodeTooError){
print(error)
}

func scanQRCodeSuccess(resultStrs: [String]){
print(resultStrs.first)
}
```
如果你需要二维码描边，你只需要设置这几个属性
```
open var isDrawQRCodeRect: Bool    true    是否描绘二维码边框 默认true
open var drawRectColor: UIColor    UIColor.red    二维码边框颜色 默认红色
open var drawRectLineWith: CGFloat    2    二维码边框线宽 默认2
```

如果你需要添加蒙板，你只需要设置这几个属性
```
open var isShowMask: Bool    true    是否展示黑色蒙版板层 默认开启
open var maskColor: UIColor    Black.alpha 0.5    蒙板层 默认黑色 alpha 0.5
open var centerWidth: CGFloat    200    中心非蒙板区域的宽
open var centerHeight: CGFloat    5.0    中心非蒙板区域的宽
open var centerPosition: CGPoint?    nil    中心非蒙板区域的中心点 默认Veiw的中心
```
哪里需要扫码，直接接入工具类，没多少行代码搞定，就问你爽不爽。
另外，项目里还提供了两个扩展，用来识别二维码图片，以及图片生成二维码，需要的各位看官老爷自取。
![yeah.png](https://upload-images.jianshu.io/upload_images/1447375-f471166df94d37f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还支持Cocoapods哦


