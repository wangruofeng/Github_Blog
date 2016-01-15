# GET&POST

> GET和POST是两种最常用的与服务器进行交互的HTTP方法 .

### GET
* GET的语义是获取指定的URL资源, 将数据按照 variable = value 的形式, 添加到action所指向的URL后面, 并且两者使用 ' ? '连接, 各变量之间使用 ' & '连接 .
* 对用户来说不安全, 因为在传输过程中, 数据被放在请求的URL中.
* 传输的数据量小, 这主要是因为受URL长度限制.

### URL长度限制

 在http协议中，其实并没有对url长度作出限制，往往url的最大长度和用户浏览器和Web服务器有关，不一样的浏览器，能接受的最大长度往往是不一样的，当然，不一样的Web服务器能够处理的最大长度的URL的能力也是不一样的。

    IE浏览器对URL的最大限制为2083个字符，如果超过这个数字，提交按钮没有任何反应。

    Firefox浏览器URL的长度限制为65,536个字符 ;

    Apache(Server)能够接受的最大URL长度为8192个字符 ;

    如果浏览器的编码为UTF8的话，一个汉字最终编码后的字符长度为9个字符。

GET请求示例

![GetRequest](http://static.oschina.net/uploads/space/2014/0602/164040_3SOT_1774273.png)

### POST
* POST语义是向指定URL的资源添加数据.
* 将数据放在数据体中, 按照变量和值相对应的方式, 传递到action所指向的URL.
* 所有数据对用户来说不可见.
* 可以传输大量数据, 上传文件只能使用POST.

POST请求示例

![PostRequest](http://static.oschina.net/uploads/space/2014/0602/164119_VRKY_1774273.png)

### 在浏览器中判断GET&POST请求

因为POST请求会向服务器发送数据体, 因此刷新页面时会出现提示窗口. 而GET请求不会向服务器发送数据体, 因此没有提示 .

从请求本质来看, GET请求要比POST更安全, 效率也会更高 .(对服务器而言)

### iOS网络发送网络请求的步骤
1. 实例化URL( 网络资源 ) ;

2. 根据URL建立URLRequest ( 网络请求 ) ;

    默认为GET请求; 对于POST请求,  需要创建请求的数据体 .

3. 利用 URLConnection 发送网络请求(发送请求并获得结果) ;

NSURLConnection提供了两个静态方法可以直接以同步或异步的方式向服务器发送网络请求.

   	同步请求:

    sendSynchronousRequest : returningResponse : error :

    异步请求:

    sendAsynchronousRequest : queue : completionHandler :

在网络请求过程中, 接收数据的过程实际上是通过 NSURLConnectionDataDelegate来实现的, 常用代理方法包括:

```objectice-c
// 服务器开始返回数据
-(void)connection:didReceiveResponse:
// 收到服务器返回的数据，本方法会被调用多次
-(void)connection:didReceiveData:
// 数据接收完毕，做数据的最后处理
-(void)connectionDidFinishLoading:
// 网络连接错误
-(void)connection:didFailWithError:
```

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>
