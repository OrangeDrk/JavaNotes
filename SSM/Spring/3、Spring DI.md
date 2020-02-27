[TOC]

# DI

创建对象的过程中Spring可以依据配置对对象的属性进行设置，这个过称之为依赖注入,也即DI。

# set方法注入（==常用==）

> 通常的javabean属性都会私有化，而对外暴露`setXxx()`、`getXxx()`方法，此时spring可以通过这样的`setXxx()`方法将属性的值注入对象。
>
> 注意：
>
> - 类属性：类中的成员变量
> - bean属性：类中的get或set方法
>   - `getName()`:bean属性为name
>   - `setAge()`:bean属性为age

## 基本类型注入

> int，String，set，list，map，properties

1. **javaBean**

   ```java
   public class Hero {
       private int id;
       private String name;
       private List<String> list;
       private Set<String> set;
       private Map<String,String> map;
       private Properties props;
   
       @Override
       public String toString() {
           return "Hero{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   ", list=" + list +
                   ", set=" + set +
                   ", map=" + map +
                   ", props=" + props +
                   '}';
       }
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public List<String> getList() {
           return list;
       }
   
       public void setList(List<String> list) {
           this.list = list;
       }
   
       public Set<String> getSet() {
           return set;
       }
   
       public void setSet(Set<String> set) {
           this.set = set;
       }
   
       public Map<String, String> getMap() {
           return map;
       }
   
       public void setMap(Map<String, String> map) {
           this.map = map;
       }
   
       public Properties getProps() {
           return props;
       }
   
       public void setProps(Properties props) {
           this.props = props;
       }
   }
   
   ```

   

2. **Spring配置文件**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="hero" class="cn.tedu.domain.Hero">
           <!--基本类型注入（int，String，…………）-->
           <property name="id" value="3"></property>
           <property name="name" value="亚瑟"></property>
           
           <!--List类型注入：-->
           <property name="list">
               <list>
                   <value>1v1</value>
                   <value>2v2</value>
                   <value>3v3</value>
               </list>
           </property>
           
           <!--Set类型注入-->
           <property name="set">
               <set>
                   <value>sv1</value>
                   <value>sv2</value>
                   <value>sv3</value>
               </set>
           </property>
   
           <!--Map类型注入：-->
           <property name="map">
               <map>
                   <entry key="k1" value="v1"></entry>
                   <entry key="k2" value="v2"></entry>
                   <entry key="k3" value="v3"></entry>
                   <entry key="k3" value="v4"></entry>
               </map>
           </property>
           
           <!--Properties类型注入-->
           <property name="props">
               <props>
                   <prop key="pk1">pk1</prop>
                   <prop key="pk2">pk2</prop>
                   <prop key="pk3">pk3</prop>
               </props>
           </property>
       </bean>
   </beans>
   ```

3. **测试**

   ```java
   public class Test {
       /**
        * 基本数据类型 String，map，set,list property注入
        */
       @org.junit.Test
       public void test(){
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           Hero hero = (Hero) context.getBean("hero");
           System.out.println(hero);
   
           ((ClassPathXmlApplicationContext) context).close();
       }
   }
   //
   /*Hero{id=3, name='亚瑟', list=[1v1, 2v2, 3v3], 
   	set=[sv1, sv2, sv3], map={k1=v1, k2=v2, k3=v4}, 
   	props={pk3=pk3, pk2=pk2, pk1=pk1}}
   */
   ```

   

## 自定义bean的注入

> javaBean类中引用其他的javaBean类

1. Dog的javaBean

   ```java
   public class Dog {
       private int age;
       private String name;
   
       @Override
       public String toString() {
           return "Dog{" +
               "age=" + age +
               ", name='" + name + '\'' +
               '}';
       }
   
       public int getAge() {return age;}
   
       public void setAge(int age) {this.age = age;}
   
       public String getName() {return name;}
   
       public void setName(String name) {this.name = name;}
   
   }
   
   ```

   

2. Hero的javaBean

   ```java
   public class Hero {
       private int age;
       private String name;
       private Dog dog;
   
       public Dog getDog() {return dog;}
   
       public void setDog(Dog dog) {this.dog = dog;}
   
       @Override
       public String toString() {
           return "Hero{" +
                   "age=" + age +
                   ", name='" + name + '\'' +
                   ", dog=" + dog +
                   '}';
       }
   
       public int getAge() {return age;}
   
       public void setAge(int age) {this.age = age;}
   
       public String getName() {return name;}
   
       public void setName(String name) {this.name = name;}
   }
   
   ```

   

3. Spring配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="hero" class="cn.tedu.domain.Hero">
           <property name="name" value="亚瑟"></property>
           <property name="age" value="55"></property>
           <property name="dog" ref="dog"></property><!--引用其他的bean-->
       </bean>
   
       <bean id="dog" class="cn.tedu.domain.Dog">
           <property name="age" value="3"></property>
           <property name="name" value="旺财"></property>
       </bean>
   
   </beans>
   ```

   

