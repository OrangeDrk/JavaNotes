[TOC]

# 注解回顾

1. 注释

   给人看的提示信息，人看了提示信息了解程序的内容

   java中注释的格式：`//` ，`/**/`， `/** */`

2. 注解

   jdk5.0提供的新特性，可以用来实现轻量化配置，具有广泛的应用场景

## JDK内置注解

> sun提供了jdk内置注解

1. `@Override`：重写

   声明重写父类方法的注解，要求编译器帮我们检查是否成功的覆盖，如果没有成功覆盖父类方法，编译器将会进行报错提示

2. `@Deprecated`：标记过时

   声明方法被过时，不再建议使用，要求编译器在编译的过程中对于这样的方法的调用提出警告，提示方法过时。

3. `@SuppressWarnings`：压制警告

   压制警告，提示编译器，在编译的过程中对指定类型的警告不再提示



------



## 自定义注解

> 开发自定义注解的过程非常类似开发一个接口

### 1.声明一个注解类，由`@interface`修饰

### 2.通过==元注解==修饰注解的定义

> 元注解：用来修饰注解声明的注解，可以控制被修饰的注解的特性。

#### `@Target`：（常用）

> 作用：==用于描述注解的使用范围（即”被描述的注解可以用在什么地方）==
>
> 使用：`@Target(value=ElementType.TYPE)`；多个用`{}`==数组==表示
>
> | **所修饰范围**                               | **取值ElementType**                                          |
> | -------------------------------------------- | ------------------------------------------------------------ |
> | package 包                                   | PACKAGE                                                      |
> | 类、接口、枚举、Annotation类型               | TYPE                                                         |
> | 类型成员（方法、构造方法、成员变量、枚举值） | CONSTRUCTOR：用于描述构造器<br />FIELD：用于描述域<br />METHOD：用于描述方法 |
> | 方法参数和本地变量                           | LOCAL_VARIABLE：用于描述局部变量<br />PARAMETER：用于描述参数 |
>
> 

实例：

```java
@Target(
    {
        ElementType.TYPE, /*可以用在类上*/
        ElementType.METHOD, /*可以用在方法上*/
        ElementType.FIELD, /*可以用在成员变量上*/
        ElementType.CONSTRUCTOR, /*可以用在构造方法上*/
        ElementType.LOCAL_VARIABLE, /*可以用在方法中的本地变量上*/
        ElementType.PARAMETER,/*可以用在方法的参数上*/
        ElementType.PACKAGE/*可以用在包上*/
    }
)
```

#### `@Retention`:（常用）

> 作用：==表示需要在什么级别保存该注解信息，用于描述注解的生命周期==
>
> 使用：`@Retention(RetentionPolicy.SOURCE)`
>
> | **取值   RetentionPolicy** | **作用**                                                     |
> | -------------------------- | ------------------------------------------------------------ |
> | SOURCE                     | 在源文件中有效（即源文件保留）(给编译器看)                   |
> | CLASS                      | 在class文件中有效（即源码、class保留）（给类加载器看）       |
> | ==RUNTIME==（常用）        | 在运行时有效（源码、class、运行时保留）<br />为Runtime可以被反射机制读取<br />==只有RUNTIME级别的注解才可以通过反射技术进行反射。== |
>
> 

