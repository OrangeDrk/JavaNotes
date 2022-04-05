[TOC]

> Spring编程风格

schema-based：xml风格

annotation-based：注解风格

java-based：javaconfig风格



# 1. 单例bean依赖多例bean问题

在Spring中，可能会存在一个单例对象中，依赖一个多例的对象。

如果只用`@scope(“prototype”)`，单例对象中的多例对象会失效，导致`@scope(“prototype”)`失效

> 原因

因为单例对象加载时只会初始化一次，那么单例中的多例对象不会再重新注入了，导致每次回去单例对象时，其中的多例对象失效，也变成了单例

```java
@Component
@Scope("prototype")
public class ShopeCart {
}

@Component
public class ShopeUser {
   
    @Autowired
    private ShopeCart shopeCart;

    public void printCart(){
        System.out.println(shopeCart.hashCode());
    }
}

// printCart()每次打印都是相同的结果
//367130878
//367130878
//367130878
// ↑单例失效
```



> 解决：Spring官方给出的解决办法。通过`@Lookup`解决

<font size=5 color=red>@Lookup</font>

**通过一个不用实现的抽象方法来获取多例对象，此时需要把单例对象也声明为abstract**

```java
@Component
@Scope("prototype")
public class ShopeCart {
}

@Component
public abstract class ShopeUser {
   
    private ShopeCart shopeCart;

    public void printCart(){
        System.out.println(getShopeCart().hashCode());
    }
    
    @Lookup
    public abstract ShopeCart getShopeCart();
}

// printCart()每次打印都是不同的结果
//561133045
//1262573693
//1256975947
// ↑单例生效
```

> 不用声明abstract的方式

```java
@Component
@Scope("prototype")
public class ShopeCart {
}

@Component
public class ShopeUser {
    @Autowired
    private ShopeCart shopeCart;

    public void printCart(){
        System.out.println(getShopeCart().hashCode());
    }

    @Lookup
    public ShopeCart getShopeCart(){
        return null;
    }
}

// printCart()每次打印都是不同的结果
//1769605448
//2096511898
//1198973449
// ↑单例生效
```



