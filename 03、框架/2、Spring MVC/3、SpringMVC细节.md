[TOC]







# `@RequestMapping`

## 基本使用

> 1. 通过注解方式实现路径到处理器方法的映射。
> 2. 可以用在类或方法上。
>    - 用在方法上表示将该方法变为一个处理器，且和指定路径做映射。
>    - 用在类上则配置的路径会作为这个类中所有处理器的路径的父路径使用。

### 用在方法上

> 访问路径： `http://localhost/SpringMVC3/test01.action` 

```java
@Controller
public class MyController01 {

    @RequestMapping("/test01.action")
    public void test01() {
        System.out.println("test01..");
        System.out.println("test01..1");
    }
}
```



### 用在类上

> 访问路径： `http://localhost/SpringMVC3/my01/test01.action` 

```java
@Controller
@RequestMapping("/my01")
public class MyController01 {

    @RequestMapping("/test01.action")
    public void test01() {
        System.out.println("test01..");
        System.out.println("test01..1");
    }
}
```



## 注解属性

###  `value`

> `String [] value() default {};`
>
> 作用：
>
> - 指定要将当前处理器绑定到哪个访问路径上。
> - 可以配置多个路径，将多个路径绑定到当前方法
> - 路径中也可以使用*号作为通配符匹配部分路径。
>
> `@RequestMapping(value = {"/test02.action","/test02z.action","/test02*x.action"})`
>
> - 可以通过 `/test02.action` 访问
> - 可以通过 `/test02z.action` 访问
> - 可以通过 `/test02++x.action` 访问

```java
/**
 *	绑定路径到控制器方法
 *  可以配置多个值，将多个路径绑定到当前方法
 *  可以在路径中使用*号作为通配符
 */
@RequestMapping(value = {"/test02.action","/test02z.action","/test02*x.action"})
public void test02() {
    System.out.println("test02..");
    System.out.println("test02..2");
}
```



### `method `

> `RequestMethod[] method() default {}`
>
> 作用：
>
> - 限定【允许/不允许】哪种请求方式的请求能访问此方法。
> - 如果不作限制，所有请求方式都可以访问此控制器方法
> - 可以通过配置`method`属性指定当前处理器处理哪种提交方式提交的请求。
>
> `@RequestMapping(value="/test03.action",method=RequestMethod.GET)`
>
> - 可以通过 `/test03.action` 访问
> - 只接收`get`请求

```java
    /**
     * @RequestMapping的method属性
     *      限定只允许哪种请求方式的请求访问此方法
     *      如果不做配置，所有请求方式都可以访问此控制器方法
     *      可以通过配置method属性限定哪此控制器方法只接受哪些中方式的请求
     * http://localhost/SpringMVCDay01_03_RequestMapping/test03.jsp
     */
    @RequestMapping(value="/test03.action",method=RequestMethod.GET)
    public void test03(){
        System.out.println("test03...");
    }
```



### `params`

> `String[] params() default {}`
>
> 作用：
>
> - 用来限定当前请求中必须包含指定名称的请求参数才会被当前处理器处理
> -  通过params属性指定只处理请求参数符合指定要求的请求
>
> 格式：
>
> - 格式1:只指定名称，要求必须具有该名称的请求参数
> - 格式2:以"名称=值"或"名称!=值"的方式指定必须具有某个请求参数，且值必须等于或不等于给定值
> - 格式3:以"!名称"的方式指定必须不包含指定名称的请求参数
>
> `@RequestMapping(value="/test04.action",params={"uname","gender=male","country!=cn","!age"})`
>
> - 路径：http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action ： 不能被访问
> - http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action?uname=zs ： 不能被访问
> -  http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action?uname=zs&gender=male ： ==可以被访问==
> -  http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action?uname=zs&gender=male&country=cn ：不能被访问
> - http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action?uname=zs&gender=male&country=us ： ==可以被访问==
> - http://localhost/SpringMVCDay01_03_RequestMapping/my01/test04.action?uname=zs&gender=male&country=us&age=99 ： 不能被访问

```java
@RequestMapping(value="/test04.action",params={"uname","gender=male","country!=cn","!age"})
public void test04()
```



### `headers`

> `String[] headers() default {}`
>
> 作用：
>
> - 用来限定当前请求中必须包含指定名称的请求头才会被当前处理器处理
>
> 格式：
>
> - 格式1:只指定名称，要求必须具有该名称的请求头
> -  格式2:以"名称=值"或"名称!=值"的方式指定必须具有某个请求头，且值必须等于或不等于给定值
> - 格式3:以"!名称"的方式指定必须不包含指定名称的请求头
>
> 【@RequestMapping(value="/test05.action",headers={"Cookie","!Refresh","Connection=keep-alive","Host!=localhost"})】
>
> - 不能被访问，因为是通过localhost访问，但是请求头规定Host!=localhost，所以不能访问

