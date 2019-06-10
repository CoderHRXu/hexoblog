---
title: iOS代码规范
---

iOS Coding Guidelines（代码指南）
===

| 版本号 | 编写人 | 日期 | 
| ------ | ------ | ------ | 
| 2.0 | 抢手的哥 | 2019-06-05 |  

# 一、命名基础

### 驼峰命名
* 针对属性、变量、方法等均采用小写字母开头的驼峰命名准则。
* 针对类、枚举、宏等均采用大写字母开头的驼峰命名准则。

### 清晰性
* 最好是既清晰又简短，但不要为简短而丧失清晰性
* 示例：`removeObject:`
* 反例：`remove`

* 名称通常不缩写，即使名称很长，也要拼写完全
* 示例：`destinationSelection `
* 反例：`destSel `

###### *我们尽可能遵守 Apple 的命名约定，其推荐使用长的，描述性强的方法和变量名，使其阅读起来更加清晰易懂。不能随意使用缩写，导致他人阅读代码困难。*

### 一致性
* 尽可能使用与 Cocoa 编程接口命名保持一致的名称。如果你不太确定某个命名的一致性，请浏览一下头文件。
* 在使用多态方法的类中，命名的一致性非常重要。在不同类中实现相同功能的方法应该具有相同的名称
* 示例：`- (NSInteger)tag`
* 示例：`- (void)setStringValue:(NSString *)`

### 前缀
**前缀是名称的重要组成部分，它们可以区分应用的功能范畴。前缀可以防止与第三方开发者之间的命名冲突**

* 前缀有规定的格式。它由两到三个大写字符组成，不能使用下划线与子前缀
* 命名 class， protocol， struct， 函数，常量时使用前缀；
* 命名成员方法时不使用前缀，因为方法已经在它所在类的命名空间中；
* 同理，命名结构体字段时也不使用前缀

# 二、方法命名【强制】
### 一般性规则
**为方法命名时，请考虑如下一些一般性规则:**

* 小写第一个单词的首字符，大写随后单词的首字符，不使用前缀。有两种例外情况:
1. 方法名以广为人知的大写字母缩略词(如 TIFF or PDF)开头
2. 私有方法可以使用统一的前缀来分组和辨识

* 表示对象行为的方法，名称以动词开头
* 示例：`- (void)invokeWithTarget:(id)target:`
* 示例：`- (void)selectTabViewItem:(NSTableViewItem *)tableViewItem`

* 如果方法返回方法接收者的某个属性，直接用属性名称命名。不要使用 get，除非是间接返回一个或多个值。
* 示例：`- (NSSize)cellSize`
* 反例：`- (NSSize)getCellSize`

* 不要使用 and 来连接用属性作参数关键字
* 示例：`- (NSInteger)runModalForDirectory:(NSString *)path file:(NSString *)name`
* 反例：`- (NSInteger)runModalForDirectory:(NSString *)path andFile:(NSString *)name`

### 方法参数
**命名方法参数时要考虑如下规则:**

* 如同方法名，参数名小写第一个单词的首字符，大写后继单词的首字符。

* 示例：`removeObject:(id)anObject`
* 不要在参数名中使用 pointer 或 ptr，让参数的类型来说明它是指针
* 避免使用 one， two，…，作为参数名
* 避免为节省几个字符而缩写


# 三、实例变量与数据类型命名【强制】
描述性的单词（有意义）+ 变量类型
### 私有变量：
私有变量放在 .m 文件中声明 以 _ 开头，第一个单词首字母小写
例子：`NSString * _somePrivateVariable`
### 属性变量：
正例：`@property (nonatomic, strong) UILabel *nameLabel；`
注意nonatomic和strong之间有一个空格，*号之前有空格，之后没有
反例：`@property(strong,nonatomic)UILabel * lab;`


**如果实例变量别设计为可被访问的，确保编写了访问方法并注意读写权限。**

### 枚举常量
* 使用枚举来定义一组相关的整数常量

```
typedef NS_ENUM(NSInteger, NSMatrixMode) { 
NSRadioModeMatrix       = 0, 
NSHighlightModeMatrix   = 1,
NSListModeMatrix        = 2, 
NSTrackModeMatrix       = 3
};
```

