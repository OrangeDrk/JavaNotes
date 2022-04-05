[TOC]

# JDBC

## 基本概念

- jdbc：(Java data base connectivity)  Java数据库链接技术
- jdbc这个技术是由Java提供好的jar包，程序员需要学习的是：
  - jar包的API（常用的类和接口）
  - 调用jar包中的方法，操作数据库
- jdbc是由2个主要的jar包组成
  - rt.jar/java/java.sql包
  - rt.jar/java/javax.sql包

## 为什么学习jdbc

- 因为jdbc是web服务器端的技术，是一段Java程序
- 该程序支持用户的数据访问以及用户的数据存储
- 在整个web流程中，jdbc是不可或缺的一个环节（入库和出库的环节）

## jdbc实现原理：主要作用

1. 可视化工具能操作的步骤以及sql，jdbc同样可以。支持数据库的增删改查

2. jdbc专门负责数据库的访问与链接。需要==基本的配置信息==：

   - 驱动包
   - 登陆名
   - 密码
   - url路径
   - 端口号
   - ip地址

3. jdbc本身是jar包，jar包中存在了大量的接口

   > jdbc是一种支持数据库的规范，规定了连接数据库的常用接口

   不同的数据库厂商，他们的sql需要不同的驱动包，所以，只要使用Java操作某一个数据库，该数据库厂商就必须要提供一个套程序，实现jdbc接口（对应数据库厂商的==驱动包==）

4. jdbc技术是目前web开发的常用技术，即使在学习后期：

   - dao层框架底层还是jdbc



------



# JDBC执行过程中出现的问题

## 登陆访问被拒绝

【java.sql.SQLException: Access denied for user 'root'@'localhost' (using password: YES)】

> 解决方案：
>
> - 需要找到配置信息 user变量/password变量把信息修改正确



## 数据库不存在

【com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException:Unknown database 'javawep02'】

> 解决方案：
>
> - 找到url路径，查看url是否书写正确

## 尝试连接失败

【com.mysql.jdbc.exceptions.jdbc4.CommunicationsException:Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago.The driver has not received any packets from the server.】

> 解决方案：
>
> - 找到url路径，检查端口号

## SQL语法不正确

【com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'false teacher' at line 1】

> 解决方案：
>
> - 找到SQL语句，检查修改

## 获取结果集报错

【java.sql.SQLException: Column 'name' not found.】

> 解决方案：
>
> - 找到while循环体，检查获取结果及的参数是否和sql列名一致

## 资源提前关闭

【java.sql.SQLException: Operation not allowed after ResultSet closed】

> 解决方案：
>
> - 检查关闭资源的代码位置是否正确





------



# 入门-基本操作

> 最基本的jdbc操作数据库
>
> - 注册驱动程序
>   - mysql:`new Driver();`
>   - oracle:`new OracleDriver();`
> - 获取数据库连接：配置基本信息
> - 创建SQL语句对象
> - 通过SQL对象，编写SQL语句，并接受结果集
> - 处理返回的结果集
> - 关闭连接

1. 注册驱动程序

   > DriverManager可以支持多个不同的数据库驱动！！！

   ```java
   DriverManager.registerDriver(new Driver());
   ```

2. 获取数据库连接：配置基本信息

   ```java
   String user = "root";
   String password = "123456";
   //url:使用的是jdbc的协议去连接的，连接到某一个database
   String url = "jdbc:mysql://localhost:3306/javaweb02?useSSL=false";
   Connection conn = DriverManager.getConnection(url,user,password);
   ```

3. 创建SQL语句对象

   ```java
   Statement statement = conn.createStatement();
   ```

4. 通过SQL语句对象编写SQL语句，并接收结果集

   ```java
   ResultSet resultSet = 
       statement.executeQuery("select * from teacher");
   ```