```java
/**
     * @RequestMapping的headers属性
     *      要求请求头必须符合指定要求才能访问此控制器方法
     *          格式1:只指定头名称，要求必须具有该名称的请求头
     *          格式2:以"头名称=值"或"名称!=值"的方式指定必须具有某个请求头，且值必须等于或不等于给定值
     *          格式3:以"!头名称"的方式指定必须不包含指定名称的请求头
     * http://localhost/SpringMVCDay01_03_RequestMapping/my01/test05.action
     */
@RequestMapping(value="/test05.action",headers={"Cookie","!Refresh","Connection=keep-alive","Host!=localhost"})
public void test05(){
    System.out.println("test05..");
}
```





# 获取请求参数

## 通过Request对象获取

> 可以选择性的接收Request和Response对象来使用，可以用request对象来获取请求参数
>
> 和Servlet中获取参数一样，通过`getParameter()`方法获取

```java
@RequestMapping("/test01.action")
public void test01(HttpServletRequest request){
    String uname = request.getParameter("uname");
    System.out.println(uname);
}
//zs
```



## 通过参数名直接获取

### 方法参数名 = 请求参数名

> 可以在Controller方法中直接接收请求参数相同名称的方法形参，可以直接得到请求参数的值
>
> - 要求：方法参数名必须和请求参数名对应

```java
@RequestMapping("/test02.action")
public void test02(String uname,int uage){
    System.out.println(uname);
    System.out.println(uage);
}
//zs
//19
```

### 方法参数名 != 请求参数名

> 方法参数名和请求参数名不一致，通过使用注解`@RequestParam("xxx")`设置映射关系
>
> - 请求参数：uname=zs&uage=19
> - 方法参数：
>   - `@RequestParam("uname") String name`
>   - `@RequestParam("uage") int age`
>
> `@RequestParam`
>
> - 属性：
>
>   | 属性         | 解释                                                         |
>   | ------------ | ------------------------------------------------------------ |
>   | value        | 参数名字。即入参的请求参数名字，如：value=“delld” 表示请求的参数其中的名字为delld的参数的值将被传入 |
>   | required     | 是否必须。默认为true，表示请求中一定要有对应的参数，否则将报400错误代码 |
>   | defaultValue | 默认值。表示如果请求中没有同名参数时的默认值                 |
>
> - `value`：指定 将那个请求参数赋值给当前形参
>
> - `required`：声明为true，则请求参数中必须有该属性，如果没有客户端将受到400
>
> - `defaultValue`：可以设定当前形参的默认值

```java
@RequestMapping("/test03.action")
public void test03(@RequestParam("uname") String name, @RequestParam("uage") int age){
    System.out.println(name);
    System.out.println(age);
}
//zs
//19
```



## 获取并自动封装到bean

> SpringMVC框架可以自动将请求参数封装到bean中，要求bean中必须提供属性的setXxx方法，且bean的属性名和请求参数中请求参数的名字必须一致，才可以自动设置。

### 全部为基本数据类型

#### User

```java
public class User {
    private String uname;
    private int uage;
    private String uaddr;
    
    public User(....) {}//有参构造
    public User() {}//无参构造
    public String toString() {...}
    //各个参数的get，set方法↓↓↓↓↓↓↓↓
}
```



#### Controller

> 请求：xxx?uname=zs&uage=19&uaddr=bj
>
> SpringMVC自动将请求参数和bean的set方法对应，自动生成bean对象

```java
@RequestMapping("/test04_1.action")
public void test04_1(User user){
    System.out.println(user);
}
//User{uname='zs', uage=19, uaddr='bj'}
```



### 有复杂数据类型

> 如果自动封装的bean中存在复杂类型，只要该复杂类型的属性同样具有setXxx方法，则可以在请求参数中包含`[bean中复杂类型].[属性]`的方式为该复杂类型的参数赋值，从而实现自动封装bean的过程中处理其中的复杂类型

#### Dog

```java
public class Dog {
    private String name;
    private int age;

    public Dog() {}
    public Dog(String name, int age) { }
    public String toString() { }
	//各个参数的get ， set方法
}
```



#### User

```java
public class User {
    private String uname;
    private int uage;
    private String uaddr;
    private Dog dog;//复杂数据类型
    
    public User(....) {}//有参构造
    public User() {}//无参构造
    public String toString() {...}
    //各个参数的get，set方法↓↓↓↓↓↓↓↓
}
```



