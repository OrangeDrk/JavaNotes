[TOC]



# 一对一查询

![](img\MyBatis一对一数据库视图.png)

## 创建表

```mysql
create table room(id int primary key,name varchar(255));
	insert into room values (1,'梅花屋'),(2,'兰花屋'),(3,'桃花屋');
create table grade(id int primary key,name varchar(255), rid int);
	insert into grade values (999,'向日葵班',2),(888,'玫瑰花班',3),(777,'菊花班',1);
```



## 创建JavaBean

> Grade和Room，与数据库表对应

### Grade

```java
public class Grade {
    private int id;
    private String name;
    private Room room;

    public String toString() {}
    public Grade() {}
    
    //属性的get，set方法-----此处省略

}
```

### Room

```java
public class Room {
    private int id;
    private String name;
    private Grade grade;
    
    public String toString() {}
    public Room(int id, String name, Grade grade) {...}
    public Room() {}
    
    //属性的get，set方法-----此处省略
}
```



## 配置映射文件

> 在通过MyBatis实现一对一的查询时，需要通过`resultMap`指定如何将结果集中的列名对应到目标`bean`中，在一对一的`bean`中，如果包含了另一个表的对应对象，则可以在`resultMap`中通过`association`标签来声明映射方式
>
> 技巧：SQL语句可以先通过SQL运行，查看所有的运行结果，通过select筛选出想要的内容，有重名的起别名，然后在手动映射时方便书写
>
> - SQL语句运行结果
>
>   ![](img\MyBatis一对一SQL语句查询结果.png)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="o2oMapper">
    <!--手动映射结果集，以Grade角度输出-->
    <resultMap id="rm01" type="cn.tedu.domain.Grade">
        <id column="gid" property="id"></id>
        <result column="gname" property="name"></result>
        <association property="room" javaType="cn.tedu.domain.Room">
            <id property="id" column="rid"></id>
            <result property="name" column="rname"></result>
        </association>
    </resultMap>

    <select id="o2o01" resultMap="rm01">
        select
            grade.id as gid,
            grade.name as gname,
            room.id as rid,
            room.name as rname
        from
            grade inner join room
        on room.id = grade.rid;
    </select>
</mapper>
```



## 测试

```java
// 手动映射结果集
@Test
public void test01(){
    //创建SqlSessionFactory
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //创建sqlsession
    SqlSession sqlSession = factory.openSession();

    //============主要代码============
    List<Grade> list = sqlSession.selectList("o2oMapper.o2o01");
    System.out.println(list);

    //关闭资源
    sqlSession.close();
}

/*
[
    Grade{id=777, name='菊花班', room=Room{id=1, name='梅花屋', grade=null}},
    Grade{id=999, name='向日葵班', room=Room{id=2, name='兰花屋', grade=null}},
    Grade{id=888, name='玫瑰花班', room=Room{id=3, name='桃花屋', grade=null}}
]
*/
```







------



# 一对多查询

![](img\MyBatis一对多数据库视图.png)

## 创建表

```mysql
create table dept(id int primary key,name varchar(255));
	insert into dept values (1,'财务部'),(2,'行政部'),(3,'人事部'),(4,'销售部');
create table emp(id int primary key,name varchar(255), deptid int);
	insert into emp values (999,'孙悟空',4),(888,'萨达姆',3),(777,'哈利波特',1),(666,'特朗普',2),(555,'鑫三胖',3);
```



## 创建JavaBean

> Dept和Emp，和数据库表对应

### Dept

```java
public class Dept {
    private int id;
    private String name;
    private List<Emp> emp;

    public String toString() {}
    public Dept(int id, String name, List<Emp> emp) {}
    public Dept() {}
    
    //属性的get，set方法-----此处省略
}

```



### Emp

```java
public class Emp {
    private int id;
    private String name;
    private Dept dept;