5. 处理结果集

   > `resultSet.next()`:类似迭代器，判断是否还有下一个数据
   >
   > `result.getInt(列号/列名);`
   >
   > `result.getString(列号/列名);`
   >
   > `result.getObject(列号/列名);`
   >
   > ………………

   ```java
   while (resultSet.next()) {
       int id = resultSet.getInt(1);//通过列号获取
       String name = resultSet.getString("t_name");//通过列名获取
       int age = (int) resultSet.getObject("t_age");
       System.out.println(id+"\t\t"+name+"\t"+age);
   }
   ```

6. 关闭连接

   > 从最下面往上面，倒序关闭连接

   ```java
   resultSet.close();
   statement.close();
   conn.close();
   ```

## 完整代码

```java
public class Demo01_jdbc入门 {
    public static void main(String[] args) throws Exception{
        try {
            //DriverManager可以支持多个不同的数据库驱动！！！
            DriverManager.registerDriver(new Driver());

            //2.获取数据库连接：配置基本信息
            String user = "root";
            String password = "123456";
            String url = "jdbc:mysql://localhost:3306/javaweb02?useSSL=false";
            Connection conn = DriverManager.getConnection(url,user,password);

            //3.编写sql语句，并且先创建sql语句的对象
            //创建一个语句对象statement:执行sql语句
            Statement statement = conn.createStatement();

            //4.使用statement对象去执行sql语句
            ResultSet resultSet = statement.executeQuery("select * from teacher");
            while (resultSet.next()) {
                int id = resultSet.getInt(1);//通过列号获取
                String name = resultSet.getString("t_name");//通过列名获取
                int age = (int) resultSet.getObject("t_age");
                System.out.println(id+"\t"+name+"\t"+age);
            }
            resultSet.close();
            statement.close();
            conn.close();
        }
    }
}

```





------



# JDBC封装：JDBCUtil

> 为了代码的可重用性设计的
>
> 工具类是提供公共的方法。一般来说，是静态方法
>
> - 因为，类在加载时会直接加载静态的方法，进入到方法区。不需要根据对象的创建去调用，直接根据`类名.方法名();`

- 把重复的代码提取成一个公用的方法：
  - `getConnection()`方法：返回数据库连接的对象
  - `close()`方法：关闭连接
- 提取公共资源：用户名，密码，url信息、数据库驱动
  - 把这些信息写入到配置文件中即可，以后数据库信息发生变动只需要修改配置文件，而不需要修改代码
  
  - 在`src/jdbc-config.properties`，内容以键值对存在
  
  - 提取出一个信息：驱动的类型com.mysql.jdbc.Driver
  
    - 配置文件中维护：driver=com.mysql.jdbc.Driver
  
    ```properties
    user=root
    password=123456
    url=jdbc:mysql://localhost:3306/javaweb02?useSSL=false
    driver=com.mysql.jdbc.Driver
    ```



## getConnection()

> 通过类加载器获取配置文件
>
> `pros.load(JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc-config.properties"));`

```java
static {
    try {
        //类加载器：是可以通过其他的类获取得到
        //pros.load()会把输入流加载到pros对象中，就拿到了文本中的key与value
        pros.load(JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc-config.properties"));
    } catch (IOException e) {
        e.printStackTrace();
    }
}


/*1.获取连接的方法:返回值是Connection*/
public static Connection getConnection() {
    Connection connection = null;
    try {
        //1.1   注册驱动程序
        Class.forName(pros.getProperty("driver"));
        //2.2   连接数据库
        String url = pros.getProperty("url");
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        connection = DriverManager.getConnection(url,user,password);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return connection;
}
```

## close()

> 为了防止关闭资源是出现异常，在catch中将资源赋值为null
>
> close()方法直接有效，比置为null快速

```java
/*2.关闭连接的方法：参数是：结果集，语句对象，连接对象*/
public static void close(ResultSet resultSet, Statement statement,Connection connection){
    if (resultSet!=null) {
        try {
            resultSet.close();/*close()方法直接有效，比null快速*/
        } catch (SQLException e) {
            resultSet = null;
            /*如果close时出异常，但必须要关掉资源，所以置为null，被jvm虚拟机回收,回收时间不确定*/
            e.printStackTrace();
        }
    }
    if (statement != null) {
        try {
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
            statement = null;
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
            connection = null;
        }
    }

}
```

