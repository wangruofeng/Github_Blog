# iOS中的触摸事件和手势处理

### iOS触摸事件分类
* 触摸事件
* 加速事件
* 远程事件

### 谁能处理触摸事件?
**响应者对象**

在iOS中不是任何对象都能处理事件,只有继承了UIResponder的对象才能接收并处理事件.我们称之为**响应者对象**.

`UIApplication`,`UIViewController`,`UIView`都继承自`UIResponder`,因此它们都是响应者对象,都能够接收并处理事件.

### UIResponder
UIResponder内部提供了方法来处理事件

1. 触摸事件

	一次完成的触摸过程,会经历3个状态;
	UIView的触摸事件处理

	1.一根或多根手指开始触摸view,系统会自动调用view下面的方法:
		- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;  //触摸开始
	2.一根或者多根手指在view上移动，系统会自动调用view下面的方法（随着手指的移动，会持续调用该方法）
		- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;  //触摸移动
	3.一根或者多根手指离开view，系统会自动调用view下面的方法
		- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;  //触摸结束
	4.触摸结束前，某个系统事件（例如电话呼入	）会打断触摸过程，系统会自动调用view下面的方法
		- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event; //触摸取消(可能会经历)

	4个触摸事件的处理方法中，都有 NSSet *touches 和 UIEvent *event 两个参数:
	* 一次完整的触摸过程，只会产生一个事件对象，4个触摸方法都是同一个event参数
	* 如果两根手指同时触摸一个view，那么view只会调用一次 touchesBegan:withEvent: 方法，touches参数中装着两个UITouch对象；
	* 如果这两根手指一前一后分开触摸同一个view，那么view会分别调用两次 touchesBegan:withEvent:方法， 并且每次调用时的touches参数只包含一个UITouch对象；
	* 根据touches中UITouch个数可以判断出使单点触摸还是多点触摸

	提示：touches中存放的都是UITouch对象。


2. 加速计事件
		- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
		- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
		- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;

3. 远程控制事件

		- (void)remoteControlReceivedWithEvent:(UIEvent *)event;

### UITouch
当用户用一根手指触摸屏幕时，会创建一个与手指相关联的UITouch对象；一根手指对应一个UITouch对象
UITouch的作用:

* 保存跟手指相关的信息，比如触摸的位置、时间、阶段；
* 当手指移动时，系统会更新同一个UITouch对象，使之能够一直保存该手指的触摸位置；
* 当手指离开屏幕时，系统会销毁相应的UITouch对象。

提示：iPhone开发中，要避免使用双击事件。

UITouch的属性:
触摸产生时所处的窗口:

 	@property(nonatomic,readonly,retain) UIWindow *window;

触摸产生时所处的视图

	 @property(nonatomic,readonly,retain) UIView *view;

短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多地点击

	 @property(nonatomic,readonly) NSUInteger tapCount;

记录了触摸事件产生或变化时的时间，单位是秒

	 @property(nonatomic,readonly) NSTimeInterval timestamp;

当前触摸事件所处的状态

	@property(nonatomic,readonly) UITouchPhase phase;
	/*
	UITouchPhase是一个枚举类型，包含：
	UITouchPhaseBegan（触摸开始）
	UITouchPhaseMoved（接触点移动）
	UITouchPhaseStationary（接触点无移动）
	UITouchPhaseEnded（触摸结束）
	UITouchPhaseCancelled（触摸取消）*/

UITouch的方法：

	- (CGPoint)locationInView:(UIView *)view;

1.返回值表示触摸在view上的位置；

2.这里返回的位置是针对view坐标系的,（以view的左上角为原点（0，0））；

3.调用时传入的view参数为nil 的话，返回的是触摸点在UIWindow的位置。

	- (CGPoint)previousLocationInView:(UIView *)view;
该方法记录了前一个触摸点的位置；

### UIEvent
> 每产生一个事件，就会产生一个UIEvent对象；
>
> UIEvent:称为事件对象，记录事件产生的时刻和类型。

常见属性：
1.事件类型

```objective-c
@property(nonatomic,readonly) UIEventType  type;
@property(nonatomic,readonly) UIEventSubtype  subtype;

typedef
NS_ENUM(NSInteger, UIEventType) {
    UIEventTypeTouches,
    UIEventTypeMotion,
    UIEventTypeRemoteControl,
};
typedef
NS_ENUM(NSInteger, UIEventSubtype) {
    // available in iPhone OS 3.0
    UIEventSubtypeNone                              = 0,    
    // for UIEventTypeMotion, available in iPhone OS 3.0
    UIEventSubtypeMotionShake                       = 1,
    // for UIEventTypeRemoteControl, available in iOS 4.0
    UIEventSubtypeRemoteControlPlay                 = 100,
    UIEventSubtypeRemoteControlPause                = 101,
    UIEventSubtypeRemoteControlStop                 = 102,
    UIEventSubtypeRemoteControlTogglePlayPause      = 103,
    UIEventSubtypeRemoteControlNextTrack            = 104,  
    UIEventSubtypeRemoteControlPreviousTrack        = 105,
    UIEventSubtypeRemoteControlBeginSeekingBackward = 106,   
    UIEventSubtypeRemoteControlEndSeekingBackward   = 107,  
    UIEventSubtypeRemoteControlBeginSeekingForward  = 108,  
    UIEventSubtypeRemoteControlEndSeekingForward    = 109,
};
```

