[TOC]

# AOP面向编程介绍

## 常用术语

| 切面（Aspect）      | 其实就是共有功能的实现                                       |
| ------------------- | ------------------------------------------------------------ |
| 连接点（Joinpoint） | 就是程序在运行过程中能够插入切面的地点【层与层之间的调用过程】 |
| 切入点（Pointcut）  | 用于定义通知应该切入到哪些连接点上【切入点表达式】           |
| 切面（Aspect）      | 一个类，Spring会把拦截下来的切入点放入该类中执行【自定义编写的一个类】 |
| 通知（Advice）      | 是切面的具体实现【切面中的具体实现方法】                     |
| 目标对象（Target）  | 就是那些即将切入切面的对象，也就是那些被通知的对象           |
| 代理对象（Proxy）   | 将通知应用到目标对象之后被动态创建的对象                     |
| 织入（Weaving）     | 将切面应用到目标对象从而创建一个新的代理对象的过程           |

## 图解

![](img\Spring AOP图解.png)



------



# Spring AOP 入门案例

1. 导入aop相关的开发包

   ![](img\Spring AOP所需要的jar包.png)

2. 配置Spring的配置文件

   > 添加配置：
   >
   > - `xmlns:aop="http://www.springframework.org/schema/aop"`
   > - `xsi:schemaLocation="
   >          http://www.springframework.org/schema/aop
   >          http://www.springframework.org/schema/aop/spring-aop.xsd">`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!--开启包扫描-->
       <context:component-scan base-package="cn.tedu"></context:component-scan>
       <!--开启注解支持-->
       <context:annotation-config></context:annotation-config>
   
   </beans>
   ```

   

3. 创建基本层级结构（==连接点==就是每层之间包含的对象【UserServlet中有UserServiceImpl对象】）

   ![](img\Spring AOP的三层结构.png)

4. 创建一个==切面类==

   ![](img\Spring AOP切面类.png)

   ```java
   // 切面
   @Component
   public class FirstAspect {}
   
   ```

   

5. 定义==通知==

   ```java
   // 切面
   @Component
   public class FirstAspect {
   
       //通知
       public void myFirst(){
           System.out.println("first。。。");
       }
   }
   
   ```

   

6. Spring中==配置切面、通知、切入表达式==

   > 有两种形式（结果是一样的）
   >
   > 方式一：配置切面和切入点在一个标签内
   >
   > 方式二：切面和切入点单独配置

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
                              http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              http://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/aop
                              http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!--开启包扫描-->
       <context:component-scan base-package="cn.tedu"></context:component-scan>
       <!--开启注解支持-->
       <context:annotation-config></context:annotation-config>
   
       <!--配置AOP形式一：切面和切入点写一起-->
       <aop:config>
           <!--配置切面-->
           <aop:aspect ref="firstAspect">
               <!--配置通知，通过切入点表达式声明，该通知切入哪一个切入点-->
               <aop:before method="myFirst" pointcut="within(cn.tedu.service.UserServiceImpl)"></aop:before>
           </aop:aspect>
       </aop:config>
       
       <!--======================================================================-->
   
       <!--配置AOP形式二：切面和切入点单独配置-->
       <aop:config>
           <!--配置切点-->
           <aop:poincut id="pt1" expression="execution(void cn.tedu.service.*.*(java.lang.String))"></aop:poincut>
           <!--配置切面-->
           <aop:aspect ref="firstAspect">
               <aop:before method="myFirst" pointcut-ref="pt1"></aop:before>
           </aop:aspect>
       </aop:config>
   
   </beans>
   ```

   

7. 测试，发现切面已经起作用

   ```java
   public class Test {
   
       @org.junit.Test
       public void test(){
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           UserServlet userServlet = (UserServlet) context.getBean("userServlet");
           userServlet.addUser();
           userServlet.queryUser();
           ((ClassPathXmlApplicationContext) context).close();
       }
   
   }
   
   ```

   ![](img\AOP运行结果.png)



------



# 切入点表达式

> 一共有两种：
>
> - `within()`表达式：粗粒度的切入表达式
> - `execution()`表达式：细粒度的切入表达式

## `within()`表达式

> 粗粒度的切入表达式（理解：沙子筛网）
>
> - 最多只能控制到类
>
> ==语法==：`within(类的全路径名)`

1. 在within表达式中可以使用*号匹配符，匹配指定包下所有的类，

   > 注意，只匹配当前包，不包括当前包的子孙包。
   >
   > ==当前包下的所有类==

   ![](img\Spring AOP的within语法.png)

