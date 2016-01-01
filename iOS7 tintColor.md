
# iOS7 tintColor 详解

### 为什么需要出现tintColor ？
> 解决以前不方便统一设置视图颜色风格的通点，方便自定义系统控件外观
 跟`UIAppearance`协议设计有点类似，`UIAppearance`是为了方便统一 设置一类控件的外观，而`tintColor`是为方便设置某个控件的外观，或者 说某个容器内所有控件的风格。

 > 像在`UIViewController` 中，通过这段代码可以实现容器内，所有的子view风格统一化,这样在这个控制器中的所有子view都会以`tintColor`显示
 > ```objective-c
 > self.view.tintAdjustmentMode = UIViewTintAdjustmentModeNormal
 > ```


`UIView`的`tintAdjustmentMode`需要说明一下，这是一个`UIViewTintAdjustmentMode`枚举，
* UIViewTintAdjustmentModeAutomatic（着色调整模式自动）
* UIViewTintAdjustmentModeNormal（着色调整模式正常）
* UIViewTintAdjustmentModeDimmed（着色调整模式变暗，打开控风格会变成灰白模式）



### 先看看官方API说明
![tintColor官方说明](http://ww3.sinaimg.cn/mw690/64124373gw1eyvl7cyzvxj20du0kpn43.jpg)

> iOS7以后所有的UIView以及它的子类都新增了一个叫tintColor的接口，方便我们对视图进行颜色调整


### 注意事项
> `UIImageView`需要设置`renderingMode`为`UIImageRenderingModeAlwaysTemplate`才能生效。
`renderingMode`是一个类型为`UIImageRenderingMode`的枚举
* UIImageRenderingModeAutomatic （默认渲染模式，自动模式）
* UIImageRenderingModeAlwaysOriginal（总是绘制原来的图片，不把它当成临时图片来处理）
* UIImageRenderingModeAlwaysTemplate （总是绘制临时图片，会忽略它原本的颜色信息，也就是根据`tintColor`生产图片）

### UIImageView的使用
```objective-c
UIImage *image = [UIImage imageNamed:@"xxx.png"];
image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
```

### - (void)tintColorDidChange的使用
在子类中重写`- (void)tintColorDidChange` 方法，就可以实现每次更新`tintColor`的时候调用相关配置
```objective-c
 - (void)tintColorDidChange
 {
    _tintColorLabel.textColor = self.tintColor;
    _tintColorBlock.backgroundColor = self.tintColor;
 }
```


### 参考链接 [Demo - iOS7 tintColor day by day](https://www.shinobicontrols.com/blog/ios7-day-by-day-day-6-tint-color)
