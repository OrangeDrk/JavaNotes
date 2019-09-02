[TOC]

# File方法介绍

## 创建File对象

1. File(String pathname)

   ```java
   public static void test1() {
   		//File(String pathname)
   		File file = new File("D:\\a.txt");
   		System.out.println(file);
   	}
   ```

   

2. File(File parent, String child) 

   ```java
   public static void test2() {
   		//File(File parent, String child) 
   		File fileParent = new File("D:\\a\\b");//表示路径
   		File file = new File(fileParent,"a.txt");//文件名
   		System.out.println(file);
   	}
   ```

   

3. File(String parent, String child) 

   ```java
   public static void test3() {
   		//File(String parent, String child) 
   		File file = new File("D:\\a\\b","a.txt");
   		System.out.println(file);
   	}
   ```

   

## 判断文件/路径是否存在

> boolean : exists()  

```java
public static void test1() {
		File file = new File("E:\\a");
		//判断文件/路径是否存在
		boolean b = file.exists();
		
	}
```



## 创建单个/多个文件夹

> boolean : mkdir():创建单个文件夹
>
> boolean : mkdirs():创建多个文件夹

```java
	/**
	 * exists():判断文件/路径是否存在
	 * mkdir():创建单个文件夹
	 * mkdirs():创建多个文件夹
	 */
public static void test1() {
		File file = new File("E:\\a");
		//判断文件/路径是否存在
		boolean b = file.exists();
		if(!b) {
			//若文件夹不存在，创建单个文件夹
			boolean b2 = file.mkdir();
			System.out.println(b2);
		}
	}
	
	public static void test2() {
		File file = new File("E:\\day04\\a\\b");
		boolean b = file.exists();
		if(!b) {
			boolean b2 = file.mkdirs();
			System.out.println(b2);
		}
	}
```

## 删除文件夹/文件

> boolean : delete()

```java
	/**
	 * 删除文件夹
	 */
	public static void test3() {
		File file = new File("E:\\a");
		//存在
		if(file.exists()) {
			boolean b = file.delete();
			System.out.println(b);
		}
	}
```

## 创建文件

> boolean : createNewFile()
>
> ==前提必须保证文件夹存在==

```java
	/**
	 * createNewFile():创建文件
	 * @throws IOException 
	 */
	public static void test4() throws IOException {
		File file = new File("E:\\a.txt");
		//判断文件是否存在
		if(!file.exists()) {
			//创建文件的方法
			boolean b = file.createNewFile();
			System.out.println(b);
		}
	}
	
```

### 在当前工程下创建文件(通常使用这种方式)

> File file = new File("a.txt");//默认在当前工程下
>
> 直接写文件名，就默认在当前工程下

```java
	/**
	 * 在工程下创建文件
	 * 通常使用这种方式
	 * @throws IOException 
	 */
	public static void test6() throws IOException{
		File file = new File("a.txt");//默认在当前工程下
		if(!file.exists()) {
			file.createNewFile();
		}
		
	}
```



## 获取当前文件夹下的所有目录和文件的信息

> listFiles()
>
> 获取当前文件夹下的所有目录和文件的信息

```java
	/**
	 * listFiles():获取当前文件夹下的所有目录和文件的信息
	 */
	public static void test7() {
		File file = new File("E:\\");
		File[] files = file.listFiles();
		for(File f:files) {
			System.out.println(f);
		}
	}
```

|                             结果                             |
| :----------------------------------------------------------: |
| E:\$RECYCLE.BIN<br/>E:\360downloads<br/>E:\BaiduNetdiskDownload<br/>E:\day04<br/>E:\leigodAPapers<br/>E:\leigodDownLoad<br/>E:\others<br/>E:\qycache<br/>E:\System Volume Information<br/>E:\迅雷下载 |



## 获取当前文件夹下的所有目录和文件的名字

> list()
>
> 获取当前文件夹下的所有目录和文件的名字

```java
	/**
	 * list():获取当前文件夹下的所有目录和文件的名字
	 */
	public static void test8() {
		File file = new File("E:\\");
		String[] files = file.list();
		for(String name:files) {
			System.out.println(name);
		}
	}
```

|                             结果                             |
| :----------------------------------------------------------: |
| $RECYCLE.BIN<br/>360downloads<br/>BaiduNetdiskDownload<br/>day04<br/>leigodAPapers<br/>leigodDownLoad<br/>others<br/>qycache<br/>System Volume Information<br/>迅雷下载 |



# IO流

## IO

### input：输入流

> ==以应用程序为核心==
>
> 文件------>应用程序

### output：输出流

> ==以应用程序为核心==
>
> 应用程序----->文件



## 字节流

> Stream结尾的类
>
> `InputStream`是所有输入流的父类（抽象类）
>
> 核心方法：`read（）`



> `OutputStream`是所有输出流的父类（抽象类）
>
> 核心方法：`write（）`



### 输出流

#### 使用文件的输出流写数据到文件

