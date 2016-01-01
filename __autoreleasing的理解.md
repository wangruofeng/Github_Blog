#  __autoreleasing 修饰符

将对象赋值给附有__autoreleasing 修饰符的变量等同于ARC 无效时调用对象的autorelease方法。我们通过以下源代码来看一下

```objective-c
@autoreleasepool {
id __autoreleasing obj  = [[NSObject alloc] init];
}
```
该源代码主要将NSObject 类对象注册到autoreleasepool 中，可作如下变换：

```objective-c
/* 编译器的模拟代码 */
id pool = objc_autoreleasePoolPush();
id obj = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(obj, @selector(init));
objc_autorelease(obj);
objc_autoreleasePoolPop(pool);
```

这与苹果的autorelease 实现中的说明（参考1.2.7 节）完全相同。虽然ARC 有效和无效时，其在源代码上的表现有所不同，但autorelease 的功能完全一样。

在alloc/new/copy/mutableCopy 方法群之外的方法中使用注册到autoreleasepool 中的对象会如何呢？下面我们来看看NSMutableArray 类的array 类方法。

```objective-c
@autoreleasepool {
id __autoreleasing obj = [NSMutableArray array];
}
```

这与前面的源代码有何不同呢？

```objective-c
/* 编译器的模拟代码 */
id pool = objc_autoreleasePoolPush();
id obj = objc_msgSend(NSMutableArray, @selector(array));
objc_retainAutoreleasedReturnValue(obj);
objc_autorelease(obj);
objc_autoreleasePoolPop(pool);
```
虽然持有对象的方法从alloc 方法变为objc_retainAutoreleasedReturnValue 函数， 但注册autoreleasepool 的方法没有改变，仍是objc_autorelease 函数。