4. 测试

   ```java
   public class Test01 {
       //Setter 方式实现DI，自定义Bean注入
       @Test
       public void test01(){
           ApplicationContext context = 
               new ClassPathXmlApplicationContext("applicationContext.xml");
           Hero hero = (Hero) context.getBean("hero");
           System.out.println(hero);
   
           ((ClassPathXmlApplicationContext) context).close();
       }
   }
   //Hero{age=55, name='亚瑟', dog=Dog{age=3, name='旺财'}}
   ```

   

## 自动装配

> 一个JavaBean中应用多个其他的JavaBean，按照自定义bean注入太过繁琐，通过`<bean>`属性进行自动装配
>
> `autowire`:
>
> - `byName`:会自动找和当前bean属性名一致的bean对象来注入
>   - 找到唯一的一个对象：注入
>   - 找不到：不注入
>   - 不可能找到多个（id不可重复）
> - `byType`:会自动找和当前bean属性类型一致的bean对象来注入
>   - 找到唯一的一个对象：注入
>   - 找不到：不注入
>   - 找到多个：报异常—NoUniqueBeanDefinitionException

1. Dog的JavaBean

   ```java
   public class Dog {
       private int age;
       private String name;
       @Override
       public String toString() {
           return "Dog{" +"age=" + age +", name='" + name + '\'' +'}';
       }
       public int getAge() {return age;}
       public void setAge(int age) {this.age = age;}
       public String getName() {return name;}
       public void setName(String name) {this.name = name;}
   }
   
   ```

2. Hero的JavaBean

   ```java
   public class Hero {
       private int age;
       private String name;
       private Dog dog;
   
       public Dog getDog() {return dog;}
       public void setDog(Dog dog) {this.dog = dog;}
       @Override
       public String toString() {
           return "Hero{" + "age=" + age +", name='" + name + '\'' +
               ", dog=" + dog +", cat=" + cat +'}';
       }
       public int getAge() {return age;}
       public void setAge(int age) {this.age = age;}
       public String getName() {return name;}
       public void setName(String name) {this.name = name;}
   }
   
   ```

   

3. Spring配置

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       
       <bean id="hero" class="cn.tedu.domain.Hero" autowire="byName">
           <property name="name" value="亚瑟"></property>
           <property name="age" value="55"></property>
           <!--<property name="dog" ref="dog"></property>  
   		配置自动装配后，这句话就不用再写了-->
       </bean>
   
       <bean id="dog" class="cn.tedu.domain.Dog">
           <property name="age" value="3"></property>
           <property name="name" value="旺财"></property>
       </bean>
   </beans>
   ```

   

4. 测试

   ```java
   @Test
   public void test01(){
       ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       Hero hero = (Hero) context.getBean("hero");
       System.out.println(hero);
   
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```

   

# 构造方法注入

> 对象属性设置的另一种方式是在对象创建的过程中通过构造方法传入并设置对象的属性。
>
> spring也可以通过这样的构造方法实现属性的注入。
>
> 通过`<bean>`标签的子标签`<constructor-arg>`实现
>
> - 属性
>   - `index`：指定构造器中第几个参数==（优先使用）==
>   - `name`：指定构造器中叫什么名字的参数
>   - `type`：指定构造器中什么类型的参数
>   - `value`：指定要设置的值
>   - `ref`：指定要设置的值——通过引用设置

1. Student的JavaBean

   ```java
   public class Student {
       private int id;
       private String name;
       private Dog dog;
   
       @Override
       public String toString() {
           return "Student{" +"id=" + id +", name='" + name + '\'' +
                   ", dog=" + dog +'}';
       }
       
       public Student(int id, String name, Dog dog) {
           this.id = id;
           this.name = name;
           this.dog = dog;
       }
       
       public Student() {}
   }
   
   ```

2. Dog的JavaBean

   ```java
   public class Dog {
       private int id;
       private String name;
   
       public String getName() {return name;}
       public void setName(String name) {this.name = name;}
       public int getId() {return id;}
       public void setId(int id) {this.id = id;}
       @Override
       public String toString() {
           return "Dog{" +"id=" + id +", name='" + name + '\'' +'}';
       }
   
   }
   ```

3. Spring的配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="student" class="cn.tedu.domain.Student">
           <constructor-arg index="0" value="3"></constructor-arg>
           <constructor-arg index="1" value="张三"></constructor-arg>
           <constructor-arg index="2" ref="dog"></constructor-arg>
       </bean>
       
       <bean id="dog" class="cn.tedu.domain.Dog">
           <property name="name" value="旺财"></property>
           <property name="id" value="10"></property>
       </bean>
   
   </beans>
   ```

4. 测试

   ```java
   public class test01 {
       //基于构造器的注入
       @Test
       public void test(){
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           Student student = (Student) context.getBean("student");
           System.out.println(student);
           ((ClassPathXmlApplicationContext) context).close();
       }
   }
   //Student{id=3, name='张三', dog=Dog{id=10, name='旺财'}}
   ```

   