> write(int b) : 将指定的字节写入此文件
>
> write(byte[] b) : 将b.length个字节从指定字节数组写入此文件
>
> byte[] buffer = getBytes() ：把字符串转换成byte类型的数组
>
> ==FileOutputStream(File file)==
>
> ==FileOutputStream(String name)==

```java
	//使用文件的输出流写数据到文件
	public static void test1() throws IOException {
		//1.确定流:FileOutputStream()
		//如果是输出流，文件不存在，会自动创建文件
		FileOutputStream output = new FileOutputStream("a.txt");
		
		//2.选择合适的方法，处理数据
		String str = "Hello,Students!";
		byte[] buffer = str.getBytes();//把字符串转换成byte类型的数组---getBytes()
		
		//把数组中的数据写到数据文件
		output.write(buffer);
		
		//关闭流
		output.close();
	}
```

> 简洁写法

```java
	//使用文件的输出流写数据到文件
	public static void test1() throws IOException {
		//1.确定流:FileOutputStream()
		//如果是输出流，文件不存在，会自动创建文件
		FileOutputStream output = new FileOutputStream("a.txt");
		//2.选择合适的方法，处理数据
		//把数组中的数据写到数据文件
		output.write("hello,students!".getBytes());//简单方法
		//关闭流
		output.close();
	}
```



### 输入流

#### 把数据文件中的数据读到应用中

> read()方法读到数据末尾会返回-1
>
> 将n的值强转为char，输出字符
>
> ==FileInputStream(File file)==
>
> ==FileInputStream(String name)==

> 方法1：read()

```java
	//把数据文件中的数据读到应用中
	public static void test1() throws IOException {
		File file = new File("a.txt");
		//选择文件流
		FileInputStream inputStream = new FileInputStream(file);
		if(file.exists()) {
			//选择处理方法，处理数据
			int n = 0;
			//一个字节一个字节的读取
			while((n = inputStream.read())!=-1) {
				//在一行输出数据
				System.out.print((char)n);
			}
		}
	}
```



> 方法2：read(byte[] b)  
>
> 可读汉字

```java
	public static void test2() throws IOException {
		FileInputStream input = new FileInputStream("a.txt");
		byte[] buffer = new byte[1024];
		int n = 0;
		//n表示读到数组中的元素个数
		n = input.read(buffer);
		
		String str = new String(buffer,0,n);
		System.out.println(str);
	}
```



> 方法3：file.length()、read(byte[] b, int off, int len) 
>
> 可读汉字
>
> 最佳用法

```java
	public static void test3() throws IOException {
		File file = new File("a.txt");
		FileInputStream input = new FileInputStream(file);
		
		//文件的长度
		byte[] b = new byte[(int)file.length()];//创建数组只能放int长度
		input.read(b);//或者：input.read从(b,0,(int)file.length())
		String str = new String(b);
		System.out.println(str);
	}
```



#### 向文本中追加数据

> FileOutputStream(String name,  boolean append)

```java
	public static void test1() throws IOException {
		//如果构造方法的第二个参数为true，表示追加文本数据
		//默认值为false，表示覆盖
		FileOutputStream output = new FileOutputStream("a.txt",true);
		output.write("Hello\n".getBytes());//自己换行
		output.close();
	}
```



## 字符流



### 输出流

> `FileWriter()`,第二个参数为true，会在原有数据上追加数据
>
> `write(String str)`

```java
	//字符流输出:写
	public static void test1() throws IOException{
        //输出会自动创建文件
		FileWriter fw = new FileWriter("a2.txt");
		fw.write("Hello");
		fw.close();
		
	}
```



### 输入流

> `FileReader()`
>
> `read(char[] arr)`

```java
	//字符流输入：读
	public static void test2() throws IOException {
        //读取不会自动创建文件
		File file = new File("a2.txt");
		FileReader fr = new FileReader(file);
		
		char[] data =new char[(int)file.length()];
		fr.read(data);
		String str = new String(data);
		System.out.println(str);
	}
```



### 练习：把a2.txt数据复制到a3.txt

```java
	//复制到a3.txt
	public static void test3() throws IOException {
		File file = new File("a2.txt");
		FileReader fr = new FileReader(file);
		
		char[] data =new char[(int)file.length()];
		fr.read(data);
		String str = new String(data);
		FileWriter fw = new FileWriter("a3.txt");
		fw.write(str);
		fw.close();
		fr.close();
		
	}
```





# 对象的输入输出


> 序列化和反序列化，只要继承Serializable接口就可以
>
> - 以对象写数据的时候，要提供写数据的规则，按规则写数据到文件，叫序列化
>
>  * 读对象数据的时候，按照写数据的规则读取，叫反序列化
>
>  ==标识一个类是序列化的类，实现Serializable接口==
>
> 类的序列化由实现java.io.Serializable接口的类启用。 
>
> - 不实现此接口的类将不会使任何状态序列化或反序列化。
>
> - 可序列化类的所有子类型都是可序列化的。
>
> - 序列化接口没有方法或字段，仅用于标识可串行化的语义。 
>
>  



## 输出：写

> Student类

