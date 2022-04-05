[TOC]



# MyBatis的结构

![](img\MyBatis结构.png)



# MyBatis入门案例

## 创建Java项目，导入相关开发包

```

```

![](img\MyBatis开发包.png)



## 创建数据库、表、bean

### 创建数据库、表

```mysql
create database mydb;
use mydb;
create table user (
	id int primary key auto_increment,
	name varchar(255),
	age int
);
insert into user values (null,'aaa',19),(null,'bbb',29),(null,'ccc',39),(null,'ddd',22),(null,'eee',33);

```

### 创建bean

```java
public class User {
	private int id;
	private String name;
	private int age;
    
	public User() {}
	public User(int id, String name, int age) {}
    
    //成员变量的get，set方法
    
	public String toString() {....}
}
```



## 编写配置文件

### 核心配置：SqlMapConfig.xml

> 配置数据源

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 配置数据源 -->
    <environments default="mysqldb">
        <environment id="mysqldb">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mydb"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!--配置映射文件-->
    <mappers>
        <mapper resource="UserMapper.xml"></mapper>
    </mappers>
</configuration>
```



### 映射文件：UserMapper.xml

> 描述bean和表sql的映射关系
>
> ==该文件需要配置到SqlMapConfig.xml中==

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.UserMapper">

    <!--基础查询-->
    <select id="select01" resultType="cn.tedu.domain.User">
        select * from user;
    </select>

    <!--有参数的查询-->
    <select id="select02" resultType="cn.tedu.domain.User">
        select * from user where id = 3;
    </select>
</mapper>
```



## 测试：

```java
// 有参数的查询
@Test
public void test02() throws IOException {
    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
    //创建sqlsession
    SqlSession sqlSession = factory.openSession();
    //调用sql，得到结果
    User user = sqlSession.selectOne("cn.tedu.UserMapper.select02");
    //处理结果
    System.out.println(user);
    //关闭资源
    sqlSession.close();
}

// 最基础的查询
@Test
public void test01() throws IOException {
    //1.创建SQL sessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
    //2.创建sqlsession
    SqlSession sqlSession = factory.openSession();
    //3.调用sql，得到结果
    List<User> list = sqlSession.selectList("cn.tedu.UserMapper.select01");
    //4.处理结果
    System.out.println(list);
    //5.关闭资源
    sqlSession.close();
}
```





------



# 值传递

## Map传递

> 可以通过Map集合传递值，在配置文件中 通过 `#{}` 或 `${}`进行获取

### 映射文件

```xml
<!--值传递 - Map传递-->
<select id="select03" resultType="cn.tedu.domain.User">
    select * from user where id = #{uid};
</select>
```



### 测试

```java
//值传递 - Map传递
@Test
public void test03() throws IOException {
    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    ////==============准备Map，添加数据==============
    Map<String, Integer> map = new HashMap<>();
    map.put("uid", 2);

    //调用sql，得到结果
    User user = sqlSession.selectOne("cn.tedu.UserMapper.select03",map);

    //处理结果
    System.out.println(user);

    //关闭资源
    sqlSession.close();
}
```



## 对象传递

> 可以通过传递对象，在配置文件中 通过 `#{}` 或 `${}`进行获取

### 映射文件

```xml
<!--值传递 - 对象传递-->
<insert id="insert01">
    insert into user values (null,#{name},#{age});
</insert>
```



### 测试

```java
//值传递 - 对象传递
@Test
public void test04() throws IOException {
    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //==============准备User对象==============
    User user = new User(1,"zzzzz",99);

    //调用sql，得到结果
    sqlSession.insert("cn.tedu.UserMapper.insert01",user);

    sqlSession.commit();


    //关闭资源
    sqlSession.close();
}
```



## 单值传递

> 如果程序中只有一个参数需要传递给sql，则不需要封装到bean或map中，可以直接传入。在sql中可以使用任意名称获取到这个参数，虽然名称可以任意，但通常仍然使用该属性的名称，以便于阅读。

### 映射文件

```xml
	<!--值传递 - 单值传递-->
	<select id="select04" resultType="cn.tedu.domain.User">
		select * from user where name = #{name};
	</select>
```



### 测试

```java
//值传递 - 单值传递
@Test
public void test05() throws IOException {
    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //调用sql，得到结果   =====================直接传递单个值===========
    User user = sqlSession.selectOne("cn.tedu.UserMapper.select04", "eee");

    System.out.println(user);

    //关闭资源
    sqlSession.close();
}
```



# `#{}` 和 `${}`的区别

> 1. **如果传递的是一个非字符串值，则两者等效**
> 2. **如果是一个==字符串值==，则**
>    - `#{}`：会将其值作为一个字符串拼接在sql上,即拼接时自动包裹引号
>    - `${}`：不会作为字符串处理，拼接在sql时不会自动包裹引号

## `#{}`

> 通常情况下，使用`#{}`

1. `insert into user values (null,#{name},55);` 
   - ➡`insert into user values (null,'fff',55);`
2. `insert into user values (null,${name},55);` 
   - ➡ `insert into user values (null,fff,55);`
   - SQL语句错误，字符串在SQL语句中要添加单引号（`''`）



### 案例：

1. 映射文件

   > 条件参数是字符串类型，需要通过`#{}`接收

   ```xml
   <select id="select05" resultType="cn.tedu.domain.User">
       select * from user where name = #{name};
   </select>
   ```

   

2. 测试

   ```java
   @Test
   public void test06() throws IOException {
       //创建SqlSessionFactory
       InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
       SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
   
       //创建sqlsession
       SqlSession sqlSession = factory.openSession();
   
       //===========通过map传递一个字符串
       Map<String, String> map = new HashMap<>();
       map.put("name","eee");
       //调用sql，得到结果
       User user = sqlSession.selectOne("cn.tedu.UserMapper.select05", map);
   
       System.out.println(user);
   
       //关闭资源
       sqlSession.close();
   }
   ```

   

## `${}`

> 如果需要引用的是一个列名，使用`${}`

1. `select * from user order by #{cname};` 
   - ➡`select * from user order by 'age';`/
   - SQL语句错误，order by 后面跟的是列名，不能加单引号
2. `select * from user order by ${cname};` 
   - ➡ `select * from user order by age;`



### 案例：

1. 映射文件

   > 条件参数是一个列名，不能加单引号，需要使用`${}`获取

   ```xml
   <select id="select06" resultType="cn.tedu.domain.User">
       select * from user order by ${c};
   </select>
   ```

   

2. 测试

   ```java
   @Test
   public void test07() throws IOException {
       //创建SqlSessionFactory
       InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
       SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
   
       //创建sqlsession
       SqlSession sqlSession = factory.openSession();
   
       //===========通过map传递一个列名===========
       Map<String, String> map = new HashMap<>();
       map.put("c","age");
       //调用sql，得到结果
       List<User> list = sqlSession.selectList("cn.tedu.UserMapper.select06", map);
   
       System.out.println(list);
   
       //关闭资源
       sqlSession.close();
   }
   ```

   