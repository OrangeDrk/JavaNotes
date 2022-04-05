[TOC]

# Spring整合JDBC — 管理数据源

## 导入相关开发包

![](img\Spring整合JDBC.png)



## 将数据源交给Spring管理

### 两个文件分开写

1. 连接池配置文件（c3p0-config.properties）

   ```properties
   driverClass=com.mysql.jdbc.Driver
   jdbcUrl=jdbc:mysql://localhost:3306/ssmdb
   user=root
   password=123456
   ```

   

2. Spring配置文件（applicationContext.xml）

   > 引入连接池配置文件：`c3p0-config.properties`

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
   
       <!--引入外部文件-->
       <context:property-placeholder location="classpath:/c3p0-config.properties"></context:property-placeholder>
       <!--配置数据源-->
       <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <property name="driverClass" value="${driverClass}"></property>
           <property name="jdbcUrl" value="${jdbcUrl}"></property>
           <property name="user" value="${user}"></property>
           <property name="password" value="${password}"></property>
       </bean>
   
   
   </beans>
   ```

### 两个文件合并

> 把数据源直接配置在Spring配置文件中

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

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///ssmdb"></property>
        <property name="user" value="root"></property>
        <property name="password" value="123456"></property>
    </bean>

</beans>
```



## 通过Spring获取数据源、连接，操作数据库

```java
public void test02(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    DataSource dataSource = (DataSource) context.getBean("dataSource");

    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        //1.注册数据库驱动
        //2.获取数据库连接
        conn = dataSource.getConnection();
        //3.获取传输器对象
        ps = conn.prepareStatement("select * from user where id < ?");
        ps.setInt(1,4);
        //4.传输sql执行获取结果集
        rs = ps.executeQuery();
        //5.遍历结果集
        while(rs.next()){
            String name = rs.getString("name");
            System.out.println(name);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }finally {
        if(rs!=null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                rs = null;
            }
        }
        ....
            ps.close();
        ....
            conn.close();
        ....
    }
}
```





------



# Spring整合JDBC — JDBC模板

> ==使用模版类能够极大的简化原有JDBC的编程过程。==

## 在Spring配置文件中配置JDBC模板

> 使用`JdbcTemplate`类
>
> 需要配置数据源（c3p0连接池）

```xml
<!--SpringJDBC-数据源-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///ssmdb"></property>
    <property name="user" value="root"></property>
    <property name="password" value="123456"></property>
</bean>

<!--配置JDBC模板类 需要配置数据源-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>

```

## 使用JDBC模板类实现CRUD

> 模板类操作：
>
> - 添加、删除、修改：`update()`
> - 查询：`query`、`queryForList`、`queryForMap`、`queryForRowSet`、`queryForObject`
>
> 为方便Test单元测试，把公共代码提取出来复用
>
> - `@Before`：Junit注解，表示运行时==先运行before标注的==
> - `@After`：Junit注解，表示==最后运行after标注的==
>
> ```java
> public class Test01 {
>     private ApplicationContext context = null;
>     private JdbcTemplate jdbcTemplate = null;
> 
>     @Before
>     public void beofre(){
>         context = new ClassPathXmlApplicationContext("applicationContext.xml");
>         jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");
>     }
> 
>     @After
>     public void after(){
>         ((ClassPathXmlApplicationContext)context).close();
>     }
> 
> }
> ```
>
> 

### 添加操作

```java
//新增
@Test
public void test01(){
    jdbcTemplate.update(
        "insert into user values (null,?,?)","zzz",99);
}
```



### 删除操作

```java
//删除
@Test
public void test03(){
    jdbcTemplate.update("delete from user where id=?",15);
}
```



### 修改操作

```java
//修改
@Test
public void test02(){
    jdbcTemplate.update("update user set age=? where id=?",88,15);
}
```



### 查询操作

#### 查询一个Map：`queryForMap`

> 特点：
>
> - `queryForMap`只支持查询一条信息
> - 将查询到的信息存放到Map中并返回

```java
//查询为一个Map
@Test
public void test04(){
    Map<String, Object> map = jdbcTemplate.queryForMap("select * from user where id = ?", 3);
    
    System.out.println(map);
}
//结果：{id=3, name=王五, age=21}
```



#### 查询一个List<Map>：`queryForList`