实例：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation01 {
}
```



#### `@Documented`

> 用来声明被修饰注解是否要被文档提取工具提取到文档中。
>
> ==默认不提取。==

#### `@inherited`

> 被修饰的注解是否具有继承性
>
> ==默认没有继承性==



### 3.为注解增加属性

> 注解类中可以声明属性

1. 为注解类声明属性的过程非常类似于为接口定义方法。

   但要求，注解中的所有的==属性==必须是==public==的，可以显式声明，也可以不声明，==不声明默认就是public的==。

2. ==注解中的属性只能是八种基本数据类型、String类型、Class类型、枚举类型、其他注解类型、及以上类型的一维数组。==

   ```java
   @Target(
       {ElementType.TYPE, 
        ElementType.METHOD, 
        ElementType.FIELD, 
        ElementType.CONSTRUCTOR, 
        ElementType.LOCAL_VARIABLE, 
        ElementType.PARAMETER})
   @Retention(RetentionPolicy.RUNTIME)
   
   public @interface First {
       public String name() default "张三";
       public int age() default 18;
       String[] addr() default {};
       String value();
   }
   ```

   

3. 注解中声明的==属性== 需要在==使用注解时为其赋值== 

   > 赋值的方式 :
   >
   > - 在注解后跟一对小括号 ,在其中通过 `属性名=属性值` 的方式指定属性的值

   ```java
   @First(name = "属性值")
   public class test {}
   ```

   

4. 在声明注解时，在注解的属性后通过`default`关键字声明属性的默认值。声明过默认值的属性可以在使用注解时不赋值，默认采用默认值 ，也可以手动赋值覆盖默认值

   ```java
   //注解
   public @interface First {
       public String name() default "张三";
       public int age() default 18;
       String[] addr() default {};
       String value();
   }
   
   //实体类
   @First(value = "属性值",name = "王五")
   public class test {}
   ```

   

5. 如果属性是 一维数组类型：在传入的数组中只有一个值，则包括数组的大括号可以省略

   ```java
   //注解
   public @interface First {
       public String name() default "张三";
       public int age() default 18;
       String[] addr() default {};
       String value();
   }
   
   //实体类
   @First(addr = "数组1")//只有一个属性
   //@First(addr = {"数组","数组2","数组3"})//多个属性
   public class test {
   }
   ```

   

6. 如果注解的属性==只有一个需要赋值==，==且该属性的名称叫做value==。 则在使用注解时 `value=` 可以不写

   ```java
   //注解
   public @interface First {
       public String name() default "张三";
       public int age() default 18;
       String[] addr() default {};
       String value();
   }
   
   //实体类
   @First("属性值")//对应的就是value=“属性值”
   public class test {}
   ```

   

------



## 反射注解

> 只有RUNTIME级别的注解可以被反射
>
> 可以通过反射技术获取注解信息，控制程序执行

### 反射注解的原理

> `RetentionPolicy.RUNTIME`级别的注解会保留到运行其，可以通过反射技术获取，从而可以根据是否有注解 或 注解属性值的不同，进而控制程序按照不同方式运行。

反射相关的类型中都提供了反射注解的方法：

| 返回值                   | 方法                           | 介绍                                                         |
| ------------------------ | ------------------------------ | ------------------------------------------------------------ |
| boolean                  | isAnnotationPresent(xxx.class) | 如果指定类型的注释存在于此元素上，则返回 true，否则返回 false。 |
| <A extends Annotation> A | getAnnotation(xxx.class)       | 如果存在该元素的指定类型的注释，则返回这些注释，否则返回 null。 |
| Annotation[]             | getAnnotations()               | 返回此元素上存在的所有注释。                                 |

### 反射注解案例

```java
enum PoliceLevel{协警,交警,刑警,警督;}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)//保留在什么阶段
@interface Level{ PoliceLevel value();}

@Level(PoliceLevel.协警)
class Police{
    public static void fakuan(){
        System.out.println("您好，您超速，罚款200");
    }
}


// 反射注解
public class Test03 {
    public static void main(String[] args) {
        Police.fakuan();
        if (Police.class.isAnnotationPresent(Level.class)) {
            Level lvl = Police.class.getAnnotation(Level.class);
            PoliceLevel value = lvl.value();
            switch (value) {
                case 交警:
                    System.out.println("交200罚款");break;
                case 协警:
                    System.out.println("少点");break;
                case 警督:
                    System.out.println("交200罚款,给根烟");break;
                case 刑警:
                    System.out.println("交200闪现");break;
            }
        }else {
            System.out.println("KO!!!");
        }
    }
}

```







------

# Spring IOC

## Spring注解方式实现IOC

1. 导入开发包

   ![](img\Spring注解实现IOC.png)

2. 修改Spring配置文件：applicationContext.xml；导入context约束

   > 1. `xmlns="http://www.springframework.org/schema/beans"`:等号后面的字符串叫做：名称空间
   >
   > 2. 可以起别名`xmlns:abc="http://www.springframework.org/schema/beans"`
   >
   >    - 通过abc别名代替后面复杂的名称
   >    - 该xml文件下的标签使用变成：`<abc:bean> </abc:bean>`
   >
   > 3. `xsi:schemaLocation`作用：
   >
   >    - 将名称空间和对应的xml约束关联起来
   >
   > 4. ==修改的部分==：
   >
   >    - 添加名称空间：`xmlns:context="http://www.springframework.org/schema/context"`
   >
   >    - 添加该名称空间的约束（在`xsi:schemaLocation`中添加）：
   >    
   >      ```xml
   >      http://www.springframework.org/schema/context
   >      http://www.springframework.org/schema/context/spring-context.xsd"
   >      ```
   >

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
   
   </beans>
   ```

   