2. 在within表达式中也可以用*号匹配符

   > 匹配包
   >
   > ==子包下的所有类==

   ![](img\Spring AOP的within语法2.png)

3. 在within表达式中也可以用..*号匹配符

   > 匹配指定包下及其子孙包下的所有的类
   >
   > ==当前包下的所有类和子孙包中的所有类==

   ![](img\Spring AOP的within语法3.png)



## `execution()`表达式：（==常用==）

> 细粒度的切入表达式
>
> - 能控制到方法上
>
> ==语法==：`execution(返回值类型 包名.类名.方法名(参数类型,参数类型…))`

1. 案例一

   > 切出【指定包】下【指定类】下【指定名称】【指定参数】【指定返回值】的方法
   >
   > 切出：
   >
   > - 返回类型为void
   > - 在`cn.tedu.service.UserServiceImpl`类下
   > - `addUser`方法，参数为String类型

   ```xml
   <aop:pointcut expression=
      "execution(void cn.tedu.service.UserServiceImpl.addUser(java.lang.String))" id="pc1"/>
   ```

   

2. 案例二

   > 切出【指定包】下【所有的类】中的【query方法】，要求【无参】，但【返回值类型不限】。
   >
   > 切出：
   >
   > - 返回值类型不限
   > - 在`cn.tedu.service`包下所有类中
   > - 没有参数的`query()`方法，

   ```xml
   <aop:pointcut 
        expression="execution(* cn.tedu.service.*.query())" id="pc1"/>
   ```

   

3. 案例三

   > 切出【指定包及其子孙包】下【所有的类】中的【query方法】，要求【无参】，但【返回值类型不限】。
   >
   > 切出：
   >
   > - 返回值类型不限
   > - 在`cn.tedu.service`包下及其子孙包中的所有类中
   > - 没有参数的`query()`方法

   ```xml
   <aop:pointcut expression="execution(* cn.tedu.service..*.query())" id="pc1"/>
   ```

   

4. 案例四

   > 切出【指定包及其子孙包】下【所有的类】中的【query方法】，要求【参数为int java.langString类型】，但【返回值类型不限】。
   >
   > 切出：
   >
   > - 返回值类型不限
   > - 在`cn.tedu.service`包及其子孙包中的所有类中
   > - 参数为`int`，`String`类型的`query()`方法

   ```xml
   <aop:pointcut 
        expression="execution(* cn.tedu.service..*.query(int,java.lang.String))" id="pc1"/>
   ```

   

5. 案例五

   > 切出【指定包及其子孙包】下【所有的类】中的【query方法】，【参数数量及类型不限】，【返回值类型不限】。
   >
   > 切出：
   >
   > - 返回值类型不限
   > - 在`cn.tedu.service`包及其子孙包下所有类中
   > - 参数类型/数量不限的`query()`方法

   ```xml
   <aop:pointcut
        expression="execution(* cn.tedu.service..*.query(..))" id="pc1"/>
   ```

   

6. 案例六

   > 切出【指定包及其子孙包】下【所有的类】中的【任意方法】，【参数数量及类型不限】，【返回值类型不限】。这种写法等价于within表达式的功能。
   >
   > 切出：
   >
   > - 返回值类型不限
   > - 在`cn.tedu.service`包及其子孙包下所有类中
   > - 参数类型/数量不限的==所有方法==

   ```xml
   <aop:pointcut 
        expression="execution(* cn.tedu.service..*.*(..))" id="pc1"/>
   ```

   



# Spring的通知类型

## 五种通知类型

> 方法返回值/参数
>
> - 前置通知 `@Before`：
>   - 返回值：void
>   - 如果接收JoinPoint，第一个参数类型：`JoinPoint`
> - 环绕通知 `@Around`：
>   - 返回值：Object（如果知道返回值是什么类型，可以自己规定，不知道就用object）
>   - 如果接收JoinPoint，第一个参数类型：`ProceedingJoinPoint`
> - 后置通知 `@AfterReturning`:
>   - 返回值：void
>   - 如果接收JoinPoint，第一个参数类型：`JoinPoint`
> - 异常通知 `@AfterThrowing`:
>   - 返回值：void
>   - 如果接收JoinPoint，第一个参数类型：`JoinPoint`
> - 最终通知 `@After`:
>   - 返回值：void
>   - 如果接收JoinPoint，第一个参数类型：`JoinPoint`
>
> 结论：五种通知中，如果接收JoinPoint，第一个参数类型只有==环绕通知==是`ProceedingJoinPoint`类型，并且有返回值，如果没有返回值，调用者将拿不到方法执行后的返回值

### 前置通知

