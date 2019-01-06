# HTTP协议

 浏览器输入URL后HTTP请求返回的完整过程
 ![](http://p7b5cwgjy.bkt.clouddn.com/15405451264177.jpg)
# HTTP协议基础及发展史

## 5层模型
![](http://p7b5cwgjy.bkt.clouddn.com/15405454020075.jpg)
### 低三层
物理层：定义物理设备如何传输数据
数据链路层：在通信的实体间建立数据链路连接
网络层：为数据在结点之间传输创建逻辑链路

### 传输层
向用户提供可靠地端到端（End-to-End）服务
传输层向高层屏蔽了下层数据通信的细节

### 应用层
为应用软件提供很多服务
构建于TCP协议之上
屏蔽网络传输相关细节

## HTTP 协议发展史
### HTTP/0.9
只有一个GET请求
没有HEADER等描述数据的信息
服务器发送完毕，就关闭TCP连接

### hTTP/1.0
增加了 POST、PUT、HEADER等命令
增加了status code和header等内容
多字符集支持、多部分发送、权限、缓存等

### hTTP/1.1
支持了持久连接
pipeline
增加post和其他一些命令

### HTTP2
所有数据是以二进制传输
同一个连接发送的多个请求不在需要按顺序来
头部信息压缩以及推送等提高效率的功能
推送

HTTP2是主要解决了http1.1性能低下的问题

## HTTP三次握手

![](http://p7b5cwgjy.bkt.clouddn.com/15405471363731.jpg)
## URI URL URN
URI: Uniform Resource Identifier 统一资源标识符
用来唯一的标识互联网上的资源
包括了URL和URN  

URL：Uniform Resource Locator 统一资源定位器

URN：永久统一资源定位符

## HTTP报文格式
![](http://p7b5cwgjy.bkt.clouddn.com/15407731035602.jpg)

http code：定义服务器对请求处理的结果
各个区间的code有各自的语义
好的http服务可以通过code判断结果的

### 创建一个简单的server
这里我们使用node
```node
const http = require('http')

http.createServer(function(request, response){
    console.log('request come', request.url)

    response.end('hello')
}).listen(8888)

console.log('server listening 8888')
```

### http client
浏览器、 curl、 爬虫

使用curl观察http请求响应的详细信息
```
zhangyangs-MacBook-Pro:Desktop leon$ curl -v baidu.com
* Rebuilt URL to: baidu.com/
*   Trying 220.181.57.216...
* TCP_NODELAY set
* Connected to baidu.com (220.181.57.216) port 80 (#0)
> GET / HTTP/1.1
> Host: baidu.com
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Mon, 29 Oct 2018 00:56:59 GMT
< Server: Apache
< Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
< ETag: "51-47cf7e6ee8400"
< Accept-Ranges: bytes
< Content-Length: 81
< Cache-Control: max-age=86400
< Expires: Tue, 30 Oct 2018 00:56:59 GMT
< Connection: Keep-Alive
< Content-Type: text/html
<
<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
* Connection #0 to host baidu.com left intact
```

### 跨域