3. 开启包扫描

   > 在配置文件中，开启包扫描，指定spring自动扫描哪些个包下的类。只有在指定的扫描包下的类上的IOC注解才会生效。
   >
   > `<context:component-scan base-package="包名"></context:component-scan>`
   >
   > - Spring启动时，会扫描指定包名下的所有文件，包括子包。扫描到有配置注解`@Component`的，将自动注册为Spring的容器中，相当于`<bean>`标签

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
   
       <!--开启包扫描-->
       <context:component-scan base-package="cn.tedu.domain"></context:component-scan>
   </beans>
   ```

   

4. 使用注解注册bean

   > `@Component`:表明要注册为bean标签
   >
   > - `id`：默认是该类名的小写（如果类名第二个字母大写，id和类名相同）
   > - `class`：默认该类的全类名

   ```java
   import org.springframework.stereotype.Component;
   
   
   /*表示要注册为bean标签
       <bean id="person" class="cn.tedu.domain.Person"></bean>
           id默认为类名小写，也可以指定
   */
   @Component
   public class Person {}
   
   @Component
   public class Dog {}
   
   @Component
   public class Cat {}
   
   
   ```

   

5. bean的id和class

   1. 通常情况下注解注册bean使用==类名首字母小写==(其它位置字母不变)为bean的id

      - 但是如果类名的==第二个字母为大写==则==类名保留原样==做为id.

        ```java
        //cn.tedu.beans.Person
        <bean id="person" class="cn.tedu.beans.Person"/>
            
        //cn.tedu.beans.PErson
        <bean id="PErson" class="cn.tedu.beans.PErson"/>
            
        //cn.tedu.beans.NBA
        <bean id="NBA" class="cn.tedu.beans.NBA"/>
        ```

   2. 也可以通过在`@Component`中配置`value`属性，==明确的指定bean的id==

      > 一旦手动指定id，则不再自动推断id

      ```java
      @Component("per")
      public class PErson{}
      ```



## 注解方式：工厂配置Bean

1. 将工厂注册为Bean

2. 将生产对象的方法通过`@Bean`标签标注即可

   > 默认通过方法名自动推断出id，也可以自定义id
   >
   > `@Bean`:用于把当前方法的返回值作为bean对象存入Spring的IOC容器中
   >
   > - 属性：
   >   - name：用于指定bean 的id（不写，默认值是当前方法的名称）

   ```java
   @Component
   public class PersonFactory {
   
       @Bean()
       /*默认id是方法名
       * 可以自定义id
       * */
       public Person getInstance(){
           return new Person("zs");
       }
   }
   ```

3. Person的JavaBean

   ```java
   public class Person {
       public Person(String name) {
           System.out.println("Person 创建了");
       }
   }
   ```

4. 测试

   ```java
   public class Test01 {
   
       @Test
       public void test(){
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           //默认使用了工厂中获取Bean对象的方法名
           Person p = (Person) context.getBean("getInstance");
           System.out.println(p);
           ((ClassPathXmlApplicationContext) context).close();
       }
   }
   ```

   



------

# Spring DI

## Spring注解方式实现DI

1. 导入开发包

   ![](img\Spring注解实现IOC.png)

2. 编写配置文件，导入context、util约束;开启IOC包扫描、开启注解配置DI

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:utils="http://www.springframework.org/schema/util"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/util
          http://www.springframework.org/schema/util/spring-util.xsd">
   	
       <!--开启包扫描-->
       <context:component-scan base-package="cn.tedu.domain"></context:component-scan>
   
       <!--开启注解方式DI-->
       <context:annotation-config></context:annotation-config>
   
   
   </beans>
   ```
   
