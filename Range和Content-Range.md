## Range and Content-Range

HTTP头中一般断点下载时才用到`Range`和`Content-Range`实体头，
`Range`用户请求头中，指定第一个字节的位置和最后一个字节的位置，如（Range：200-300）
`Content-Range`用于响应头

请求下载整个文件:
---
---
* GET  /test.rar  HTTP/1.1
* Connection:  close
* Host:  116.1.219.219
* Range:  bytes=0-100


1. Range头域可以请求实体的一个或者多个子范围，
2. Range的值为0表示第一个字节，也就是Range计算字节数是从0开始的
3. 表示头500个字节：bytes=0-499
4. 表示第二个500字节：bytes=500-999
5. 表示最后500个字节：bytes=-500
6. 表示500字节以后的范围：bytes=500-
7. 第一个和最后一个字节：bytes=0-0,-1
8. 同时指定几个范围：bytes=500-600,601-999

---
一般正常回应
---
HTTP/1.1 206 OK
Content-Length:  801      
Content-Type:  application/octet-stream
Content-Location: http://www.onlinedown.net/hj_index.htm
Content-Range:  bytes  0-100/2350 //2350:文件总大小
Last-Modified: Mon, 16 Feb 2009 16:10:12 GMT
Accept-Ranges: bytes
ETag: "d67a4bc5190c91:512"
Server: Microsoft-IIS/6.0
Date: Wed, 18 Feb 2009 07:55:26 GMT

*注意：如果用户的请求中含有range ，则服务器的相应代码为206。
206 - Partial Content 客户发送了一个带有Range头的GET请求，服务器完成了它（HTTP 1.1新）。*

备注：欢迎转载，但请一定注明出处！ <https://github.com/wangruofeng/Github_Blog>