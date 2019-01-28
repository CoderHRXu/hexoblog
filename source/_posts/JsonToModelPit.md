---
title: iOS项目中Json转Model的坑
---


## Json转Model
json转model，是个开发都会遇到过。都已经9102年了，谁还不会用个第三方框架搞。拿起键盘就是干！![就是干.png](http://upload-images.jianshu.io/upload_images/1447375-7e42bf91528876c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打开podfile，把大名顶顶的YYModel写上，pod install一下。再用上ESJsonFormat，直接根据json，都能把model生成好。

## 特殊处理
**啥?返回的字段值不是我们所需的**
在日常开发中，经常会遇到一些接口字段返回的值，并不是我所需要的类型的情况，这个时候，我们都会对这个字段进行处理。
举个栗子：
```
/** 错误代码 */
@property (nonatomic, assign) NSInteger error_code;
/** 错误消息 */
@property (nonatomic, copy) NSString *error_msg;
/** 是否成功 */
@property (nonatomic, assign) BOOL isSuccess;
```
接口的json中的error_code字段，接口会用这个字段告诉我这次请求是否成功。比方说成功的error_code是1，平时我们为了方便开发，会在model里自己加一个自定义的属性isSuccess，来表示本次网络请求回来之后的结果是否成功。通常的做法，要么重写error_code的set方法，在set的时候，做一次error_code==1的判断，将判断的结果，赋值给isSuccess，要么就重写isSuccess的get方法，get的时候，返回error_code==1的结果。
相信这些对于老司机们而言，都属于常规操作了。那我们来看看坑在什么地方？
## 入坑
我们来看这个案例：
接口返回了4个字段值，每个字段都用得到，所以新建一个model类来解析。
```
@interface ExamSubjectVo : NSObject

/**  考试学科ID */
@property (nonatomic, assign) NSInteger examSubjectValue;
/**  考试学科名称 */
@property (nonatomic, strong) NSString *examSubjectName;
/** 学科分数  */
@property (nonatomic, strong) NSString *subjectScore;
/** 基础学科Id  */
@property (nonatomic, assign) NSInteger subjectBaseId;
@end
```
但是由于有业务需求，且为了方便开发过程区分，需要对考试名称的字段examSubjectName为全科或者语数外的情况，要特殊处理。所以，按照一贯的思维，我们要重写set方法
```
- (void)setExamSubjectName:(NSString *)examSubjectName{
_examSubjectName = examSubjectName;
if ([examSubjectName isEqualToString:@"全科"]) {
self.subjectBaseId = -100;
}
if ([examSubjectName isEqualToString:@"语数外"]) {
self.subjectBaseId = -200;
}
}
```
乍一看，也没什么问题，解析的过程中，把字段的值转化为我们需要的。**而且真机实测的时候，所有的测试机都没问题，除了一台iPhone5之外**
就除了一台iPhone5，debug的时候看到set方法确实也走了，可是最终的subjectBaseId并没有转化成-100或者-200，可见subjectBaseId又被json本身的值覆盖了，也就是说 **set方法的执行顺序，在不同CPU架构设备上存在差异。**

## 出坑
那么如何解决问题呢？
正是因为存在这样的差异，所以我们只能在model所有的字段全部set完毕之后，再做一些特殊的字段处理，那么如何来处理呢？
翻阅YYModel源码，肯定能有所发现，果不其然，有所收获。
```
/**
If the default json-to-model transform does not fit to your model object, implement
this method to do additional process. You can also use this method to validate the 
model's properties.

@discussion If the model implements this method, it will be called at the end of
`+modelWithJSON:`, `+modelWithDictionary:`, `-modelSetWithJSON:` and `-modelSetWithDictionary:`.
If this method returns NO, the transform process will ignore this model.

@param dic  The json/kv dictionary.

@return Returns YES if the model is valid, or NO to ignore this model.
*/
- (BOOL)modelCustomTransformFromDictionary:(NSDictionary *)dic;
```
YYModel提供了这么个方法，它会在`+modelWithJSON:`, `+modelWithDictionary:`, `-modelSetWithJSON:` and `-modelSetWithDictionary:`方法结束的时候调用。
所以我们对model特殊字段的处理，都应该放到这个方法去执行
```
- (BOOL)modelCustomTransformFromDictionary:(NSDictionary *)dic{
if ([self.examSubjectName isEqualToString:@"全科"]) {
self.subjectBaseId = -100;
}
if ([self.examSubjectName isEqualToString:@"语数外"]) {
self.subjectBaseId = -200;
}
return YES;
}
```
这么一来，问题就解决了。
注意，YYModel还有一个``` - (NSDictionary *)modelCustomWillTransformFromDictionary:(NSDictionary *)dic;```
这个方法很类似，但是执行的时机不一样，这个方法是在model转化之前执行，虽不符合本案例的需求，但是很有可能在其他类似的情况能用的上。


