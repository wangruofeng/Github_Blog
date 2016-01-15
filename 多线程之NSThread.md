## 本文目录
* 前言
* 1.获取当前线程
* 2.获取主线程
* 3.NSThread的创建
* 4.暂停当前线程
* 5.线程的其他操作
* 6.优缺点

### 前言
每个iOS应用程序都有个专门用来更新显示UI界面、处理用户触摸事件的主线程，因此不能将其他太耗时的操作放在主线程中执行，不然会造成主线程堵塞(出现卡机现象)，带来极坏的用户体验。一般的解决方案就是将那些耗时的操作放到另外一个线程中去执行，多线程编程是防止主线程堵塞，增加运行效率的最佳方法。

iOS中有3种常见的多线程编程方法
1. `NSThread`
这种方法需要管理线程的生命周期、同步、加锁问题，会导致一定的性能开销

2. `NSOperation`和`NSOperationQueue`
是基于OC实现的。NSOperation以面向对象的方式封装了需要执行的操作，然后可以将这个操作放到一个NSOperationQueue中去异步执行。不必关心线程管理、同步等问题。

3. `Grand Centeral Dispatch`
简称GCD，iOS4才开始支持，是纯C语言的API。自iPad2开始，苹果设备开始有了双核CPU，为了充分利用这2个核，GCD提供了一些新特性来支持多核并行编程

这篇文章简单介绍`NSThread这`个类，一个`NSThread`实例就代表着一条线程

### 1.获取当前线程

```objective-c
    NSThread *current = [NSThread currentThread];
```

### 2.获取主线程
```objective-c
    NSThread *main = [NSThread mainThread];
    NSLog(@"主线程:%@", main);    
```

打印结果是：

    2013-04-18 21:36:38.599 thread[7499:c07] 主线程:<NSThread: 0x71434e0>{name = (null), num = 1}

num相当于线程的id，主线程的num是为1的

### 3.NSThread的创建
#### a.动态方法
    - (id)initWithTarget:(id)target selector:(SEL)selector object:(id)argument;

在第2行创建了一条新线程，然后在第4行调用`start`方法启动线程，线程启动后会调用self的`run:`方法，并且将@"mj"作为方法参数

```objective-c
// 初始化线程
NSThread *thread = [[[NSThread alloc] initWithTarget:self selector:@selector(run:) object:@"mj"] autorelease];
// 开启线程
[thread start];
```    

假如run:方法是这样的：

```objective-c
- (void)run:(NSString *)string {
     NSThread *current = [NSThread currentThread];
     NSLog(@"执行了run:方法-参数：%@，当前线程：%@", string, current);
}
```

打印结果为：

    2013-04-18 21:40:33.102 thread[7542:3e13] 执行了run:方法-参数：mj，当前线程：<NSThread: 0x889e8d0>{name = (null), num = 3}

可以发现，这条线程的num值为3，说明不是主线程，主线程的num为1

#### b.静态方法
```objective-c
    + (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(id)argument;
```

```objective-c
[NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:@"mj"];
```

#### c.隐式创建线程
```objective-c
[self performSelectorInBackground:@selector(run:) withObject:@"mj"];
```
会隐式地创建一条新线程，并且在这条线程上调用self的run:方法，以@"mj"为方法参数

### 4.暂停当前线程

```objective-c
    [NSThread sleepForTimeInterval:2];
```

```objective-c
NSDate *date = [NSDate dateWithTimeInterval:2 sinceDate:[NSDate date]];  
[NSThread sleepUntilDate:date];
```

上面两种做法都是暂停当前线程2秒

### 5.线程的其他操作
#### a.在指定线程上执行操作

```objective-c
 [self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES];
```

* 上面代码的意思是在thread这条线程上调用self的run方法
* 最后的YES代表：上面的代码会阻塞，等run方法在thread线程执行完毕后，上面的代码才会通过

#### b.在主线程上执行操作

```objective-c
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES];  
```

在主线程调用self的run方法

#### c.在当前线程执行操作

```objective-c
[self performSelector:@selector(run) withObject:nil];
```
在当前线程调用self的run方法

### 6.优缺点
1. 优点：`NSThread`比其他多线程方案较轻量级，更直观地控制线程对象
2. 缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>
