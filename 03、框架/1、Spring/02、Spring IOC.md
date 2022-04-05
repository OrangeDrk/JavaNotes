[TOC]



------



# IOC（DI）

> IOC(DI)：控制反转（依赖注入）
>
> 所谓的IOC称之为控制反转，==简单来说就是将对象的创建的权利及对象的生命周期的管理过程交由Spring框架来处理，从此在开发过程中不再需要关注对象的创建和生命周期的管理，而是在需要时由Spring框架提供，这个由spring框架管理对象创建和生命周期的机制称之为控制反转。==而在创建对象的过程中Spring可以依据配置对对象的属性进行设置，这个过称之为依赖注入,也即DI。

1. IOC+接口可以实现软件分层时的每个层之间的彻底解耦，用接口接收实现类（多态）



------



# 入门案例

1. 创建一个java项目

   - Spring并不是非要在javaweb环境下才可以使用，一个普通的java程序中也可以使用Spring

2. 将需要用到的jar包导入Spring项目下的lib文件夹

   ![](img\lib包.png)

3. 创建Spring配置文件：applicationContext.xml

   > Spring采用xml文件作为配置文件，xml文件名任意，但通常取名为applicationContext.xml。
   >
   > - 通常将该文件放置在类加载的目录下（src目录），方便后续使用
   > - 右键src—New—XML Configuration File—Spring Config
   >   - （如果没有此选项，检查Spring所需要的jar包是否导入正确）

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   </beans>
   ```

4. 创建bean类，并在Spring中进行配置，交由Spring容器管理

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--id不能重复-->
       <bean id="person" class="cn.tedu.domain.Person"></bean>
       <bean id="personx" class="cn.tedu.domain.Person"></bean>
   </beans>
   ```

5. 在程序中通过Spring容器获取对象并使用

   ```java
   /**
    * SpringIOC方式创建并管理
    */
   @Test
   public void test01(){
       //1.初始化容器
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       //2.通过Spring容器获取bean
       Person p = (Person) context.getBean("person");
       p.eat();
       p.say();
       //3.关闭Spring容器
       ((ClassPathXmlApplicationContext)context).close();
   
   }
   ```

   



------



# IOC的实现原理

1. 在初始化一个Spring容器时，Spring会去解析指定的xml文件，当解析到其中的`<bean>`标签时，会根据该标签中的class属性指定的类的全路径名，==通过反射创建该类的对象==，并将该对象==存入内置的Map==中管理。

   > 注意：底层通过反射创建该类的对象：`Person p = Class.forName("cn.tedu.domain.Person")`
   >
   > - 默认是调用了该类的无参构造，如果没有无参构造，Spring的配置文件applicationContext.xml中的`<bean>`标签就会报错
   > - ==javaBean中必须要有无参构造==

2. 其中==键==就是该==标签的id值==，==值==就是==该对象==。之后，当通过`getBean()`方法来从容器中获取对象时，其实就是根据传入的条件在内置的Map中寻找是否有匹配的键值，如果有则将该键值对中保存的对象返回，如果没有匹配到则抛出异常。

>  ==**由此可以推测而知：**==
>
> - 默认情况下，多次获取同一个id的bean，得到的将是同一个对象。
>
> - 不可以配置多个id相同的bean
> - 可以配置多个id不同但class相同的bean

```java
/**
 * 1.id不能相同
 * 2.在id不同时，class可以相同
 */
@Test
public void test03(){
    //1.初始化容器
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    //2.通过Spring容器获取bean
    Person p = (Person) context.getBean("person");
    Person p1 = (Person) context.getBean("personx");
    System.out.println(p);
    System.out.println(p1);
    System.out.println(p==p1);//false：id不同获取的对象也不相同
    ((ClassPathXmlApplicationContext)context).close();

}
```



------

# 获取bean的方式

> 三种方式：
>
> 1. 通过id获取
> 2. 通过class获取
> 3. 通过id+class获取

## 通过id获取

> 返回结果：
>
> 1. 找到对应的唯一对象：返回该对象
> 2. 找不到对应的对象：抛出异常：NoSuchBeanDefinitionException
> 3. 不可能找到多个对象：==因为id不能重复，不可能出现此情况==

```java
@Test
public void test04(){
    ApplicationContext context = 
        new ClassPathXmlApplicationContext("applicationContext.xml");

    //方式一：通过id获取

    Person p = (Person) context.getBean("person11");
    System.out.println(p);

    ((ClassPathXmlApplicationContext) context).close();
}
```



## 通过class获取

