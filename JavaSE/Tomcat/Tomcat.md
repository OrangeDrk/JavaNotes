[TOC]

# Tomcat介绍

## 应用程序

> 1. 按应用类型分：办公程序，系统程序，游戏，社交程序，教学程序……
>
> 2. 按应用设备分：windows软件(c/c++)，安卓软件，IOS软件，单片机嵌入式软件
>
> 3. 按服务器分：CS(Client-Server):CF,LOL,PUBG……服务器和设备各一个软件
>
>    ​				  BS(Browser-Server):JD,天猫……只用服务器软件

### CS

> 特点：
>
> - 除了服务器端的数据以外，必须开发一个客户端，再给客户端电脑装上，然后教他怎么用
> - 网络协议：TCP/IP 
> - 应用层协议：自定义------安全

### BS

> 特点：
>
> - 只写服务器端的代码，客户端无需开发，客户通过自带的浏览器访问即可
> - 网络协议：TCP/IP
> - 应用层协议：HTTP（W3C规定的）------相对HTTPS不安全

### Tomcat就是BS中的S

> Tomcat是一个服务器软件，可以通过HTTP协议对外提供数据服务。客户端只要通过浏览器即可访问到Tomcat
>
> Tomcat是半个软件（互联网应用容器）
>
> 我们自己开发的网站，是另外半个软件（互联网应用）
>
> Tomcat+web应用 = BS结构应用程序



# Tomcat代码

## 定义服务器套接字，并初始化

> ServerSocket这个类支持TCP协议，我们只要在这个TCP协议的基础上，自己写HTTP协议就可以接受浏览器信息了
>
> 启动服务器，对8080端口进行监听:
>
> `ServerSocket server = new ServerSocket(8080);`------8080端口号
>
> 



## 等待客户端连接

> `accept()`是一个阻塞线程，直到有客户端访问。返回一个套接字，封装了客户端的请求
>
> `Socket client = server.accept();`



## 接收客户端发来的信息---详情看HTTP

> HTTP协议默认的编码方式是ISO8859-1

> ```java
> InputStream in = client.getInputStream();//来自客户端的【字节】信息
> 
> //因为请求中的字节数不清楚，所以暂时定义先定义那么大，不要过小，过小数据读的不完整
> byte[] data = new byte[1024*10];
> 
> in.read(data);//将字节里的数据传给data
> 
> //http协议的默认编码方式是iso8859-1
> String data_s = new String(data,"ISO8859-1");
> System.out.println(data_s);//输出内容则是客户端发来的请求信息
> ```
>
>
> 输出所有数据：（行数每个人不相同）==可以每次读一行，用": "分割==
>
> 1、GET / HTTP/1.1
> 2、Host: localhost:8080
> 3、Connection: keep-alive
> 4、Upgrade-Insecure-Requests: 1
> 5、User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36
> 6、Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
> 7、Accept-Encoding: gzip, deflate, br
> 8、Accept-Language: zh-CN,zh;q=0.9
> 消息正文 ,此时没有

### 请求行：

> ```
> GET / HTTP/1.1
> ```
>
> 分三部分：
>
> - 请求方式：GET、POST
> - URI：/、/index.html
> - 协议版本：HTTP/1.1

### 消息头：

> - Host: localhost:8080------==服务器地址==
> - Connection: keep-alive------==只有HTTP1.1版本才有==
> - Upgrade-Insecure-Requests: 1
> - User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36------==浏览器版本和各种信息==
> - Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
> - Accept-Encoding: gzip, deflate, br
> - Accept-Language: zh-CN,zh;q=0.9

### 请求正文：

> 视情况，有时有，有时没有，取决于请求方式
>
> get请求方式，没有请求正文
>
> post请求方式，有请求正文





## 给浏览器响应---详情看HTTP

> 浏览器请求以后，服务器必须给相应，如果不响应，浏览器就会一直转圈，转到一定程度，报无法访问的错误

> 响应也包含三部分
>
> - 状态行
>   - 协议版本------HTTP1.1
>   - 状态码------200
>   - 状态描述------OK
> - 响应头
>   - Content-Type:==空格==text/html(字符串，代表发给浏览器的文件类型)
>     - text/html       文件是html
>     - images/jpeg  文件是 jpg
>     - images/gif     文件是gif
>   - Content-Length:==空格==长度(整数，代表发给浏览器的文件大小)
> - 响应正文
>   - 就是一个字节流，字节流是没有编码方式的，但一定是由有编码方式(UTF-8、GBK,文件的编码方式)的字符流转换过来的
>   - HTML的编码方式，不同于文件的编码方式，这个编码说的是请求和响应中文字的编码，比如说请求行，请求头，状态行，响应行，响应头，响应正文，这些文字也有编码方式，是固定不变的------ISO8859-1
>   - 读文件，写文件

### 给浏览器响应文件

> 1. 读取第一行客户端发来的信息，提取URI
>
>    ```java
>    InputStream in = client.getInputStream();//来自客户端的【字节】信息
>    			
>    InputStreamReader isr = new InputStreamReader(in,"ISO8859-1");
>    BufferedReader reader = new BufferedReader(isr);
>    
>    String requestLine = reader.readLine();
>    
>    System.out.println(requestLine);
>    String[] requestLines = requestLine.split(" ");
>    String uri = requestLines[1];
>    ```
>
> 2. 通过URI定位文件
>
>    ```java
>    String str = "File Not Found 404";
>    if(uri.equals("/")) {
>        uri = "/index.html";
>    }
>    File file = new File("webapps"+uri);
>    ```
>
> 3. 状态行
>
>    ```java
>    os.write("HTTP1.1 200 OK".getBytes("ISO8859-1"));
>    os.write(13);
>    os.write(10);
>    ```
>
> 4. 响应头
>
>    ```java
>    //告知浏览器响应的内容是一个html文本文档
>    os.write("Content-Type: text/html".getBytes("ISO8859-1"));
>    os.write(13);
>    os.write(10);
>    //告知浏览器相应正文的长度
>    if(!file.exists()) {
>    	os.write(("Content-Length: "+str.length()).getBytes("ISO8859-1"));
>    }else {
>    	os.write(("Content-Length: "+file.length()).getBytes("ISO8859-1"));
>    }
>    os.write(13);
>    os.write(10);
>    
>    //告知浏览器响应头结束（其实是发了一个空行）
>    os.write(13);
>    os.write(10);
>    ```
>
> 5. 响应正文
>
>    ```java
>    if(!file.exists()) {
>    	os.write(str.getBytes("UTF-8"));
>    }else {
>        //将流读出来
>        FileInputStream fis = new FileInputStream(file);
>        byte[] tmp = new byte[(int) file.length()];
>        fis.read(tmp);
>        os.write(tmp);
>        fis.close();
>    }
>    os.flush();//刷新
>    os.close();//关闭
>    System.out.println("发送响应完毕");
>    ```
>
>    