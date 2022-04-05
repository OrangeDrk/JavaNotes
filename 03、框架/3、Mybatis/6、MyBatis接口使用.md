[TOC]

# 接口的使用（==开发中经常使用==）

> 为了简化MyBatis的使用，MyBatis提供了接口方式 自动化 生成调用过程的机制，可以大大简化MyBatis的开发
>
> 开发规则：
>
> 1. 接口的全路径名应为映射文件中声明的名称空间          
> 2. 接口中应该声明和映射文件中sql对应的id相同名称的方法 
> 3. 方法接收的参数应该和sql中接收的参数一致 
> 4. 方法的返回值应该和sql中声明的返回值类型一致         
>
> 对应图解：
>
> ![](img\MyBatis接口开发图解.png)



# 实现过程

## MyBatis核心文件(sqlMapConfig.xml)

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



## 开发映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.tedu.mapper.UserMapper">

    <select id="selectOrderByAge" resultType="cn.tedu.domain.User">
        select * from user order by age;
    </select>

    <select id="selectBetweenAge" resultType="cn.tedu.domain.User">
        select * from user where age between #{min} and #{max}
    </select>

</mapper>
```



## 开发接口

> 如果SQL语句仅需要单独的几个参数，可以通过`@Param("键值")`标注这个参数传给哪个SQL占位符

```java
public interface UserMapper {

    //方法名：名称空间     返回值：resultType	参数：SQL语句所需要的参数
    public List<User> selectBetweenAge(@Param("min") int min, @Param("max") int max);

    public List<User> selectOrderByAge();
}

```



## 测试

```java
@Test
public void test02(){
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = userMapper.selectBetweenAge(15,30);

    System.out.println(users);
}
/*结果：
	[User{id=12, name='aaa', age=19},
    User{id=13, name='bbb', age=29}, 
    User{id=15, name='ddd', age=22}]
*/
@Test
public void test03(){
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = userMapper.selectOrderByAge();

    System.out.println(users);
}
/*结果：
	[User{id=12, name='aaa', age=19}, 
	User{id=15, name='ddd', age=22}, 
	User{id=13, name='bbb', age=29}, 
	User{id=16, name='eee', age=33}, 
	User{id=14, name='ccc', age=39}]
*/
```





# 实现原理

![](img\MyBatis接口开发实现原理.png)




