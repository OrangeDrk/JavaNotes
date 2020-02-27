[TOC]



# JDK1.5新特性

1. switch支持枚举类型
2. 自动拆箱、装箱
3. 增强for循环
4. 静态导入
5. 可变参数
6. 枚举

## 静态导入

> 导入类的静态方法
>
> 特性：
>
> 1. 使用时不用`类名.`，直接写方法名
> 2. 会优先加载

```java
//导入静态方法;
import static java.lang.Math.*;

public class StaticImportDemo {
    public static void main(String[] args) {
        System.out.println(random());
        System.out.println(ceil(4.5));
    }
}
```



## 可变参数

> `...`   表示可变参数，表示可以接收任意多个参数的传入
>
> 可变参数的底层是由一个数组实现的；把传入的参数值一一赋值到数组的对应元素中
>
> 特性：
>
> 1. 可以接收任意多个参数
> 2. 底层基于数组实现
> 3. 可变参数在其他参数右边，处于最后一位
> 4. 在参数列表中只能出现一次

```java
package cn.tedu.jdk.jdk5;

public class VariableDemo {
    public static void main(String[] args) {
        System.out.println(sum(1, 2));
        System.out.println(sum(1.2, 3, 4, 5));
    }

    /** ...   表示可变参数，表示可以接收任意多个参数的传入
     * 可变参数的底层是由一个数组实现的
     * 把传入的参数值一一赋值到数组的对应元素中
     */
    public static int sum(double d ,int... i) {
        int sum = 0;
        for (int j : i) {
            sum += j;
        }
        return sum;
    }
}

```



## 枚举

> 特性：
>
> 1. 一一列举值
> 2. 用enum表示枚举
> 3. 

### 正常枚举

```java
//枚举类：代表季节，只能产生四个对象
enum Season{
    //SPRING--->public static final Season SPRING = new Season();
    //枚举常量,保证要在类的首行，并且在同行
    SPRING,SUMMER,AUTUMN,WINTER;
}
```

### 可以正常的定义属性和方法

```java
//枚举类：代表季节，只能产生四个对象
enum Season{
    //SPRING--->public static final Season SPRING = new Season();
    //枚举常量,保证要在类的首行，并且在同行
    SPRING,SUMMER,AUTUMN,WINTER;
    //可以正常的定义属性和方法
    int i = 1;
    public void m(){}
}
```

### 没有无参构造的枚举

```java
enum Season1{
    SPRING(10),SUMMER(2),AUTUMN(7),WINTER(8);
    private Season1(int month) {}
}
```

### 可以定义抽象方法

```java
enum Season2{
    SPRING(10){
        @Override
        public void play() {
            System.out.println("踏春");
        }
    },
    SUMMER(2) {
        @Override
        public void play() {
            System.out.println("游泳");
        }
    },
    AUTUMN(7) {
        @Override
        public void play() {
            System.out.println("爬山");
        }
    },
    WINTER(8) {
        @Override
        public void play() {
            System.out.println("吃冰激凌");
        }
    };
    private Season2(int month) {}
    //可以定义重写方法
    public abstract void play();
}
```

### switch-case支持枚举类型

```java
public static void main(String[] args) {
        Season2 spring = Season2.SPRING;
        switch (spring) {
            case SPRING:
                System.out.println("春季");spring.play();break;
            case SUMMER:
                System.out.println("夏季");break;
            case AUTUMN:
                System.out.println("秋季");break;
            case WINTER:
                System.out.println("冬季");break;
        }
    }
```





# JDK1.8新特性

1. Lambda表达式
2. 函数式接口
3. Stream(操作集合的流式结构)

## Stream(操作集合的流式结构)（深入了解）

> 提供了大量的函数式接口：使用lambda表达式

```java
package cn.tedu.jdk.jdk8;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.stream.Stream;

public class StreamDemo {
    public static void main(String[] args) {
        //创建集合对象
        List<String> list = new ArrayList<>();
        list.add("C");
        list.add("C++");
        list.add(".Net");
        list.add("python");
        list.add("java");
        list.add("C#");
        list.add("GO");
        //---------------正常写法------------------
        //筛选出以C开头的元素
        for (String s:list) {
            //判断
            if (s.startsWith("C")) {
                System.out.println(s);
            }
        }

        //----------------Stream------------------
        //把集合对象所有元素放到这个流式结构中操作
        Stream<String> s = list.stream();
        //lambda表达式
        //筛选之后进行排序再进行转小写
        s.filter(s1->s1.startsWith("C")).
            sorted((s2,s3)->s3.compareTo(s2)).
            map(str1->str1.toLowerCase()).
            forEach(str-> System.out.println(str));

    }
}

```



## 时间日期类

> Time包：`LocalTime`
>
> Date包：`LocalDate`

### 获取当前日期

```java
LocalTime l = LocalTime.now();
System.out.println(l);//17:54:25.507
```

### 增加10天

```java
LocalTime l = LocalTime.now();
//增加10天
System.out.println(l.plus(10, ChronoUnit.DAYS));
```



### jdk1.8以后提供的日期时间类

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

