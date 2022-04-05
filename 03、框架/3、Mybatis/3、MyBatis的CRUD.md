[TOC]



# CRUD

> MyBatis的CRUD对于插入、删除需要手动提交：`sqlSession.commit();`

## update修改

> update修改也可以使用之前的机制在配置文件中直接编写sql
>
> 但是update语句 的 set字句中，拼接哪些字段是根据传入的值决定，此时可以通过MyBatis提供的标签，实现判断、动态拼接update语句

### 映射文件

> 通过`<set>`标签内的`<if>`标签，实现动态的拼接SQL语句。
>
> ==MyBatis会自动将SQL语句中多出来的`,`删掉==

```xml
<update id="update01">
    update user
    <set>
        <if test="name!=null">name=#{name},</if>
        <if test="age!=0">name=#{age},</if>
    </set>
    where id=#{id};
</update>
```



### 测试

```java
// update
@Test
public void test01() throws IOException {

    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //===========主要代码===================
    User user = new User(6, "zzz", 80);
    //调用sql，得到结果
    sqlSession.update("cn.tedu.UserMapper.update01", user);
	//================⬆⬆⬆⬆⬆================
    
    //关闭资源
    sqlSession.close();
}
```



## select查询

> select查询也可以使用之前的机制在配置文件中直接编写sql
>
> 但是select语句 的 where字句中 拼接哪些查询字段 是根据传入的值决定，此时可以通过MyBatis提供的标签 实现判断、动态拼接select语句

### 映射文件

> 通过`<where>`标签内的`<if>`标签，实现动态的拼接SQL语句。
>
> MyBatis会自动将SQL语句中多出来的`and`删掉

```xml
<select id="select01" resultType="cn.tedu.domain.User">
    select * from user
    <where>
        <if test="id!=0">id = #{id}</if>
        <if test="name!=null">and name = #{name}</if>
        <if test="age!=0">and age = #{age}</if>
    </where>

</select>
```



### 测试

> 由于映射文件通过标签书写，可以动态拼接SQL语句，实现了传递不同的对象可以使用同一段SQL配置

```java
//select
@Test
public void test01() throws IOException {

    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //===========主要代码===================
    //User user = new User(2, "bbb", 29);
    //User user = new User(0, "bbb", 0);
    User user = new User(2, "bbb", 0);
    //调用sql，得到结果
    List<User> list = sqlSession.selectList("cn.tedu.UserMapper.select01", user);
    System.out.println(list);
    //================⬆⬆⬆⬆⬆================

    //关闭资源
    sqlSession.close();
}
```



## insert插入

> insert插入也可以使用之前的机制在配置文件中直接编写sql
>
> 但是insert语句 的参数和值的列表 拼接哪些字段 是根据传入的值决定，此时可以通过MyBatis提供的标签 实现判断、动态拼接insert语句

### 映射文件

> 通过`<trim>`标签内的`<if>`标签，实现动态的拼接SQL语句。
>
> - `<trim>`属性：
>   - `prefix`：前缀
>   - `suffix`：后缀
>   - `suffixOverrides`：分隔符
>
> MyBatis会自动将SQL语句中多出来的`and`删掉

```xml
<insert id="insert01">
    insert into user
    <trim prefix="(" suffix=")" suffixOverrides=",">
        id,
        <if test="name != null">name,</if>
        <if test="age != 0">age,</if>
    </trim>
    values
    <trim prefix="(" suffix=")" suffixOverrides=",">
        null ,
        <if test="name!=null">#{name},</if>
        <if test="age!=0">#{age},</if>
    </trim>
</insert>
```



### 测试

> 需要手动递交

```java
// update
@Test
public void test01() throws IOException {

    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //===========主要代码===================
    User user = new User(0, "qqq", 88);//此处传入的值可以是任意的，可变的
    sqlSession.insert("cn.tedu.UserMapper.insert01", user);
    sqlSession.commit();
    //================⬆⬆⬆⬆⬆================

    //关闭资源
    sqlSession.close();
}
```



## delete删除

> delete删除也可以使用之前的机制在配置文件中直接编写sql
>
> 但是delete语句 的删除条件 拼接哪些字段 是根据传入的值决定，此时可以通过MyBatis提供的标签 实现判断 动态拼接delete语句：

### 映射文件

```xml
<delete id="delete01">
    delete from user
    <where>
        <if test="id!=0">id=#{id}</if>
        <if test="name!=null">and name=#{name}</if>
        <if test="age!=0">and age=#{age}</if>
    </where>
</delete>


<delete id="delete02">
    delete from user where id in
    <foreach collection="list" open="(" close=")" separator="," item="id">
        #{id}
    </foreach>
</delete>
```



### 测试