    public String toString() {}
    public Emp(int id, String name, int deptid, Dept dept) { }
    public Emp() {}
    //属性的get，set方法-----此处省略
}
```



## 方式一：以员工为角度

> 在每个员工中保存其所在的部门
>
> SQL语句运行结果：
>
> ![](img\MyBatis一对多SQL语句查询结果.png)

### 配置映射文件

> 在通过MyBatis实现一对多的查询时，通过`resultMap`指定如何将结果集中的列名对应到目标`bean`中，在一对多的`bean`中，如果包含了另一个表的对应对象的集合，则可以在`resultMap`中通过`collection`标签来声明映射方式：

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



### 测试

```java
//以员工为角度
@Test
public void test01() throws IOException {
    //获取sqlsessionFactory工厂
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //获取SqlSession
    SqlSession sqlSession = factory.openSession();

    //调用SQL语句，处理结果
    List<Emp> list = sqlSession.selectList("o2mMapper.o2m01");
    System.out.println(list);
}
/*	结果
[
	Emp{id=777, name='哈利波特', deptid=0, dept=Dept{id=1, name='财务部', emp=null}}, 
	Emp{id=666, name='特朗普', deptid=0, dept=Dept{id=2, name='行政部', emp=null}}, 
	Emp{id=555, name='鑫三胖', deptid=0, dept=Dept{id=3, name='人事部', emp=null}}, 
	Emp{id=888, name='萨达姆', deptid=0, dept=Dept{id=3, name='人事部', emp=null}}, 
	Emp{id=999, name='孙悟空', deptid=0, dept=Dept{id=4, name='销售部', emp=null}}]

*/
```



## 方式二：以部门为角度

> 以部门为角度，在部门bean中保存emp的集合
>
> SQL语句运行结果
>
> ![](img\MyBatis一对多SQL语句查询结果.png)

### 配置映射文件

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



### 测试

```java
// 以部门为角度
@Test
public void test02() throws IOException {
    //获取SqlSessionFactory工厂
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //获取SqlSession对象
    SqlSession sqlSession = factory.openSession();
    
    //调用SQL语句，处理结果
    List<Dept> list = sqlSession.selectList("o2mMapper.o2m02");
    System.out.println(list);
}
/*	结果
[
	Dept{id=1, name='财务部', emp=[Emp{id=777, name='哈利波特', deptid=0, dept=null}]}, 
	Dept{id=2, name='行政部', emp=[Emp{id=666, name='特朗普', deptid=0, dept=null}]}, 
	Dept{id=3, name='人事部', emp=[Emp{id=555, name='鑫三胖', deptid=0, dept=null}, 
								  Emp{id=888, name='萨达姆', deptid=0, dept=null}
								 ]}, 
	Dept{id=4, name='销售部', emp=[Emp{id=999, name='孙悟空', deptid=0, dept=null}]}
]
*/
```





# 多对多查询

![](img\MyBatis多对多数据库视图.png)

## 创建表

```mysql
create table stu(id int primary key,name varchar(255));
	insert into stu values (1,'小新'),(2,'小白'),(3,'美伢'),(4,'风间');
create table teacher(id int primary key,name varchar(255));
	insert into teacher values (999,'孙悟空'),(888,'猪八戒'),(777,'萨达姆'),(666,'哈利波特');
create table stu_teacher (sid int,tid int);
	insert into stu_teacher values (1,999),(1,888),(2,999),(2,777),(3,666),(4,888),(4,666);
```



## 创建JavaBean

> 在通过MyBatis实现多对多的查询时，需要通过`resultMap`指定如何将结果集中的列名对应到目标`bean`中，在多对多的`bean`中，如果包含了另一个表的对应对象的集合，则可以在`resultMap`中通过`collection`标签来声明映射方式

### Student

```java
public class Student {
    private int id;
    private String name;
    private List<Teacher> teachers;//学生被哪些老师教过

    public String toString() {}
    public Student(int id, String name, List<Teacher> teachers) {.....}
    public Student() {}
    