> 返回结果：
>
> 1. 找到对应的唯一对象：返回该对象
> 2. 找不到对应的对象：抛出异常：NoSuchBeanDefinitionException
> 3. 找到多个对象：抛出异常：NoUniqueBeanDefinitionException
>
> 特点：
>
> - 如果Spring IOC在通过class获取bean时，找不到该类型的bean时，会去检查是否存在该类型的子孙类型的bean
>   - 有：返回该之孙类型的bean
>   - 没有/找到多个：抛出异常
> - 符合java面向对象思想中的多态的特性

```java
@Test
public void test04(){
    ApplicationContext context = 
        new ClassPathXmlApplicationContext("applicationContext.xml");

    //方式二：通过class获取
    Person p1 = context.getBean(Person.class);
    p1.say();
    System.out.println(p1);

    ((ClassPathXmlApplicationContext) context).close();
}
```



## 通过id+class获取

> 返回结果：
>
> 1. 找到对应的唯一对象：返回该对象
> 2. 找不到对应的对象：抛出异常：NoSuchBeanDefinitionException
> 3. 找到多个对象：==id+class能唯一定位一个bean==，不可能出现查到多个bean的情况

```java
@Test
public void test04(){
    ApplicationContext context = 
        new ClassPathXmlApplicationContext("applicationContext.xml");

    //方式三：通过id+class获取
    Person p = context.getBean("person1", Person.class);
    System.out.println(p);

    ((ClassPathXmlApplicationContext) context).close();
}
```





------



# 别名标签

> 在 Spring中提供了别名标签`<alias>`可以为配置的`<bean>`起一个别名，要注意的是这仅仅是对指定的`<bean>`起的一个额外的名字，并不会额外的创建对象存入map。
>
> 【可以为bean指定别名，之后可以通过这个别名取代id操作bean】
>

1. Spring配置文件

   > `<alias name="要起别名的bean的id"  alias="要指定的别名"/>`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="person" class="cn.tedu.domain.Person"></bean>
       <alias name="person" alias="pers"></alias><!--别名-->
   </beans>
   ```

2. 测试

   ```java
   /**
    * 别名标签
    * 可以通过别名标签为bean的id起一个别名，可以通过id获取bean，也可以通过别名取代id获取bean
    */
   @Test
   public void test05(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Person p = (Person) context.getBean("person");
       System.out.println("p:"+p);//p:cn.tedu.domain.Person@57536d79
       Person p1 = (Person) context.getBean("pers");
       System.out.println("p1:"+p);//p1:cn.tedu.domain.Person@57536d79
   	//p 和 p1 其实是同一个对象
       
       ((ClassPathXmlApplicationContext) context).close();
   
   }
   ```

   





------



# Spring创建对象的方式

## 无参构造创建对象

> javaBean中有无参构造，通过id、class、id+class创建bean时底层默认使用反射，调用无参构造创建bean对象

1. javaBean类

   ```java
   public class Person {
       public Person() {
       }
   
       public Person(String name) {
           System.out.println("我被创建了");
       }
   }
   
   ```

2. Spring配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--反射创建bean(底层使用无参构造)-->
       <bean id="person" class="cn.tedu.domain.Person"></bean>
   
   </beans>
   ```

3. 测试

   ```java
   /**
    * 无参构造创建对象，必须有无参构造
    */
   @Test
   public void test(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Person p = (Person) context.getBean("person");
       System.out.println(p);
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```

   



## 静态工厂创建对象

1. 静态工厂类

   ```java
   public class PersonStaticFactory {
       private PersonStaticFactory(){
   
       }
       public static Person getInstance(){
           return new Person("zzz");
       }
   }
   
   ```

