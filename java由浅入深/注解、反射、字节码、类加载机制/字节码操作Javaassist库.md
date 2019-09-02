[TOC]



# 字节码介绍

> - Java动态性的两种常见实现方式
>   - 字节码操作
>   - 反射
>
> - 运行时操作字节码可以让我们实现如下功能
>   - 动态生成新的类
>   - 动态改变某个类的结构（添加、删除、修改新的属性/方法）
> - 字节码操作的优势
>   - 比反射开销小，性能高
>   - JAVAasist性能高于反射，低于ASM



## 常见的字节码操作类库

> - BCEL
>   - BCEL是Java classworking 广泛使用的一种框架，它可以让您深入JVM汇编语言进行类操作的细节，BCEL于Javaassist 有不同的处理字节码方法，BCEL在实际的JVM 指令层上进行操作（BCEL拥有丰富的JVM指令集支持）而Javaassist所强调的源代码级别的工作
> - ASM
>   - 是一个轻量级Java字节码操作框架，直接涉及到JVM底层的操作和指令
> - CGLIB（Code Generation Library）
>   - 是一个强大的，高性能，高质量的Code生成类库，基于ASM实现
> - Javassist
>   - 是一个开源的分析、编辑和创建Java字节码的类库。性能较ASM差，跟CGLIB差不多，但是使用简单。很多开源框架都在使用它
>
> 
>
> AOP：面向切面编程：动态的在代码的前、后添加代码



# javassist库API

## 创建一个类

> 前提：导入javassist类库
>
> - ClassPool是一个CtClass对象的容器，一个CtClass对象被构建后，它被记录在ClassPool中
> - 创建类（包括项目名称/包名+类名）
>
> ```java
> ClassPool pool = ClassPool.getDefault();
> CtClass cc = pool.makeClass("TestJavaAssist.Emp");
> ```



## 创建属性

> - 创建属性，并通过`addField()`方法添加到创建的==字节码文件==中
>
>   ```java
>   CtField f1 = CtField.make("private int empno;", cc);
>   CtField f2 = CtField.make("private String ename;", cc);
>   
>   //添加到创建的类中
>   cc.addField(f1);
>   cc.addField(f2);
>   ```



## 创建方法

> - 创建方法，并通过`addMethod()`方法添加到创建的==字节码文件==中
>
>   ```java
>   CtMethod m1 = CtMethod.make("public int getEmpno(){return empno;}", cc);
>   CtMethod m2 = CtMethod.make("public void setEmpno(int empno){this.empno = empno;}", cc);
>   
>   cc.addMethod(m1);
>   cc.addMethod(m2);
>   ```
>



## 添加构造器

> - 创建构造器，并通过`addConstructor()`方法添加到创建的==字节码文件==中
>
>   ```java
>   //创建有参构造器（int,String）
>   CtConstructor constructor = new CtConstructor(new CtClass[] {CtClass.intType,pool.get("java.lang.String")}, cc);
>   //添加构造体
>   constructor.setBody("{this.empno = empno;this.ename = ename;}");
>   
>   //将构造器添加到字节码文件中
>   cc.addConstructor(constructor);
>   
>   ```
>
> - 创建构造器的代码解析：
>
>   - `new CtConstructor(new CtClass[]{CtClass.intType,pool.get("java.lang.String")}, cc);`
>     - new CtClass[]{}规定写法
>       - 如果是无参构造，`{}`中就不用写东西
>       - 有参构造，8中基本类型：`CtClass.***Type`;不是基本类型：`pool.get(对应类型路径/包名)`
>     - 第二个参数：要添加到的字节码文件对象
>
> - 添加构造器内容
>
>   - `constructor.setBody("{this.empno = empno;this.ename = ename;}");`
>
> - 通过`add.Constructor()`添加到字节码文件中



## 将上述构造好的字节码文件导出

> - 通过`writeFile(path)`把指定字节码文件写入到路径==path==中
>
>   ```java
>   cc.writeFile("E:MyJava");//将上面构造好的类写入到E:/MyJava中
>   ```
>
>   



## 使用XJAD反编译工具，生成Java文件

