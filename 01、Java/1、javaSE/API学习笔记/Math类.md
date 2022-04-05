[TOC]

# Math类

> 构造方法为私有，不能使用，不代表没有
>
> 属性和方法都是静态的
>
> 每个类都有构造方法（抽象类也有构造方法，接口没有构造方法）

## 属性

### PI

> 默认值：3.14159265358979323846
>
> 圆周率

### E

> 默认值：2.7182818284590452354
>
> 比任何其他值都更接近 e（即自然对数的底数）的 double 值。

## 方法

|          方法名          |                             介绍                             |
| :----------------------: | :----------------------------------------------------------: |
|      abs(double a)       |                   返回 double 值的绝对值。                   |
|      ceil(double a)      |    向上取整<br />返回 >=a 的最小整数值（double类型，6.0）    |
|     floor(double a)      |   向下取整<br />返回 <=a 的最大整数值 （double类型，5.0）    |
|    max(int a, int b)     |                      返回a和b中的最大值                      |
|        random（）        | 返回[0.0,1.0)的随机数<br />==API提供的Random类：<br />Random random = new Random();<br />int i = random.nextInt(10);== |
|    round（double a）     |          四舍五入<br />在参数基础上+0.5，再向下取整          |
| pow（double a, double b) |                        返回a^b的结果                         |

### random()

> 根据底层的伪随机算法得来

```java
public static void test2(){
    int i = (int) (Math.random()*20+20);
    System.out.println(i);

    char[] chars = {'职','1','2','a','5','发','中'};
    String s = "";
    //验证码：
    for (int j = 0; j < 4; j++) {
        s = s + chars[(int) (Math.random() * chars.length)];
    }
    System.out.println(s);


}
```



## 精确运算

> double d = 2.1-1.9;不是精确运算

### strictfp

> 把运算过程从64位提升到80位进行运算
>
> 但这样依然不能精确运算

```java
public strictfp static void test3(){
	double d = 2.1-1.9;
}
```

### 使用BigDecimal类

> 可以实现任意精确运算，参数要写成==字符串==的形式
>
> 加法：add
>
> 减法：subtract
>
> 乘法：multiply
>
> 除法：divide
>
> - divisor - 此 BigDecimal 要除以的值。
> - scale - 要返回的 BigDecimal 商的位数。
> - roundingMode - 要应用的舍入模式。 

```java
public strictfp static void test3(){
    BigDecimal bc1 = new BigDecimal("2.3");
    BigDecimal bc2 = new BigDecimal("6.3");
    System.out.println(bc1.add(bc2));//加
    System.out.println(bc1.subtract(bc2));//减
    System.out.println(bc1.multiply(bc2));//乘
    System.out.println(bc1.divide(bc2,1,BigDecimal.ROUND_HALF_UP));//除
}
```

#### 格式化输出

```java
BigDecimal bc3 = new BigDecimal("2.312");
BigDecimal bc4 = new BigDecimal("1.534");
DecimalFormat df = new DecimalFormat("##.###");
System.out.println(df.format(bc3.multiply(bc4)));
```



### 超大数`BigInteger`

> 支持超大数之间的运算

```java
public static void test4(){
    BigInteger bi1 = new BigInteger("2312312312312314254768672342345435345345345345234123488");
    BigInteger bi2 = new BigI nteger("2312312312142547684768672342345414254768345142547685234");
    System.out.println(bi1.multiply(bi2));
}
```



# 实例代码

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

