[TOC]

# request对象的介绍

- 解释：

  服务器接收到浏览器的请求后，会创建一个Request对象，对象中存储了此次请求相关的请求数据。服务器在调用Servlet时会将创建的Request对象作为实参传递给Servlet的方法，比如：service方法

- 作用：request对象中封存了当前请求的所有请求信息

  - 实现请求转发
  - 实现请求包含
  - 获取session对象
  - 作为作用域对象使用(==Attribute==,`set/get Attribute();`)
  - 获取请求头信息

- 使用：

  - 获取请求行数据
    - `req.getMethod()`——获取请求方式
    - `req.getRequestURL()`——获取请求URL信息
    - `req.getRequestURI()`——获取请求URI信息
    - `req.getScheme()`——获取协议
    - `req.getQueryString()`获取请求行中的参数
    - `req.getRemoteAddr()`获取请求端IP地址
    - `req.getContextPath()`获取当前web应用的虚拟目录名
  - 获取请求头数据
    - `req.getHeader("键名");`——返回指定的请求头信息
    - `req.getHeaderNames();`——返回请求头的键名的枚举集合
  - 获取用户数据
    - `req.getParameter();`——返回指定的用户数据
    - `req.getParameterValues();`——返回同键不同值的请求数据(多选)，数组接收
    - `req.getParameterNames();`——返回所有用户请求数据的枚举集合

- 注意：==request对象由Tomcat服务器创建，并作为实参传递给请求的servlet的service方法==

## 获取请求行数据

> 在servlet中的service方法中

### `req.getMethod()`获取请求方式

> POST

```java
String method = req.getMethod();
System.out.println(method);
```

### `req.getRequestURL()`获取请求URL

> `http://localhost:8080/ServletDemo/req`

```java
StringBuffer url = req.getRequestURL();
System.out.println(url);
```

### `req.getRequestURI()`获取请求URI

> /ServletDemo/req

```java
String uri = req.getRequestURI();
System.out.println(uri);
```

### `req.getScheme()`获取请求协议

> http

```java
String h = req.getScheme();
System.out.println(h);
```

### `req.getQueryString()`获取请求行中的参数

> name=10&pas=123

```java
String queryString = request.getQueryString();
System.out.println("请求行中参数："+queryString);
```

### `req.getRemoteAddr()`获取请求端IP地址

> 127.0.0.1：80

```java
String remoteAddr = request.getRemoteAddr();
System.out.println("客户端IP地址：" + remoteAddr);
```

### ⭐`req.getContextPath()`获取当前web应用的虚拟目录名

> /day10：当前项目的虚拟名称（浏览器访问的虚拟路径）

```java
String contextPath = request.getContextPath();
System.out.println("虚拟目录名称"+contextPath);
```



------



## 获取请求头数据

### `req.getHeader("键名");`获取指定的请求行信息

> localhost:8080

```java
String value = req.getHeader("host");
```

### ⭐`req.getHeaderNames();`获取所有的请求行的键的枚举和对应值

>  `Enumeration e = req.getHeaderNames();`//获取所有的键
>
> host:localhost:8080
> connection:keep-alive
> upgrade-insecure-requests:1
> user-agent:Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
> sec-fetch-mode:navigate
> sec-fetch-user:?1
> …………
>
> …………

```java
Enumeration e = req.getHeaderNames();
while (e.hasMoreElements()) {/*判断集合中是否存在元素*/
    
    //获取元素对象，返回值是请求头的key
    String name = (String) e.nextElement();
    
    //获取对应key的value值
    String value2 = req.getHeader(name);
    System.out.println(name + ":" + value2);
}
```



### `req.getIntHeader("键值")`获取指定key，value为int的数据

> 原有数据：`upgrade-insecure-requests,value:1`
>
> 结果：`int类型的值：1`

```java
int intHeader = request.getIntHeader("upgrade-insecure-requests");
System.out.println("int类型的值："+intHeader);
```







------



## 获取用户数据

### ⭐`req.getParameter(“键值”);`返回指定的用户数据

```java
String name = req.getParameter("uname");
String pwd = req.getParameter("pwd");
```

### `req.getParameterValues();`返回同键不同值的请求数据（多选）,数组接收

```java
String[] favs = req.getParameterValues("fav");
if (favs != null) {
    for(String fav:favs){
        System.out.println(fav);
    }
}
```

### `req.getParameterNames();`返回所有用户请求数据的枚举集合

```java
Enumeration es = req.getParameterNames();
while (es.hasMoreElements()) {
    String names = (String) es.nextElement();
    if (names.equals("fav")) {
        String[] favs1 = req.getParameterValues(names);
        if (favs != null) {
            for(String fav:favs){
                System.out.println(fav);
            }
        }
    }else{
        System.out.println(req.getParameter(names));
    }
}
```