> 特点：
>
> - 能==查询多条信息==。
> - 将查询到的信息放到List集合中并返回

```java
// 查询为一个List<Map>
@Test
public void test05(){
    List<Map<String, Object>> list = 
        jdbcTemplate.queryForList("select * from user where id < ?", 3);
    System.out.println(list);
}
//结果：[{id=1, name=张三, age=20}, {id=2, name=李四, age=18}]
```



#### 查询一个SqlRowSet：`queryForRowSet`

> 特点：
>
> - 能==查询多条信息==。
> - 类似于JDBC的Query，返回的是ResultSet

```java
// 查询为一个SqlRowSet
@Test
public void test06(){
    SqlRowSet rs = jdbcTemplate.queryForRowSet("select * from user where id < ?", 3);
    while(rs.next()){
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setAge(rs.getInt("age"));
        System.out.println(user);
    }
}

//结果：
/*
User{id=1, name='张三', age=20}
User{id=2, name='李四', age=18}
*/
```



#### 查询一个Object：`queryForObject`

> 1. 特点：
>    - 只能==查询一条信息==。
>    - 把查询到的信息封装到一个指定的Bean中
>      - 如何存放，通过第二个参数（`new RowMapper<User>{}`）的匿名内部类中重写的`mapRow`方法规定
> 2. 使用共有两种方式：
>    - 方法一：自定义`RowMapper`实现映射（==书写麻烦，效率高==）
>    - 方法二：通过`BeanPropertyRowMapper`自动实现映射（==书写简单，效率低==）
>      - `new BeanPropertyRowMapper<T>(Class<T>)`
>      - 参数传入要返回的类型的`Xxx.Class`，因为底层要通过反射解析Bean中有什么方法，会将方法名（`set Xxx()`）与数据库中的字段一一对应，自动把对应值set到Bean中，并返回
>      - 注意：如果`setXxx()`==方法名与数据库字段不对应，将无法设置对应值到Bean中==

1. 方法一：自定义`RowMapper`实现映射（==书写麻烦，效率高==）

   ```java
   //查询为一个对象 - 通过自定义RowMapper实现映射
   @Test
   public void test07(){
       User user = jdbcTemplate.queryForObject(
           "select * from user where id = ?", 
           new RowMapper<User>{
               @Override
               public User mapRow(ResultSet rs, int i) throws SQLException {
                   User user = new User();
                   user.setId(rs.getInt("id"));
                   user.setName(rs.getString("name"));
                   user.setAge(rs.getInt("age"));
                   return user;
               }
           }, 
           3);
   
       System.out.println(user);
   }
   //结果：User{id=3, name='王五', age=21}
   ```

2. 方法二：通过`BeanPropertyRowMapper`自动实现映射（==书写简单，效率低==）

   ```java
   //查询为一个对象 - 通过BeanPropertyRowMapper自动实现映射
   @Test
   public void test9(){
       User user = jdbcTemplate.queryForObject(
           "select * from user where id = ?",
           new BeanPropertyRowMapper<User>(User.class),
           3);
       System.out.println(user);
   }
   ////结果：User{id=3, name='王五', age=21}
   ```

   

#### 查询一个Object集合：`query`

> 特点：
>
> - 可以==查询多条信息==
> - 把每个Bean存放在`List`中并返回

1. 方法一：自定义`RowMapper`实现映射（==书写麻烦，效率高==）

   ```java
   // 查询为一个对象集合 - 通过自定义RowMapper实现映射
   @Test
   public void test08(){
       List<User> list = jdbcTemplate.query(
           "select * from user where id < ?", 
           new RowMapper<User>{
               @Override
               public User mapRow(ResultSet rs, int i) throws SQLException {
                   User user = new User();
                   user.setId(rs.getInt("id"));
                   user.setName(rs.getString("name"));
                   user.setAge(rs.getInt("age"));
                   return user;
               }
           }, 
           3);
   
       System.out.println(list);
   }
   ```

   

2. 方法二：通过`BeanPropertyRowMapper`自动实现映射（==书写简单，效率低==）

   ```java
   // 查询为一个对象集合 - 通过BeanPropertyRowMapper自动实现映射
   @Test
   public void test10(){
       List<User> list = jdbcTemplate.query(
           "select * from user where id < ?",
           new BeanPropertyRowMapper<User>(User.class),
           3);
       System.out.println(list);
   }
   ```

   



