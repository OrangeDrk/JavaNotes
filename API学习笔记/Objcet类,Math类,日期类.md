[TOC]



# Object方法

> equals返回true，hashCode必须相同，所以hashCode和equals必须同时重写（集合Set中应用）

1. equals（Object obj）：判断对象是否相同
2. hashCode（）：返回该对象的hashCode码
3. toString：以字符串的形式返回对象

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

# Math类

> 构造方法为私有，不能使用，不代表没有
>
> 每个类都有构造方法（抽象类也有构造方法，接口没有构造方法）

|          方法名          |                             介绍                             |
| :----------------------: | :----------------------------------------------------------: |
|      ceil(double a)      |           返回 >=a 的最小整数值（double类型，6.0）           |
|     floor(double a)      |          返回 <=a 的最大整数值 （double类型，5.0）           |
|    max(int a, int b)     |                      返回a和b中的最大值                      |
|        random（）        | 返回[0.0,1.0)的随机数<br />==API提供的Random类：<br />Random random = new Random();<br />int i = random.nextInt(10);== |
|    round（double a）     |                           四舍五入                           |
| pow（double a, double b) |                        返回a^b的结果                         |

> Random示例

```java
		package demoMath;
		 
		import java.util.Random;
		 
		public class DemoMath {
		        public static void test1() {
		                System.out.println(Math.max(Math.max(3, 4), 5) );
		        }
		        public static void test2() {
		                System.out.println(Math.ceil(5.8));//>=5.8的最小值6.0
		                System.out.println(Math.floor(5.8));//<=5.8的最大值5.0
		        }
		        public static void testRandom() {
		                Random random = new Random();
		                //返回[0,10)
		                int i = random.nextInt(10);
		                System.out.println(i);
		        }
		        public static void test3() {
		                System.out.println(Math.round(5.8));//6
		        }
		 
		        public static void main(String[] args) {
		                test2();
		 
		        }
		 
		}
		 

```

# 日期类

> new Date():最常用
>
> Sat May 18 11:49:38 CST 2019

| Sat  | May  | 18   | 11:49:38 | CST                            | 209  |
| ---- | ---- | ---- | -------- | ------------------------------ | ---- |
| 星期 | 月   | 日   | 时间     | 中央标准时间<br />（北京时间） | 年   |



## 对日期的规范

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//规范格式
System.out.println(sdf.format(new Date()));
```



## 字符串转换成日期类型

```java
parse( str );
```



# Calendar类

> 更加详细全面的日历信息

## Calendar rightNow = Calendar.getInstance();

```java
java.util.GregorianCalendar[time1558159814215（从1970年1月1日到现在时刻的毫秒值）,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2019,MONTH=4（月份从0开始到11）,WEEK_OF_YEAR=20,WEEK_OF_MONTH=3,DAY_OF_MONTH=18,DAY_OF_YEAR=138,DAY_OF_WEEK=7,DAY_OF_WEEK_IN_MONTH=3,AM_PM=1,HOUR=2,HOUR_OF_DAY=14,MINUTE=10,SECOND=14,MILLISECOND=215,ZONE_OFFSET=28800000,DST_OFFSET=0]
```



## 获取年月日

```java
System.out.println(rightNow.get(Calendar.YEAR));//年
System.out.println(rightNow.get(Calendar.MONTH)+1);//月
System.out.println(rightNow.get(Calendar.DAY_OF_MONTH));//日

```



# System的静态方法获取系统时间的毫秒值

```java
System.currentTimeMillis();
```



# jdk1.8以后提供的日期时间类

|                 方法名                 |             结果              |
| :------------------------------------: | :---------------------------: |
|     LocalDate d = LocalDate.now()      |          2019-05-18           |
|     LocalDate t = LocalTime.now()      |      14:40:22.852980800       |
| LocalDateTime dt = LocalDateTime.now() | 2019-05-18T14:42:33.007551500 |

```java
package DemoDate;
 
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.Calendar;
import java.util.Date;
 
public class DemoDate {
 
	public static void test1() {
		System.out.println(new Date());
	}
	
	/**
	 * 日期的规范     最常用
	 */
	public static void test2() {
	        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	        System.out.println(sdf.format(new Date()));
	}
	
	/**
	 * Calendar时间日期类
	 */
	public static void test3() {
		Calendar rightNow = Calendar.getInstance();
		System.out.println(rightNow.get(Calendar.YEAR));
		System.out.println(rightNow.get(Calendar.MONTH)+1);
		System.out.println(rightNow.get(Calendar.DAY_OF_MONTH));
		String strTime = rightNow.get(Calendar.YEAR)+"-"+(rightNow.get(Calendar.MONTH)+1)+"-"+rightNow.get(Calendar.DAY_OF_MONTH);
		System.out.println(strTime);
	}
	
	/**
	 * System的静态方法获取系统时间的毫秒值
	 * @param args
	 */
	public static void test4() {
		long start = System.currentTimeMillis();
		method();
		long end = System.currentTimeMillis();
		System.out.println(end-start);
	}
	//模拟业务方法
	public static void method() {
		for(int i = 0;i<10000000;i++) {
		}
		System.out.println("end");
	}
	
	/**
	 * jdk1.8以后提供的日期时间类
	 * LocaDate
	 * LocalTime
	 * LocalDateTime
	 */
	public static void test5() {
		LocalDate d = LocalDate.now();
		System.out.println(d);
		LocalTime t = LocalTime.now();
		System.out.println(t);
		LocalDateTime dt = LocalDateTime.now();
		System.out.println(dt);
	}
 
	public static void main(String[] args) {
		test5();
	}
 
}

```