> <!--使用XJAD反编译工具，将生成的class文件反编译成Java文件-->
>
> XJAD是一个软件，在软件中打开生成的字节码文件，就可以自动反编译成`.Java`文件，并保存在和字节码文件同级目录下



# Javassist库API详解

## 处理类的基本用法

> - `pool.get()`;获取已存在文件的class文件
>
> - `cc.toBytecode()`;转换成字节码
>
> - `cc.getName()`;获取类名
>
> - `cc.getSimpleName()`;获取简要类名
>
> - `cc.getSuperclass()`;获取父类
>
> - `cc.getInterfaces()`;获取接口
>
> - 实例代码：
>
>   ```java
>   	/**
>   	 * 处理类的基本用法
>   	 * @throws Exception 
>   	 */
>   	public static void test01() throws Exception {
>   		ClassPool pool = ClassPool.getDefault();
>   		CtClass cc = pool.get("TestJavaAssist.Emp");
>   		
>   		byte[] bytes = cc.toBytecode();
>   		System.out.println(Arrays.toString(bytes));
>   		
>   		System.out.println(cc.getName());//获取类名
>   		System.out.println(cc.getSimpleName());//获取简要类名
>   		System.out.println(cc.getSuperclass());//获取父类
>   		System.out.println(cc.getInterfaces());//获取接口
>   		
>   		
>   	}
>   ```



## 对方法的操作

### 产生新的方法，并测试

> - 添加新方法
>
>   - 方式一：
>
>  ```java
>  CtMethod m = CtNewMethod.make("public int add(int a,int b){return a+b;}", cc);
>  cc.addMethod(m);
>  ```
>
>   - 方式二：
>
>  ```java
>  CtMethod m = new CtMethod(CtClass.intType, "add",new CtClass[] {CtClass.intType,CtClass.intType},cc);
>  m.setModifiers(Modifier.PUBLIC);
>  m.setBody("{System.out.println(\"www.baidu.com\");return $1+$2;}");
>  
>  cc.addMethod(m);
>  ```
>
>     - new CtMethod(arg1,arg2,arg3);
>     
>       - 参数1：方法的返回值类型 int:`CtClass.intType`
>       - 参数2：方法名
>       - 参数3：新添加的方法的参数   `new CtClass[]{……(同上述参数1)};`
>     
>     - 添加方法时用到的符号
>
> | **符号**       | **说明**                                                     |
> | -------------- | ------------------------------------------------------------ |
> | $0 ,$1,$2,$3…… | $0代表this<br />$1代表第一个参数<br />$2代表第二个参数<br />……………… |
> | $args          | 把所有参数放到这个数组中<br />$args[0]对应的是$1<br />……………… |
> | $cflow         | 一个方法调用的深度                                           |
> | $r             | 方法返回值的类型                                             |
> | $_             | 方法的返回值（修改方法时不支持）                             |
> | addCatch()     | 方法中加入try catch<br />$e代表异常对象                      |
> | $class         | this的类型(class),也就是$0的类型                             |
> | $sig           | 方法参数的类型(class)数组,数组的顺序为参数的顺序             |
>
> - 通过反射调用新生成的方法
>
> ```java
> //通过反射调用新生成的方法
> Class clazz = cc.toClass();//CtClass的方法，获取到Class对象
> Object obj = clazz.newInstance();//通过调用Emp无参构造，创建新的Emp对象
> 
> //获取方法，名称为“add”,两个参数为int类型
> Method method = clazz.getDeclaredMethod("add", int.class,int.class);
> 
> //执行该方法，对象为obj，参数为200，300
> Object result = (Object) method.invoke(obj, 200,300);//方法有返回值，为int型
> System.out.println(result);//如果方法无返回值，该result为null
> ```
>
> - 结果：
>
>   **www.baidu.com**
>   **500**
>



### 修改已存在的方法，并测试