## 获取配置文件信息

> ==通过反射获取==：
>
> `pros.load(JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc-config.properties"));`

```java
//定义一个Properties对象，负责解析.properties文本
private static Properties pros = new Properties();
//解析的内容通过静态代码块去执行。因为静态代码块，在JDBCUtil这个类加载时就会执行，把数据写入到了内存中
static {
    try {
        //类加载器：Java的jdk内置的
        //类的加载过程：比如JDBCUtil这个类中的静态方法被加载到内存中，这个过程就是类加载器处理完成的
        //类加载器：是可以通过其他的类获取得到

        //pros.load()会把输入流加载到pros对象中，就拿到了文本中的key与value
        pros.load(JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc-config.properties"));
        System.out.println(pros);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```



------



# 数据库连接池

连接池技术：

- 主要目的是提供多个connection连接，减少内存不断开关的消耗
- 这些连接connection是一直在池中存在，用的时候直接拿到

常用的数据库连接池

- DBCP
- C3P0（常用）

图解：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110814.jpg)

## DBCP

> DBCP技术，在早期的场景中会经常用到，比如Tomcat服务器中
>
> 需要用到的jar包：commons-dbcp-xx.jar + commons-pool-xx.jar
>
> 封装一个DBCP类
>
> - 静态的BasicDataSource对象bds，默认为null
> - `init()`方法：初始化BasicDataSource对象
> - `getConnection()`方法：==获取连接对象bds.getConnection();==
> - `close()`方法：关闭连接
>
> 书写dbcp-config.properties配置文件
>
> - mysql的基本信息和DBCP数据库连接池的信息

- `DBCPUtil`类

  ```java
  public class DBCPUtil {
      private static BasicDataSource bds = null;
      //私有化的构造方法，不能new创建对象
      private DBCPUtil(){}
  
      //初始化连接池，把连接池的基本信息获得
      public static void init(){
          //1.定义一个properties对象
          Properties pros = new Properties();
          //2.通过流加载dbcp配置文件
          try {
              pros.load(DBCPUtil.class.getClassLoader().getResourceAsStream("dbcp-config.properties"));
          } catch (IOException e) {
              e.printStackTrace();
          }
          //3.通过pros对象获取参数：配置信息
          try {
              String driver = pros.getProperty("jdbc.driver");
              String url = pros.getProperty("jdbc.url");
              String user = pros.getProperty("jdbc.user");
              String password = pros.getProperty("jdbc.password");
              String initSizes = pros.getProperty("jdbc.dataSource.initialSize");
              String maxIdle = pros.getProperty("jdbc.dataSource.maxIdle");
              String minIdle = pros.getProperty("jdbc.dataSource.minIdle");
              String maxActive = pros.getProperty("jdbc.dataSource.maxActive");
              String maxWait = pros.getProperty("jdbc.dataSource.maxWait");
  
              //4.创建一个连接池对象
              bds = new BasicDataSource();
              //5.把配置信息添加在连接池对象中
              bds.setDriverClassName(driver);
              bds.setUrl(url);
              bds.setUsername(user);
              bds.setPassword(password);
              //如果没有配置连接池的信息，给默认值
              if (initSizes == null || "".equals(initSizes)) {
                  bds.setInitialSize(40);
              }else {
                  bds.setInitialSize(Integer.parseInt(initSizes));
              }
              if (maxIdle == null || "".equals(maxIdle)) {
                  bds.setMaxIdle(60);
              }else {
                  bds.setMaxIdle(Integer.parseInt(maxIdle));
              }
              if (minIdle == null || "".equals(minIdle)) {
                  bds.setMinIdle(20);
              }else {
                  bds.setMinIdle(Integer.parseInt(minIdle));
              }
  
              if (maxActive == null || "".equals(maxActive)) {
                  bds.setMaxActive(50);
              }else {
                  bds.setMaxActive(Integer.parseInt(maxActive));
              }
              if (maxWait == null || "".equals(maxWait)) {
                  bds.setMaxWait(1000);
              }else {
                  bds.setMaxWait(Integer.parseInt(maxWait));
              }
  
          } catch (Exception e) {
              System.out.println("初始化连接池失败");
          }
  
      }
  
      public static Connection getConnection(){
          if (bds == null) {//如果为null，需要先初始化连接池
              init();//调用本类的静态方法，进行初始化
          }
  
          Connection connection = null;
          try {
              connection =  bds.getConnection();
          } catch (SQLException e) {
              e.printStackTrace();
          }
          return connection;
      }
      public static void close(...){...}
  
  
      public static void main(String[] args) {
          Connection connection = getConnection();
          System.out.println(connection);
  
      }
  
  }
  
  ```

