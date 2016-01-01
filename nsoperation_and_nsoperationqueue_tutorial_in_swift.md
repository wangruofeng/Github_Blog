# NSOperation and NSOperationQueue Tutorial in Swift

* 译自 Written by [ Richard Turton](http://www.raywenderlich.com/u/jrturton) — on October 7, 2014
* 原文链接：<http://www.raywenderlich.com/76341/use-nsoperation-nsoperationqueue-swift>
* 译者[@oneruofeng](https://twitter.com/oneruofeng)

**Post info**: Updated for Xcode 7.1 and Swift 2.1 -- 1 January 2016

**Update note**: This tutorial was updated to iOS 8, Xcode 6.1 and Swift by Richard Turton. [Original post](http://www.raywenderlich.com/76341/use-nsoperation-nsoperationqueue-swift) by Tutorial Team member [Soheil Azarpour](http://www.raywenderlich.com/u/Canopus).

每个人都有点击一个按钮或者进入一些文本在iOS或者Mac App上的令人沮丧的经历，就是突然-WHAM，用户交互停止了响应。

在Mac上，你的用户开始盯着一个沙漏或者一个七彩的轮子开始旋转直到它们再次恢复UI交互为止。在一个iOS app中，用户期望App立即响应它们的触摸事件，无响应的app给人的感觉是笨重和缓慢，这样通常会导致收到差评。

保持你的app的可交互的状态说起来容易做起来难，一旦你的app需要执行不仅仅是少量的任务，事情很快就变得很复杂，我们并没有多少事件在主事件循环执行繁重的工作并且同时提供一个可以响应的UI。

低级的开发者是怎样做的呢？解决方案就是把工作从主线程中移除通过并发。并发意味着你应用所有的操作同时执行在多个流（或者线程）中--这样的话用户界面就能保持响应的要执行的工作。

一种实现并发操作在iOS是通过`NSOperation`和`NSOperationQueue`这两个类。在这个教程中，你将学会怎样使用它们！你将从一个没有使用并发的App开始，所以它将出现的非常迟钝和无响应。然后你将重做你的应用给它们添加并发操作并且--希望--呈现一个更加响应良好的可交互界面给用户！

### 我们开始吧

这份样品工程的总的目标的就是展示一个滤镜处理过的图片的表视图。图片将从网络上下载，经过一个滤镜处理后，然后在表视图中显示。

下面是这个app模型的示意图

![app模型的示意图](http://ww2.sinaimg.cn/mw690/64124373gw1ezk3rh3ebyj20i20nmq4k.jpg)

<!--
<img src = "http://ww2.sinaimg.cn/mw690/64124373gw1ezk3rh3ebyj20i20nmq4k.jpg" with= 300 height = 600>-->


### 第一个版本尝试
下载你将在这个教程中使用的[第一个版本的项目](http://cdn5.raywenderlich.com/wp-content/uploads/2014/10/ClassicPhotos-Starter63.zip)

> 注意：所以的图片来自[ stock.xchng.](http://sxc.hu/)。一下图片在数据源故意错误命名，以便这里有些例子是图片下载失败好处理失败的情况。

构建并且运行这个工程，最终你将看到这个app运行起来显示一列照片。尝试滚动这个列表，很痛苦，不是吗？
![list of photoes](http://ww3.sinaimg.cn/mw690/64124373gw1ezk45qmdigj205x08wdga.jpg)

*传统的相册，运行缓慢*

所有的操作都发生在`ListViewController.swift`里，并且最主要的是发生在` tableView(_:cellForRowAtIndexPath:)`方法里。看一下那个方法和注释这里有两件相当集中的事情需要思考：

1. `从网络载入图片数据`。即使网络状况良好，app将仍然必须等待直到下载完成了才能继续。
2. 使用*Core Image*给图片加滤镜。这个方法给图片应用一个深褐色的滤镜，假如你想了解更多关于`Core Image`滤镜的知识，请点击[Beginning Core Image in Swift](http://www.raywenderlich.com/76285/beginning-core-image-swift)

还有，你将载入一系列图片请求从网络当它第一次被请求时:

```swift
  lazy var photos = NSDictionary(contentsOfURL:dataSourceURL)
```

所有的工作发生在应用的主线程。由于主线程也负责用户交互，让它一直忙于从网络下载东西和给图片加滤镜消磨掉了响应中的app。你可以通过使用Xcode的仪表测量视图获得这样一个快速的概述。你可以通过显示`调试导航`(Commnad+6)接入这个仪表视图,让后选择`CPU`当app正在运行的时候。

![gauges](http://ww4.sinaimg.cn/mw690/64124373gw1ezk4seggwgj20jg0a0ta0.jpg)

*Xcode的仪表视图表明，主线程的任务非常重*

你会看到那些所有的长钉在`Thread 1`，那就是app的主线程。更多详细的信息你可以运行app的`Instruments`,但是那个在[ whole other tutorial :\].](http://www.raywenderlich.com/?p=23037)中

是时候考虑怎样改善一下你的用户体验了！

### 任务，线程和进程
在你专研这篇教程之前，这里有一下技术上的慨念需要理一下，我将定义一些专用术语：

* **任务** ：一个简单单个的需要完成的工作
* **线程** ：操作系统提供的机制允许多个用户操作同时进行在一个应用中
* **进程** ：一个可执行的代码块，它可以由多个线程构成

> 注意：在iOS和OS X中，线程的功能由POSIX线程API（或者 pthreads）实现，并且它使操作系统的一部分。而这个又是相当底层的东西，你将发现它很容易犯错误；或许关于线程最糟糕的事情是那些很难被发现的错误！

> `Foundation`框架包含了一个叫做`NSThread`的类，这让我们处理事情更容易，但是用`NSThread`管理多个线程仍是一件令人头疼的事情。`NSOperation`和`NSOperationQueue`是为了最大化简化处理多线程的更高级的类。

在这个图解中，你会看到进程，线程和任务之间的关系：

![the relationship between a process, threads, and tasks](http://ww3.sinaimg.cn/mw690/64124373gw1ezk5dwtbhuj20go03n0t2.jpg)

*进程，线程和任务*

正如你所见，一个进程可以包涵多个执行的线程，每个线程能够同时执行多个任务。

在这个图集中, `thread 2`执行文件的读的工作，同时`thread 1`执行UI相关的代码。这和你应该怎样在iOS中构建你的代码（主线程执行任何和UI相关的工作，第二线程应当执行慢的或者长时间运行的耗时操作例如读取文件，接入网络等等）有点相似。

### NSOperation vs. Grand Central Dispatch (GCD)

你应该听说过[Grand Central Dispatch (GCD).](http://developer.apple.com/library/ios/#documentation/Performance/Reference/GCD_libdispatch_Ref/Reference/reference.html)。简单的来说，`GCD`由语言的特征，运行时库和系统增强组成来提供一个在`iOS`和`OS X`多核硬件中支持并发系统并且综合的改良。假如你想了解更多关于GCD相关的知识，有可以阅读我们的[Multithreading and Grand Central Dispatch on iOS for Beginners Tutorial](http://www.raywenderlich.com/?p=4295)。

`NSOperation`和`NSOperationQueue`构建在GCD之上。普遍来说，苹果推荐使用最高级别抽象，当需要显示他们一些需要的测量工作时回到最底层。

下面是关于这两者的一些简单比较，将帮助你决定何时何地选择使用`GCD`或者`NSOperation`：

* **GCD** 是一个轻量级的方式来描述将要被并发执行的工作单元。你不必定制这个工作单元的时刻表；系统为你定制时刻表。在blocks中增加依赖是一件头疼的事情。取消或者暂停一个block来进行额外的工作为作为开发者的你！ :]

* **NSOperation** 和GCD相比增加了一些额外开支，但是你能够在各种操作之间增加依赖并且恢复，取消，暂停他们。

这个教程将使用`NSOperation`,因为你将处理一个列表为了好的表现并且由于它大量的消耗的资源你需要能够取消一个操作针对某个图片，假如用户已经将那张图片滚出屏幕。即使这些操作在后台线程，假如这里有一打事情在队列里等着它们去处理，这将表现得跟糟糕。

### 重构 App Model

是时候重构开始的非多线程的模型了！假如你仔细观察先前的模型，你会发现这里有三个可以被改进的线程受困区域。通过切割这三个区域让后把他们放在单独的线程里，主线程的压力将得到缓解并且能够保持和用户交互。

![NSOperation_model_improved](http://ww3.sinaimg.cn/mw690/64124373gw1ezk6m9a5xzj20i20nm416.jpg)

*改进后的 model*

为了摆脱你应用的瓶颈，你需要一个指定一个线程来响应你的用户时间，一个线程专注于下载资源和图片，一个线程执行图片滤镜操作。在新的模型中，app从主线程启动让后载入一个空的表视图。同时，app启动第二个线程开始下载数据资源。

一旦数据资源下载完成，你将通知表视图重新载入。这些事情必须在主线程完成，因为它涉及到用户界面相关的操作。从这一点来说，表视图知道有多少行，并且它知道他将显示图片的URL地址，但是她不知道它是否真的有图片！假如你立即开始下载所有的图片在这个点上，这可能导致效率极其低下，因为你不需要一次性把所有图片下载完！

为了让这个变得更好我们能够做什么？

一个更好的模型就是可交互的行在屏幕范围内可见时才开始下载图片。所以你的代码开始将询问表视图有多少行可见。因此，代码应该直到这里有一个未加滤镜的图片等待处理时才开始处理图片加滤镜操作。

为了使app更加快速响应，代码将要让图片一旦下载完毕立即显示。让后才开始进行图片加滤镜操作，让后更新UI界面来显示已经经过滤镜处理后的图片。下面的图标显示了这个过程的控制流：

![Control Flow](http://ww3.sinaimg.cn/mw690/64124373gw1ezk75nd4bdj20i206yq3q.jpg)

*Cotroll Flow*

为了获得这些对象，你需要跟踪这张图片现在是否正在被下载，一旦完成下载，假如图片的滤镜被应用上。你需要跟跟踪每个操作的状态，它是否正在下载中或者执行滤镜操作，以便你能够取消，暂停或者恢复每个操作当用户滚动的时候。

Okey！ 现在你准备好开始码代码了！ :]

打开你下载的工程，添加一个新的**Swift File**到你的工程中命名为** PhotoOperations.swift**。添加下面代码：


```swift
class PendingOperations {
  lazy var downloadsInProgress = [NSIndexPath:NSOperation]()
  lazy var downloadQueue:NSOperationQueue = {
    var queue = NSOperationQueue()
    queue.name = "Download queue"
    queue.maxConcurrentOperationCount = 1
    return queue
    }()

  lazy var filtrationsInProgress = [NSIndexPath:NSOperation]()
  lazy var filtrationQueue:NSOperationQueue = {
    var queue = NSOperationQueue()
    queue.name = "Image Filtration queue"
    queue.maxConcurrentOperationCount = 1
    return queue
    }()
}
```

这回类包含了2个字典为了跟踪激活和正在进行中的下载和滤镜操作对表中的每一行，并且两个操作队列都有各自的操作类型。

所有的值被懒加载的方式创建，意味着他们不会被初始化知道他们第一次被接入。这样改善了你app的表现性能。

创建一个`NSOperationQueue`是非常简单的，正如你所见，给你的队列命名是非常有用的，因为名字会在仪器或者调速器中显示。`maxConcurrentOperationCount`在这里由于这个教程的缘故被设置成1，是为了让你看到操作一个接一个完成。你可以离开这一部分允许队列决定他一次处理多少个操作--这样会进一步改善性能。

队列是怎样决定一次运行多少个操作的呢？这是一个非常好的问题！ :] 这取决于硬件。默认，`NSOperationQueue`将要处理一写计算在屏幕背后，决定什么是最好的需要看代码是运行在某个具体的平台，和将载入的最大数量的线程数。

考虑到下面的例子，假设系统此时是空闲的，这里有很多资源可用，所以这个队列能够载入可能8条并发的线程。下一个时刻你运行程序，系统可能忙于其他不相关的正在抢夺资源的操作，这时队列就仅仅载入2个并发的线程。因为你已经设置了一个最大并发操作数，在这个app中一次只会进行一个操作。

> 注意：你可能想知道为什么你必须跟踪所有的激活的和正在进行中操作。队列有一个`operations`的方法，它将返回一个操作的数组，所有为什么不用它呢？在这个工程中这样做效果不是很好。你需要跟踪更表视图行数关联的操作，它可能会重复执行数组每次你需要一个的时候。把它们储存在一个字典中用index path来作为他的key方便快速和高效的查找。

是时候考虑下载和过滤操作了。添加下列代码到` PhotoOperations.swift:`文件的末尾：

```swift
class ImageDownloader: NSOperation {
  //1
  let photoRecord: PhotoRecord

  //2
  init(photoRecord: PhotoRecord) {
    self.photoRecord = photoRecord
  }

  //3
  override func main() {
    //4
    if self.cancelled {
      return
    }
    //5
    let imageData = NSData(contentsOfURL:self.photoRecord.url)

    //6
    if self.cancelled {
      return
    }

    //7
    if imageData?.length > 0 {
      self.photoRecord.image = UIImage(data:imageData!)
      self.photoRecord.state = .Downloaded
    }
    else
    {
      self.photoRecord.state = .Failed
      self.photoRecord.image = UIImage(named: "Failed")
    }
  }
}
```

`NSOperation`是一个抽象类，为它的子类而设计。每个子类代表了一个特别的`任务`正如呈现在列表早期那样。

下面是上面代码每行注释到底发生了什么的说明：

1. 添加一个常量引用到和操作相关的`PhotoRecord`对象
2. 创建一个设计初始化方法允许`photo record`参数可以被传进来
3. `main`是你在`NSOperation`子类中需要重写的方法，用来执行相关工作
4. 在启动开始前检查是否被取消，操作应该定期检查是否已经被取消在尝试长时间或密集的工作之前
5. 下载图片数据
6. 再次检查是否被取消
7. 假如这里有数据，创建一个图片对象然后把它添加到记录中，同时改变它的状态，假如这里没有数据，把这条记录标记成失败然后设置适当的图片

下一步，你将创建另一个操作来处理图片加滤镜的操作！添加下面的代码到`PhotoOperations.swift`文件末尾：

```swift
class ImageFiltration: NSOperation {
  let photoRecord: PhotoRecord

  init(photoRecord: PhotoRecord) {
    self.photoRecord = photoRecord
  }

  override func main () {
    if self.cancelled {
      return
    }

    if self.photoRecord.state != .Downloaded {
      return
    }

    if let filteredImage = self.applySepiaFilter(self.photoRecord.image!) {
      self.photoRecord.image = filteredImage
      self.photoRecord.state = .Filtered
    }
  }
}
```
除了你对图片应用滤镜（使用一个未实现的方法，因此编译1错误）而不是下载它之外，这个看起来和下载操作非常相像。

添加遗失的图片滤镜处理方法到`ImageFiltration`类中:

```swift
(image:UIImage) -> UIImage? {
  let inputImage = CIImage(data:UIImagePNGRepresentation(image))

  if self.cancelled {
    return nil
  }
  let context = CIContext(options:nil)
  let filter = CIFilter(name:"CISepiaTone")
  filter.setValue(inputImage, forKey: kCIInputImageKey)
  filter.setValue(0.8, forKey: "inputIntensity")
  let outputImage = filter.outputImage

  if self.cancelled {
    return nil
  }

  let outImage = context.createCGImage(outputImage, fromRect: outputImage.extent())
  let returnImage = UIImage(CGImage: outImage)
  return returnImage
}
```

图片添加滤镜操作使用先前`ListViewController`中相同的实现。已经把它移动到这里来了以至于它它能在后台的一个单独的操作中完成。再次强调，你应该非常频繁的检查取消操作；最佳实践是在进行任何耗时操作调用之前和之后。一旦加滤镜操作完成，你应该立即设置photo record 实例的值。

很棒！现在你已经有了所有的工具和基础为了处理后台任务进程的操作。是时候回到控制器修改它以便能利用所有这些新的福利。

切换到`ListViewController.swift`文件，然后删除`lazy var photos`接口声明。添加下面声明：

```swift
var photos = [PhotoRecord]()
let pendingOperations = PendingOperations()
```

这些将持有一个数组的你在开始创建的`PhotoDetails`对象，`PendingOperations`对象来管理操作。

添加一个下载`photos property`列表新的方法到类中：
```swift
func fetchPhotoDetails() {
  let request = NSURLRequest(URL:dataSourceURL!)
  UIApplication.sharedApplication().networkActivityIndicatorVisible = true
 
  NSURLConnection.sendAsynchronousRequest(request, queue: NSOperationQueue.mainQueue()) {response,data,error in
    if data != nil {
      let datasourceDictionary = NSPropertyListSerialization.propertyListWithData(data, options: Int(NSPropertyListMutabilityOptions.Immutable.rawValue), format: nil, error: nil) as! NSDictionary
 
      for(key : AnyObject,value : AnyObject) in datasourceDictionary {
        let name = key as? String
        let url = NSURL(string:value as? String ?? "")
        if name != nil && url != nil {
          let photoRecord = PhotoRecord(name:name!, url:url!)
          self.photos.append(photoRecord)
        }
      }
 
      self.tableView.reloadData()
    }
 
    if error != nil {
      let alert = UIAlertView(title:"Oops!",message:error.localizedDescription, delegate:nil, cancelButtonTitle:"OK")
      alert.show()
    }
    UIApplication.sharedApplication().networkActivityIndicatorVisible = false
  }
}
```

这个方法创建一个异步的网络请求，当完成的时候，将执行`completion block`在主线程。当下载完成property list的数据被萃取成一个`NSDictionary`,然后再次处理一个数组的`PhotoRecord`的对象。你不能直接使用这里的`NSOperation`,你应该在主线程中接入它使用`NSOperationQueue.mainQueue()`。

在`viewDidLoad`调用新方法
> fetchPhotoDetails()

下一步找到` tableView(_:cellForRowAtIndexPath:)`然后替换它用下面的实现：

```swift
override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCellWithIdentifier("CellIdentifier", forIndexPath: indexPath) as! UITableViewCell
 
  //1
  if cell.accessoryView == nil {
    let indicator = UIActivityIndicatorView(activityIndicatorStyle: .Gray)
    cell.accessoryView = indicator
  }
  let indicator = cell.accessoryView as! UIActivityIndicatorView
 
  //2
  let photoDetails = photos[indexPath.row]
 
  //3
  cell.textLabel?.text = photoDetails.name
  cell.imageView?.image = photoDetails.image
 
  //4
  switch (photoDetails.state){
  case .Filtered:
    indicator.stopAnimating()
  case .Failed:
    indicator.stopAnimating()
    cell.textLabel?.text = "Failed to load"
  case .New, .Downloaded:
    indicator.startAnimating()
    self.startOperationsForPhotoRecord(photoDetails,indexPath:indexPath)
  }
 
  return cell
}
```

花点时间来通读注释区域下面的解释：

1. 为了给用户提供反馈，创建`UIActivityIndicatorView`让后把它设成cell的accessory view。
2. 数据源包含`PhotoRecord`实例。抓取正确的数据基于当前行的`indexPath`。
3. cell的文本标签总是一样的，图片被正确的设置在`PhotoRecord`中当它被处理的时候，以便你能够设置他们两者，而不管记录的状态。
4. 检查记录，正确的设置activity indicator和文本，然后开始操作（暂时还没实现）

你可以移除`applySepiaFilter`的实现，因为那个将不再被调用了，添加下面的方法到类中来开始操作：

```swift
func startOperationsForPhotoRecord(photoDetails: PhotoRecord, indexPath: NSIndexPath){
  switch (photoDetails.state) {
  case .New:
    startDownloadForRecord(photoDetails, indexPath: indexPath)
  case .Downloaded:
    startFiltrationForRecord(photoDetails, indexPath: indexPath)
  default:
    NSLog("do nothing")
  }
}
```

这里，你将传递一个`PhotoRecord`类型的实例带有它的`index path`。
依据`photo record`的状态，你选开始进行下载还是加滤镜的步骤。

> 注意：下载图片和给图片加滤镜的方法分开实现，因为这里有可能出现当一个图片正在被下载，用户可能把它滚开了，这样你就不必对它应用滤镜操作。当下一次用户又来到相同的哪行时，你不必重新下载图片；你只需对它应用
图片滤镜即可！ Efficiency rocks! :]

现在你需要实现你在上面调用的方法。记住你创建的自定义的类，`PendingOperations`,保持跟踪操作；现在实际上你可以使用它了！添加下面的方法到类中：

```swift
func startDownloadForRecord(photoDetails: PhotoRecord, indexPath: NSIndexPath){
  //1
  if let downloadOperation = pendingOperations.downloadsInProgress[indexPath] {
    return
  }
 
  //2
  let downloader = ImageDownloader(photoRecord: photoDetails)
  //3
  downloader.completionBlock = {
    if downloader.cancelled {
      return
    }
    dispatch_async(dispatch_get_main_queue(), {
      self.pendingOperations.downloadsInProgress.removeValueForKey(indexPath)
      self.tableView.reloadRowsAtIndexPaths([indexPath], withRowAnimation: .Fade)
    })
  }
  //4
  pendingOperations.downloadsInProgress[indexPath] = downloader
  //5
  pendingOperations.downloadQueue.addOperation(downloader)
}
 
func startFiltrationForRecord(photoDetails: PhotoRecord, indexPath: NSIndexPath){
  if let filterOperation = pendingOperations.filtrationsInProgress[indexPath]{
    return
  }
 
  let filterer = ImageFiltration(photoRecord: photoDetails)
  filterer.completionBlock = {
    if filterer.cancelled {
      return
    }
    dispatch_async(dispatch_get_main_queue(), {
      self.pendingOperations.filtrationsInProgress.removeValueForKey(indexPath)
      self.tableView.reloadRowsAtIndexPaths([indexPath], withRowAnimation: .Fade)
      })
  }
  pendingOperations.filtrationsInProgress[indexPath] = filterer
  pendingOperations.filtrationQueue.addOperation(filterer)
}
```

Okey!下面是一个快速列表来帮助你立即上面的代码到底做了什么：

1. 首先，检查特定的`indexPath`来看这里是否已经有一个操作在`downloadsInProgress`中。假如有，忽略它。
2. 假如没有，通过`designated initializer`创建一个`ImageDownloader`实例
3. 添加一个当操作完成时执行的完成block。这是最好的地方让你其余的应用知道一个操作已经完成。注意完成block必须被执行即使这个操作被取消，这相当的重要，所以你需要使用GCD来触发重新载入表视图在主线程。
4. 添加操作到`downloadsInProgress`中来保持跟踪一些事情。
5. 添加操作到下载队列中。这是你实际获取这些操作开始运行的方法--队列需要注意的是一旦你添加了操作它就会执行。

过滤图片的方法遵循下面相同的类型，除了它使用`ImageFiltration`和`filtrationsInProgress`来跟踪操作。作为经验，你应该尝试不要重复这个区域的代码 :]

你做到了！你的工程完成了。构建让后运行看有什么改进在操作上！当你滚动表视图的时候，app并没有停止，还是像它们变得可见一样继续下载图片和给图片加滤镜操作。

![classicphotos-stalled-screenshot](http://ww2.sinaimg.cn/mw690/64124373gw1ezkc6fh23sj205x08wdga.jpg)

*原来的图片，现在可以滚动了*

是不是很酷？你能看到随做你的进步让你的应用更易于响应能做出的努力 --对用户来说更有趣！

### 细微的调整

你已经随着这个教程走过漫长的路！你的小项目现在反应灵敏表明了在原来的版本基础上有很多改进。然而，这里任然有一些遗留的细节我们需要考虑。你是想成为一名伟大的程序员，而不仅仅是一名优秀的程序员！

你可能已经注意到了当你滚动表视图时，那些离屏的cell仍然在处理下载和给图片加滤镜的操作。假如你快速滚动，app将忙于下载和给图片处理滤镜的操作，在列表中从最前面甚至到不可见的地方。理想的情况下app应该对离开屏幕的cells就是现在不可见的取消滤镜操作。

难道你没有把取消的规定放进你的代码里？ 是的，你做了--现在你应该充分利用它们！:]

回到Xcode，让后打开`ListViewController.swift`文件。去到`tableView(_:cellForRowAtIndexPath:)`方法的实现，封装`startOperationsForPhotoRecord`调用在一个if条件向下面的：

```swift
if (!tableView.dragging && !tableView.decelerating) {
  self.startOperationsForPhotoRecord(photoDetails, indexPath: indexPath)
}
```
你需要告诉表视图开始操作仅仅当表视图没有滚动的时候。这实际上是`UIScrollView`的接口，由于`UITableView`是`UIScrollView`的子类，你自动继承了这些接口。

下一步，添加到下面`UIScrollView`代理方法的实现到类中：

```swift
override func scrollViewWillBeginDragging(scrollView: UIScrollView) {
  //1
  suspendAllOperations()
}
 
override func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
  // 2
  if !decelerate {
    loadImagesForOnscreenCells()
    resumeAllOperations()
  }
}
 
override func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
  // 3
  loadImagesForOnscreenCells()
  resumeAllOperations()
}
```

快速走查上面的代码显示在下面：

1. 当用户开始滚动，你想暂停所有的操作让后看看用户想看到什么。你将要实现`suspendAllOperations`在一会儿工夫。
2. 假如`decelerate`的值是`false`,那就意味着停止拖拽表视图，因此你想恢复暂停的，因为cell离开屏幕取消的操作，启动在屏幕内cells的操作。你将要一起实现`loadImagesForOnscreenCells`和`resumeAllOperations`。
3. 代理方法告诉你表视图已经停止滚动，所以你将做和#2条相同的处理。

现在，添加下面这些遗失的方法的实现到`ListViewController.swift`:

```swift
func suspendAllOperations () {
  pendingOperations.downloadQueue.suspended = true
  pendingOperations.filtrationQueue.suspended = true
}
 
func resumeAllOperations () {
  pendingOperations.downloadQueue.suspended = false
  pendingOperations.filtrationQueue.suspended = false
}
 
func loadImagesForOnscreenCells () {
  //1
  if let pathsArray = tableView.indexPathsForVisibleRows() {
    //2
    var allPendingOperations = Set(pendingOperations.downloadsInProgress.keys.array)
    allPendingOperations.unionInPlace(pendingOperations.filtrationsInProgress.keys.array)
 
    //3
    var toBeCancelled = allPendingOperations
    let visiblePaths = Set(pathsArray as! [NSIndexPath])
    toBeCancelled.subtractInPlace(visiblePaths)
 
    //4
    var toBeStarted = visiblePaths
    toBeStarted.subtractInPlace(allPendingOperations)
 
    // 5
    for indexPath in toBeCancelled {
      if let pendingDownload = pendingOperations.downloadsInProgress[indexPath] {
        pendingDownload.cancel()
      }
      pendingOperations.downloadsInProgress.removeValueForKey(indexPath)
      if let pendingFiltration = pendingOperations.filtrationsInProgress[indexPath] {
        pendingFiltration.cancel()
      }
      pendingOperations.filtrationsInProgress.removeValueForKey(indexPath)
    }
 
    // 6
    for indexPath in toBeStarted {
      let indexPath = indexPath as NSIndexPath
      let recordToProcess = self.photos[indexPath.row]
      startOperationsForPhotoRecord(recordToProcess, indexPath: indexPath)
    }
  }
}
```

`suspendAllOperations`和`resumeAllOperations`有一个简单的实现。`NSOperationQueues`能够被暂停，通过设置`suspended`接口为`true`。这将暂停队列里面的所有操作--你不能单独暂停一个操作。

`loadImagesForOnscreenCells`有一点小复杂。这里发生了什么事？

1. 以一个包含了现在表视图可见的`index paths`的数组开始开始
2. 构造一个所有进行中的操作的集合通过结合所有在下载的进度+所有在处理滤镜的进度。
3. 构造一个`index paths`集合用来取消操作。开始所有的操作，然后移除可见行的`index paths`，这样讲留下一个离开屏幕的行正在执行的操作集合
4. 构造一个`index paths`集合，需要操作启动，用所有可见行的`index paths`启动，让后移除它们中在进行的操作。
5. 遍历那些被取消的操作，取消它们，然后移除它们的引用从`PendingOperations`。
6. 遍历那些将要开始的操作，然后对他们每个调用`startOperationsForPhotoRecord`。


构建运行然后你发现一个更流畅，资源管理德更好的应用！给你自己一轮掌声！

![improved app](http://ww2.sinaimg.cn/mw690/64124373gw1ezkdl5q55nj20jg09qq4h.jpg)

*原来的相册，载入东西一次一个*

注意到当你完成滚动表视图，在可见区域行的cell的图片立即开始处理。

### 何去何从？
这里是[completed version of the project](http://cdn2.raywenderlich.com/wp-content/uploads/2014/10/ClassicPhotos-Final63.zip)。

> 注意：此教程写于`Update 17 April 2015: Updated for Xcode 6.3 and Swift 1.2`,现在Swift最新版本2.1使用Xcode7+以上编辑会报错，这里打包一个新语法修改版[completed fixed version of the project](https://github.com/wangruofeng/ClassicPhotos-Starter-fixed/archive/master.zip)。
> 
> 

假如你完成这个工程应该花时间来真正理解它，恭喜你！你可以认为你自己是一位更有价值iOS开发者了比起在教程刚开始的时候！大多数开发的公司是非常幸运的有一个或者两个人正在知道这个东西。

当时请当心 -- 像多层嵌套的blocks,不必要的使用多线程可能让一个工程变得难以理解对维护你代码的人来说。线程可能引入一些难以捉摸的bugs，将永远不出现知道你网络非常慢，或者代码运行在一个更快（或更慢）的设备上，或一个不同数量的核的芯片上时。小心测试，尽量使用`Instruments`（或者你自己的观察）来确定引入多线程真的有很大改进。

一个有用的特征使用操作时在这里没涉及到就是依赖（`dependency`）。你可以给一个操作添加一个或者更多的操作的依赖。这个操作不会开始直到它所有依赖的操作完成时。例如：

```swift
// MyDownloadOperation is a subclass of NSOperation
let downloadOperation = MyDownloadOperation()
// MyFilterOperation  is a subclass of NSOperation
let filterOperation = MyFilterOperation()
 
filterOperation.addDependency(downloadOperation)
```

移除依赖：

```swift
filterOperation.removeDependency(downloadOperation)
```

这个工程是否能使用依赖简化呢？把你学到的新技能用起来试一试 ：]
有件非常重要的事需要注意的就是一个依赖操作将仍然启动假如它依赖的操作被取消，还有它将自然完成。你需要牢记在心。

假如你有任何评论或者问题关于这个教程或者`NSOperations`,请加[Pull request](https://github.com/wangruofeng/Github_Blog/pulls)。





