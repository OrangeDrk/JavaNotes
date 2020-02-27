[TOC]



# Object中的方法

> Object类是所有类的父类，所有的类在底层默认继承Object
>
> equals返回true，hashCode必须相同，所以hashCode和equals必须同时重写（集合Set中应用）

## `equals(Object obj)`：判断对象是否相同

> ==重写equals()方法必须要重写hashCode()==

- 重写equals()时，如果对象里包含对象属性

  - 要不在创建对象时将对象属性初始化

  - 要不在equals()方法中判断该对象属性是否全为null；

    ```java
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Person person = (Person) o;
            return age == person.age &&
                    gender == person.gender &&
                    (name==null && person.name==null)|| name.equals(person.name);
        }
    ```

    

- 返回的是对象的哈希码值
- 不同对象的哈希码值不一样
- 哈希码值：**哈希码值几乎不会重复**-----哈希码值一定唯一 ---- 代表对象的地址值
  - 取值范围：和int范围一样大，哈希值没有负数[0-41亿左右]
  - 散列分布：均匀分布



## `toString()`：以字符串的形式返回对象

> 不重写：调用的是Object类里的toString()，返回对象的拼接地址值
>
> 重写：调用重写的toString()方法

- `System.out.println(demo3);` 和 `System.out.println(demo3.toString());`效果一样
  - 底层默认让对象调用Object类里的toString()返回对象的地址值



## `clone()`：创建并返回此对象的副本

- 需要继承`Cloneable`接口（此接口仅仅作为一个标记）才能调用`clone()`方法
- ==`clone()`方法只能在Object子类本身中调用，其他类中调用别的类对象的`clone()`方法，会报错==。原因：protected特性

- `CloneNotSupportedException`：克隆不支持异常（没有实现cloneable接口）

> 克隆代码实例
>
> ```java
> package cn.tedu.objectdemo;
> 
> /**
>  * Cloneable接口没有内容，仅仅只是作为一个标记
>  * 类实现Cloneable接口，类产生的对象就能支持克隆操作
>  */
> public class ObjectDemo1 implements Cloneable{
>     int i = 6;
>     public static void main(String[] args) throws CloneNotSupportedException {
>         //创建对象
>         ObjectDemo1 od = new ObjectDemo1();
>         //调用克隆方法
>         ObjectDemo1 od1 = (ObjectDemo1) od.clone();
>         //克隆对象调用属性
>         System.out.println(od1.i);
>         System.out.println(od==od1);
>     }
> }
> 
> ```



## `finalize()`:通知系统进行回收

> 当垃圾收集确定没有对对象的引用时，由对象上的垃圾收集器调用。



## `getClass()`:返回Object运行时的类

> 包名+类名：全路径名
>
> 哪个对象调用就返回该对象的**全路径名**



## 实例代码

> Student类

```java
package demoObject;
 
public class Student {
	private int age;
 
	public int getAge() {
		return age;
	}
 
	public void setAge(int age) {
		this.age = age;
	}
 
	public Student(int age) {
		super();
		this.age = age;
	}
	public Student() {
 
	}
	
 
	@Override
	public String toString() {
		return "Student [age=" + age + "]";
	}
 //重写hashCode和equals方法------------------------------------
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + age;
		return result;
	}
 
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Student other = (Student) obj;
		if (age != other.age)
			return false;
		return true;
	}
}

```

> DemoObject测试类

```java
package demoObject;
 
public class DemoObject {
	//toString()方法
	public static void test1() {
		Student stu = new Student(20);
		String str = new String("Hello");
		System.out.println(stu);
		System.out.println(str);
	}
	//hashCode()方法
	public static void test2() {
		Student stu1 = new Student(20);
		Student stu2 = new Student(20);
		String str1 = new String("Hello");
		String str2 = new String("Hello");
		System.out.println(stu1.hashCode()+","+stu2.hashCode());
		System.out.println(str1.hashCode()+","+str2.hashCode());
	}
	//equals()方法
	public static void test3() {
		Student stu1 = new Student(20);
		Student stu2 = new Student(20);
		String str1 = new String("Hello");
		String str2 = new String("Hello");
		System.out.println(stu1.equals(stu2));
		System.out.println(str1.equals(str2));
	}
 
	public static void main(String[] args) {
		test3();
	}
 
}
```



