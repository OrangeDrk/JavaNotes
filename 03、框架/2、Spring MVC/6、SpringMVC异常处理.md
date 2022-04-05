[TOC]



------



# 为当前Controller配置错误处理

> 该错误处理只对当前Controller有效

```java
@Controller
@RequestMapping("/my01")
public class MyController {

    /**
     * 当前控制器类内部的异常处理机制 - 只对当前控制器内部的方法有效
     * @param e
     * @return
     */
    @ExceptionHandler
    public String my01ExceptionHandler(Exception e){
        System.out.println("######发生异常" + e.getMessage());
        return "my01err";//记录完日志后跳转到指定的错误页面
    }

    /**
     * 异常处理
     */
    @RequestMapping("/test01.action")
    public void test01(){
        int i = 1/0;
    }

}

```

## 结果

![](img\SpringMVC异常处理.png)







------



# 注解方式配置全局错误处理

> 需要自定义一个类，并通过`@ControllerAdvice`和`@ExceptionHandler`
>
> 可以为所有Handler提供错误处理，==当有Handler有单独的错误处理时，采用单独的错误处理（就近原则）==

```java
/**
 * 全局错误处理器
 */
@ControllerAdvice
public class MyGlobalExceptionHandler {

    @ExceptionHandler
    public String myExceptionHandler(Exception e){
        System.out.println("######全局发生异常" + e.getMessage());
        return "err";//跳转指定的错误页面
    }
}
```





------



# 配置文件方式配置全局错误处理

> 注解方式要单独创建一个类，比较麻烦，可以通过配置文件配置错误处理：在`springmvc.xml`中配置
>
> - 每个key：可能出现的异常错误
> - 值：要跳转的错误页面
>
> 就近原则：有Handler有单独的错误处理，就执行单独的错误处理

```xml
<!--配置文件方式：配置全局错误处理器-->
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
            <prop key="java.lang.NullPointerException">null_err</prop>
            <prop key="java.io.IOException">io_err</prop>
            <prop key="java.lang.Throwable">g_err</prop>
        </props>
    </property>
</bean>
```