### 其他常量
* 通常不使用 #define 来创建常量。如上面所述，整数常量请使用枚举。
* 使用大写字母来定义预处理编译宏。如:`#ifdef DEBUG`
* 为 notification 名定义大驼峰字符串常量，从而能够利用编译器的拼写检查，减少书写错误。

# 四、图片资源命名
模块名+功能名+控件名+状态
`photoBrowser_btn_selected 
photoBrowser_btn_unselected
`

# 五、可接受的缩略语
**在设计编程接口时，通常名称不要缩写。下面列出的缩写要么是固定下来的要么是过去被广泛使用的，所以可以继续使用。关于缩写有一些额外的注意事项:**

* 标准 C 库中长期使用的缩写形式是可以接受的。如:”alloc”， “getc”
* 你可以在参数名中更自由地使用缩写。如: imageRep, col(column), obj, otherWin

### 常见的缩写
| 缩写 | 含义 | 缩写 | 含义 |
| :------ | :------ | :------ | :------ |
| alloc | Allocate | alt | Alternate |
| app | Application | calc | Calculate |
| dealloc | dealloc | func | Function |
| horiz | Horizontal | info | Information |
| init | Initialize | int | Integer |
| max | Maximum | min | Minimum |
| msg | Message | nib | Interface Builder archive |
| pboard | Pasteboard | rect | Rectangle |
| Rep | Representation | temp | Temporary |
| vert | Vertical |||

### 补充的缩写
| 缩写 | 含义 | 缩写 | 含义 |
| :------ | :------ | :------ | :------ |
| idx | Index | btn | Button |
| proj | Project | pro | Professional |
| pwd | Password | pic | Picture |
| str | String | img | Image |
| VC | ViewController |||

# 五、编码格式

### 星号(*)位置

* 定义一个对象时，*靠近变量

* 示例：`NSString *userName;`

### 统一的ViewController模板（代码缩进、Method进行分组、大括号写法）
* import引用分组划分
* 使用property与局部变量，不使用用全局变量（不能显性区分作用域）
* 空格与换行建议（参照iOS SDK API）
* 方法功能区域性划分

```
#import "GMBaseViewController.h"
/**
HomePage
*/
@interface HomePageViewController : GMBaseViewController
/** 项目Id */
@property (nonatomic, copy) NSString *projectId;
@end
```

```
#import "HomePageViewController.h"
#import "HomePageTableViewCell"     //View
#import "HomePageListRequest"        //Model数据类
#import "ProjectViewController"     //非本页面所用

@interface GMHPViewController ()

@end

@implementation GMHPViewController
#pragma mark - LifeCycle
- (void)viewDidLoad {
[super viewDidLoad];
[self setupView];
[self loadData];
}

#pragma mark - Custom Methods
- (void)setupView {

}
#pragma mark - Event Response


#pragma mark - Data && Net
- (void)loadData {

}

#pragma mark - UITableViewDataSource && UITableViewDelegate

#pragma mark - CustomDelegate

#pragma mark - Getter && Setter

```

### 条件语句
* 条件判断语句执行主体不管是 if 里面还是 else 里面都必须要使用大括号包住, 即使代码是一行的情况（Apple SSL/TLS gotofail）。for 循环同样。
* if-else中，else不另起一行，与if的反括号在同一行，如下段代码所示，注意else前后均有空格。
* switch-case中，建议所有case和default后的代码块均用{}包裹。

```
if（success）{
...
} else {
...
}

for（int i = 0 ; i < 10 ; i++）{
...
}

```

### 单一原则
* 每个函数的职责都应该划分明确（如类一样）

#### 模型解析（MJExtension）
```
+ (NSDictionary *)mj_replacedKeyFromPropertyName {
return @{@"Id" : @"id"};
}

+ (NSDictionary *)mj_objectClassInArray {
return @{@“block” : @"GMBlockModel"};
}
``` 

### 其他

* 子类重写父类方法一般需先调用父类方法

```
- (void)viewDidLoad {
[super viewDidLoad];
...
}
```
* 单模块多页面可新建模块Header类，统一导入共用类，设定常量等。非全局不可定义在`PCH`中


