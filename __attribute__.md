# __attribute__

* 译自 Written by [Mattt Thompson](http://nshipster.com/authors/mattt-thompson/) — January 14th, 2013
* 原文链接：<http://nshipster.com/__attribute__/>
* 译者[@oneruofeng](https://twitter.com/oneruofeng)

重复发布这个主题已经说明了同编译器保存健康关系的重要性,像任何草稿一样，作为一个实践者的效率取决于他们怎样对待他们的工具，你照顾好它们，它们反过来也会对你有好处。

 `__attribute__`是一个编译器的指令在声明的时候指明了一些参数，这些参数允许更多的检查错误和高级的优化。

 语法关键字是`__attribute__`紧跟2套圆括号（双圆括号让出现的宏更容易辨认，特别是在有多个属性的时候）。在括号内部是一个以逗号分隔的属性列表，`__attribute__`指令被放在函数，变量和类型声明后面。

```objective-c
 // Return the square of a number
int square(int n) __attribute__((const));

// Declare the availability of a particular API
void f(void)
  __attribute__((availability(macosx,introduced=10.4,deprecated=10.6)));

// Send printf-like message to stderr and exit
extern void die(const char *format, ...)
  __attribute__((noreturn, format(printf, 1, 2)));
```

假如这个让你想起ISO C语言的 `#pragma`,你就不会感到孤单了。

实际上，当`__attribute__`被第一次引入到`GCC`编译器时，它面临一些阻力，有人建议使用专用的`#pragma`因为相同的目的。

这里，然而，有2个非常好的理由为什么`__attribute__`被添加进来
> * 从一个宏中产生`#pragma`命令几乎是不能的（在C99 _Pragma 预算符以前）。
> * 这里没人知道相同的`#pragma`在另一个编译器中可能的意思。

 引用[GCC Documentation for Function Attributes](http://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html)

 > * 这里有2个原因被应用到几乎所有的应用推荐使用`#pragma`,这犯了一个低级错误就是把`#pragma`使用到任何地方。

 确实，假如你在苹果的框架中和牛逼工程师的开源项目中的头文件看一下现代的`Objective-c`--`__attribute__`被大量使用。（相反，`#pragma`的主要声明名声来着这些天是装饰:`#pragma mark`）

 所以为了以后不费力，我们还是先看一下最重要的属性：

 ------
GCC
---
**format**
> `format`属性指定了一个函数像`printf`,`scanf`,`strftime`或者`strfmo`风格的参数，这个参数应该是可以进行类型检查的一个格式化字符串。

```objective-c
extern int
my_printf (void *my_object, const char *my_format, ...)
  __attribute__((format(printf, 2, 3)));
```

Objective-C程序员也可使用`__NSString__`来格式化来做到相同的格式化规则，像在`NNString`中通过 ``+stringWithFormat:`` 和 `NSLog()`格式化字符串一样。

**nonnull**
> 这个`nonnull`属性指定了某些函数的参数必须是非空的指针。

```objective-c
extern void *
my_memcpy (void *dest, const void *src, size_t len)
  __attribute__((nonnull (1, 2)));
```

使用`nonnull`编码期望这个值遵守一个明确的约定中，这样能帮助捕获潜伏在任何代码调用的NULL指针bugs，请记住：
`编译时的错误 >>  运行时的错误。`

**noreturn**
> 一些标准库函数，例如`abort`和`exit`,是不能返回的。GCC自动知道这些东西，这个noreturn属性用于指定任何其他函数永远不会返回的情况。

例如，AFNetworking 使用`noreturn`属性在它的[网络请求线程进入点的方法](https://github.com/AFNetworking/AFNetworking/blob/1.1.0/AFNetworking/AFURLConnectionOperation.m#L157)里面,这个方法用在当大量产生专用的网络的线程里用来保证分离的线程持续执行在应用的整个生命周期中。

**pure/const**
> `pure`属性指定了一个函数除了返回值没有副作用，例如它的返回值仅仅依赖参数和/或者全局变量。这样的函数可以用公共子表达式消除并且循环优化就像一个算数操作符那样。

> `pure`属性指定了一个函数不会检查任何值除了它们的参数，并且返回值没有副作用。注意到一个函数有一个指针参数并且需呀检查数据的指向不能声明成`const`。同样的，一个函数调用一个非`nonst`函数通常不能为`const`,一个`const`函数返回`void`并没有什么意义。

```objective-c
int square(int n) __attribute__((const));
```

`pure`和`const`是两个执行在一个函数式编程惯例中的参数为了允许有效性能优化。`const`可以被认为是严格形式的`pure`因为它不依赖全局变量或者指针。

例如，因为一个函数声明为`const`的结果并不依赖任何东西除了传进来的参数。函数的结果能够缓存那个结果并且当函数被调用时返回，这样的函数叫做相同的组合参数（也就是说，我们知道一个数字的平方是一个常量，所以我们仅仅需要只计算它一次)。

**unused**
> 这个属性，附着在一个函数后面，意味着那个函数很可能不会被使用，GCC不会对这个函数产生警告。

用`__unused`关键词可以获得相同的效果，声明这个在方法实现中不会被使用的参数中。知道那以后一些上下文就可以允许编译器来做相应的优化。你很可能喜欢在delegate方法实现李勉使用`__unused`,因为协议频繁的提高更多的上下文比通常必要的情况，为了满足大量的潜在使用案例。

### LLVM

像GCC的很多特征一样，Clang也支持`__attribute__`,添加到它自己的小范围的扩展。为了检查某个属性的可用性，你可以直接使用`__has_attribute`属性。

**availability**
> Clang引进了availability属性，这个可以被取代在声明描述的生命周期中声明相对于操作系统的版本。思考对一个简单函数f：的函数声明

```objective-c
void f(void) __attribute__((availability(macosx,introduced=10.4,deprecated=10.6,obsoleted=10.7)));
```

> `availability`属性声明f在OS X老虎系统中被引入，在OS X雪豹系统中被弃用，在OS X 山狮系统中被废弃。

> 这个信息被Clang用来决定什么时候使用f：函数式安全的，例如，假如Clang在OS X 美洲豹系统上编译，调用f()函数将成功。假如Clang在OS X雪豹系统中编译，函数调用将成功但是Clang会发出一个警告指明这个函数被弃用了。最后，假如Clang被引进编译OS X山狮系统的代码，函数调用将失败，因为f()函数已经不再可用了。

> `availability`属性是一个逗号分隔的列表以平台名开始然后引入一些定语列举出生命周期内的重要里程碑事件附加额外的信息（以任何顺序）。

* introduced：声明被引入的第一个版本
* deprecated：声明被弃用的第一个版本，这意味着用户应该把这个API移走
* obsoleted： 声明被废弃的第一个版本，这意味着它将被完全移除并且不能再使用
* unavailable：声明在这个平台上将永远不可用
* message：额外的消息将被Clang提供当忽略一个警告或者一个错误在使用一个被弃用或者被废弃的声明。对引导用户替换APIs很有用。

> 在声明时可以使用多个availability属性，每个对应不同的平台，仅当availability属性对应相应的目标平台被使用的时候，任何其他才将被忽略。假如没有availability 属性指定可用性对现在的目标平台，availability 属性将被忽略。

#### 支持的平台：
* ios：苹果的iOS操作系统。最小的部署目标被指定通过`-mios-version-min=*version*`或者`-miphoneos-version-min=*version*`命令行参数。
* macosx：苹果的OS X 操作系统，最小的部署目标被指定通过`-mmacosx-version-min=*version*`命令行参数

**overloadable**

> Clang提供对C++函数在C中重载的支持。在C中函数重载被引进使用`overloadable`属性。例如，一个可能提供一个重载版本的`tgsin`函数来精确执行相关的标准函数计算`float`,`double`,`long double`的正弦值：

```objective-c
#include <math.h>
float __attribute__((overloadable)) tgsin(float x) { return sinf(x); }
double __attribute__((overloadable)) tgsin(double x) { return sin(x); }
long double __attribute__((overloadable)) tgsin(long double x) { return sinl(x); }
```
请注意`overloadable`只对函数起作用。你可以重载方法声明在某种范围内通过使用通用的返回值和参数类型，想`id`或者`void *`.

----

上下文是国王当它遇到编译器优化时。通过提供限制在怎样解析你的代码，增加你参数尽可能高效代码的可能性。遇到编译器把你打断，这将是一项奖励。

还有`__attribute__`并不仅仅对编译器有用：下一个人看代码也将感谢这些额外的上下文。所以多走几英尺远将对你的合作中和接替者或者从现在算2年以后的你(那个时候你已经忘记了所以的事情关于这份代码)自己有用

你付出了多少爱,最终你会得到多少爱。


译者注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>