```java
public class Test01 {
    SqlSessionFactory factory = null;
    SqlSession sqlSession = null;

    @Before
    public void befor() {
        //创建SqlSessionFactory
        try {
            InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
            factory = new SqlSessionFactoryBuilder().build(in);

            //创建sqlsession
            sqlSession = factory.openSession();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @After
    public void after() {
        //关闭资源
        sqlSession.close();
    }



    //delete: 配置文件：foreach
    @Test
    public void test04_2(){
        List<Integer> list = new ArrayList<>();
        list.add(5);
        list.add(6);
        sqlSession.delete("cn.tedu.UserMapper.delete02", list);
        sqlSession.commit();
    }
     
    //delete: #{name}......
    @Test
    public void test04() throws IOException {
        User user = new User(6, "zzz", 99);
        sqlSession.delete("cn.tedu.UserMapper.delete01", user);
        sqlSession.commit();
    }
}
```





------



# 手动映射结果集

> MyBatis可以自动将查询结果封装到bean中，前提条件是bean的属性名和查询的结果列名相同， 就会依次对应存储。
>
> 如果查询结果的列名和bean的属性名不一致，则需要手动映射结果集

## 映射文件

> 🔑主键：通过`<id></id>`配置
>
> - 属性：`column`：库的列名
> - 属性：`property`：JavaBean属性名
>
> 📑其他属性：` <result></result>`配置
>
> - 属性：`column`：库的列名
> - 属性：`property`：JavaBean属性名
>
> 📑复杂类型：`<association></association>`配置
>
> - `<association>`属性
>   - `property`：JavaBean中的属性名
>   - `javaType`：JavaBean中属性的类型
>
> - 子标签：`<id>`,`<result>`标签，标签属性和上述相同
>
> 📑集合类型：`<collection>`配置
>
> - `<collection>`属性
>   - `property`：JavaBean中的属性名
>   - `ofType`：JavaBean中属性的泛型
> - 子标签：`<id>`,`<result>`标签，标签属性和上述相同

## 其他属性

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.UserMapper">
    <!--手动映射-->
    <resultMap id="rm01" type="cn.tedu.domain.User2">
        <id column="id" property="uid"></id>
        <result column="name" property="uname"></result>
        <result column="age" property="uage"></result>
    </resultMap>
    
    <select id="select01" resultMap="rm01">
        select * from user;
    </select>

</mapper>
```



### 测试

```java
public class Test01 {

    // 手动映射结果集
    @Test
    public void test01(){
        //创建SqlSessionFactory
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

        //创建sqlsession
        SqlSession sqlSession = factory.openSession();
        
        //============主要代码============
        List<User2> list = sqlSession.selectList("cn.tedu.UserMapper.select01");
        System.out.println(list);

        //关闭资源
        sqlSession.close();
    }

}

/*结果
    [
        User2{uid=12, uname='aaa', uage=19}, 
        User2{uid=13, uname='bbb', uage=29}, 
        User2{uid=14, uname='ccc', uage=39}, 
        User2{uid=15, uname='ddd', uage=22}, 
        User2{uid=16, uname='eee', uage=33}
    ]
*/

```



## 复杂类型

> 员工类：`Dept dept;`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="o2mMapper">

    <!--手动映射结果集：以员工为角度-->
    <resultMap id="rm01" type="cn.tedu.domain.Emp">
        <id property="id" column="eid"></id>
        <result property="name" column="ename"></result>
        <association property="dept" javaType="cn.tedu.domain.Dept">
            <id property="id" column="did"></id>
            <result property="name" column="dname"></result>
        </association>
    </resultMap>
    <select id="o2m01" resultMap="rm01">
        select
            emp.id as eid,
            emp.`name` as ename,
            dept.id as did,
            dept.name as dname
        from emp left join dept on emp.deptid=dept.id
    </select>
</mapper>
```



## 集合类型

> 部门类中：集合`List<Emp> emp`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="o2mMapper">

    <!--以部门为角度-->
    <resultMap id="rm02" type="cn.tedu.domain.Dept">
        <id property="id" column="did"></id>
        <result property="name" column="dname"></result>
        <collection property="emp" ofType="cn.tedu.domain.Emp">
            <id property="id" column="eid"></id>
            <result property="name" column="ename"></result>
        </collection>
    </resultMap>

    <!--配置sql映射-->
    <select id="o2m02" resultMap="rm02">
        select
            emp.id as eid,
            emp.`name` as ename,
            dept.id as did,
            dept.name as dname
        from emp left join dept on emp.deptid=dept.id
    </select>

</mapper>
```



------



# 多表设计&多表查询

## 多表设计

1. 一对一
   - 在任意一方设计外键保存另一张表的主键，维系表和表的关系
2. 一对多
   - 在多的一方设计外键保存一的一方的主键，维系表和表的关系
3. 多对多
   - 设计一张第三方关系表，存储两张表的主键的对应关系，将一个多对多拆成两个一对多来存储

### 图解：

![](img\多表设计图解.png)

## 多表查询

> 1. 笛卡儿积
> 2. 内连接查询：`inner join ... on ...`
> 3. 外连接查询
>    - 左外连接：`left join ... on....`
>    - 右外连接：`right join ... on....`
>    - 全外连接：MySQL通过`union`进行全连接

### 图解

![](img\多表查询图解.png)