> 在目标方法执行之前执行执行的通知
>
> 前置通知方法，可以没有参数，也可以额外接收一个`JoinPoint`，Spring会自动将该对象传入，代表当前的连接点，通过该对象可以获取【目标对象】 和【 目标方法相关的信息】。
>
> 注意，如果接收`JoinPoint`，必须保证其为方法的==第一个参数==，否则报错。

1. Spring配置文件的配置

   > `<aop:before></aop:before>`
   >
   > - `method`：通知方法
   > - `pointcut`：切入点

   ```xml
   <!--配置AOP-->
   <aop:config>
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <!--配置切面-->
       <aop:aspect ref="firstAspect">
           <aop:before method="myFirst" pointcut-ref="pc01"></aop:before>
       </aop:aspect>
   
   
   </aop:config>
   ```

   

2. 切面类

   ```java
   public void myFirst(JoinPoint jp){
       Object target = jp.getTarget();//获取目标对象
       System.out.println(target);
       
       //获取目标对象的方法
       MethodSignature signature = (MethodSignature) jp.getSignature();
       System.out.println(signature);
       System.out.println("前置通知。。。");
   }
   ```

3. 执行结果

   ![](img\前置通知执行结果.png)



### 环绕通知

> 在目标方法==执行之前==和==之后==都可以执行额外代码的通知。
>
> 在环绕通知中==必须显式的调用目标方法==，否则目标方法不会执行。
>
> - 这个显式调用是通过`ProceedingJoinPoint`来实现的，可以在环绕通知中接收一个此类型的形参，spring容器会自动将该对象传入，这个参数必须处在环绕通知的第一个形参位置。
>
> `ProceedingJoinPoint`是`JoinPoint`的子类
>
> - 要注意，==只有环绕通知可以接收==`ProceedingJoinPoint`，而==其他通知====只能接收JoinPoint。==
>
> ==**特点**==：
>
> -  控制目标方法是否执行
> - 在目标方法之前和之后对执行代码进行增强
> - 控制是否返回返回值
> - 改变返回值
>
> 注意：环绕通知虽然有改变返回值的能力，但一定要慎用，要小心不要破坏了软件分层的“高内聚 低耦合”的目标。

1. Spring配置文件的配置

   > `<aop:around></aop:around>`
   >
   > - `method`：通知方法
   > - `pointcut`：切入点

   ```xml
   <!--配置AOP-->
   <aop:config>
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <!--配置切面-->
       <aop:aspect ref="firstAspect">
           <aop:around method="myAround" pointcut-ref="pc01"></aop:around>
       </aop:aspect>
   
   
   </aop:config>
   ```

2. 切面类

   ```java
   /**
        * 环绕通知
        *      控制目标方法是否执行
        *      在目标方法之前和之后对执行代码进行增强
        *      控制是否返回返回值
        *      改变返回值
        *
        * @param pjp
        * @return 需要手动把返回值返回给调用者
        */
   public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
       System.out.println("环绕通知。。。目标方法执行前。。。");
       Object retObj = pjp.proceed();//调用目标方法
       System.out.println("环绕通知。。。目标方法执行之后。。");
       return retObj;//手动返回结果返回值
   }
   ```

3. 执行结果

   ![](img\环绕通知执行结果.png)



### 后置通知

> 在目标【方法执行成功之后】执行的通知。
>
> 在后置通知中也可以【选择性的接收一个`JoinPoint`】来获取连接点的额外信息，但是这个参数必须处在==参数列表的第一个==。否则会报异常
>
> ==**特点**：==
>
> -  在==目标方法成功执行==之后执行的通知
> - 可以通过`JoinPoint`获取目标对象和目标方法
> - 可以==接收==目标==方法==执行之后返回的==返回值==

1. Spring配置文件的配置

   > `<aop:after-returning></aop:after-returning>`
   >
   > - `method`：通知方法
   > - `pointcut`：切入点
   > - `returning`：声明返回值参数名
   >   - 属性值与通知方法参数名一致
   >
   > 图解：
   >
   > ![](img\后置通知配置文件与通知方法.png)

   ```xml
   <!--配置AOP-->
   <aop:config>
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <!--配置切面-->
       <aop:aspect ref="firstAspect">
           <aop:after-returning 
                method="myAfterReturning" 
                pointcut-ref="pc01" 
                returning="retObj">
               <!--此处的returning和切面类中的通知方法参数对应-->
           </aop:after-returning>
       </aop:aspect>
   
   </aop:config>
   ```

