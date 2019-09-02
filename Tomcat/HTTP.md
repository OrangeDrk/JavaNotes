[TOC]

# HTTP协议--超文本传输协议

> 该协议应用在浏览器与服务器之间。HTTP协议是应用层协议，
> 规定了浏览器与服务端之间传输数据的方式，以及数据内容定义
> 等规则。而HTTP协议要求必须建立在可靠的传输协议基础上，通
> 常我们使用的传输协议为TCP协议。
>
> HTTP协议规定了客户端(浏览器)与服务器之间的交互规则:
> 要求客户端发起请求(Request)，服务端接收请求并处理，
> 然后响应(Response)。不允许服务端主动响应客户端。
>
> 
>
>
> HTTP协议要求所使用的字符集为ISO8859-1，该字符集不支持
> 中文字符，所以对于传输中文内容需要特别处理
>
>
> HTTP协议由万维网制定(w3c),详细更多可以参看其官网。
>
> URL:统一资源定位
> 我们在浏览器上输入的"网址"就是一个URL。

## URL的格式如

> http://www.baidu.com/index.html
> http://localhost:8088/shop/login.html

## URL分为三部分

> protocol://host/abs_path
> 即:
> 应用协议://服务器地址信息/请求资源的抽象路径
> 参见图:url.png
> URL=协议+服务器地址+URI



## HTTP请求:

> 请求是客户端发送给服务端的内容。
>
> 一个请求由三部分构成:
>
> - 请求行
> - 消息头
> - 消息正文

### 请求行:

> 请求行是一行字符串，以CRLF结尾
>
> 注:
>
> - CR，LF是asc编码中的回车符与换行符。
> - CR:对应编码值为13
> - LF:对应编码值为10
>
> 请求行由三部分组成,格式为:
>
> method uri protocol(CRLF)
>
> - 请求方式------method
> - URL中的抽象路径------uri 
> - 使用的协议------protocol
>
> 如:
> GET /index.html HTTP/1.1(CRLF)
>
> 请求方式常见的:
> GET:地址栏方式请求，通常用户传递的参数会被包含在抽象路径中
> POST:打包传输，用户传递的数据会包含在消息正文中。
>



### 消息头:

> 消息头是由若干行组成，每一行为一个具体的消息头信息。
> 消息头是客户端发送给服务端的附加信息，用于说明很多信息，
> 比如告知服务端客户端使用的浏览器信息，请求的路径，交互
> 方式，消息正文内容及长度等等。
> 每一行字符串(一个消息头内容)都以CRLF结尾。最后一个消息头
> 结束后会单独发送一个CRLF表示消息头部分结束。
>
> 各式如:
> name: value(CRLF)
> 消息头名字: 对应的值(CRLF)
>
> 例如:
> Host: localhost:8088(CRLF)
> Connection: keep-alive(CRLF)
> Upgrade-Insecure-Requests: 1(CRLF)
> User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36(CRLF)
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8(CRLF)
> Accept-Encoding: gzip, deflate, br(CRLF)
> Accept-Language: zh-CN,zh;q=0.9(CRLF)(CRLF)

### 消息正文

> 消息正文是2进制数据(不一定是文本数据)，是用户传递给服务端
> 的内容。
> ==一个请求中可以不包含消息正文。==
> 是否含有消息正文可以根据消息头中是否存在:
> Content-Type,Content-Length这两个头来判定。若消息头不
> 含有这两个头信息，则该请求不含有消息正文。
> 注:
>
> - Content-Type用来说明消息正文的数据类型
> - Content-Length用来说明消息正文总共多少字节



### 一个完整的请求大致如下:

> GET /index.html HTTP/1.1(CRLF)
> Host: localhost:8088(CRLF)
> Connection: keep-alive(CRLF)
> Upgrade-Insecure-Requests: 1(CRLF)
> User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36(CRLF)
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8(CRLF)
> Accept-Encoding: gzip, deflate, br(CRLF)
> Accept-Language: zh-CN,zh;q=0.9(CRLF)(CRLF)
> 101001010101010001...



## 响应

> 响应是服务端发送给客户端的内容
>
> 一个响应包含三部分内容:
>
> - 状态行
> - 响应头
> - 响应正文



### 状态行

> 状态行由一行字符串构成(CRLF结尾)
>
> 格式：
> protocol statusCode statusReason(CRLF)
>
> | protocol | statusCode | statusReason | (CRLF) |
> | -------- | ---------- | ------------ | ------ |
> | 协议版本 | 状态代码   | 状态描述     | (CRLF) |
>
> 状态代码是一个3位数字，客户端通过这个代码判断自己的请求
> 是否被服务器处理，以及处理结果如何。
>
> 状态代码分为5类:
> 1xx:保留
> 2xx:成功，表示服务端成功处理了该请求
> 3xx:重定向,服务端希望客户端继续后续操作
> 4xx:客户端错误,客户端的请求有误
> 5xx:服务端错误,服务端处理请求时出现了错误
>
> | 代码 | 描述                  |
> | ---- | --------------------- |
> | 200  | OK                    |
> | 201  | Created               |
> | 202  | Accepted              |
> | 204  | No Content            |
> | 301  | Moved Permanently     |
> | 302  | Moved Temporarily     |
> | 304  | Not Modified          |
> | 400  | Bad Request           |
> | 401  | Unauthorized          |
> | 403  | Forbidden             |
> | 404  | Not Found             |
> | 500  | Internal Server Error |
> | 501  | Not Implemented       |
> | 502  | Bad Gateway           |
> | 503  | Service Unavailable   |
>
> 
>
> 例如:
>
> HTTP/1.1 200 OK(CRLF)



### 响应头

> 响应头的格式与请求中的消息头一样。是服务端发送给客户端的
> 附加信息。



### 响应正文

> 响应正文是2进制数据，是服务端实际响应给客户端请求的结果
> 一个响应中也可以不包含响应正文，客户端也是通过响应头中
> 是否包含:Content-Type与Content-Length来读取的。



### 一个响应内容大致如下:

> HTTP/1.1 200 OK(CRLF)
> Content-Type: text/html(CRLF)
> Content-Length: 125==(CRLF)(CRLF)==
> 1101010100101010100101011......