[TOC]

# 别名

> 如果在映射文件中，大量使用类名比较长，可以在`sqlMapConfig.xml`声明别名，在映射文件中可以使用别名缩短配置，注意此配置要放在最前面

## SqlMapConfig.xml

```xml
<!--别名标签-->
<typeAliases>
    <typeAlias type="cn.tedu.domain.User" alias="userAlias"></typeAlias>
</typeAliases>
```

## 映射文件

```xml
<select id="select01" resultType="userAlias">
    select * from user where id=#{id}
</select>
```



# SQL的复用

> 如果某段sql语句的片段在映射文件中重复出现，可以将其单独配置为一个引用，从而在需要时直接引用，减少配置量

## 映射文件

> SQL语句通过标签`<include refid="sqlid">`来引入

```xml
<!--SQL的复用-->
<sql id="sfu">
    select * from user
</sql>

<select id="select02" resultType="userAlias">
    <include refid="sfu"/> where age between #{min} and #{max}
</select>
```





# MyBatis的缓存机制

> ==（不用，不要随意改动）==
>
> 缓存机制可以减轻数据库的压力，原理是在第一查询时，将查询结果缓存起来，之后再查询同样的sql，不是真的去查询数据库，而是直接返回缓存中的结果。
>
> 缓存可以降低数据库的压力，但同时可能无法得到最新的结果数据。
>
> 开发中会用第三方的缓存机制：Redis

## 数据库缓存的实现

1. 通过第三方工具实现缓存：Redis内存数据库，可以实现缓存

2. 通过MyBatis提供的缓存机制来实现缓存

   - 一级缓存：

     > 缓存只==在一个事务中有效==，即同一个事务中先后执行多次同一个查询，只在第一次真正去查库，并将结果缓存，之后的查询都直接获取缓存中的中数据。
     >
     > ==如果是不同的事务，则缓存是隔离的。==

   - 二级缓存：

     > 缓存在==全局有效==，一个事务查询一个sql得到结果，会被缓存起来，之后只要缓存未被清除，则其他事务如果查询同一个sql，得到的将会是之前缓存的结果。
     >
     > ==二级缓存作用范围大，作用时间长，可能造成的危害也更大==，所以在开发中一般==很少启用Mybatis的二级缓存。==



## MyBatis的一级缓存

> MyBatis的一级缓存默认是开启状态，且无法手动关闭

```java
// 一级缓存 默认关闭,需要手工开启
@Test
public void test02() throws IOException {

    User user = sqlSession.selectOne("UserMapper.select01", 15);
    System.out.println(user);

    User user2 = sqlSession.selectOne("UserMapper.select01", 15);
    System.out.println(user2);

    sqlSession.commit();
}
```



> 结果 — 图解

![](img\MyBatis一级缓存.png)



## MyBatis的二级缓存

> MyBatis的二级缓存默认是关闭的

![](img\MyBatis二级缓存.png)

> 不开启二级缓存，不同的事务要单独查询SQL语句，开启二级缓存后，只有第一个SQL在库中查询，其他的直接在缓存中拿取数据

![](img\MyBatis二级缓存查询结果.png)





### 配置

#### SqlMapConfig.xml配置

```xml
<!--配置二级缓存-->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

#### 映射文件

```xml
<!--二级缓存开关-->
<cache/>
```

#### JavaBean类

> 想要被二级缓存缓存的bean必须实现序列化接口

```java
public class User implements Serializable {}
```