> - 加载已有的类class文件
>
> ```java
> ClassPool pool = ClassPool.getDefault();
> CtClass cc = pool.get("TestJavaAssist.Emp");
> ```
>
> - 已存在的类中的`sayHello()`方法
>
>   ```java
>   	public void sayHello(int a) {
>   		System.out.println("sayHello,"+a);
>   	}
>   ```
>
>   
>
> - 获取指定的方法，并更改方法
>
>   - `getDeclaredMethod()`
>     - 第一个参数：方法名
>     - 第二个参数：方法的参数
>
>   ```java
>   //获取指定-名称、参数的方法
>   CtMethod cm = cc.getDeclaredMethod("sayHello",new CtClass[] {CtClass.intType});
>   
>   //(insertBefore)在方法体前面插如代码
>   cm.insertBefore("System.out.println($1);System.out.println(\"start\");");
>   //(insertAfter)在方法体后面插入代码
>   cm.insertAfter("System.out.println(\"end\");");		
>   ```
>
> - 通过反射调用新生成的方法
>
>   ```java
>   //通过反射调用新生成的方法
>   Class clazz = cc.toClass();//获取该类的Class对象
>   Object obj = clazz.newInstance();//通过反射创建对象
>   Method method = clazz.getDeclaredMethod("sayHello",int.class);//获取指定的方法
>   Object result = (Object) method.invoke(obj, 100);//反射调用方法
>   System.out.println(result);//因为方法无返回值，所以result为null
>   ```
>
> - 结果：
>
>   **100**
>   **start**
>   **sayHello,100**
>   **end**
>   **null**



## 对属性的操作

### 创建新的属性

> - 方法一：
>
>   ```java
>   CtField f1 = CtField.make("private int empno;", cc);
>   ```
>
> - 方法二：
>
>   - `new CtField(*,*,*)`
>     - 第一个参数：属性的类型
>     - 第二个参数：属性名称
>     - 第三个参数：为哪个class文件添加
>
>   ```java
>   CtField f1 = new CtField(CtClass.intType,"salary",cc);
>   f1.setModifiers(Modifier.PRIVATE);
>   cc.addField(f1,"1000");
>   ```



### 对新的属性添加get/set方法

> - 添加set方法
>
>   ```java
>   cc.addMethod(CtNewMethod.getter("getSalary", f1));
>   ```
>
> - 添加get方法
>
>   ```java
>   cc.addMethod(CtNewMethod.getter("setSalary", f1));
>   ```
>
>   

### 获取已有属性

> - `cc.getDeclaredField("ename");`//获取指定的属性





## 对构造器的操作

> - 获取所有构造器
>
>   ```java
>   	/**
>   	 * 对构造器的操作
>   	 * @param args
>   	 * @throws Exception 
>   	 * @throws Exception
>   	 */
>   	public static void test05() throws Exception {
>   		ClassPool pool = ClassPool.getDefault();
>   		CtClass cc = pool.get("TestJavaAssist.Emp");
>   		
>   		CtConstructor[] cs = cc.getConstructors();
>   		for (CtConstructor c : cs) {
>   			System.out.println(c.getLongName());
>   		}
>   	}
>   ```
>
> - 对构造器方法体的操作和对普通方法的操作相同



## 对注解的操作

> - 自定义注解
>
>   ```java
>   public @interface Author{
>       String name();
>       int year();
>   }
>   ```
>
> - 对方法添加注解
>
>   ```java
>   @Author(name="Chiba",year=2005)
>   public class Point{
>       int x,y;
>   }
>   ```
>
> - 通过字节码操作获取注解
>
>   ```java
>   CtClass cc = ClassPool.getDefault().get("Point");
>   Object[] all = cc.getAnnotations();
>   Author a = (Author)all[0];
>   String name = a.name();
>   int year = a.year();
>   System.out.println("name="+name+",year="+year);
>   ```



# javassist库的API的局限性

> - JDK5.0以后不支持（包括泛型，枚举），不支持注解修改
>
> - 不支持数组的初始化，如String[]{“1”,“2”},除非只有数组的容量为1
>
> - 不支持内部类和匿名类
>
> - 不支持continue和break表达式
>
> - 对于继承关系，有些不支持，例如
>
>   class A{}
>
>   class B extends A{}
>
>   class C extends B{}