------



# 声明式事务处理：xml方式

> 在AOP案例中有一个<!--事务控制-->案例，该案例中通过自定义：
>
> - 注解：`Trans`
> - 事务管理器：`TransactionManager`
>   - 通过`ThreadLocal`保证每个线程各自使用各自的`Connection`
> - 切面：`TransAspect`
>
> 来实现了Spring中对JDBC的事务管理，事实上，==SpringJDBC中提供了内置的事务处理机制，可以经过简单的配置完成事务管理，称之为声明式事务管理==

## 创建基本的MVC结构

![](img\Spring声明式事务管理MVC结构.png)

## 在Spring配置文件导入相关约束

### 导入相关包

![](img\Spring声明式事务管理的相关jar包.png)

### 导入相关约束

> 导入相关的约束：
>
> - `xmlns:tx="http://www.springframework.org/schema/tx"`
> - `xsi:schemaLocation=" http://www.springframework.org/schema/tx   http://www.springframework.org/schema/tx/spring-tx.xsd"`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd">
   
</beans>
```



## 详细Spring配置

### 基本配置

> 开启包扫描、开始注解DI、配置数据源、配置SpringJDBC模板

```xml
<!--IOC包扫描-->
<context:component-scan base-package="cn.tedu"></context:component-scan>

<!--注解方式DI-->
<context:annotation-config/>

<!--SpringJDBC-数据源-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///ssmdb"></property>
    <property name="user" value="root"></property>
    <property name="password" value="123456"></property>
</bean>

<!--SpringJDBC模板-JdbcTemplate-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>

```



### 配置事务管理器

> Spring内置的事务管理器：`DataSourceTransactionManager`
>
> - 需要配置数据源：`<property name="dataSource" ref="dataSource"></property>`

```xml
<!--事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

### 配置事务切面

> Spring 会自动提供一个切面，但是需要配置
>
> - 需要切入点表达式：`<aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>`
>   - 筛选要拦截的方法（切入点）
> - 需要配置事务通知 ：`<aop:advisor advice-ref="txAdvice" pointcut-ref="pc01"></aop:advisor>`

```xml
<!--配置事务切面-->
<aop:config>
    <aop:pointcut id="pc01" expression="execution(* cn.tedu.service..*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pc01"></aop:advisor>
</aop:config>
```

### 配置事务通知

> 需要被事务切面引用。事务通知配置具体拦截下来的那些方法需要事务还是不需要事务
>
> - 需要事务：`propagation="REQUIRED"`
> - 不需要事务：`propagation="NEVER"`
>
> <!--propagation【p rua pe gei shen】-->

```xml
<!--配置事务通知逻辑-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="registUser" propagation="REQUIRED"/>
        <tx:method name="add*" propagation="REQUIRED"/>
        <tx:method name="update*" propagation="REQUIRED"/>
        <tx:method name="query*" propagation="NEVER"/>
    </tx:attributes>
</tx:advice>
```



### 事务管理、切面、通知的关系

![](img\Spring声明式事务管理中事务管理、切面、通知的关系图.png)



## 事务管理策略

1. 异常的种类

   > java.lang.Throwable
   > 		|-Exception
   > 				|-RuntimeException
   > 				|-其他Exception
   >   	 |-Error

2. SpringJDBC事务处理策略

   > SpringJDBC内置的事务处理策略，只在底层==抛出运行时异常时才会回滚==，其他异常不会滚，需要手动处理。
   >
   > 可以在配置中指定在原有规则的基础上，哪些异常额外的回滚或不回滚



### 单独配置哪些异常回滚/不回滚

> 哪些需要回滚【`rollback-for`】：`rollback-for="java.io.IOException"`
>
> 哪些不需要回滚【`no-rollback-for`】：`no-rollback-for="java.lang.NullPointerException"`

```xml
<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="registUser" propagation="REQUIRED" rollback-for="java.io.IOException"/>
        <tx:method name="add*" propagation="REQUIRED" no-rollback-for="java.lang.NullPointerException"/>
    </tx:attributes>
</tx:advice>
```



### 配置所有异常都回滚