#### Controller

> 通过`.` 设定自定义bean类型中带有复杂类型的数据
>
> 请求：`xxx?uname=zs&uage=19&uaddr=bj&dog.name=wc&dog.age=5`

```java
@RequestMapping("/test05.action")
public void test05(User user){
    System.out.println(user);
}
```



## 获取多个同名请求参数

> 请求：`xxxx?like=zq&like=lq&like=ppq`

### 直接获取

> 直接获取，会得到一个用逗号分隔的字符串

```java
@RequestMapping("/test06.action")
public void test06(String like){
    System.out.println(like);
}
//zq,lq,ppq
```



### 通过数组获取

> 修改Controller方法的形参为==数组==类型，则直接接收到一个数组

```java
@RequestMapping("/test06.action")
public void test06(String[] like){
    System.out.println(Arrays.asList(like));
}
//[z1,lq,ppq]
```



# 请求参数乱码

### SpringMVC内置乱码解决过滤器

> SpringMVC提供了过滤器用来解决全站乱码
>
> ==这种方式只能解决POST提交的乱码，对GET方式提交的乱码无效！==
>
> 此时只能手动进行编解码 解决GET方式请求参数乱码

```xml
<!--配置乱码解决过滤器-->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 手动进行编码

```java
username = new String(username.getBytes("iso8859-1"), "utf-8");
```



### 直接修改Tomcat配置（不推荐）

> 这种方式不建议大家使用，因为生产环境下不一定允许修改此项
>
> `URIEncoding="UTF-8"`

```xml
<Connection port="8080" protocol="HTTP/1.1"
            connectionTimeout="20000"
            redirectPort="8443" URIEncoding="UTF-8"/>
```





# 日期数据的处理

> 在SpringMVC中解析页面提交的请求参数时，无法自动获取封装日期到Data。
>
> 如果想要实现自动封装，必须==手动注册适配器自己来指定转换方式。==
>
> - 自定义类型转换器：通过注解：`@InitBinder`实现

```java
@InitBinder
//自定义类型转换器
public void myDateInitBinder(ServletRequestDataBinder binder){
    binder.registerCustomEditor(
        Date.class,
        new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"),true));
}
```



### controller

> 没有上面的自定义转换器，Spring是无法自动封装Date的

```java
@RequestMapping("/test08.action")
public void test08(String username,Date birthday){
    System.out.println(username);
    System.out.println(birthday);
}
//石晓浩
//Fri Jan 03 00:00:00 CST 2020
```



------



# SpringMVC文件上传

> 需要jar包：commons-io-2.0.1.jar 	commons-fileupload-1.2.2.jar

## 上传表单

> 文件上传表单必须满足如下三个条件
>
> 1. 文件上传项必须有`name`属性
> 2. 表单必须是`Post`提交
> 3. 表单必须是`enctype="multipart/form-data"`

```html
<form action="${pageContext.request.contextPath}/my01/test09.action"
      enctype="multipart/form-data"
      method="post">
    选择文件：<input type="file" name="fx">
    <input type="submit" />
</form>
```



## 在配置文件中配置文件上传工具

> 其中可以配置很多属性
>
> - 上传总文件大小：`maxUploadSize`
> - 上传单个文件大小：`maxUploadSizePerFile`
> - 最大内存大小：`maxInMemorySize`（值范围之内都会先把文件加载到内存中，在操作）
> - 文件过大，无法加载到内存中，需要在硬盘生成临时文件进行上传：`uploadTempDir`

```xml
<!--配置文件上传工具类-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
</bean>
```



## 在Controller中实现文件上传

> 将用户上传的文件保存在服务器本地

```java
//文件上传
@RequestMapping("/test09.action")
public void test09(MultipartFile fx) throws IOException {
    fx.transferTo(new File("E:/"+fx.getOriginalFilename()));
}
```





------



# 路径动态数据的获取

> `RESTFul`风格的请求参数处理

## RESTFul风格的请求

1. 普通get请求

   > Url:`localhost/XXXX/addUser.action?name=tom&age=18`

2. RESTFul风格的请求

   > 将表单数据以路径的形式提交
   >
   > Url:`localhost/XXXX/addUser/tom/18.action`

   

## SpringMVC对RESTFul风格请求的处理

> URL:`xxxx/zs/bj/test10.action`

```java
// 获取路径中的数据
@RequestMapping("/{uname}/{uaddr}/test10.action")
public void test10(@PathVariable("uname") String uname, @PathVariable("uaddr") String uaddr){
    System.out.println(uname);//zs
    System.out.println(uaddr);//bj
}
```






