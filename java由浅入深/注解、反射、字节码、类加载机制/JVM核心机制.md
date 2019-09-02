[TOC]



# 类加载全过程

## 类加载机制

> JVM把class文件加载到内存，并对数据进行校验、解析和初始化，最终形成JVM可以直接使用的Java类型的过程
>
> - **加载**(Loading)——**链接**[验证(Verification)——准备(Preparation)——解析(Resolution)]———**初始化**(Initalization)——**使用**(Using)——**卸载**(UnLoading)
>
> 
>
> 1. 加载
>
>    > 将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区中运行时数据结构，在堆中生成一个代表这个类的java.lang.Class对象，作为方法区，类数据的访问入口。
>    >
>    > 这个过程需要==类加载器==参与
>
> 2. 链接
>
>    > - 验证
>    >
>    >   确保加载的类信息符合JVM规范，没有安全方面的问题
>    >
>    > - 准备
>    >
>    >   正式为类变量(static变量)分配内存并设置类变量初始值(0,null)的阶段，这些内存都将在方法区中进行分配
>    >
>    >   此处赋初值如果代码为：`int a = 3;`，则会把a赋初值为==0==
>    >
>    > - 解析
>    >
>    >   虚拟机常量池内的符号引用替换为直接引用的过程
>
> 3. 初始化
>
>    > - 初始化阶段时执行类构造器`<clinit>()`方法的过程。类构造器<clinit>()方法是由编译器自动收集类中所有==静态变量的赋值动作和静态语句块(static块)中的语句合并产生的==
>    >
>    > - 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化
>    >
>    > - 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确===加锁==和==同步==
>    >
>    >   ```java
>    >   package TestJVM;
>    >   
>    >   public class Demo01 {
>    >   
>    >   	public static void main(String[] args) {
>    >   		A a = new A();
>    >   		System.out.println(A.width);
>    >   	}
>    >   }
>    >   
>    >   class A{
>    >   	public static int width = 100;
>    >   	
>    >   	static {
>    >   		System.out.println("静态初始化类A");
>    >   		width = 300;
>    >   	}
>    >   	
>    >   	public A(){
>    >   		System.out.println("创建A类的对象");
>    >   	}
>    >   	
>    >   }
>    >   
>    >   //结果：
>    >   	静态初始化类A//先初始化类变量
>    >   	创建A类的对象
>    >   	300//说明静态变量和静态块由上到下合并
>    >   
>    >   
>    >   ```



## 类加载过程示意图

> ![](C:\Users\橙汁儿Drk\Desktop\java资料及其他学习资料\达内java资料\java笔记\java自学笔记\注解、反射、字节码、类加载机制\类加载过程.png)

## 引申：类执行顺序

> - 父类静态变量、静态块
> - 父类非静态代码块
> - 父类无参构造
> - 子类静态变量、静态块
> - 子类非静态代码块
> - 子类的无参构造
> - 子类中被调用的重写父类的方法



# 初始化时机

> 代码：
>
> ```java
> package TestJVM;
> 
> public class Demo01 {
> 	
> 	static {
> 		System.out.println("静态初始化Demo01");
> 	}
> 
> 	public static void main(String[] args) {
> 		System.out.println("Demo01的mian方法");
> 		A a = new A();
> 		System.out.println(A.width);
> 		
> 		A a2 = new A();
> 	}
> }
> 
> class A extends A_Father{
> 	public static int width = 100;//静态变量，静态域 field
> 	
> 	static {
> 		System.out.println("静态初始化类A");
> 		width = 300;
> 
> 	}
> 	
> 	public A(){
> 		System.out.println("创建A类的对象");
> 	}
> 	
> }
> 
> class A_Father{
> 	static {
> 		System.out.println("静态初始化A_Father");
> 	}
> }
> 
> ```
>
> 结果：
>
> ```java
> 静态初始化Demo01
> Demo01的mian方法
> 静态初始化A_Father
> 静态初始化类A
> 创建A类的对象
> 300
> 
> 创建A类的对象//对象a2的输出结果
> ```
>
> 
>
> - 上述代码执行顺序：
>   - 加载Demo01类
>   - 加载Demo01静态代码
>   - 加载A父类的静态代码
>   - 加载A的静态代码
>   - new A();加载A的构造方法
>   - 加载main方法中的输出语句

