# Animating the Drawing of a CGPath With CAShapeLayer
* [原文链接](http://oleb.net/blog/2010/12/animating-drawing-of-cgpath-with-cashapelayer/)


# 这是什么？
> 此文将讲解通过形状图层`CAShaperLayer`的`strokeStart`和`strokeEnd`来实现动画绘制路径`CGPath`,此文是[By Ole Begemann](http://oleb.net)创建于December 18, 2010,当时是发布iOS SDK 4.2时`CAShapeLayer`新增加的两个属性[strokeStart](http://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/Reference/Reference.html#//apple_ref/doc/uid/TP40008314-CH1-SW16)和[strokeEnd](http://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/Reference/Reference.html#//apple_ref/doc/uid/TP40008314-CH1-SW15),这两个值是两个浮点数取值范围0.0~1.0,用来表明形状图层所指向的路径在绘制开始和结束路径中的相对位置。

`strokeStart`默认值是0.0，`strokeEnd`默认值是1.0，显然这会导致形状图层的路径将一整个被绘制。假如，你想说，如果设置了layer.strokeEnd = 0.5f,只让她绘制前半部分，那该多好。

真正有趣的事情是这些接口都是可动画的。通过动画绘制`strokeEnd`从0.0到1.0在几秒内，我们就能很容易自己绘制路径像下面这样写：

```objective-c
CABasicAnimation *pathAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
pathAnimation.duration = 10.0;
pathAnimation.fromValue = [NSNumber numberWithFloat:0.0f];
pathAnimation.toValue = [NSNumber numberWithFloat:1.0f];
[self.pathLayer addAnimation:pathAnimation forKey:@"strokeEndAnimation"];
```

最后，再添加第二个图层包含一个铅笔图片，使用关键帧动画` CAKeyframeAnimation`来让它随着这个路径以相同的速度绘制，就可以达到完美的错觉效果：

```objective-c
CAKeyframeAnimation *penAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
penAnimation.duration = 10.0;
penAnimation.path = self.pathLayer.path;
penAnimation.calculationMode = kCAAnimationPaced;
[self.penLayer addAnimation:penAnimation forKey:@"penAnimation"];
```

[绘制普通路径效果视频](http://oleb.net/media/AnimatedPathsHausVomNikolaus.mp4)

[下载地址](http://oleb.net/media/AnimatedPathsHausVomNikolaus.mp4)

这个对文本一样有效；我们只需要把字符转化成`CGPath`。`Core Text`提供了那样的功能的函数，[CTFontCreatePathForGlyph( )](http://developer.apple.com/library/ios/documentation/Carbon/Reference/CTFontRef/Reference/reference.html#//apple_ref/c/func/CTFontCreatePathForGlyph)。为了使用它，我们需要创建一个带属性的字符串用我们想要渲染的文本，先把它们分割成行在分割成一个个字符。在把所有的字符转换成路径后，我们以子路径方式把它添加到一个单个的`CGPath`路径中。更多细节可以查看[Ohmu](http://www.codeproject.com/script/Membership/View.aspx?mid=2887692)写的[Low-level text rendering](http://www.codeproject.com/KB/iPhone/Glyph.aspx)这篇文章。结果看以来非常的炫酷：

[绘制文字路径效果视频](http://oleb.net/media/AnimatedPathsHelloWorld.mp4)

[下载地址](http://oleb.net/media/AnimatedPathsHelloWorld.mp4)

从Github上获得[iPad版的样品工程](http://github.com/ole/Animated-Paths)

# 你将学到的知识点
* 使用`CAShapeLayer`的`strokeStart`和`strokeEnd`来实现路径动画,比较高级复杂的效果像google的下拉刷新转圈就可以从这里引申去实现。
* `CABasicAnimation`和`CABasicAnimation`使用
* 深入理解`CAShapeLayer`和`CALayer`
* 通过文本创建路径，核心函数`CTFontCreatePathForGlyph()`


# 补充说明

```objective-c
CAShapeLayer *pathLayer = [CAShapeLayer layer];
    pathLayer.frame = self.animationLayer.bounds;
    pathLayer.bounds = pathRect;
    pathLayer.geometryFlipped = YES;
    pathLayer.path = path.CGPath;
    pathLayer.strokeColor = [[UIColor blackColor] CGColor];
    pathLayer.fillColor = nil;
    pathLayer.lineWidth = 10.0f;
    pathLayer.lineJoin = kCALineJoinBevel;
    
    [self.animationLayer addSublayer:pathLayer];
```

有一点非常重要，CALayer在iOS系统中相对坐标系是以屏幕左上`top-left`为坐标原点的，在Mac OS X上以坐下`bottom-left`为坐标原点,但是可以通过`CALayer`的接口`geometryFlipped`垂直翻转坐标系，这个值默认是`NO`，设置成`YES`就可以把坐标系转换成左下`bottom-left`了，这里作者使用的左下`bottom-left`的坐标系。


```objective-c
@property(getter=isGeometryFlipped) BOOL geometryFlipped;
```
关于这个属性使用时需要特别注意

1. 翻转会同时作用于它的子图层
2. 即使这个属性设置成`YES`,图片的`orientation`仍然是不变的（也就是说当设置`flipped=YES`和`flipped=NO`时一个`CGImageRef`储存在`contents`接口中的内容将会显示一致，赋值并不会真正变换底层的图层）

### pathLayer动画实现原理
1. 先创建一个动画用的图层`animationLayer`类型`CALayer`，用来充当动画的画布。
2. 创建真正的路径图层`pathLayer`类型为`CAShapeLayer`,让它的坐标系垂直翻转，并且让图层宽高同时向内收缩100个点,通过`CGRectInset(CGRect rect, CGFloat dx, CGFloat dy)`实现
3. 把`pathLayer`添加到`animationLayer`的子图层中去
4. 创建一个铅笔图层`penLayer`类型为`CALayer`,把它添加到`pathLayer`去
5. 对`pathLayer`添加`CABasicAnimation`动画，动画属性为`strokeEnd`
6. 对`penLayer`添加`CAKeyframeAnimation`动画，动画属性为`position`

### textLayer动画实现原理
1. 先创建一个动画用的图层`animationLayer`类型`CALayer`，用来充当动画的画布
2. Create path from text,See:<http://www.codeproject.com/KB/iPhone/Glyph.aspx>，最终保存到一个类型为`CGMutablePathRef`的letter中
3. 通过letter来创建文字`UIBezierPath`类型的`path`
4. 通过path再创建`CAShapeLayer`pathLayer,并且把pathLayer添加到`animationLayer`中去
5. 创建一个铅笔图层`penLayer`类型为`CALayer`,把它添加到`pathLayer`去
5. 对`pathLayer`添加`CABasicAnimation`动画，动画属性为`strokeEnd`
6. 对`penLayer`添加`CAKeyframeAnimation`动画，动画属性为`position`



### 修复一处bug
重复点击`UISegmentedControl`导致铅笔消失，这是设置了` penAnimation.delegate = self;`在代理方法里面没有判断结束直接将设置`self.penLayer.hidden = YES`，导致连续切换时铅笔不见了，要修复这个bug只需加一个判断`    if (flag)   self.penLayer.hidden = YES;
`即可,这样的意思是只有当动画完成时才设置`self.penLayer.hidden`的值，好了现在已经非常完美了，快去动手自己试试吧！🍺












