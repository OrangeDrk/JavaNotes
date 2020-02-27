[TOC]





# SpringMVC中域的使用

> 四个作用域：Request、Session、servletContext、pag





------



# Request域

> 在JSP页面中通过`${requestScope.k1}`获取Request域中的数据

## 传统方式

```java
// 传统方式
@RequestMapping("/test01.action")
public String test01(HttpServletRequest request) {
    request.setAttribute("k1","v1");
    request.setAttribute("k2","v2");
    return "my01test01";
}
```

## SpringMVC方式

> 向Model对象中添加数据，默认就是写到了Request域中

```java
/**
 * 通过model将数据存入request域
 *      存入Model中的数默认就是存入了request域
 */
@RequestMapping("/test02.action")
public String test02(Model model){
    model.addAttribute("k1", "v1");
    model.addAttribute("k2", "v2");
    return "my01test02";
}
```





------



# Session域

> 在JSP页面中通过`${sessionScope.k1}`获取Request域中的数据

## 传统方式

```java
    /**
     * 传统方式
     */
    @RequestMapping("/test02.action")
    public String test02(HttpSession session){
        session.setAttribute("session1","session1");
        session.setAttribute("session2","session2");
        return "my02test02";
    }
```

## SpringMVC方式

> 通过【`Model`+`@SessionAttributes`】实现将数据写入session域中
>
> - `@SessionAttributes`：将Model中的值相同的复制到session中一份
>   - 参数可以传多个值，底层通过数组接收
> - 注意：该注解用在==类==上，类中所有的model键值对，都会复制到session中，所以==该注解慎用==！！

```java
@Controller
@RequestMapping("/my02")
@SessionAttributes({"kx1","kx2"})
//将下面model中的kx1,kx2键值对复制到session中一份
public class MyController02 {

    //springmvc方式
    @RequestMapping("/test03.action")
    public String test03(Model model){
        model.addAttribute("kx1", "vvv1");
        model.addAttribute("kx2", "vvv2");
        return "my02test03";
    }
}
```





------



# ServletContext域

> 在JSP页面中通过`${applicationScope.k1}`获取Request域中的数据
>
> ==该作用域只有传统方式==

```java
//传统方式
@RequestMapping("/test01.action")
public String test01(HttpServletRequest request){
    ServletContext servletContext = request.getServletContext();
    servletContext.setAttribute("k1", "servletContext1");
    servletContext.setAttribute("k2", "servletContext2");
    return "my03test01";
}
```





------



# @ModelAttribute

> 该注解可以用在两个位置：
>
> - 方法上
> - 方法参数前
>
> `@ModelAttribute`的参数为存入model中的键

## 用在方法上

> 被修饰的方法将会在当前类的==任意handler方法执行之前执行==，该方法返回的==返回值会自动存入model供后续使用==

```java
//用在方法上
@ModelAttribute("k1")//k1 是键，方法中return的abc为值
public String myfunc01(){
    return "abc";//自动加到model中
}

@RequestMapping("/test01.action")
public void test01(Model model){
    Map<String, Object> map = model.asMap();
    System.out.println(map);//{k1=abc}
}
```

## 用在方法参数前

> 则会从model中获取属性值，赋值到被修饰的方法参数上

```java
@ModelAttribute("k1")//k1 是键，方法中return的abc为值
public String myfunc01(){
    return "abc";//自动加到model中
}   

//用在控制器方法测参数上
@RequestMapping("/test02.action")
public void test03(@ModelAttribute("k1") String str){
    System.out.println(str);//abc
}
```