3. 注解方式注入==内置支持的类型数据==

   - ==原始注入方式==

     > 注入Spring内置支持的类型数据（基本数据类型，String类型）
     >
     > 通过`@Value("值")`注解实现Spring内置支持的类型的属性的注入

     ```java
     @Component
     public class Hero {
         @Value("亚瑟")
         private String name;
     
         @Value("55")
         private int age;
     
         @Override
         public String toString() {
             return "Hero{" +"name='" + name + '\'' +
                     ", age=" + age +'}';
         }
     }
     ```

   - ==通过properties配置文件==，注入基本数据类型

     - my.properties

       ```properties
       name=兰陵王
       age=33
       ```

     - 修改Spring配置文件

       > 添加标签，配置加载外部文件

       ```xml
       <!--配置加载外部文件-->
       <context:property-placeholder location="my.properties"></context:property-placeholder>
       ```

     - JavaBean

       ```java
       @Component
       public class Hero {
           @Value("${name}")
           private String name;
       
           @Value("${age}")
           private int age;
       
           @Override
           public String toString() {
               return "Hero{" +"name='" + name + '\'' +
                       ", age=" + age +'}';
           }
       }
       ```

4. 注解方式注入==集合类型数据==

   > 引入util名称空间，通过适当的util标签注册数据（上面已经配置）

   - Spring配置文件

     ```xml
     <!--配置集合-->
     <!--配置List-->
     <utils:list id="l1">
         <value>list1</value>
         <value>list2</value>
         <value>list3</value>
     </utils:list>
     
     <!--配置Set-->
     <utils:set id="s1">
         <value>s1</value>
         <value>s2</value>
         <value>s3</value>
     </utils:set>
     
     <!--配置Map-->
     <utils:map id="m1">
         <entry key="k1" value="v1"></entry>
         <entry key="k2" value="v2"></entry>
         <entry key="k3" value="v3"></entry>
     </utils:map>
     <!--配置Properties-->
     <utils:properties id="p1">
         <prop key="p1">v1</prop>
         <prop key="p2">v2</prop>
         <prop key="p3">v3</prop>
     </utils:properties>
     ```

   - JavaBean中通过`@Value`注入

     ```java
     @Component
     public class Hero {
         
         @Value("#{@l1}")
         private List<String> list;
         
         @Value("#{@s1}")
         private Set<String> set;
         
         @Value("#{@m1}")
         private Map<String,String> map;
         
         @Value("#{@p1}")
         private Properties pros;
     
     }
     ```

5. 注解方式注入==自定义Bean类型数据==

   > 在Bean中的属性上通过`@Autowired`、`@Qualifier("id值")`实现自定义Bean类型的属性注入
   >
   > - 只配置了`@Autowired`：
   >   - 先按照类型查找，找到唯一的对象：注入
   >   - 找不到/找到多个：开始按照id查找
   >   - 把当前属性名当作id，寻找bean。
   >     - 找到：注入
   >     - 找不到：抛出异常
   > - 同时配置了`@Autowired`、`@Qualifier("id值")`
   >   - 按照`@Qualifier("id值")`中配置的id值去找
   >     - 找到：注入
   >     - 找不到：抛出异常
   > - 配置`@Resource(name = "id值")`（不常用）
   >   - Java官方提供的，相当于`@Autowired`+`@Qualifier("id值")`

   ```java
   @Component
   public class Hero {  
       @Autowired
       @Qualifier("dog")  //定向查找
       private Dog dog;
   
   }
   ```

   



------



# 其他注解

## @Scope(value="prototype")

> 配置修饰的类的bean是单例还是多例
>
> - 不配置：默认为单例

```java
@Scope("prototype")//配置多例
public class Person {
    private String name;
    private int age;
}
```



## @Lazy

> 配置修饰的类的bean采用懒加载机制
>
> - 不配置：默认不使用懒加载

```java
@Lazy//配置懒加载
public class Person {
    private String name;
    private int age;
}

```



## @PostConstruct

> 在bean对应的类中 修饰某个方法 将该方法==声明为初始化方法==，对象创建之后立即执
>
> - 相当于applicationContext.xml中`<bean>`标签的`init-method`属性

```java
public class Person {
    private String name;
    private int age;
    
    @PostConstruct
    public void init(){
        System.out.println("初始化方法...");
    }
}
```



## @PreDestroy