- `dbcp-config.properties`配置文件

  ```properties
  #1.数据库连接的基本信息
  jdbc.driver=com.mysql.jdbc.Driver
  jdbc.url=jdbc:mysql://localhost:3306/javaweb02?useSSL=false
  jdbc.user=root
  jdbc.password=123456
  
  #2.数据库连接池的信息
  #初始化连接
  dataSource.initialSize=10
  #最大空闲连接
  dataSource.maxIdle=20
  #最小空闲连接
  dataSource.minIdle=5
  #最大连接数量
  dataSource.maxActive=50
  #超时等待时间以毫秒为单位 ???6000毫秒/1000等于60秒
  dataSource.maxWait=1000
  
  ```

  

## C3P0

> 优点：
>
> - C3p0的性能比DBCP更好
> - C3P0的内置方法，会自动加载.xml配置文件，不需要手动（写代码）加载
> - C3P0使用起来更加方便
>
> 封装`C3P0Util`类
>
> - 创建静态的ComboPooledDataSource对象，==该对象会自动加载.xml文件==
> - `getConnection()`方法：获取连接对象
> - `close()`方法：关闭连接
>
> 配置`c3p0-config.xml`文件

- `C3P0Util`类

  ```java
  public class C3P0Util {
      /*1.定义一个成员变量*/
      //当进行new的对象创建时，会自动找到src/c3p0-config.xml文件。不需要进行配置信息初始化
      private static ComboPooledDataSource cpd = new ComboPooledDataSource();
  
      /*2.定义一个connection连接的方法*/
      public static Connection getConnection() {
          Connection conn = null;
          try {
              conn = cpd.getConnection();
          } catch (SQLException e) {
              e.printStackTrace();
          }
          return conn;
      }
  
      /*3.定义一个close()方法，此方法是把连接释放回连接池*/
      public static void close(Connection conn, Statement state, ResultSet res){
  
          if(res!=null){
              try {
                  res.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          if(state!=null){
              try {
                  state.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          if(conn!=null){
              try {
                  conn.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  
      public static void main(String[] args) {
          Connection connection = getConnection();
          System.out.println(connection);
      } 
  }
  
  ```

- c3p0-config.xml

  > 标签的name值，不能变，规定这样写
  
  ```xml
  <c3p0-config>
      <!--default默认指的是：c3p0会直接加载该标签中的内容-->
      <default-config>
          <property name="driverClass" >com.mysql.jdbc.Driver</property>
          <property name="jdbcUrl" >jdbc:mysql://localhost:3306/javaweb02?useSSL=false</property>
          <property name="user" >root</property>
          <property name="password" >123456</property>
      </default-config>
</c3p0-config>
  ```
  
  



# JDBC核心API

## Statement

