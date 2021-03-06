## 通知中心(NSNotificationCenter)
* 每一个应用程序都有一个通知中心(NSNotificationCenter)实例，专门负责协助不同对象之间的消息通信
* 任何一个对象都可以向通知中心发布通知(NSNotification)，描述自己在做什么。其他感兴趣的对象(Observer)可以申请在某个特定通知发布时(或在某个特定的对象发布通知时)收到这个通知

##通知(NSNotification)
* 一个完整的通知一般包含3个属性：
  - (NSString *)name; // 通知的名称
  - (id)object; // 通知发布者(是谁要发布通知)
  - (NSDictionary *)userInfo; // 一些额外的信息(通知发布者传递给通知接收者的信息内容)

* 初始化一个通知（NSNotification）对象
```objc
  + (instancetype)notificationWithName:(NSString *)aName object:(id)anObject;
  + (instancetype)notificationWithName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary    *)aUserInfo;
  - (instancetype)initWithName:(NSString *)name object:(id)object userInfo:(NSDictionary *)userInfo;
```
## 发布通知
* 通知中心(NSNotificationCenter)提供了相应的方法来帮助发布通知
```objc
   - (void)postNotification:(NSNotification *)notification;
发布一个notification通知，可在notification对象中设置通知的名称、通知发布者、额外信息等

   - (void)postNotificationName:(NSString *)aName object:(id)anObject;
发布一个名称为aName的通知，anObject为这个通知的发布者

   - (void)postNotificationName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;
发布一个名称为aName的通知，anObject为这个通知的发布者，aUserInfo为额外信息
```
## 注册通知监听器
* 通知中心(NSNotificationCenter)提供了方法来注册一个监听通知的监听器(Observer)
```objc
   - (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
observer：监听器，即谁要接收这个通知
   aSelector：收到通知后，回调监听器的这个方法，并且把通知对象当做参数传入
   aName：通知的名称。如果为nil，那么无论通知的名称是什么，监听器都能收到这个通知
   anObject：通知发布者。如果为anObject和aName都为nil，监听器都收到所有的通知

   - (id)addObserverForName:(NSString *)name object:(id)obj queue:(NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block;
   name：通知的名称
   obj：通知发布者
   block：收到对应的通知时，会回调这个block
   queue：决定了block在哪个操作队列中执行，如果传nil，默认在当前操作队列中同步执行
```
## 取消注册通知监听器
* 通知中心不会保留(retain)监听器对象，在通知中心注册过的对象，必须在该对象释放前取消注册。否则，当相应的通知再次出现时，通知中心仍然会向该监听器发送消息。因为相应的监听器对象已经被释放了，所以可能会导致应用崩溃

```objc
通知中心提供了相应的方法来取消注册监听器
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(NSString *)aName object:(id)anObject;
一般在监听器销毁之前取消注册（如在监听器中加入下列代码）：
- (void)dealloc {
	//[super dealloc];  非ARC中需要调用此句
    [[NSNotificationCenter defaultCenter] removeObserver:self];
```
## UIDevice通知
* UIDevice类提供了一个单粒对象，它代表着设备，通过它可以获得一些设备相关的信息，比如电池电量值(batteryLevel)、电池状态(batteryState)、设备的类型(model，比如iPod、iPhone等)、设备的系统(systemVersion)
```objc
   通过[UIDevice currentDevice]可以获取这个单粒对象
   UIDevice对象会不间断地发布一些通知，下列是UIDevice对象所发布通知的名称常量：
   UIDeviceOrientationDidChangeNotification // 设备旋转
   UIDeviceBatteryStateDidChangeNotification // 电池状态改变
   UIDeviceBatteryLevelDidChangeNotification // 电池电量改变
   UIDeviceProximityStateDidChangeNotification // 近距离传感器(比如设备贴近了使用者的脸部)
   ```
## 键盘通知
* 我们经常需要在键盘弹出或者隐藏的时候做一些特定的操作,因此需要监听键盘的状态
   键盘状态改变的时候,系统会发出一些特定的通知
```objc
   UIKeyboardWillShowNotification // 键盘即将显示
   UIKeyboardDidShowNotification // 键盘显示完毕
   UIKeyboardWillHideNotification // 键盘即将隐藏
   UIKeyboardDidHideNotification // 键盘隐藏完毕
   UIKeyboardWillChangeFrameNotification // 键盘的位置尺寸即将发生改变
   UIKeyboardDidChangeFrameNotification // 键盘的位置尺寸改变完毕
   ```
* 系统发出键盘通知时,会附带一下跟键盘有关的额外信息(字典),字典常见的key如下:
```objc
UIKeyboardFrameBeginUserInfoKey // 键盘刚开始的frame
UIKeyboardFrameEndUserInfoKey // 键盘最终的frame(动画执行完毕后)
UIKeyboardAnimationDurationUserInfoKey // 键盘动画的时间
UIKeyboardAnimationCurveUserInfoKey // 键盘动画的执行节奏(快慢)
```


## 代理
* 代理设计模式的作用:
    * 1.A对象监听B对象的一些行为，A成为B的代理
    * 2.B对象想告诉A对象一些事情，A成为B的代理

* 代理设计模式的总结：
    * 如果你想监听别人的一些行为，那么你就要成为别人的代理
    * 如果你想告诉别人一些事情，那么就让别人成为你的代理

* 代理设计模式的开发步骤
    * 1.拟一份协议（协议名字的格式：控件名 + Delegate），在协议里面声明一些代理方法（一般代理方法都是@optional）
    * 2.声明一个代理属性：@property (nonatomic, weak) id<代理协议> delegate;
    * 3.在内部发生某些行为时，调用代理对应的代理方法，通知代理内部发生什么事
    * 4.设置代理：xxx.delegate = yyy;
    * 5.yyy对象遵守协议，实现代理方法

## 代理和通知的区别
- 代理：1个对象只能告诉另1个对象发生了什么事
- 通知：1个对象可以告诉N个对象发生了什么事

## KVC\KVO
- KVC(Key Value Coding)常见作用：给模型属性赋值
- KVO(Key Value Observing)常用作用：监听模型属性值的改变
- KVO的使用步骤<br>

```objc
// cc监听了aa的name属性的改变
[aa addObserver:cc forKeyPath:@"name" options: NSKeyValueObservingOptionOld context:nil];

// cc得实现监听方法
/**
 * 当监听到object的keyPath属性发生了改变
 */
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    NSLog(@"监听到%@对象的%@属性发生了改变， %@", object, keyPath, change);
}
```