## 类的主动引用（一定会发生类的初始化）

> - new一个类的对象：`A a = new A();`
> - 调用类的静态成员(除了final常量)和静态方法：`System.out.println(A.width);`
> - 使用反射调用：`Class.forName("TestJVM.A");`
> - 当虚拟机启动，java Hello ,则一定会初始化Hello类，说白了就是先启动main方法所在的类
> - 当初始化一个类，如果其父类没有被初始化，则会先初始化他的父类
>
> ```java
> public class Demo01 {
> 	
> 	static {
> 		System.out.println("静态初始化Demo01");
> 	}
> 
> 	public static void main(String[] args) throws Exception {
> 		System.out.println("Demo01的mian方法");
>         
> 		//主动引用
> 		//A a = new A();
> 
> 		//System.out.println(A.width);
> 		//Class.forName("TestJVM.A");
> 	}
> }
> ```



## 类的被动引用（不会发生类的初始化）

> - **当访问一个静态域时，只有真正声明这个域的类才会被初始化**
>
>   ==通过子类引用父类的静态变量，不会导致子类初始化==
>
>   ```java
>   public class Demo01 {
>   	
>   	static {
>   		System.out.println("静态初始化Demo01");
>   	}
>   
>   	public static void main(String[] args) throws Exception {
>   		System.out.println("Demo01的mian方法");
>   		
>   		//被动引用
>   		System.out.println(B.width);
>   	}
>   }
>   
>   class B extends A{
>   	static {
>   		System.out.println("静态初始化B");
>   	}
>   }
>   
>   class A extends A_Father{
>   	public static int width = 100;//静态变量，静态域 field
>   	
>   	static {
>   		System.out.println("静态初始化类A");
>   		width = 300;
>   	}
>   }
>   class A_Father{
>   	static {
>   		System.out.println("静态初始化A_Father");
>   	}
>   	
>   }
>   
>   //结果：------------------------------------------
>   静态初始化Demo01
>   Demo01的mian方法
>   
>   静态初始化A_Father//没有初始化B的静态块
>   静态初始化类A
>   300
>   ```
>
> - **通过数组定义类引用，不会触发此类的初始化**
>
>   ```java
>   public class Demo01 {
>   	
>   	static {
>   		System.out.println("静态初始化Demo01");
>   	}
>   
>   	public static void main(String[] args) throws Exception {
>   		System.out.println("Demo01的mian方法");
>   
>   		//被动引用
>   		A[] a = new A[10];
>   	}
>   }
>   
>   
>   class A {
>   	public static int width = 100;//静态变量，静态域 field
>   	
>   	static {
>   		System.out.println("静态初始化类A");
>   		width = 300;
>   	}
>   }
>   //结果：---------------------------------------
>   静态初始化Demo01
>   Demo01的mian方法
>   ```
>
> - **引用常量不会触发此类的初始化(==常量在编译阶段就存入调用类的常量池中了==)**
>
>   ```java
>   public class Demo01 {
>   	
>   	static {
>   		System.out.println("静态初始化Demo01");
>   	}
>   
>   	public static void main(String[] args) throws Exception {
>   		System.out.println("Demo01的mian方法");
>   		
>   		//被动引用
>   		System.out.println(A.heigth);
>   	}
>   }
>   
>   class A{
>   	public static final int heigth = 200;//静态变量，静态域 field
>   	
>   	static {
>   		System.out.println("静态初始化类A");
>   	}
>   	
>   	public A(){
>   		System.out.println("创建A类的对象");
>   	}
>   	
>   }
>   //结果：--------------------------------------
>   静态初始化Demo01
>   Demo01的mian方法
>   200//没有初始化A类，直接在常量池中引用常量
>   
>   ```



# 深入类加载器

> - 类加载器的作用
>   - 将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区中的运行时数据结构，在堆中生成一个代表这个类的Class对象，作为方法区类数据的访问入口
> - 类缓存
>   - 标准的Java SE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过，JVM垃圾收集器可以回收这些Class对象