> 提供了两个方法：
>
> - `execute();`：运行语句，返回boolean类型的值，表示是否有结果集
>
> - `executeQuery();`select语句
> - `executeUpdate();`Insert,Update,Delete
>
> ==容易出现的问题：==
>
> - SQL语句太多时，==占用内存资源较大==（会把大量的SQL语句放入数据库的缓冲区，消耗数据库性能）
>
> - SQL语句是使用 变量去拼接字符串（容易出现==SQL注入问题==）
>
> - 图解：
>
> ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110815.jpg)

- Statement使用

  ```java
  try {
      //1.获取连接：c3p0连接池
      conn = C3P0Util.getConnection();
      //2.创建一个statement的语句对象
      sta = conn.createStatement();
      //3.创建4个语句，CRUD
      String sql1 = "select * from teacher";
      String sql2 = "insert into teacher values(default,'测试用户',22)";
      String sql3 = "update teacher set t_name='测试' where t_id=1";
      String sql4 = "delete from teacher where t_id=2";
  
      //4.执行SQL语句
      resultSet = sta.executeQuery(sql1);
      int row1 = sta.executeUpdate(sql2);
      int row2 = sta.executeUpdate(sql3);
      int row3 = sta.executeUpdate(sql4);
  } catch (SQLException e) {
      e.printStackTrace();
  }
  ```

- 出现的问题：SQL注入

  > statement对有参数的SQL语句要是有字符串拼接，如下代码。会出现问题：
  >
  > - 通过字符串拼接后，在原有语句后面，增加了：or 1=1
  >
  > - 本想得到：`select * from user where name='xuning' and password=??` 
  >
  > - 拼接后，改变了原有的SQL语句：
  >
  >   `select * from user where name='xuning' and password=' ' or 1=1` 
  ```java
  String name="xuning";
  String password=" '' or 1=1 ";
  
  String sql = 
      " select * from user where name="+name+
      " and password="+password;
  
  ```
  


## preparedStatement

> 与Statement的关系：
>
> - preparedStatement implement Statement接口
>
> 特点：预编译：
>
> - 为了防止SQL注入：先规定了一个SQL语句的基本结构
>
>     `select * from user where name=? and password=?`
>
>     `?`：占位符，会替换变量
>
> 优点：
>
> - 只定义一个SQL语句，变量问题用占位符？
> - 防止SQL注入

- preparedStatement使用

  ```java
  try {
      //1.获取连接
      connection = C3P0Util.getConnection();
      ps = connection.prepareStatement("select * from teacher where t_id=?");
      ps.setInt(1, 1);
      resultSet = ps.executeQuery();
      while (resultSet.next()) {
          int id = resultSet.getInt("t_id");
          String name = resultSet.getString("t_name");
          int age = resultSet.getInt("t_age");
          System.out.println(id + "--" + name + "--" + age);
      }
  } catch (SQLException e) {
      e.printStackTrace();
  } finally {
      C3P0Util.close(connection, ps, resultSet);
  }
  ```



## ResultSet

1. 属性
   
   - `Statement stmt = conn.createStatement(type, concurrency);`
   
   - `PreparedStatement stmt = conn.prepareStatement(sql, type, concurrency);`
   
   - ==Type==取值：
   
     | 值                      | 解释                   |
     | ----------------------- | ---------------------- |
     | TYPE_FORWARD_ONLY       | 只能向前移动，默认参数 |
     | TYPE_SCROLL_INSENSITIVE | 可滚动，不感知数据变化 |
     | TYPE_SCROLL_SENSITIVE   | 可滚动，感知数据变化   |
   
   - ==concurrency==取值
   
     | 值               | 解释   |
     | ---------------- | ------ |
     | CONCUR_READ_ONLY | 只读   |
     | CONCUR_UPDATABLE | 可更新 |