> 在bean对应的类中 修饰某个方法 。将该方法==声明为销毁的方法==，对象销毁之前调用的方法。
>
> - 相当于applicationContext.xml中`<bean>`标签的`destroy-method`属性

```java
public class Person {
    private String name;
    private int age;
    
    @PreDestroy
    public void destroy(){
        System.out.println("销毁方法");
    }  
}
```



## 注册Bean的注解

> @Controller、@Service、@Repository、@Component
>
> 这四个注解的功能是==完全相同==的，都是用来修饰类，将类==声明为Spring管理的bean==的。

1. `@Component`一般认为是通用的注解
2. `@Controller`用在软件分层中的控制层，一般用在web层
3. `@Service`用在软件分层中的业务访问层，一般用在service层
4. `@Repository`用在软件分层中的数据访问层，一般用在dao层



## @Configuration

> ==Spring新注解==
>
> 作用：指定当前类是一个配置类（可以把这个类当做applicationContext.xml使用）

```java
@Configuration
public class SpringConfiguration {
}
```



## @ComponentScan

> ==Spring新注解==
>
> 作用：用于通过注解指定Spring在创建容器时要扫描的包
>
> 属性：底层是数组 `String [] value`,可传多个参数，
>
> - `value`：它和`basePackages`的作用是一样的，都是用于指定创建容器时要扫描的包
> - `basePackages`:和value作用一样
>
> 等同于Spring配置文件applicationContext.xml中的：
>
> `<context:component-scan base-package="com.sxh.spring"></context:component-scan>`

```java
@Configuration //声明为Spring的配置类
@ComponentScan(basePackages = "com.sxh.spring") //声明要扫描的包
public class SpringConfiguration {
}
```



## @Bean

> `@Bean`:用于把当前方法的返回值作为bean对象存入Spring的IOC容器中
>
> - 属性：
>   - name：用于指定bean 的id（不写，默认值是当前方法的名称）
> - 细节：
>   - 当我们使用注解配置方法时，如果方法有参数，Spring框架会去容器中查找有没有可用的bean对象
>   - 查找方式：同`@Autowired`相同

```java
@Configuration
@ComponentScan(basePackages = "com.sxh.spring")
public class SpringConfiguration {

    @Bean(name = "person")
    public Person createPerson(Cat cat){
        return new Person();
    }

    @Bean(name = "cat")
    public Cat createCat() {
        return new Cat();
    }

}
```

图解：

![image-20191229175226516](img\Java类替代Spring的xml细节.png)



# 注解Java类替换xml

> 通过Spring新注解：`@configuration`和`@ComponentScan`可以把Spring的配置文件（ApplicationContext.xml）替换成Java类

1. Spring的配置文件：ApplicationContext.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              http://www.springframework.org/schema/context/spring-context.xsd">
   
       <!--开启包扫描-->
       <context:component-scan base-package="com.sxh.spring"></context:component-scan>
   
       <bean id="person" class="com.sxh.spring.Person">
           <property name="cat" ref="cat"></property>
       </bean>
   
       <bean id="cat" class="com.sxh.spring.di.Cat">
           <property name="name" value="猫咪"></property>
       </bean>
   
   </beans>
   ```

   

2. 通过注解声明的Java类

   ```java
   package config;
   
   import com.sxh.spring.Person;
   import com.sxh.spring.di.Cat;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration //声明这个Java类是一个spring配置类
   @ComponentScan(basePackages = "com.sxh.spring") //声明包扫描
   public class SpringConfiguration {
   
       @Bean(name = "person") //把方法返回值添加到spring容器
       //spring会自动在容器中匹配该方法参数类型相同的bean，并添加进来
       //自动匹配机制和@Autowired一样
       public Person createPerson(Cat cat){
           return new Person(cat);
       }
   
       @Bean(name = "cat") //配置一个id为cat的bean
       public Cat createCat() {
           return new Cat();
       }
   
   }
   
   ```

3. 测试

   > 获取容器时，和之前xml配置时获取不一样
   >
   > `ApplicationContext ctx = `
   >
   > - 配置文件为xml时：参数为：xml文件名称
   >
   >   `new ClassPathXmlApplicationContext("applicationContext.xml");`
   >
   > - 配置文件为Java时：参数为`Java配置类.class`
   >
   >   `new AnnotationConfigApplicationContext(SpringConfiguration.class);`

