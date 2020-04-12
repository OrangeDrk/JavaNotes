[TOC]

# 返回一段数据`@ResponseBody`

## 返回字符串数据

### 方式一：通过response返回

```java
@RequestMapping("/test01.action")
public void test(HttpServletResponse response) throws IOException {
    response.getWriter().write("hello,my friends...");
}
```



### 方式二：直接获取PrintWriter返回

```java
@RequestMapping("/test02.action")
public void test02(PrintWriter writer){
    writer.write("hello,my01test02");
}
```



### 方式三：通过`@ResponseBody`返回

> 没有@ResponseBody这个注解，返回的是一个view视图，添加后才是返回数据

```java
@RequestMapping("/test03.action")
@ResponseBody
public String test03(){
    return "hello,test03";
}
```



- 注意：通过@ResponseBody注解输出数据，存在编码问题

  > 设定`response.setContextType("text/html;charset=utf-8");`是无效的
  >
  > 此时只能通过`@RequestMapping`注解中的`produces`属性来设置输出的数据格式
  >
  > - `produces = "text/html;charset=utf-8"`

  ```java
  @RequestMapping(value = "/test04.action",produces = "text/html;charset=utf-8")
  @ResponseBody
  public String test04(){
      return "hello 中国";
  }
  ```

  



## 返回json数据

### 方式一：手动拼接json

> 了解即可，不可能手写，太麻烦

```java
@ResponseBody
@RequestMapping(value="/test04.action")
public String test04() throws Exception {
	User user = new User(99,"zs",Arrays.asList("bj","sh","gz"));
	String str = "";
	for(String addr : user.getAddrs()) {
		str += "'"+addr+"',";
	}
	str = str.substring(0, str.length()-1);
	String json = "{id:'"+user.getId()+"',name:'"+user.getName()+"',addrs:["+str+"]}";
	return json;
}
```



### 方式二：配置@ResponseBody，利用内置jackson

```java
@ResponseBody
@RequestMapping(value="/test05.action",produces="application/json;charset=utf-8")
public User test05() throws Exception {
    User user = new User(99,"张三",Arrays.asList("bj","sh","gz"));
    return user;
}
```





# 处理器方法支持的参数类型和返回值类型总结

## 支持的方法参数类型

1. `HttpServletRequest`：代表当前请求的对象
2. `HttpServletResponse`：代表当前响应的对象
3. `HttpSession`：代表当前会话的对象
4. `WebRequest`：SpringMVC提供的对象，相当于是request和session的合体，可以操作这两个域中的属性。
5. `InputStream`     `OutputStream Reader Writer`：代表request中获取的输入流和response中获取的输出流
6. 通过`@PathVariable`     `@RequestParam`声明的方法参数
   - @PathVariable可以将请求路径的指定部分获取赋值给指定方法参数
   - @RequestParam可以将指定请求参数赋值给指定方法参数
   - 如果不写此注解，则默认会将同名的请求参数赋值给方法参数
7. 通过`@CookieValue`和`@RequestHeader`声明的方法参数
   - @CookieValue可以将请求中的指定名称的cookie赋值给指定方法参数
   - @RequestHeader可以将请求参数中的指定名称的头赋值给指定方法参数
8. `Model`和`ModelMap`和`java.util.Map`
   - 向这些Model ModelMap Map中存入属性，相当于向模型中存入数据
9. `Bean`类：SpringMVC自动将请求参数封装到bean
10. `MultipartFile`：实现文件上传功能时，接收上传的文件对象
11. `Errors`      `BindingResult` 
    - 实现数据验证的参数

## 支持的返回值类型

1. ModelAndView：可以返回一个ModelAndView对象，在其中封装Model和View信息
2. View：可以直接返回一个代表视图的View对象
3. 字符串：直接返回视图的名称
4. void：如果返回值类型是void，则会自动返回和当前处理器路径名相同的视图名
5. 方法被@ResponseBody修饰：当方法被@ResponseBody修饰时，默认将返回的对象转为json写入输出
6. 除以上之外返回的任何内容都会被当做模型中的数据来处理，值为返回的数据，键为返回类型名首字母转小写，而返回的视图名等同于返回值为void的时的视图名