2. 方法：

   - `next()`：移动到下一条（==最常用的==）

   - 不常用的：了解即可

     | 方法            | 解释                       |
     | --------------- | -------------------------- |
     | first()         | 指针移动到第一条           |
     | last()          | 指针移动到最后一条         |
     | beforeFirst()   | 指针移动到第一条之前       |
     | afterLast()     | 指针移动到最后一条之后     |
     | isFirst()       | 判断指针是否指向第一条     |
     | isLast()        | 判断指针是否指向最后一条   |
     | isBeforeFirst() | 判断指针是否在第一条之前   |
     | isAfterLast()   | 判断指针是否在最后一条之后 |
     | relative()      | 移动到当前指针的相对位置   |
     | previous()      | 移动到前一条               |
     | absolute()      | 移动到绝对位置             |

   - 实例代码

     > 实现了从后往前查询输出

     ```java
     try {
         //1.获取连接
         conn = C3P0Util.getConnection();
         ps = conn.prepareStatement("select * from teacher",
                                    ResultSet.TYPE_FORWARD_ONLY,/*向前查找结果集*/
                                    ResultSet.CONCUR_READ_ONLY);/*只读*/
         res = ps.executeQuery();
         while (res.previous()) {
             System.out.println(res.getString("t_name"));
         }
     
     } catch (Exception e) {
         e.printStackTrace();
     } finally {
         C3P0Util.close(conn, ps, res);
     }
     ```

     



# 事务

> 数据库中保证交易可靠的机制
>
> 问题：给另一个人转钱，过程中出现异常，如何保证原有账户的钱不减少。
>
> 解决方案：==事务==

- ==JDBC处理事务的通常模式：==

  1. **将事务提交模式改为手动**：`connection.setAutoCommit(false);`：默认true，自动提交
  2. 执行事务中的SQL语句
  3. **提交事务**`connection.commit();`
     - 如果SQL失败或者出现异常，将事务**回滚**`connection.rollback();`
  4. 恢复JDBC的事务提交状态；释放资源

- 实例代码

  ```java
  try {
      /*1.先获取事务自动提交的状态*/
      boolean flag = connection.getAutoCommit();//默认为true，自动提交
      connection.setAutoCommit(false);/*2.手动修改状态为false，禁止自动提交*/
  
      //t_id=1用户减少10岁
      ps = connection.prepareStatement("update teacher set t_age=t_age-10 where t_id=1 ");
      ps.executeUpdate();
  
      int i = 1/0;//为了测试，造成一个异常
  
      //t_id=4的用户增加10岁
      ps = connection.prepareStatement("update teacher set t_age=t_age+10 where t_id=4 ");
      ps.executeUpdate();
  
      connection.commit();/*2.手动提交前面执行的所以的SQL语句*/
  
      /*3.最后需要把事务的手动提交 切换为 自动状态，以免出现其他的情况*/
      connection.setAutoCommit(flag);
  
  } catch (Exception e) {
      e.printStackTrace();
      try {
          connection.rollback();//所有的交易数据都恢复到最初的状态
      } catch (SQLException ex) {
          ex.printStackTrace();
      }
  } finally {
      C3P0Util.close(connection, ps, null);
  }
  ```

  



# 批量处理

> 批处理：发送到数据库作为一个单元执行的一组更新语句
>
> 优点：
>
> - 批处理降低了应用程序和数据库之间的网络调用
> - 相比单个SQL语句的处理，批处理更有效
>
> 特点：
>
> - ==批处理使SQL语句执行的效率更高==
>
> ==操作流程==：
>
> - 将要运行的SQL语句添加到批处理队列：`preparedStatement.addBatch();`
> - 当队列满后，一次性写入数据库：`preparedStatement.executeBatch();`
> - 为下一次的批处理，将队列清空：`preparedStatement.clearBatch();`

- 图示

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110816.jpg)

