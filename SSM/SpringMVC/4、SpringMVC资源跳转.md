[TOC]





# SpringMVC中的资源跳转

## 请求转发、重定向、定时刷新的区别

> 转发、请求重定向 和 定时刷新 都是web开发中资源跳转的方式。

- 请求转发：
  - 请求转发是服务器内部的跳转
  - 地址栏不发生变化
  - 只有一个请求响应
  - 可以通过request域传递数据
- 重定向：
  - 请求重定向是浏览器自动发起对跳转目标的请求
  - 地址栏会发生变化
  - 两次请求响应
  - 无法通过request域传递对象
- 定时刷新：
  - 定时刷新是浏览器自动发起对跳转目标的请求
  - 地址栏会发生变化
  - 两次请求响应无法通过request域传递对象
  - 可以在资源跳转期间提示额外信息





------



# 请求转发

## 传统方式

```java
@RequestMapping("/test01.action")
public void test01(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    request.getRequestDispatcher("/index.jsp").forward(request,response);
}
```



## SpringMVC方式

> 可以通过返回`forward:/xxxx.xxx`格式的字符串表明要转发到指定地址
>

```java
@RequestMapping("/test02.action")
public String test02(){
    return "forward:/index.jsp";
}
```





------



# 重定向

## 传统方式

> 注意：在重定向的路径前面要添加应用名：`request.getContextPath()`

```java
@RequestMapping("/test03.action")
public void test03(HttpServletRequest request, HttpServletResponse response) throws IOException {
    response.sendRedirect(request.getContextPath()+"/index.jsp");
}
```



## SpringMVC方式

> 可以通过返回`redirect:/xxxx.xxx`格式的字符串表明要重定向到指定地址
>
> 通过这种方式实现请求重定向时不用在路径前写应用名，SpringMVC会自动拼接应用名

```java
@RequestMapping("/test04.action")
public String  test04(){
    return "redirect:/index.jsp";
}
```





------



# 定时刷新

> SpringMVC中没有提供实现定时刷新的便捷方式，只能用传统方式实现定时刷新
>
> 注意：定时刷新的路径前要添加应用名：`request.getContextPath()`

```java
@RequestMapping("/test05.action")
public void test05(HttpServletRequest request, HttpServletResponse response) throws IOException {
    response.setContentType("text/html;charset=utf-8");
    response.getWriter().write("注册成功！3秒后回到主页");
    response.setHeader("refresh", "3;url=" + request.getContextPath() + "/index.jsp");
}
```