> 无论什么异常执行回滚操作： `rollback-for="java.lang.Throwable"`

```xml
    <!--配置事务通知逻辑-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="update*" propagation="REQUIRED" rollback-for="java.lang.Throwable"/>
        </tx:attributes>
    </tx:advice>
```



## 回滚操作注意的问题：

注意：如果在一个业务逻辑中，需要有多步不同表的操作：

- 应该在service层中完成对不同表的操作，以此保证多步操作处在同一个事务中
- 切勿将业务代码在web层中调用不同Service来实现，虽然正常情况下也可以完成功能，但一旦出问题，很可能只有部分操作被回滚，数据出问题。

出现这种问题，==不是事务管理机制的问题，而是开发者将业务逻辑错误的在web层中进行了实现==，所以切记，==web层只负责和用户交互和业务逻辑的调用，不进行任何业务逻辑的处理，任何业务逻辑都在service层中进行==。





------



# 声明式事物处理：注解方式

> 通过xml方式配置声明式事务处理太过繁琐，Spring支持使用注解的方式配置事务处理

## 开启事务注解配置

> 需要基本的JDBC配置：
>
> - 开启包扫描、开启Spring的DI注解支持
> - 配置数据源：`c3p0`
> - 配置SpringJDBC模板：`JdbcTemplate`
> - 配置Spring的事务管理器

### 基本配置

```xml
<!--IOC包扫描-->
<context:component-scan base-package="cn.tedu"></context:component-scan>

<!--注解方式DI-->
<context:annotation-config/>
```



### ==配置数据源==

```xml
<!--SpringJDBC-数据源-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///ssmdb"></property>
    <property name="user" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
```



### 配置SpringJDBC模板

```xml
<!--SpringJDBC模板-JdbcTemplate-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```



### ==配置事务管理器==

```xml
<!--事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```



### ==开启Spring事务处理注解支持==

```xml
<!--开启注解方式事务管理-->
<tx:annotation-driven/>
```



## 在方法上通过注解开始事务

> `@Transactional` 【chuan zai ke shen nou】即可以标注在接口上，也可以标注在实现类上，理论上应该表在接口上，实现面向接口编程，但实际开发中为了方便也有人写在实现类上。
>
> - `@Transactional`:开启事务，默认只对运行时异常进行回滚
> - `@Transactional(rollbackFor=java.io.IOException.class)`：指定发生IOException异常时回滚
> - `@Transactional(noRollbackFor=java.lang.NullPointerException.class)`：指定发生空指针异常时不回滚
> - `rollbackFor = java.lang.Throwable.class`：指定无论什么异常都回滚

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao = null;

    //@Transactional 
    //@Transactional(rollbackFor = java.io.IOException.class)
    //@Transactional(noRollbackFor = java.lang.NullPointerException.class)
    @Transactional(rollbackFor = java.lang.Throwable.class)
    @Override
    public void registUser(User user) throws IOException  {
        userDao.addUser(user);
        //int i = 1/0;
        throw new IOException();
    }
}
```



## 可以在类上通过注解开启事务

1. 在类上开始事务，此时类中的所有方法都会有事务

   ```java
   @Service
   @Transactional
   public class UserServiceImpl implements UserService {
   
       @Autowired
       private UserDao userDao = null;
   
       @Override
       public void registUser(User user) {
           userDao.addUser(user);
       }
   }
   ```

   

2. 类上开启事务后，可以在方法上单独配置不需要事务

   > 通过：
   >
   > - `@Transactional(propagation=Propagation.NOT_SUPPORTED)`标注此方法不需要事务
   > - `@Transactional(rollbackFor=java.io.IOException.class)`：指定发生IOException异常时回滚
   > - `@Transactional(noRollbackFor=java.lang.NullPointerException.class)`：指定发生空指针异常时不回滚
   > - ``@Transactional(rollbackFor = java.lang.Throwable.class)`：指定无论什么异常都回滚

   ```java
   @Service
   @Transactional
   public class UserServiceImpl implements UserService {
   
       @Autowired
       private UserDao userDao = null;
   
       @Override
       @Transactional(propagation=Propagation.NOT_SUPPORTED)//此方法不需要事务
       public void registUser(User user) {
           userDao.addUser(user);
       }
   }
   ```

   