2.事件产生的时间

	@property(nonatomic,readonly) NSTimeInterval  timestamp;
 UIEvent还提供了相应的方法可以获得在某个view上面的触摸对象（UITouch）。

 触摸事件的产生：

 1. 发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中；

 2. UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）；

 3. 主窗口会在视图层次结构中找到一个最合适的视图控件来处理触摸事件，这也是整个事件处理过程的第一步；

 4. 找到合适的视图控件后，就会调用视图控件的touches方法来做具体的事件处理。

触摸事件的传递：

> 触摸事件的传递是从父控件传递到子控件；
>
> 如果父控件不能接收触摸事件，那么子控件就不可能接收到触摸事件。

![触摸事件](http://ww4.sinaimg.cn/mw690/64124373gw1eznrp087anj205t07rdfx.jpg)
![触摸事件2](http://ww2.sinaimg.cn/mw690/64124373gw1eznrrf36ymj2097077dhc.jpg)

UIView不接收触摸事件的三种情况:
不接受用户交互 ：

1. userInteractionEnable = NO;

2. 隐藏 ：hidden = YES;

3. 透明：alpha = 0.0 ~ 0.01

提示：UIImageView的userInteractionEnable默认就是NO，因此UIImageView以及它的子控件默认是不能接收触摸事件的。

触摸事件处理的详细过程：

1. 用户点击屏幕后产生的一个触摸事件，经过一些列的传递过程后，会找到最合适的视图控件来处理这个事件

2. 找到最合适的视图控件后，就会调用控件的touches方法来作具体的事件处理

	touchesBegan…

    touchesMoved…

    touchedEnded…

这些touches方法的默认做法是将事件顺着响应者链条向上传递，将事件交给上一个响应者进行处理

响应者链的事件传递过程：

 1. 如果view的控制器存在，就传递给控制器；如果控制器不存在，则将其传递给它的父视图；

 2. 在视图层次结构最顶级的视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理。

 3. 如果window对象也不处理，则其将事件或消息传递给UIApplication对象；

 4. 如果UIApplication也不能处理该事件或消息，则将其丢弃

### 监听触摸事件的做法

如果想监听一个view上面的触摸事件，之前的做法是：

1. 自定义一个view；

2. 实现view的touches方法，在方法内部实现具体处理代码。

	通过touches方法监听view触摸事件，有很明显的几个缺点：

	* 必须得自定义view；

    * 由于是在view内部的touches方法中监听触摸事件，因此默认情况下，无法让其他外界对象监听view的触摸事件；

    * 不容易区分用户的具体手势行为。

iOS 3.2之后，苹果推出了手势识别功能（Gesture Recognizer）,在触摸事件处理方面，大大简化了开发者的开发难度。

### UIGestureRescognizer

为了完成手势识别，必须借助于手势识别器：UIGestureRecognizer 。

利用UIGestureRecognizer,能轻松识别用户在某个view上面做的一些常见手势。

UIGestureRecognizer是一个抽象类，定义了所有的手势基本行为，使用它的子类才能处理具体的手势

* UITapGestureRecognizer(敲击)
* UIPinchGestureRecognizer(捏合，用于缩放)
* UIPanGestureRecognizer(拖拽)
* UISwipeGestureRecognizer(轻扫)
* UIRotationGestureRecognizer(旋转)
* UILongPressGestureRecognizer(长按)

每一个手势识别器的用法都差不多，比如UITapGestureRecognizer的使用步骤如下:

1. 创建手势识别器对象；

    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] init];

2. 设置手势识别器对象的具体属性；

		// 连续敲击2次
		tap.numberOfTapsRequired = 2;
		// 需要2根手指一起敲击
		tap.numberOfTouchesRequired = 2;

3. 添加手势识别器到对应的view上

	[self.iconView addGestureRecognizer:tap];

4. 监听手势的触发

	[tap addTarget:self action:@selector(tapIconView:)];

手势识别的状态

```objective-c
typedef NS_ENUM(NSInteger, UIGestureRecognizerState) {
    // 没有触摸事件发生，所有手势识别的默认状态
    UIGestureRecognizerStatePossible,
    // 一个手势已经开始但尚未改变或者完成时
    UIGestureRecognizerStateBegan,
    // 手势状态改变
    UIGestureRecognizerStateChanged,
    // 手势完成
    UIGestureRecognizerStateEnded,
    // 手势取消，恢复至Possible状态
    UIGestureRecognizerStateCancelled,
    // 手势失败，恢复至Possible状态
    UIGestureRecognizerStateFailed,
    // 识别到手势识别
    UIGestureRecognizerStateRecognized = UIGestureRecognizerStateEnded
};
```

参考资料：[傲风凌寒的博客](http://my.oschina.net/aofe/blog/268749)
