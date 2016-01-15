### 本文目录
* 前言
* 1.NSInvocationOperation
* 2.NSBlokcOperation
* 3.NSOperation的其他用法
* 4.自定义NSOperation

## 前言
1.虽然`NSThread`也可以实现多线程编程，但是需要我们去管理线程的生命周期，还要考虑线程同步、加锁问题，造成一些性能上的开销。我们也可以配合使用`NSOperation`和`NSOperationQueue`实现多线程编程，实现步骤大致是这样的

* 先将需要执行的操作封装到一个NSOperation对象中
* 然后将NSOperation对象添加到NSOperationQueue中
* 系统会自动将`NSOperation`中封装的操作放到一条新线程中执行在此过程中，我们根本不用考虑线程的生命周期、同步、加锁等问题下面列举一个应用场景，比如微博的粉丝列表：

![微博的粉丝列表](http://images.cnitblog.com/blog/497279/201304/19122046-b40c752b60a5413290b569f5377ef7f3.png)

每一行的头像肯定要从新浪服务器下载图片后才能显示的，而且是需要异步下载。这时候你就可以把每一行的图片下载操作封装到一个`NSOperation`对象中，上面有6行，所以要创建6个`NSOperation`对象，然后添加到`NSOperationQueue`中，分别下载不同的图片，下载完毕后，回到对应的行将图片显示出来。

2 .默认情况下，`NSOperation`并不具备封装操作的能力，必须使用它的子类，使用NSOperation子类的方式有3种：

* NSInvocationOperation
* NSBlockOperation
* 自定义子类继承NSOperation，实现内部相应的方法

这讲先介绍如何用`NSOperation`封装一个操作，后面再结合`NSOperationQueue`来使用。

## 1.NSInvocationOperation

```objective-c
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run:) object:@"mj"];
[operation start];
```
*  第1行初始化了一个`NSInvocationOperation`对象，它是基于一个对象和selector来创建操作
* 第2行调用了start方法，紧接着会马上执行封装好的操作，也就是会调用self的run:方法，并且将@"mj"作为方法参数
* 这里要注意：默认情况下，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作。只有将operation放到一个NSOperationQueue中，才会异步执行操作。

## 2.NSBlockOperation
### a.同步执行一个操作

```objective-c
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^(){
         NSLog(@"执行了一个新的操作");
 }];
  // 开始执行任务
[operation start];
```
*  第1行初始化了一个NSBlockOperation对象，它是用一个Block来封装需要执行的操作
* 第2行调用了start方法，紧接着会马上执行Block中的内容
* 这里还是在当前线程同步执行操作，并没有异步执行

### b.并发执行多个操作

```objective-c
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^(){
　　NSLog(@"执行第1次操作，线程：%@", [NSThread currentThread]);
}];

[operation addExecutionBlock:^() {
　　NSLog(@"又执行了1个新的操作，线程：%@", [NSThread currentThread]);
}];

[operation addExecutionBlock:^() {
　　NSLog(@"又执行了1个新的操作，线程：%@", [NSThread currentThread]);
}];

[operation addExecutionBlock:^() {
　　NSLog(@"又执行了1个新的操作，线程：%@", [NSThread currentThread]);
}];

// 开始执行任务
[operation start];
````

* 第1行初始化了一个NSBlockOperation对象
* 分别在第5、9、13行通过addExecutionBlock:方法添加了新的操作，包括第1行的操作，一共封装了4个操作
* 在第18行调用start方法后，就会并发地执行这4个操作，也就是会在不同线程中执行

		1 2013-02-02 21:38:46.102 thread[4602:c07] 又执行了1个新的操作，线程：<NSThread: 0x7121d50>{name = (null), num = 1}
		2 2013-02-02 21:38:46.102 thread[4602:3f03] 又执行了1个新的操作，线程：<NSThread: 0x742e1d0>{name = (null), num = 5}
		3 2013-02-02 21:38:46.102 thread[4602:1b03] 执行第1次操作，线程：<NSThread: 0x742de50>{name = (null), num = 3}
		4 2013-02-02 21:38:46.102 thread[4602:1303] 又执行了1个新的操作，线程：<NSThread: 0x7157bf0>{name = (null), num = 4}

可以看出，每个操作所在线程的num值都不一样，说明是不同线程

## 3.NSOperation的其他用法
### a.取消操作
operation开始执行之后, 默认会一直执行操作直到完成，我们也可以调用cancel方法中途取消操作

	[operation cancel];

### b.在操作完成后做一些事情
如果想在一个NSOperation执行完毕后做一些事情，就调用NSOperation的setCompletionBlock方法来设置想做的事情

	operation.completionBlock = ^() {
	    NSLog(@"执行完毕");
	};
当operation封装的操作执行完毕后，就会回调Block里面的内容

## 4.自定义NSOperation
如果`NSInvocationOperation`和`NSBlockOperation`不能满足需求，我们可以直接新建子类继承NSOperation，并添加任何需要执行的操作。如果只是简单地自定义NSOperation，只需要重载`-(void)main`这个方法，在这个方法里面添加需要执行的操作。

下面写个子类DownloadOperation来下载图片
### a.继承NSOperation，重写main方法

*DownloadOperation.h*
```objective-c
#import <Foundation/Foundation.h>
@protocol DownloadOperationDelegate;

@interface DownloadOperation : NSOperation
// 图片的url路径
@property (nonatomic, copy) NSString *imageUrl;
// 代理
@property (nonatomic, assign) id<DownloadOperationDelegate> delegate;

- (id)initWithUrl:(NSString *)url delegate:(id<DownloadOperationDelegate>)delegate;
@end

// 图片下载的协议
@protocol DownloadOperationDelegate <NSObject>
- (void)downloadFinishWithImage:(UIImage *)image;
@end
```

*DownloadOperation.m
*

```objective-c
#import "DownloadOperation.h"

@implementation DownloadOperation
@synthesize delegate = _delegate;
@synthesize imageUrl = _imageUrl;

// 初始化
- (id)initWithUrl:(NSString *)url delegate:(id<DownloadOperationDelegate>)delegate {
    if (self = [super init]) {
        self.imageUrl = url;
        self.delegate = delegate;
    }
    return self;
}
// 释放内存
- (void)dealloc {
    [super dealloc];
    [_imageUrl release];
}

// 执行主任务
- (void)main {
    // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
    @autoreleasepool {
        // ....
    }
}
@end
```

* 在第22行重载了main方法，等会就把下载图片的代码写到这个方法中
* 如果这个DownloadOperation是在异步线程中执行操作，也就是说main方法在异步线程调用，那么将无法访问主线程的自动释放池，所以在第24行创建了一个属于当前线程的自动释放池

### b.正确响应取消事件
* 默认情况下，一个NSOperation开始执行之后，会一直执行任务到结束，就比如上面的DownloadOperation，默认会执行完main方法中的所有代码
* NSOperation提供了一个cancel方法，可以取消当前的操作。
* 如果是自定义NSOperation的话，需要手动处理这个取消事件。比如，一旦调用了cancel方法，应该马上终止main方法的执行，并及时回收一些资源。
* 处理取消事件的具体做法是：在`main`方法中定期地调用`isCancelled`方法检测操作是否已经被取消，也就是说是否调用了`cancel`方法，如果返回YES，表示已取消，则立即让main方法返回。
* 以下地方可能需要调用`isCancelled`方法:
	1. 在执行任何实际的工作之前，也就是在main方法的开头。因为取消可能发生在任何时候，甚至在operation执行之前。
	2. 执行了一段耗时的操作之后也需要检测操作是否已经被取消

```objective-c
- (void)main {
    // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
    @autoreleasepool {
        if (self.isCancelled) return;

        // 获取图片数据
        NSURL *url = [NSURL URLWithString:self.imageUrl];
        NSData *imageData = [NSData dataWithContentsOfURL:url];

        if (self.isCancelled) {
            url = nil;
            imageData = nil;
            return;
        }

        // 初始化图片
        UIImage *image = [UIImage imageWithData:imageData];

        if (self.isCancelled) {
            image = nil;
            return;
        }

        if ([self.delegate respondsToSelector:@selector(downloadFinishWithImage:)]) {
            // 把图片数据传回到主线程
            [(NSObject *)self.delegate performSelectorOnMainThread:@selector(downloadFinishWithImage:) withObject:image waitUntilDone:NO];
        }
    }
}
```

* 在第4行main方法的开头就先判断operation有没有被取消。如果被取消了，那就没有必要往下执行了
* 经过第8行下载图片后，在第10行也需要判断操作有没有被取消
* 总之，执行了一段比较耗时的操作之后，都需要判断操作有没有被取消
* 图片下载完毕后，在第26行将图片数据传递给了代理(delegate)对象


备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>

