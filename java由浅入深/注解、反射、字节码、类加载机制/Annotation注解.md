[TOC]



# Annotation



## Annotation的作用

> - 不是程序本身，可以对程序作出解释
> - 可以被其他程序读取（比如：编译器等）

## Annotation的格式

> 注解是以“`@注释名`”在代码中存在的，还可以添加一些参数值，例如：
>
> `@SuppressWarnings(value="unchecked")`

## Annotation在哪里使用

> 可以附加在package、class、method、field 等上面，相当于给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问



# 内置注解

## @Override

> 定义在java.lang.Override中，此注释只适用于修饰方法，==表示一个方法声明打算重写超类中的另一个方法声明==
>
> - 正确：
>
>   ```java
>   @Override//表示重写父类的方法
>   public String toString() {
>   	return "";
>   }
>   ```
>
> - 错误（Object中没有tostring方法）
>
>   ```java
>   @Override
>   public String tostring() {
>   	return " ";
>   }
>   ```
>



## @Deprecated

> 定义在java.lang.Deprecated中，此注释可用于修饰方法、属性、类；表示==不鼓励程序员使用这样的元素==，通常是因为它很危险或存在更好的选择
>
> 
>
> ```java
> 	@Deprecated
> 	public static void test001() {//test001处会有下划线	
> 	}
> 	
> 	public static void main(String[] args) {
> 		Date d = new Date();
> 		test001();//test001处会有下划线，表明不推荐使用该方法
> 	}
> ```
>
> 

## @SuppressWarnings

> 定义在java.lang.SuppressWarnings中，==用来抑制编译时的警告信息==
>
> ```java
> 	@SuppressWarnings("all")//不加这句话下面3个list就会有警告
> 	public static void test002() {
> 		List list = new ArrayList();
> 		List list1 = new ArrayList();
> 		List list2 = new ArrayList();
> 	}
> ```
>
> 与前两个注解有所不同，你需要添加一个参数才能正常使用，这些参数值都是已经定义好了的，我们选择性的使用就好了，参数入下：
>
> | **参数**    | **说明**                                           |
> | ----------- | -------------------------------------------------- |
> | deprecation | 使用了过时的类或者方法的警告                       |
> | unchecked   | 执行了未检查的转换时的警告，如使用集合时未指定泛型 |
> | fallthrough | 当在switch语句使用时发生case穿透                   |
> | path        | 在类路径、源文件路径等中有不存在路径的警告         |
> | serial      | 当在序列化的类上缺少serialVersionUID定义时的警告   |
> | finally     | 任何finally子句不能完成时的警告                    |
> | all         | 关于以上所有情况的警告                             |
>
> `@SuppressWarnings(“unchecked”)`
>
> `@SuppressWarnings(value={“unchecked”,“deprecation”})`





# 自定义注解

