[TOC]

# API

> API文档：应用程序接口
>
> Application Programing interfaces

- API里提供的所有方法被protected或public修饰



# 常用包介绍

|    包名    |                             介绍                             |
| :--------: | :----------------------------------------------------------: |
| java.lang  |          基础类：使用频率非常高的类，不用使用import          |
| java.util  | 工具包：集合类，Set,List,Map,日期相关，随机数Random，Scanner |
|  java.io   |                           输入输出                           |
|  java.sql  |                            数据库                            |
|  java.net  |                          和网络相关                          |
|  java.awt  |                                                              |
| java.swing |                                                              |



# 文档注释

```java
/**
 * 1.在类上：
 *     目的：
 *         开发者 石晓浩
 *         开始时间：2019年5月12日11:28:42
 *         结束时间：2019年5月12日11:36:20
 *         功能：测试文档注释
 * @author dell
 *
 */
public class Demo01 {
 
	/**
	 * 2. 注释常量
	 *     表示圆周率
	 */
	public static final double PI=3.1415926;
	/**
	 * 3.注释方法：
	 *     参数1：接收int的参数
	 *     参数2：接收int的参数
	 * @return：返回两个数的和
	 */
	public int sum(int x,int y) {
		return x+y;
	}
	public static void main(String[] args) {
		Demo01 d = new Demo01();
		System.out.println(Demo01.PI);
		System.out.println(d.sum(2, 3));
	}
}

```