## 类加载器的层次结构（树状结构）

> <!--知道加载器的关系就可以-->
>
> - 引导类加载器（bootstrap class loader）
>
>   - 它用来加载Java的核心库（JAVA_HOME/jre/rt.jar或sun.boot.class.path路径下的内容），是用C语言实现的，并不继承自java.lang.ClassLoader
>   - ==扩展类加载器==和==应用程序类加载器==是==该加载器的子类==
>
> - 扩展类加载器（extensions class loader）
>
>   - 用来加载Java的扩展库（JAVA_HOME/jre/*.jar或java.ext.dirs路径下的内容）。
>   - Java虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载Java类
>   - 由sun.misc.Launcher$ExtClassLoader实现
>
> - 应用程序类加载器（application class loader）
>
>   - 它根据Java应用的类路径（classpath,java.class.path路径下的内容）来加载Java类。
>
>     - 知道系统里java.class.path指的是哪个path
>
>       ```java
>       System.out.println(System.getProperty("java.class.path"));
>       //结果-------------------------------
>       C:\eclipse\.workspace\Learn_self\bin
>       ```
>
>       
>
>   - 一般来说，Java应用的类都是由它来完成的加载的。
>
>   - 由sun.misc.Launcher$AppClassLoader实现
>
> - 自定义类加载器
>
>   - 开发人员可以通过继承java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求
>
> 代码说明：
>
> ```java
> public class Demo02 {
> 	
> 	public static void main(String[] args) {
> 		System.out.println(ClassLoader.getSystemClassLoader());//应用程序类加载器
> 		System.out.println(ClassLoader.getSystemClassLoader().getParent());//应用程序类加载器的父类：扩展类加载器
> 		System.out.println(ClassLoader.getSystemClassLoader().getParent().getParent());//扩展类加载器的父类：引导类加载器
> 	}
> }
> 
> //结果：--------------------------------------
> sun.misc.Launcher$AppClassLoader@73d16e93
> sun.misc.Launcher$ExtClassLoader@15db9742
> null//因为 引导类加载器是由c语言写的，所以Java获取不到
> 
> ```
>
> 

![1567238940950](C:\Users\橙汁儿Drk\Desktop\java资料及其他学习资料\达内java资料\java笔记\java自学笔记\注解、反射、字节码、类加载机制\类加载器的层次结构.png)



## ClassLoader类

> - 作用
>
>   - Class Loader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节码代码，然后==从这些字节码代码中定义出一个Java类==，==即java.lang.Class类的一个实例==
>   - 除此之外，Class Loader还负责加载Java应用所需的资源，如图像文件和配置文件等
>
> - 相关方法
>
>   | **方法**                                            | **说明**                                                     |
>   | --------------------------------------------------- | ------------------------------------------------------------ |
>   | getParent()                                         | 返回该类加载器的父类加载器                                   |
>   | loadClass(String name)                              | ==加载==名称为name的类，返回的结果是Class类的实例（Class对象） |
>   | findClass(String name)                              | ==查找==名称为name的类，返回的结果是Class类的实例（Class对象） |
>   | findLoadedClass(String name)                        | 查找名称为name的已经被加载过的类，返回的结果是Class类的实例（Class对象） |
>   | defineClass(String name,byte[] b, int off, int len) | 把字节数组b中的内容转换成Java类，返回的结果是Class类的实例。这个方法被声明为final的 |
>   | resolveClass(Class<?> c)                            | 链接指定的Java类                                             |



## 类加载器的代理模式

> - 代理模式
>   - 交给其他加载器来加载指定的类
> - 双亲委托机制
>   - 某个特定的类加载器在接收到加载类的请求时，首先将加载任务委托给父类加载器，依次追溯，直到最高的引导类加载器。如果引导类加载器可以加载，即加载；无法加载，以此回溯，直到能加载
>   - ==双亲委托机制是为了保证Java核心库的类型安全==
>     - 这种机制就保证了不会出现用户自己能定义java.lang.Object类的情况
>   - 类加载器除了用于加载类，也是安全的最基本的保障
>   - 双亲委托机制是代理模式的一种
>     - 并不是所有的类加载器都采用双亲委托机制
>     - ==tomcat服务器类加载器也是用代理模式==，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这==与一般类加载器的顺序是相反的==



# 自定义类加载器

## 自定义类加载器的流程

> - 首先检查请求的类型是否已经被这个类装载器装载到命名空间中了，如果已经装在，直接返回
> - 双亲委托机制给父类加载器，如果父类加载器能够完成，则返回父类加载器加载的Class实例
> - 调用本类加载器的findClass(……)方法，试图获取对应的字节码
>   - 如果获取的到，则调用defineClass(……)导入类型到方法区；
>   - 如果获取不到对应的字节码或者其他原因失败，返回异常给loadClass(……)，loadClass()转抛异常，终止加载过程
> - 注意：==被两个类加载器加载的同一个类，JVM不认为是相同的类==



## 自定义类加载器

> ```java
> package TestJVM;
> 
> import java.io.ByteArrayOutputStream;
> import java.io.FileInputStream;
> import java.io.IOException;
> import java.io.InputStream;
> 
> /**
>  * 自定义文件系统加载器
>  * @author OrangeDrk
>  *
>  */
> public class FileSystemClassLoader extends ClassLoader {
> 
> 	//TestJVM.User	---> d:/MyJava/TestJVM/User.class
> 	private String rootDir;
> 	
> 	public FileSystemClassLoader(String rootDir) {
> 		this.rootDir = rootDir;//初始化路径
> 	}
> 	
> 	@Override
> 	protected Class<?> findClass(String name) throws ClassNotFoundException {
> 	
> 		Class<?> c = findLoadedClass(name);//查询是否已被加载过
> 		
> 		//应该要先查询有没有加载过这个类。如果已经加载，则直接返回加载好的类。如果没有，则加载新的类
> 		if(c!=null) {
> 			return c;
> 		}else {
> 			ClassLoader parent =  this.getParent();
> 			try {
> 				c = parent.loadClass(name);//委托给父类加载
> 			} catch (Exception e) {
> 				//e.printStackTrace();
> 			}
> 			if (c != null) {
> 				return c;
> 			}else {
> 				byte[] classData = getClassData(name);
> 				if(classData == null) {
> 					throw new ClassNotFoundException();
> 				}else {
> 					c = defineClass(name, classData, 0,classData.length);
> 				}
> 			}
> 		}
> 		
> 		return c;
> 		
> 	}
> 	
> 	private byte[] getClassData(String classname) {
> 		//TestJVM.User	---> d:/MyJava/TestJVM/User.class
> 		String path = rootDir + "/"+ classname.replace('.', '/')+".class";
> 		
> 		//IOUtils,可以使用它将流中数据转换成字节数组
> 		InputStream  is = null;
> 		ByteArrayOutputStream baos = new ByteArrayOutputStream();
> 		try {
> 			is = new FileInputStream(path);
> 			
> 			byte[] buffer = new byte[1024];
> 			int temp = 0;
> 			while((temp = is.read(buffer))!=-1) {
> 				baos.write(buffer,0,temp);
> 			}
> 			
> 			return baos.toByteArray();
> 		} catch (Exception e) {
> 			e.printStackTrace();
> 			return null;
> 		}finally {//关闭两个IO流
> 			try {
> 				if(is!=null) {
> 					is.close();
> 				}
> 				
> 				if (baos != null) {
> 					baos.close();
> 				}
> 			} catch (IOException e) {
> 				e.printStackTrace();
> 			}
> 		}
> 		
> 	}
> 	
> }
> 
> ```
>
> 
>
> - 测试及结果
>
>   ```java
>   1442407170
>   1442407170//同一个类加载器加在同一个了类，JVM视为同一个Class
>       
>   1028566121//不同的类加载器加载同一个类，JVM视为不同的Class
>   
>   TestJVM.FileSystemClassLoader@5c647e05
>   null
>   sun.misc.Launcher$AppClassLoader@73d16e93
>   ```
>
>   

