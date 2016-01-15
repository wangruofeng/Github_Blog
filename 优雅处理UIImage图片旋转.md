# 优雅处理UIImage图片旋转

## UIImag构造方式
UIImag构造方式大致有4种方式
* 从本地bundle中加载 `imageNamed:`，传入一个bundle的文件名即可
* 从本地一个文件路径读取 `imageWithContentsOfFile:`，需要传一个文件的文件路径path
* 通过二进制数据`NSData`来创建`imageWithData:`
* 通过一个`CoreGraphics`的`CGImageRef`来创建，`initWithCGImage:`
* 通过一个`CoreImage`的`CIImage`来创建`initWithCIImage`

通过查阅Apple官网文档我们发现有2个这样的方法，今天就来一探究竟

```objective-c
+ (UIImage *)imageWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(4_0);
+ (UIImage *)imageWithCIImage:(CIImage *)ciImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(6_0);

- (instancetype)initWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(4_0);
- (instancetype)initWithCIImage:(CIImage *)ciImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(6_0);
```

2个类方法2个实例方法都是类似，这里以`CGImageRef`为例

```objective-c
+ (UIImage *)imageWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(4_0);
```

1. 新建的Xcode工程选择single Application
2. 在storyboard中拖一个`UIImageView`设置它水平垂直居中对齐，宽带高度随便设一个值不要太大就行，设置`UIImageView`的`contentMode`为`Aspect Fit`方便查看以免变形
3. 在`UIImageView`下发放一个`UIButton`控件方便后面好对图片进行旋转操作
4. 在viewController中建立一个`UIImageView`引用,拉出一个rotate按钮的`IBAction`

    现在大概界面大概这样
    ![UIImageOrientation效果图](http://ww3.sinaimg.cn/mw690/64124373gw1ezhnagzyhpj20x7105doj.jpg)

5. 下面我们实现
`- (IBAction)rotateImage:(id)sender {}`这个方法

    > 在这里我们想通过点击按钮实现图片旋转
    为了方便使用我们使用Category的方式实现
    新建一个UIImage的分类取名叫Rotate

    > 这里需要传一张要处理的图片和一个待处理成的图片方向
    >
    ```objective-c
    + (UIImage *)rotateImage:(UIImage *)oldImage
             orientation:(UIImageOrientation)orientation;
    ```

    ```objective-c
    + (UIImage *)rotateImage:(UIImage *)oldImage orientation:(UIImageOrientation)orientation{

    UIImage *newImage = [UIImage imageWithCGImage:oldImage.CGImage scale:1 orientation:orientation];

    NSString *orientationStr = nil;
    switch (orientation) {
        case UIImageOrientationUp: {
            orientationStr = @"UIImageOrientationUp";
            break;
        }
        case UIImageOrientationDown: {
            orientationStr = @"UIImageOrientationDown";
            break;
        }
        case UIImageOrientationLeft: {
            orientationStr = @"UIImageOrientationLeft";
            break;
        }
        case UIImageOrientationRight: {
            orientationStr = @"UIImageOrientationRight";
            break;
        }
        case UIImageOrientationUpMirrored: {
            orientationStr = @"UIImageOrientationUpMirrored";
            break;
        }
        case UIImageOrientationDownMirrored: {
            orientationStr = @"UIImageOrientationDownMirrored";
            break;
        }
        case UIImageOrientationLeftMirrored: {
            orientationStr = @"UIImageOrientationLeftMirrored";
            break;
        }
        case UIImageOrientationRightMirrored: {
            orientationStr = @"UIImageOrientationRightMirrored";
            break;
        }

    }

    NSLog(@"current orientation: %@",orientationStr);

    return newImage;
}
    ```

在button点击事件触发时的这样使用

```objective-c
- (IBAction)rotateImage:(id)sender {

    UIImage *oldImage = self.imgView.image;

    UIImage *rotatedImage = [UIImage rotateImage:oldImage orientation:UIImageOrientationLeft];

    self.imgView.image = rotatedImage;
}
```

点击按钮测试发现第一次没问题，但是重逢点击无效
原来`+ (UIImage *)imageWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation`方法执行原理是执行前通过
`@property(nonatomic,readonly) UIImageOrientation imageOrientation;`接口先判断当前图片的方向是否为将要旋转的方向，
如果是就直接返回不做处理，如果不是再作旋转处理，
也就是说这个方法并没有实际上旋转image的数据，只是用一个枚举标记旋转的状态

如果我们想每次旋转需要直接改变原始image的数据该怎么办呢？
在这里我们通过`CGBitmapContext`,使用`CGContextRotateCTM`来设置旋转，
再把UIImage通过`drawInRect
`重新绘制出来，通过`UIGraphicsGetImageFromCurrentImageContext`获得处理后的图片
下面是具体实现

```objective-c
- (UIImage *)fixedRotation{
    if (self.imageOrientation == UIImageOrientationUp) return self;
    CGAffineTransform transform = CGAffineTransformIdentity;

    switch (self.imageOrientation) {
        case UIImageOrientationDown:
        case UIImageOrientationDownMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, self.size.height);
            transform = CGAffineTransformRotate(transform, M_PI);
            break;

        case UIImageOrientationLeft:
        case UIImageOrientationLeftMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, 0);
            transform = CGAffineTransformRotate(transform, M_PI_2);
            break;

        case UIImageOrientationRight:
        case UIImageOrientationRightMirrored:
            transform = CGAffineTransformTranslate(transform, 0, self.size.height);
            transform = CGAffineTransformRotate(transform, -M_PI_2);
            break;
        case UIImageOrientationUp:
        case UIImageOrientationUpMirrored:
            break;
    }

    switch (self.imageOrientation) {
        case UIImageOrientationUpMirrored:
        case UIImageOrientationDownMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, 0);
            transform = CGAffineTransformScale(transform, -1, 1);
            break;

        case UIImageOrientationLeftMirrored:
        case UIImageOrientationRightMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.height, 0);
            transform = CGAffineTransformScale(transform, -1, 1);
            break;
        case UIImageOrientationUp:
        case UIImageOrientationDown:
        case UIImageOrientationLeft:
        case UIImageOrientationRight:
            break;
    }

    // Now we draw the underlying CGImage into a new context, applying the transform
    // calculated above.
    CGContextRef ctx = CGBitmapContextCreate(NULL, self.size.width, self.size.height,
                                             CGImageGetBitsPerComponent(self.CGImage), 0,
                                             CGImageGetColorSpace(self.CGImage),
                                             CGImageGetBitmapInfo(self.CGImage));
    CGContextConcatCTM(ctx, transform);
    switch (self.imageOrientation) {
        case UIImageOrientationLeft:
        case UIImageOrientationLeftMirrored:
        case UIImageOrientationRight:
        case UIImageOrientationRightMirrored:
            // Grr...
            CGContextDrawImage(ctx, CGRectMake(0,0,self.size.height,self.size.width), self.CGImage);
            break;

        default:
            CGContextDrawImage(ctx, CGRectMake(0,0,self.size.width,self.size.height), self.CGImage);
            break;
    }

    // And now we just create a new UIImage from the drawing context
    CGImageRef cgimg = CGBitmapContextCreateImage(ctx);
    UIImage *img = [UIImage imageWithCGImage:cgimg];
    CGContextRelease(ctx);
    CGImageRelease(cgimg);
    return img;

}
```

现在再优化一下原来`+ (UIImage *)rotateImage:(UIImage *)oldImage orientation:(UIImageOrientation)orientation` 方法,修改成这样
```objective-c
+ (UIImage *)rotateImage:(UIImage *)oldImage orientation:(UIImageOrientation)orientation{

    UIImage *newImage = [UIImage imageWithCGImage:oldImage.CGImage scale:1 orientation:orientation];

    //fix original Image with gived orientation.
    UIImage *fixedRotationImage = [newImage fixedRotation];

    NSString *orientationStr = nil;
    switch (orientation) {
        case UIImageOrientationUp: {
            orientationStr = @"UIImageOrientationUp";
            break;
        }
        case UIImageOrientationDown: {
            orientationStr = @"UIImageOrientationDown";
            break;
        }
        case UIImageOrientationLeft: {
            orientationStr = @"UIImageOrientationLeft";
            break;
        }
        case UIImageOrientationRight: {
            orientationStr = @"UIImageOrientationRight";
            break;
        }
        case UIImageOrientationUpMirrored: {
            orientationStr = @"UIImageOrientationUpMirrored";
            break;
        }
        case UIImageOrientationDownMirrored: {
            orientationStr = @"UIImageOrientationDownMirrored";
            break;
        }
        case UIImageOrientationLeftMirrored: {
            orientationStr = @"UIImageOrientationLeftMirrored";
            break;
        }
        case UIImageOrientationRightMirrored: {
            orientationStr = @"UIImageOrientationRightMirrored";
            break;
        }

    }

    NSLog(@"current orientation: %@",orientationStr);

    return fixedRotationImage;
}
```
 现在再测试一下，well，It‘s OK。

 have fun!!!

参考资料：
* [Apple-UIImage Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/#//apple_ref/c/tdef/UIImageOrientation)
* [ios-uiimageview-how-to-handle-uiimage-image-orientation](http://stackoverflow.com/questions/8915630/ios-uiimageview-how-to-handle-uiimage-image-orientation)
* [UIImageOrientation / EXIF orientation sample images](http://www.galloway.me.uk/2012/01/uiimageorientation-exif-orientation-sample-images/)

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>