    //属性的get，set方法-----此处省略
}
```



### Teacher

```java
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;//老师教过哪些学生

    public String toString() {}
    public Teacher(int id, String name, List<Student> students) {}
    public Teacher() {}
    
    //属性的get，set方法-----此处省略
}
```



## 方式一：以Teacher为角度

> 以Teacher为角度，在教师bean中保存Student的集合
>
> 在Teacher bean中存储Stu bean的List
>
> SQL语句运行结果
>
> ![](img\MyBatis多对多SQL语句查询结果.png)

### 配置映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="m2mMapper">

    <!--方式一：以Teacher为角度-->
    <resultMap id="rm01" type="cn.tedu.domain.Teacher">
        <id property="id" column="tid"></id>
        <result property="name" column="tname"></result>
        <collection property="students" ofType="cn.tedu.domain.Student">
            <id property="id" column="sid"></id>
            <result property="name" column="sname"></result>
        </collection>
    </resultMap>

    <!--配置SQL映射-->
    <select id="m2m01" resultMap="rm01">
        select
            stu.id as sid,
            stu.name as sname,
            teacher.id as tid,
            teacher.name as tname
        from stu left join stu_teacher on stu.id = stu_teacher.sid
        left join teacher on stu_teacher.tid = teacher.id
    </select>
</mapper>
```



### 测试

```java
@Test
public void test01() throws IOException {
    //获取SqlSessionFactory工厂
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    SqlSession sqlSession = factory.openSession();

    List<Teacher> list = sqlSession.selectList("m2mMapper.m2m01");
    System.out.println(list);
}
/*结果
[
	Teacher{id=999, name='孙悟空', 
            students=[Student{id=1, name='小新', teachers=null}, Student{id=2, name='小白', teachers=null}]}, 
	Teacher{id=888, name='猪八戒', 
            students=[Student{id=1, name='小新', teachers=null}, Student{id=4, name='风间', teachers=null}]}, 
	Teacher{id=777, name='萨达姆', 
			students=[Student{id=2, name='小白', teachers=null}]}, 
	Teacher{id=666, name='哈利波特', 
            students=[Student{id=3, name='美伢', teachers=null}, Student{id=4, name='风间', teachers=null}]}
]
*/
```



## 方式二：以Student为角度

> 以Student为角度，在学生bean中保存Teacher的集合
>
> 在Stu bean中存储Teacher bean的List
>
> SQL语句运行结果
>
> ![](img\MyBatis多对多SQL语句查询结果.png)

### 配置映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="m2mMapper">

    <!--方式二：以Student为角度-->
    <resultMap id="rm02" type="cn.tedu.domain.Student">
        <id property="id" column="sid"></id>
        <result property="name" column="sname"></result>
        <collection property="teachers" ofType="cn.tedu.domain.Teacher">
            <id property="id" column="tid"></id>
            <result property="name" column="tname"></result>
        </collection>
    </resultMap>
    
    <!--配置SQL映射-->
    <select id="m2m02" resultMap="rm02">
        select
            stu.id as sid,
            stu.name as sname,
            teacher.id as tid,
            teacher.name as tname
        from stu left join stu_teacher on stu.id = stu_teacher.sid
        left join teacher on stu_teacher.tid = teacher.id
    </select>
</mapper>
```



### 测试

```java
//以Student为角度
@Test
public void test02() throws IOException {
    //获取SqlSessionFactory工厂
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

    //获取SqlSession对象
    SqlSession sqlSession = factory.openSession();

    //执行SQL语句，处理结果
    List<Student> list = sqlSession.selectList("m2mMapper.m2m02");
    System.out.println(list);
}
/*结果：
[
	Student{id=1, name='小新', 
			teachers=[Teacher{id=999, name='孙悟空', students=null}, Teacher{id=888, name='猪八戒', students=null}]}, 
	Student{id=2, name='小白', 
			teachers=[Teacher{id=999, name='孙悟空', students=null}, Teacher{id=777, name='萨达姆', students=null}]}, 
	Student{id=3, name='美伢', 
			teachers=[Teacher{id=666, name='哈利波特', students=null}]}, 
	Student{id=4, name='风间', 
			teachers=[Teacher{id=888, name='猪八戒', students=null}, Teacher{id=666, name='哈利波特', students=null}]}]
*/
```







------



# 技巧

1. 多表查询时，不管有多少表，可以从第一个开始，一直向后`join`，select内容先写`*`，查询出所有的以后，在修改select内容
2. 先书写SQL语句，通过select筛选出需要的内容，如果有重名列，通过别名区别开，再去书写映射文件，这样手动映射结果集会方便书写