2. Spring配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--静态工厂方式创建bean-->
       <bean id="person" class="cn.tedu.factory.PersonStaticFactory" factory-method="getInstance"></bean>
   
   </beans>
   ```

3. 测试

   ```java
   /**
    * 无参构造创建对象，必须有无参构造
    */
   @Test
   public void test(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Person p = (Person) context.getBean("person");
       System.out.println(p);
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```

   

## 实例工厂创建对象

1. 实例工厂类

   ```java
   public class PersonInstanceFactory {
       public Person getInstance(){
           return new Person("zzss");
       }
   }
   ```

2. Spring配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--实例工厂方式创建bean-->
       <bean id="personInstanceFactory" class="cn.tedu.factory.PersonInstanceFactory"></bean>
       <bean id="person" factory-bean="personInstanceFactory" factory-method="getInstance"></bean>
   
   </beans>
   ```

3. 测试

   ```java
   /**
    * 无参构造创建对象，必须有无参构造
    */
   @Test
   public void test(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Person p = (Person) context.getBean("person");
       System.out.println(p);
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```

   

## Spring工厂创建对象

1. Spring工厂类

   ```java
   public class PersonSpringFactory implements FactoryBean<Person> {
       //生产bean对象的方法
       @Override
       public Person getObject() throws Exception {
           return new Person("ww");
       }
   
       //获取bean类型的方法
       @Override
       public Class<?> getObjectType() {
           return Person.class;
       }
   
       //告知当前bean是否采用单例模式
       @Override
       public boolean isSingleton() {
           return true;//采用单例模式
       }
   }
   
   ```

2. Spring配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--Spring工厂方式创建bean-->
       <bean id="person" class="cn.tedu.factory.PersonSpringFactory"></bean>
   
   </beans>
   ```

3. 测试

   ```java
   /**
    * 无参构造创建对象，必须有无参构造
    */
   @Test
   public void test(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Person p = (Person) context.getBean("person");
       System.out.println(p);
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```





------



# 单例和多例

> 单例：一个bean只有一个对象，之后无论获取多少次都是获取的同一个对象
>
> 多例：每次获取时都重新创建一个新的对象

## 单例对象

> Spring容器管理的bean在默认情况下是单例的。
>
> - 即：一个bean只会创建一个对象，存在内置 map中，之后无论获取多少次该bean，都返回同一个对象。
>
> Spring默认采用单例方式，减少了对象的创建，从而减少了内存的消耗。

### bean在单例模式下的生命周期

- 容器初始化时创建对象，保存在Spring内部的map中，获取对象直接返回map内保存的对象
- 直到容器销毁时保存在Map中的对象跟着被销毁



## 配置单/多例对象（属性`scope`）

> 实际开发中是存在多例的需求的。
>
> - Spring也提供了选项可以将bean设置为多例模式。

1. Spring配置文件

   > `<bean>`标签属性：scope
   >
   > - ==singleton==:单例（大部分情况都是单例）
   > - ==prototype==：多例
   > - request：作用于web应用的请求范围
   > - session：作用于web应用的会话范围
   > - global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，他就是session

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
      <bean id="cart" class="cn.tedu.domain.Cart" scope="prototype"></bean>
   </beans>
   ```


### bean在多例模式下的生命周期：

- spring容器启动时解析xml发现该bean标签后，只是将该bean进行管理，并不会创建对象。
- 此后每次使用 `getBean()`获取该bean时，spring都会重新创建该对象返回，每次都是一个新的对象。这个对象spring容器并不会持有
- 什么销毁取决于使用该对象的用户自己什么时候销毁该对象。通过JVM的垃圾回收器进行回收



------



# 懒加载

> Spring默认会在容器初始化的过程中，解析xml，并将单例的bean创建并保存到map中
>
> - 这样的机制在bean比较少时问题不大，但一旦bean非常多时，spring需要在启动的过程中花费大量的时间来创建bean 花费大量的空间存储bean
> - 但这些bean可能很久都用不上，这种在启动时在时间和空间上的浪费显得非常的不值得。

## Spring提供的懒加载机制

> 规定指定的bean不在启动时立即创建，而是在后续第一次用到时才创建，从而减轻在启动过程中对时间和内存的消耗。
>
> ==懒加载机制只对单例bean有作用，对于多例bean设置懒加载没有意义。==

1. 为指定bean配置懒加载

   > `<bean>`标签的`lazy-init`属性：
   >
   > - default:  关闭懒加载
   >
   > - false:    关闭懒加载
   > - true:     开启懒加载

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="person" class="cn.tedu.domain.Person" lazy-init="true"></bean>
   </beans>
   ```

   

2. 为全局配置懒加载

   > `<beans>`标签`default-lazy-init`属性：
   >
   > - false:    关闭懒加载
   > - true:     开启懒加载

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
          default-lazy-init="true">
       <!--全局配置懒加载-->
   
       <bean id="person" class="cn.tedu.domain.Person"></bean>
   </beans>
   ```
   



------



# 配置初始化和销毁的方法

> 在Spring中如果某个bean在初始化之后 或 销毁之前要做一些 额外操作可以为该bean配置初始化和销毁的方法 ，在这些方法中完成要功能。

## Spring配置文件配置初始化和销毁方法

> `<bean>`标签中的属性：
>
> - 执行初始化方法的属性：`init-method="初始化方法"`
> - 执行销毁方法的属性：`destroy-method="销毁方法"`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="jdbcUtils" class="cn.tedu.domain.JDBCUtils" init-method="init" destroy-method="destory"></bean>
</beans>
```



## ==Spring中关键方法的执行顺序：==

- 在Spring创建bean对象时，先创建对象(通过无参构造或工厂)，
- 之后立即调用init方法来执行初始化操作，
- 之后此bean就可以拿来调用其它普通方法,
- 而在对象销毁之前，spring容器调用其destory方法来执行销毁操作。

