## NSRunLoop是什么？
在Cocoa中，每个线程(`NSThread`)对象中内部都有一个run loop（`NSRunLoop`）对象用来循环处理输入事件.

*NSRunloop并不真的是一个loop，的apple的文档中 也提到了需要自己写while或者for语句来实现,类似下面：*
```objective-c
while(running){
    [NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}
```
## 何为Run loop 事件源
从字面翻译来看，run loop就是一个运行循环，的确它就是一个处理输入时间的运行循环，为什么需要这样处理，难道没有事件发生的时候让线程空转浪费资源？很明显在有事件发生的时候唤醒线程，没有事件发生的时候让其sleep更好。

下面我还是拿这张百看不厌的图来说事：
![runloop](http://ww2.sinaimg.cn/mw690/64124373gw1ezgx936dhzj20dg071aav.jpg)

处理的事件包括两类
* 来自Timer sources的同步事件
* 来自Input sources的异步事件

1.Time Source. Timer sources deliver synchronous events, occurring at a scheduled time or repeating interval.
苹果文档中有句话需要注意，
> Timer sources deliver events to their handler routines but do not cause the run loop to exit.*

创建NSTimer添加到run loop中的时候，这里需要注意的是，`NSTimer`默认是处于`NSDefaultRunloopMode`，这也就可以解释为什么如果你在你的控制器中添加了一个timer定时刷新你的界面，而你在拖动视图的时候timer不回fire，因为这个时候你的runloop 是`NSEventTrackingRunloopMode`,在这个mode下timer不回fire

2.input source input source 主要是一些异步的事件，比如来自其它线程或者其它app的消息。

input source 传递异步事件到其对应的处理函数，并且使runUntilDate(与线程相关联的runloop对象调用)返回

为了能够处理input sourcr，run loops 产生notifications.通过注册成run-loop observers可以接受到这些通知（通过Core Foundation 来注册observers）.


## RunLoopMode有哪些？
run loop在处理输入事件时会产生通知，可以通过Core Foundation向线程中添加run loop observers来监听特定事件,以在监听的事件发生时做附加的处理工作。

每个run loop可运行在不同的模式下,一个run loop mode是一个集合，其中包含其监听的若干输入事件源，定时器，以及在事件发生时需要通知的run loop observers。运行在一种mode下的run loop只会处理其run loop mode中包含的输入源事件，定时器事件，以及通知run loop mode中包含的observers。

Cocoa中的预定义模式有:
1. Default模式

    定义[NSDefaultRunLoopMode](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSRunLoop_Class/index.html#//apple_ref/c/data/NSDefaultRunLoopMode) (Cocoa) [kCFRunLoopDefaultMode](https://developer.apple.com/library/ios/documentation/CoreFoundation/Reference/CFRunLoopRef/index.html#//apple_ref/c/data/kCFRunLoopDefaultMode) (Core Foundation)

    描述：默认模式中几乎包含了所有输入源(NSConnection除外),一般情况下应使用此模式，这是最常用的run loop mode。

2. Connection模式

    定义：`NSConnectionReplyMode`(Cocoa)

    描述：处理`NSConnection`对象相关事件，系统内部使用，这个mode表明`NSConnection`对象等待reply,用户基本不会使用。

3. Modal模式

    定义：`NSModalPanelRunLoopMode`(Cocoa)

    描述：处理modal panels事件,需要等待处理的input source为modal panel时设置，比如NSSavePanel和NSOpenPanel。

4. Event tracking模式
    定义：[UITrackingRunLoopMode](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/doc/uid/TP40006728-CH3-SW38)(iOS) `NSEventTrackingRunLoopMode`(cocoa)

    描述：使用该模式来处理用户界面相关的事件,例如在拖动loop或其他user interface tracking loops时处于此种模式下，在此模式下会限制输入事件的处理。例如，当手指按住UITableView拖动时就会处于此模式。

5. Common模式

    定义：[NSRunLoopCommonModes](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSRunLoop_Class/index.html#//apple_ref/c/data/NSRunLoopCommonModes) (Cocoa) [kCFRunLoopCommonModes](https://developer.apple.com/library/ios/documentation/CoreFoundation/Reference/CFRunLoopRef/index.html#//apple_ref/c/data/kCFRunLoopCommonModes) (Core Foundation)

    描述：这是一个伪模式，其为一组run loop mode的集合，将输入源加入此模式意味着在Common Modes中包含的所有模式下都可以处理。在Cocoa应用程序中，默认情况下Common Modes包含default modes,modal modes,event Tracking modes,
    可使用[CFRunLoopAddCommonMode](https://developer.apple.com/library/ios/documentation/CoreFoundation/Reference/CFRunLoopRef/index.html#//apple_ref/c/func/CFRunLoopAddCommonMode)方法向Common Modes中添加自定义modes。

    **注意这个并不是一个特定的mode，而是一个mode的集合，而runloop必须运行在一个特定的mode下**。

获取当前线程的runloop mode
``` objective-c
NSString* runLoopMode = [[NSRunLoop currentRunLoop] currentMode];
```

## NSTimer、NSURLConnection与UITrackingRunLoopMode
NSTimer与NSURLConnection默认运行在default mode下，这样当用户在拖动UITableView处于UITrackingRunLoopMode模式时，NSTimer不能fire,NSURLConnection的数据也无法处理。
NSTimer的例子:
在一个UITableViewController中启动一个0.2s的循环定时器，在定时器到期时更新一个计数器，并显示在label上。

``` objective-c
- (void)viewDidLoad {

    label =[[UILabel alloc]initWithFrame:CGRectMake(10, 100, 100, 50)];
    [self.view addSubview:label];
    count = 0;

    NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval: 0.2
                                                      target: self
                                                    selector: @selector(incrementCounter:)
                                                    userInfo: nil
                                                     repeats: YES];
}

- (void)incrementCounter:(NSTimer *)theTimer
{
    count++; label.text = [NSString stringWithFormat:@"%zd",count];
}

```

在正常情况下，可看到每隔0.2s，label上显示的数字+1,但当你拖动或按住tableView时，label上的数字不再更新，当你手指离开时，label上的数字继续更新。当你拖动`UItableView`时，当前线程run loop处于`UIEventTrackingRunLoopMode`模式，在这种模式下，不处理定时器事件，即定时器无法fire,label上的数字也就无法更新。
解决方法，一种方法是在另外的线程中处理定时器事件，可把Timer加入到`NSOperation`中在另一个线程中调度;还有一种方法时修改Timer运行的run loop模式，将其加入到`UITrackingRunLoopMode`模式或`NSRunLoopCommonModes`模式中。
即
```objective-c
[[NSRunLoop currentRunLoop] addTimer:timer forMode:UITrackingRunLoopMode];
```
或

```objective-c
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

另外一种是放到`NSThread`中
```objective-c
- (void)viewDidLoad{
    [super viewDidLoad];
    NSLog(@"主线程 %@", [NSThread currentThread]);
    //创建并执行新的线程
    NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
    [thread start];
}

- (void)newThread{
    @autoreleasepool{
        //在当前Run Loop中添加timer，模式是默认的NSDefaultRunLoopMode
        [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(timer_callback) userInfo:nil repeats:YES];
        //开始执行新线程的Run Loop，如果不启动run loop，timer的事件是不会响应的
        [[NSRunLoop currentRunLoop] run];
    }
}

- (void)timer_callback{
    NSLog(@"Timer %@", [NSThread currentThread]);
}
```

`NSURLConnection`也是如此，见[SDWebImage](https://github.com/rs/SDWebImage)中的描述,以及[SDWebImageDownloader.m](https://github.com/rs/SDWebImage/blob/master/SDWebImage/SDWebImageDownloader.m)代码中的实现。修改NSURLConnection的运行模式可使用[scheduleInRunLoop:forMode:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/Reference/Reference.html#//apple_ref/occ/instm/NSURLConnection/scheduleInRunLoop:forMode:)方法。

```objective-c
NSURL *url = [NSURL URLWithString:@"https://www.baidu.com"];
NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:15];

self.connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
[connection scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
[connection start];
```

### 参考资料：
* [Threading Programming Guide – Run Loops](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1)
* [NSRunLoop Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSRunLoop_Class/index.html)
* [NSURLConnection Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/)
* [NSTimer Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/index.html)
* [CFRunLoop wiki](http://iphonedevwiki.net/index.php/CFRunLoop)
* [SDWebImage](https://github.com/rs/SDWebImage)
* [TestButtonDown](http://hayne.net/MacDev/TestButtonDown/)
* [关于NSRunloop的学习和理解](http://billwang1990.github.io/blog/2013/12/30/nsrunloop-issue/)