```java
package demoObjectIO;

import java.io.Serializable;

/**
 * 序列化和反序列化
 * 	以对象写数据的时候，要提供写数据的规则，按规则写数据到文件，叫序列化
 * 	读对象数据的时候，按照写数据的规则读取，叫反序列化
 * 
 * 标识一个类是序列化的类，实现Serializable接口
 * 类的序列化由实现java.io.Serializable接口的类启用。 
 * 		 不实现此接口的类将不会使任何状态序列化或反序列化。
 * 		 可序列化类的所有子类型都是可序列化的。
		 序列化接口没有方法或字段，仅用于标识可串行化的语义。 
 * @author dell
 *
 */

public class Student implements Serializable{
	/**
	 * 
	 */
	private static final long serialVersionUID = -8215237806434974738L;
	private String no;
	private String name;
	private int age;
	
	public Student() {
		// TODO Auto-generated constructor stub
	}
	
	
	public Student(String no, String name, int age) {
		super();
		this.no = no;
		this.name = name;
		this.age = age;
	}


	public String getNo() {
		return no;
	}
	public void setNo(String no) {
		this.no = no;
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


	@Override
	public String toString() {
		return "Student [no=" + no + ", name=" + name + ", age=" + age + "]";
	}
	
	
	
	
}

```



> 输出：
>
> ==ObjectOutputStream(OutputStream out)==
>
> `writeObject(Object obj)` 
>
> - 对象输出不能向数据文件末尾累加数据，需要自己重新定义一个新的类来实现

```java
	//把对象写到数据文件
	public static void test1() throws IOException {
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("a1.txt"));
		
		Student stu = new Student("0211160314","sxh",23);
		
		oos.writeObject(stu);
		oos.close();
	}
```



## 输入：读

> ==ObjectInputStream(InputStream in)==
>
> `Object ：readObject()`  

```java
	public static void test2() throws Exception{
		ObjectInputStream oos = new ObjectInputStream(new FileInputStream("a1.txt"));
		
		Student stu = new Student("0211160314","sxh",23);
		
		Student stu1 = (Student) oos.readObject();
		System.out.println(stu1.toString());
		oos.close();
	}
```



> 读有多个对象的数据

```java
	//读多个对象，当出现异常是，说明读取 完毕
	public static void test2() throws IOException{
		//确定流
		ObjectInputStream input = new ObjectInputStream(new FileInputStream("a1.txt"));
		//选择方法，处理数据
		Object obj = null;
		try {
			while((obj=(input.readObject()))!=null) {
				Student stu = (Student)obj;
				System.out.println(stu);
			}
		} catch (Exception e) {
			System.out.println("读取完毕");
		}
		input.close();
	}

```



# ==缓冲流==

> ==读取速度最快，效率高，操作字符串方便==
>
> BufferedReader : readLine();------读数据，读一行
>
> PrintWriter : println();------自动换行



## 使用缓冲流PrintWriter写数据到数据文件

> PrintWriter的构造方法：
>
> - PrintWriter(File file) 
> - PrintWriter(OutputStream out) 
> - PrintWriter(String fileName) 
> - PrintWriter(Writer out) 
>
> println()

```java
	//使用缓冲流PrintWriter写数据到数据文件
	public static void test1() throws IOException {
		PrintWriter pw = new PrintWriter(new FileWriter("a.txt",true));
		String str = "admin"+"&"+"123456";
		pw.println(str);
		pw.close();
	}
```



## 使用缓冲流BufferedReader读数据文件到应用程序

> BufferedReader的构造方法
>
> - BufferedReader(Reader in)
>
> readLine()		
>
> ​	最后返回null

```java
	//使用缓冲流BufferedReader读数据文件到应用程序
	//BufferedReader(Reader in) 
	//readLine() 读取一行数据
		public static void test2() throws IOException {
			File file = new File("a.txt");
			BufferedReader br = new BufferedReader(new FileReader(file));
			if(file.exists()) {
				String str = "";
				while((str = br.readLine())!=null) {
					System.out.println(str);
				}
			}
			br.close();
		}
```



# 字节流和字符流之间的转换

1. `InputStreamReader`------输入把字节流转换成字符流

   - 要想追加数据，==应该在FileOutputStream后的第二个参数，加true==

   ```java
   public static void test() throws IOException {
   		OutputStreamWriter ow = new 
               OutputStreamWriter(new FileOutputStream("a.txt",true));
   		PrintWriter pr = new PrintWriter(ow);
   		String str = "hello!";
   		pr.println(str);
   		pr.close();
   		
   	}
   ```

   

2. `OutputStreamWriter`------输出把字节流转换成字符流

   ```java
   	/**
   	 * 使用缓冲流读取数据
   	 * @throws IOException
   	 */
   	public static void test2() throws IOException {
   		File file = new File("a.txt");
   		InputStreamReader or = new InputStreamReader(new FileInputStream(file));
   		BufferedReader br = new BufferedReader(or);
   		String str = "";
   		while((str = br.readLine())!= null) {
   			System.out.println(str);
   		}
   	}
   ```

   

# 既是输入流也是输出流

> PrintAccess****