## @Override源码分析

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
    
}
```



## 自定义注解

> - 使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口
> - 要点：
>   - @interface用来声明一个注解
>     - 格式：`public @interface 注解名{定义体}`
>   - 其中的每一个方法实际上是声明了一个配置参数
>     - 方法的名称就是参数的名称
>     - 返回值类型就是参数类型（返回值类型只能是基本类型、Class、String、enum）
>     - 可以通过default来声明参数的默认值
>     - 如果只有一个参数成员，一般参数名为value

### 元注解

> 元注解的作用：负责注解其他注解
>
> 这些类型和它们所支持的类在java.lang.annotation包中可以找到
>
> ==@Target==、==@Retention==、@Documented、@Inherited

### @Target

> - 作用：==用于描述注解的使用范围（即”被描述的注解可以用在什么地方）==
>
>   | **所修饰范围**                               | **取值ElementType**                                          |
>   | :------------------------------------------- | :----------------------------------------------------------- |
>   | package 包                                   | PACKAGE                                                      |
>   | 类、接口、枚举、Annotation类型               | TYPE                                                         |
>   | 类型成员（方法、构造方法、成员变量、枚举值） | CONSTRUCTOR：用于描述构造器<br />FIELD：用于描述域<br />METHOD：用于描述方法 |
>   | 方法参数和本地变量                           | LOCAL_VARIABLE：用于描述局部变量<br />PARAMETER：用于描述参数 |
>
> - `@Target(value=ElementType.TYPE)`
>
> - 实例1：
>
>   ```java
>   import java.lang.annotation.ElementType;
>   import java.lang.annotation.Target;
>   
>   @Target(value=ElementType.METHOD)//在方法前使用
>   public @interface MyAnnotation01 {
>   	
>   }
>   ```
>
> - 实例2：
>
>   ```java
>   import java.lang.annotation.ElementType;
>   import java.lang.annotation.Target;
>   
>   @Target(value= {ElementType.METHOD,ElementType.TYPE})//在方法、类前使用
>   public @interface MyAnnotation01 {
>   	
>   }
>   ```



### @Retention

> 作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期
>
> | **取值   RetentionPolicy** | **作用**                                                     |
> | -------------------------- | ------------------------------------------------------------ |
> | SOURCE                     | 在源文件中有效（即源文件保留）                               |
> | CLASS                      | 在class文件中有效（即class保留）                             |
> | ==RUNTIME==                | 在运行时有效（即运行时保留）<br />为Runtime可以被反射机制读取 |
>
> - 实例：
>
>   ```java
>   @Target(value= {ElementType.METHOD,ElementType.TYPE})//在方法前使用
>   @Retention(RetentionPolicy.RUNTIME)
>   public @interface MyAnnotation01 {
>   	
>   }
>   ```

## 注解内容(参数信息)

> `String studentName() default "";`	
>
> `int age() default 0;`
>
> - String——参数类型 	
> - studentName()——参数名	
> - default——默认值（没有默认值会报错）经常使用空、0，-1（表示不存在）
> - 如果参数只有一个，参数名一般用value
>
> - 实例
>
>   ```java
>   import java.lang.annotation.ElementType;
>   import java.lang.annotation.Retention;
>   import java.lang.annotation.RetentionPolicy;
>   import java.lang.annotation.Target;
>   
>   @Target(value= {ElementType.METHOD,ElementType.TYPE})//在方法前使用
>   @Retention(RetentionPolicy.RUNTIME)
>   public @interface MyAnnotation01 {
>   	String studentName() default "";
>   	int age() default 0;
>   	
>   	String[] schools() default {"北京大学"};
>   	int id();
>   	
>   }
>   ```
>
>   ```java
>   package Testannotation;
>   
>   public class Demo02 {
>   	@MyAnnotation01(age = 19,studentName = "Tom",id = 1001,
>   			schools= {"北京大学"})
>   	public void test() {
>   		
>   	}
>   ```
>
>   



# 通过反射机制读取注解

## ORM

> ORM：（Object Relationship Mapping）——对象关系映射
>
> ```java
> public class Student{
> 	int id;
> 	String sname;
> 	int age;
> }
> ```
>
> 把类转换成SQL表的形式
>
> - Student——tb_student
> - id——’id’ int(10)
> - sname——‘sname’ varchar(10)
> - age——‘age’  int(3) 
>
> 1. 规则：
>    - 类和表结构对应
>    - 属性和字段对应
>    - 对象和记录对应



## 应用（自定义注解，标明注解）

1. 类注解：

   ```java
   package Testannotation;
   
   import java.lang.annotation.ElementType;
   import java.lang.annotation.Retention;
   import java.lang.annotation.RetentionPolicy;
   import java.lang.annotation.Target;
   
   @Target(value=ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Table {
   	String value();
   }
   ```

2. 属性注解：

   ```java
   package Testannotation;
   
   import java.lang.annotation.ElementType;
   import java.lang.annotation.Retention;
   import java.lang.annotation.RetentionPolicy;
   import java.lang.annotation.Target;
   
   @Target(value=ElementType.FIELD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Field {
   	String columnName();
   	String type();
   	int length();
   	
   }
   ```

3. 类中标明注解

   ```java
   package Testannotation;
   
   @Table("tb_student")//类名注解，SQL表名
   public class Student {
   	@Field(columnName="id",type="int",length=10)//属性注解，SQL属性名
   	private int id;
   	@Field(columnName="sname",type="varchar",length=10)
   	private String name;
   	@Field(columnName="age",type="int",length=3)
   	private int age;
   	
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
   	public int getAge() {
   		return age;
   	}
   	public void setAge(int age) {
   		this.age = age;
   	}
   
   }
   ```



## 解析注解（一般都有框架来完成）

1. 获取类的所有注解信息
   `clazz.getAnnotations()`:Annotation[]

   ```java
   public class Demo03 {
   	
   	public static void main(String[] args) {
   		try {
   			Class clazz = Class.forName("Testannotation.Student");
   			Annotation[] annotations = clazz.getAnnotations();
   			
   			for (Annotation a : annotations) {
   				System.out.println(a);
   			}
   			
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   
   }
   
   ```

   结果：`@Testannotation.Table(value=tb_student)`

2. 通过类名获取注解
   `注解类名 ** = clazz.getAnnotation(注解类名.class)`

   ```java
   public class Demo03 {
   	
   	public static void main(String[] args) {
   		try {
   			Class clazz = Class.forName("Testannotation.Student");
   	
   			//获得类的指定的注解
   			Table table = (Table) clazz.getAnnotation(Table.class);
   			System.out.println(table.value());
   			
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   
   }
   ```

   结果：`tb_student`

3. 获得属性相关的值
   通过`Field f = clazz.getDeclaredField("属性名")`获取属性的对象
   再通过`MyField field = f.getAnnotation(MyField.class);`获取该属性对象的注解类对象，通过该对象就可以获取该属性对应的注解信息

   ```java
   public class Demo03 {
   	
   	public static void main(String[] args) {
   		try {
   			Class clazz = Class.forName("Testannotation.Student");
   			
   			//获得类的属性的注解
   			Field f = clazz.getDeclaredField("name");
   			MyField field = f.getAnnotation(MyField.class);
   			System.out.println(field.columnName()+","+field.type()+","+field.length());
   			
               //根据获得的表名、字段的信息，拼出DDL语句，使用JDBC执行这个SQL，在数据库中生成相关的表
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   
   }
   ```

   