- 实例代码：向person表中插入1000条数据

  ```java
  public static void main(String[] args) {
      Connection conn = null;
      PreparedStatement ps = null;
      try {
          conn = C3P0Util.getConnection();
          String sql = "insert into person values (null,?,?)";
          ps = conn.prepareStatement(sql);
          for (int i = 0; i < 1000; i++) {
              ps.setString(1,"孙悟空"+i);
              ps.setInt(2,i);
              ps.addBatch();
              if (i % 100 == 0) {
                  ps.executeBatch();//批量执行100条数据
                  ps.clearBatch();//执行完毕后，清空操作
              }
          }
          //最后一批进行批处理
          ps.executeBatch();//批量执行100条数据
          ps.clearBatch();//执行完毕后，清空操作
  
  
      } catch (Exception e) {
          e.printStackTrace();
      } finally {
          C3P0Util.close(conn, ps, null);
      }
  }
  ```

  

# 关联表插入数据

表于表的关系：

- 1对1
- 1对多
- 多对多

> 1对1举例：
>
> - user表：基础信息表  ，字段id
> - userInfo表：详细信息表  ，字段 id
> - 用户通过页面维护了基础信息+详细信息，需要把数据传到数据库
>   - 基本操作：执行2条SQL语句
>   - ==user的id必须要和userInfo的id一致==
> - 问题：
>   - `insert into user values(null,‘张三’);`
>   - `insert into userInfo values('需要用上一条的id');`
>   - 第一条执行的插入语句：id是新生成的；第二条想要获取这个新生成的id，是不能直接得到的
> - 一般处理方式：麻烦，代码量大
>   1. 插入到user表，生成一个自增id：inset
>   2. 查询这个新插入的id：select
>   3. 根据查到的id，插入到userInfo表：insert
> - ==最好的解决方案==：`statement.getGenerateKeys();`
>   - 不需要查询数据库
>

## `statement.getGenerateKeys();`

- 要用这个方法，需要用prepareStatement的2个参数的方法

  - `connection.prepareStatement(sql,value);`
    - sql：需要的SQL语句
    - value：==Statement.RETURN_GENERATED_KEYS==

- 通过该方法可以直接获取新生成的主键

    `statement.getGenerateKeys();`

- 获取主键的值：

  ```java
  result =ps.getGeneratedKeys();//返回新生成的主键
  result.next();//将游标指向下一个数据id（默认在第一个前的空白行）
  int id = result.getInt(1);//获取当前的id值
  ```

- 实例代码

  > user表插入新数据，同时向关联表userInfo表更新相应的数据
  >
  > user表的id和userInfo表的id是一一对应的

  ```java
  String sql1 = "insert into user values(default,?)";
  ps = connection.prepareStatement(sql1,Statement.RETURN_GENERATED_KEYS);
  ps.setString(1, "二师兄");
  ps.executeUpdate();
  //获取主键的方法
  res = ps.getGeneratedKeys();
  res.next();//移动一下，指向第一个数据id
  int id = res.getInt(1);
  System.out.println(id);
  
  //第二条SQL
  String sql2 = "insert into userInfo values(?,?,?,?)";
  ps = connection.prepareStatement(sql2);
  ps.setInt(1,id);
  ps.setString(2, "190@qq.com");
  ps.setString(3, "110");
  ps.setString(4, "家");
  ps.executeUpdate();
  ```

  



# 分页查询

> 不同数据库获取部分结果集的SQL是有区别的
>
> - MySQL和Oracle的分页书写方式就不一样

==语法格式（仅限MySQL）==：`select * from user limit begin,pageSize`

- begin：从第几条开始显示，但==不包括这一条数据==
- pageSize：每页多少条

## 实例代码

> 查询user表，每次查询8条数据

```java
try {
    /*分页查询的SQL语句：该语法仅限MySQL使用*/
    String sql = "select * from user limit ?,?";
    ps = conn.prepareStatement(sql);
    ps.setInt(1, 0);/*第一个？：begin，第几条开始，不包含当前条*/
    ps.setInt(2, 8);/*第二个？：pageSizes:一页中的数据量*/
    rs = ps.executeQuery();
    while (rs.next()) {
        int id = rs.getInt("id");
        String name = rs.getString("name");
        System.out.println(id + "--" + name);
    }
}
```