2. 切面类

   > 方法的参数可以声明`Object retObj`：表示接收方法执行完后的返回值

   ```java
   public void myAfterReturning(JoinPoint jp,Object retObj){
       //获取目标对象
       Object target = jp.getTarget();
       //获取目标方法
       MethodSignature signature = (MethodSignature) jp.getSignature();
       Method method = signature.getMethod();
       System.out.println(method);
       //获取方法返回值
       System.out.println(retObj);
       System.out.println("后置通知");
   }
   ```



### 异常通知

> 在目标方法抛出异常时执行的通知
>
> 可以配置传入`JoinPoint`获取==目标对象==和==目标方法==相关信息，但==必须处在参数列表第一位==。另外，还可以==配置参数==，让异常通知==接收到==目标方法==抛出的异常对象==
>
> ==**特点**：==
>
> - 在目标方法抛出异常时执行的通知
> - 可以通过`JoinPoint`获取目标对象和目标方法

1. Spring配置文件的配置

   > `<aop:after-throwing>`标签属性
   >
   > - `method`：通知方法
   > - `pointcut-ref`：切入点
   > - `throwing`：异常对象（【值】与通知方法【参数名】对应）
   >
   > 图解：
   >
   > ![](img\异常通知配置文件与通知方法.png)

   ```xml
   <!--配置AOP-->
   <aop:config>
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <!--配置切面-->
       <aop:aspect ref="firstAspect">
           <aop:after-throwing
                method="myAfterThrowing" 
                pointcut-ref="pc01" 
                throwing="e">
           </aop:after-throwing>
       </aop:aspect>
   
   </aop:config>
   ```

2. 切面类

   ```java
   public void myAfterThrowing(JoinPoint jp,Throwable e){
       Object target = jp.getTarget();
       System.out.println("异常通知..."+target+"---"+e);
   }
   ```



### 最终通知

> 区别：
>
> - ==最终通知==：
>   - 在目标方法==执行之后==执行的通知。（不管方法是否执行成功，都会执行该通知）
>   - 无法得到返回值。
> - ==后置通知==：
>   - 在方法==正常返回后==执行的通知，如果方法没有正常返-例如抛出异常，则后置通知不会执行。
>   - 可以通过配置得到返回值
>
> 可以通过`JoinPoint`获取目标对象和目标方法

1. Spring配置文件的配置

   > `<aop:after></aop:after>`
   >
   > - `method`：通知方法
   > - `pointcut`：切入点

   ```xml
   <!--配置AOP-->
   <aop:config>
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <!--配置切面-->
       <aop:aspect ref="firstAspect">
           <aop:after method="myAfter" pointcut-ref="pc01"></aop:after>
       </aop:aspect>
   
   </aop:config>
   ```

2. 切面类

   ```java
   public void myAfter(JoinPoint jp){
       Object target = jp.getTarget();
       System.out.println("最终通知...."+target);
   }
   ```



## 五种通知的执行顺序

### 在目标方法没有抛出异常的情况下

> 1. 前置通知
> 2. 环绕通知的调用目标方法之前的代码   【1、2取决于配置顺序】
>    - 目标方法
> 3. 环绕通知的调用目标方法之后的代码
> 4. 后置通知
> 5. 最终通知 【3、4、5取决于配置顺序】

### 在目标方法抛出异常的情况下

> 1. 前置通知
> 2. 环绕通知的调用目标方法之前的代码 【1、2取决于配置顺序】
>    - 目标方法 抛出异常 
> 3. 异常通知
> 4. 最终通知 【3、4取决于配置顺序】

### 存在多个切面

> 多切面执行时，采用了责任链设计模式。
>
> 切面的配置顺序决定了切面的执行顺序，多个切面执行的过程，类似于方法调用的过程，在环绕通知的proceed()执行时，去执行下一个切面或如果没有下一个切面执行目标方法，从而达成了如下的执行过程：

#### 目标方法没有抛出异常

![](img\多个切面下没有异常的5种通知执行流程.png)

#### 目标方法抛出异常

![](img\多个切面下抛出异常的5种通知执行流程.png)

## 五种通知的常见使用场景

| 通知类型 | 应用                                 |
| -------- | ------------------------------------ |
| 前置通知 | 记录日志(方法将被调用)               |
| 环绕通知 | 控制事务 权限控制                    |
| 后置通知 | 记录日志(方法已经成功调用)           |
| 异常通知 | 异常处理 控制事务                    |
| 最终通知 | 记录日志(方法已经调用，但不一定成功) |





# Spring AOP原理

> Spring在创建bean时，除了创建目标对象bean之外，会根据aop的配置，生成目标对象的代理对象，将其存储，之后获取bean时得到的其实是代理对象，在代理对象中，根据配置的切入点规则，决定哪些方法不处理直接执行目标方法，哪些方法拦截后进行增强，需要增强的方法拦截后根据配置调用指定切面中的指定通知执行增强操作。

