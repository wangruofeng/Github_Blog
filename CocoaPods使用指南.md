#   CocoaPods安装和使用
CocoaPods是iOS最常用的第三方类库管理工具，绝大部分有名的开源类库
支持CocoaPods.
CocoaPods是用Ruby实现的，要使用它首先需要有Ruby的环境。幸亏是
OS X系统默认已经可以运行Ruby了，我么只需执行以下命令：

```objective-c
sudo gem install cocoapods
```

由于某些原因，执行时会出现下面的错误提示：

```objective-c
ERROR :Could not find a valid gem `cocoapods` (>= 0), here is why:
        Unable to download data from https://rubygems.org/ - Errno::EPIPI:
        Broken pipe - SSL_connect
(https://rubygems.org/lastest_specs.4.8.gz)
```

安装成功后，接着执行命令：

```objective-c
pod setup
```

如果Ruby环境不够新，可能需要更新以下：

```objective-c
sudo gem update --system
```

至此安装就完成了，我们可以尝试搜索一个第三方类库：

```objective-c
pod search AFNetworking
```

使用CocoaPods第一步，是在当前项目下，新疆一个Podfile文件：

```objective-c
touch Podfile
```

然后利用vim打开Podfile文件编辑，加入你想要的类库，格式如下：

```objective-c
platform :ios
pod  'Reachability', '3.1.0'

platform :ios, '6.0'
pod 'JSONKit', '1.4'
pod 'AFNetworking', '~> 2.3,1'
```

如果是拷贝别人的项目，或是一个很久没打开过的项目，可能需要先执行一下：

```objective-c
pod update
```

最后一步，执行命令：

```objective-c
pod install
```

当终端出现类似下面的提示后，就代表成功了：

```objective-c
[!] From now no use `Sample0814.xcworkspace`.
```

这个时候会看到项目文件夹多了一个xxx.xcworkspace,以后要通过这个文件
打开项目，老项目xxx.xcodeproj不再使用。

> 1. 上面的每一步都可能出现问题，但大部分问题都是因为局域网的原因，用一个网速稳
定的境外VPN可破
> 2. 如果上面因为权限问题安装失败，必须每次都要删除
> 
```objective-c
rm -rf /User/loginname/Library/Caches/CocoaPods/
```
因为这个缓冲中会存下你的github的东西，造成每次调用上次权限问题的缓存。
> 3. 关于Podfile文件编辑时，第三方版本号的各种写法:


```objective-c
pod ‘AFNetworking’      //不显式指定依赖库版本，表示每次都获取最新版本
pod ‘AFNetworking’,  ‘2.0’     //只使用2.0版本
pod ‘AFNetworking’, ‘>2.0′     //使用高于2.0的版本
pod ‘AFNetworking’, ‘>=2.0′     //使用大于或等于2.0的版本
pod ‘AFNetworking’, ‘<2.0′     //使用小于2.0的版本
pod ‘AFNetworking’, ‘<=2.0′     //使用小于或等于2.0的版本
pod ‘AFNetworking’, ‘~>0.1.2′     //使用大于等于0.1.2但小于0.2的版本，相当于>=0.1.2并且<0.2.0
pod ‘AFNetworking’, ‘~>0.1′     //使用大于等于0.1但小于1.0的版本
pod ‘AFNetworking’, ‘~>0′     //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本
```
