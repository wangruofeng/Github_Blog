# HTTP POST And Multipart Forms
参考资料：

* [Sending Multipart Forms with Objective-C](http://nthn.me/posts/2012/objc-multipart-forms.html)
* [POST multipart/form-data with Objective-C](http://stackoverflow.com/questions/24250475/post-multipart-form-data-with-objective-c)
* [Sending an HTTP POST request on iOS](http://stackoverflow.com/questions/15749486/sending-an-http-post-request-on-ios)
* [The Multipart Content-Type--w3规范](http://www.w3.org/Protocols/rfc1341/7_2_Multipart.html)

注意事项：

* http Body 中的NSData 编码方式要用`NSASCIIStringEncoding`而不是`NSUTF8StringEncoding`
* 通过

	`NSString *postLength = [NSString stringWithFormat:@"%d",[postData length]];`		
计算数据的长度

#### POST参数设置

```objective-c
		//设置header Content-Length
		[request setValue:postLength forHTTPHeaderField:@"Content-Length"];
		//设置header contentType
		[request setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Current-Type"];
		//设置body
		[request setHTTPBody:postData];
```

> 备注:普通`post`的`header`的`Current-Type`为`application/x-www-form-urlencoded`

#### Multipart Forms POST参数设置

```objective-c
		//设置header contentType
		NSString *contentType = [NSString stringWithFormat:@"multipart/form-data; boundary=%@", boundary];
		[request addValue:contentType forHTTPHeaderField:@"Content-Type"];

		//设置body contentType
		[body appendData:[@"Content-Type: application/octet-stream\r\n\r\n" dataUsingEncoding:NSUTF8StringEncoding]];
```

> 备注:`Multipart Forms`的`header`的`Current-Type`为`multipart/form-data`

request body like this

		--YOUR_BOUNDARY_STRING
		Content-Disposition: form-data; name="photo"; filename="calm.jpg"
		Content-Type: image/jpeg

		YOUR_IMAGE_DATA_GOES_HERE
		--YOUR_BOUNDARY_STRING
		Content-Disposition: form-data; name="message"

		My first message
		--YOUR_BOUNDARY_STRING
		Content-Disposition: form-data; name="user"

		1
		--YOUR_BOUNDARY_STRING

I’m sending over three variables: an image named photo, a string named message, and an integer named user. It’s important to note the linebreaks and the dashes before the boundary string. These must be included in order to build a good request. Now lets write some objective-c:


```objective-c

	NSString *boundary = @"YOUR_BOUNDARY_STRING";
	NSString *contentType = [NSString stringWithFormat:@"multipart/form-data; boundary=%@", boundary];
	[request addValue:contentType forHTTPHeaderField:@"Content-Type"];
	
	NSMutableData *body = [NSMutableData data];
	
	[body appendData:[[NSString stringWithFormat:@"\r\n--%@\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];
	[body appendData:[[NSString stringWithFormat:@"Content-Disposition: form-data; name=\"photo\"; filename=\"%@.jpg\"\r\n", self.message.photoKey] dataUsingEncoding:NSUTF8StringEncoding]];
	[body appendData:[@"Content-Type: application/octet-stream\r\n\r\n" dataUsingEncoding:NSUTF8StringEncoding]];
	[body appendData:[NSData dataWithData:imageData]];
	
	[body appendData:[[NSString stringWithFormat:@"\r\n--%@\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];
	[body appendData:[[NSString stringWithFormat:@"Content-Disposition: form-data; name=\"message\"\r\n\r\n%@", self.message.message] dataUsingEncoding:NSUTF8StringEncoding]];
	
	[body appendData:[[NSString stringWithFormat:@"\r\n--%@\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];
	[body appendData:[[NSString stringWithFormat:@"Content-Disposition: form-data; name=\"user\"\r\n\r\n%d", 1] dataUsingEncoding:NSUTF8StringEncoding]];
	
	[body appendData:[[NSString stringWithFormat:@"\r\n--%@\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];
	
	[request setHTTPBody:body];

```


Now all we need to do is make a connection to the server and send the request:

	[request setHTTPBody:body];

```objective-c
	NSURLResponse *response;
	NSError *error;
	
	[NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
```

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>