1. Spring自动为目标对象生成代理对象

   - 默认情况下，如果目标对象实现过接口，则采用java的动态代理机制
   - 如果目标对象没有实现过接口，则采用cglib动态代理。

2. 开发者可以可以在spring中进行配置，要求无论目标对象是否实现过接口，都强制使用cglib动态代理。

   > `<aop:config proxy-target-class="true">`
   >
   > - `proxy-target-class="true"`：代理全部使用cglib

   ```xml
   <!--配置切面-->
   <aop:config proxy-target-class="true">
       <!--配置切入点-->
       <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
       <aop:aspect ref="firstAspect">
         
           <aop:around method="myAround" pointcut-ref="pc01"></aop:around>
   
       </aop:aspect>
   </aop:config>
   ```



# Spring AOP注解方式实现

> spring也支持注解方式实现AOP，相对于配置文件方式，注解配置更加的轻量级，配置、修改更加方便，是目前最流行的方式。

1. 开启AOP注解配置方式

   ```xml
   <!--开启主借支持AOP-->
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   ```

   

2. 将指定的类标志位一个切面

   > `@Aspect`

   ```java
   @Component
   @Aspect
   public class SecondAspect {}
   ```

   

3. 配置通知，指定切入点规则（切入点表达式）

   > 前置通知 `@Before`
   >
   > 环绕通知 `@Around`
   >
   > 后置通知 `@AfterReturning`
   >
   > 异常通知 `@AfterThrowing`
   >
   > 最终通知 `@After`
   >
   >
   > **等同于配置文件**：
   >
   > ![](img\AOP注解配置通知.png)

   ```java
   @Component
   @Aspect
   public class SecondAspect {
   
   
       //前置通知
       @Before("execution(* cn.tedu.service..*(..))")
       public void before(JoinPoint jp){
           System.out.println("second前置通知。。。");
       }
   
       @Around("execution(* cn.tedu.service..*(..))")
       public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
           System.out.println("second环绕前");
           Object proceed = pjp.proceed();
           System.out.println("second环绕后");
           return proceed;
   
       }
       @AfterReturning(value = "execution(* cn.tedu.service..*(..))",returning = "retObj")
       public void myAfterReturning(Object retObj){
           System.out.println("second后置通知"+retObj);
       }
   
       @AfterThrowing("execution(* cn.tedu.service..*(..))")
       public void myAfterThrowing(){
           System.out.println("second异常通知");
       }
   
       @After("execution(* cn.tedu.service..*(..))")
       public void myAfter(){
           System.out.println("second最终通知");
       }
   }
   
   ```

   

4. 如果一个切面中多个通知用到相同的切入点表达式，则可以将该切入点表达式单独定义，后续引用。

   > 注意，在当前切面中通过注解定义的切入点只在当前切面中起作用，其他切面看不到。
   >
   > **图示**：
   >
   > ![](img\AOP配置多个相同切入点.png)

   ```java
   @Component
   @Aspect
   public class SecondAspect {
   
       @Pointcut("execution(* cn.tedu.service..*(..))")
       public void mx(){}
   
       //前置通知
       @Before("mx()")
       public void before(JoinPoint jp){
           System.out.println("second前置通知。。。");
       }
   
       @Around("mx()")
       public Object around(ProceedingJoinPoint pjp) throws Throwable {
           System.out.println("second环绕前");
           Object proceed = pjp.proceed();
           System.out.println("second环绕后");
           return proceed;
       }
       
       @AfterReturning(value = "mx()",returning = "retObj")
       public void afterReturning(Object retObj){
           System.out.println("second后置通知"+retObj);
       }
   
       @AfterThrowing(value = "mx()",throwing = "e")
       public void afterThrowing(Throwable e){
           System.out.println("second异常通知");
       }
   
       @After("mx()")
       public void after(){
           System.out.println("second最终通知");
       }
   }
   
   ```

5. 在==后置通知==的注解中，也可以额外配置一个`returning`属性，来指定一个参数名接受目标方法执行后的返回值

   ```java
   @AfterReturning(value = "mx()",returning = "retObj")
   public void afterReturning(Object retObj){
       System.out.println("second后置通知"+retObj);
   }
   ```

   

6. 在==异常通知==的注解中，也可以额外配置一个`throwing`属性，来指定一个参数名接受目标方法抛出的异常对象

   ```java
   @AfterThrowing(value = "mx()",throwing = "e")
   public void afterThrowing(Throwable e){
       System.out.println("second异常通知");
   }
   ```

   