# 六、代码注释
**优秀的代码大部分是可以自描述的，我们完全可以用代码本身来表达它到底在干什么，而不需要注释的辅助。
但并不是说一定不需要写注释，有以下三种情况需要编写注释：**

* 公共接口（注释要告诉阅读代码的人，当前类能实现什么功能，譬如：基础类、工具类）
* 涉及到比较深层专业知识及业务逻辑的代码（注释要体现出实现原理和思想）
* 容易产生歧义的代码（但是严格来说，容易让人产生歧义的代码是不允许存在的）

### import注释
**如果有一个以上的import语句，就对这些语句进行分组，每个分组的注释是可选的。**

### 属性注释
```
/** 注释 */
@property (nonatomic, strong) UIView *headerView;
```
### 枚举注释
```
typedef NS_ENUM(NSInteger, RefreshDateType) {
/** 新建 */
RefreshDateTypeTaskMain    = 1,
/** 编辑 */
RefreshDateTypeEdit,
/** 重派 */
RefreshDateTypeAssign,
/** 新建子任务 */
RefreshDateTypeSubTask,
/** 任务派发（进度管理模块） */
RefreshDateTypeTaskDis,
/** 任务变更（进度管理模块） */
RefreshDateTypeTaskChanged,
};
```

### 方法声明注释

**公开接口，重要的方法，分类，以及协议都应该伴随注释。方法的注释使用Xcode自带注释快捷键:`Commond+option+/`或者`///`，这两种注释方式可被`Xcode`中`Quick Help`识别**

```
/**
<#Description#>

@param tableView <#tableView description#>
@param section <#section description#>
@return <#return value description#>
*/
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section {
...
}

///<#Description#>
- (void)getUserInfo {
...
}
```

<font size=2 color=#4A7EC0>参考文档：</font>[Apple Coding Guidelines for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html#//apple_ref/doc/uid/10000146-SW1)


# 七、其他事项

## 1.【推荐】条件过多、过长时应该换行
```
if  ( condition1 &&
condition2 &&
condition3 ) {
}
```
## 2.【强制】二元运算符与变量之间必须有空格
```
width = 5 + 5; 
height = width * 2;
length = width + height;
for (int i = 0; i < 10; i++)
```
## 3.【强制】NSArray泛型
如果NSArray中存储数据类型是统一的，必须如下使用
```
/** modelArray  */
@property (nonatomic, strong) NSArray <GCBModel *> *modelArray;
```

## 4.【强制】block循环引用问题<特别关注>
```
__weak_self;
[selectPerVC setSingleResultBlock:^(ContactModel *resultModel) {
__strong_self;
}];
```

## 5【强制】枚举使用
项目中两个以上的区分逻辑要使用枚举，使用枚举值赋值的时候不能使用枚举的值（1、2、3… 等）
正例：`self.dataType = RefreshDateTypeTaskMain；`
反例：`self.dataType = 1；`

## 6【强制】加载xib
xib名称用NSStringFromClass(),避免书写错误
正例：`[self.tableView registerNib:[UINib nibWithNibName:NSStringFromClass([DXRecommendTagVCell class]) bundle:nil] forCellReuseIdentifier:ID];`
反例：`[self.tableView registerNib:[UINib nibWithNibName:@"DXRecommendTagVCell" bundle:nil] forCellReuseIdentifier:ID];`
## 7【强制】三目运算符
三目运算符使用时注意?和:空格
正例：`NSString *string = model.isSelected ? @"已选中" : @"未选中";`
**禁止连续使用三目运算符**
反例：`return indexPath.row == 0 ? 109 : indexPath.row == 1 ? 138 : indexPath.row == 10 ? 45 : indexPath.row == 11 ? [PublicPhotoBrowserView getCollecHeiImageCount:self.taskModel.imgsArray.count remainCount:9 - self.taskModel.filesArray.count width: ScreenW - 30 detail:NO]+[PublicPhotoBrowserView publicPhotoGetFilesHeight:self.taskModel.filesArray.count singleHeight:0] : 55 `
