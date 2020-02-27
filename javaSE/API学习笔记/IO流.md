[TOC]



# IO流

> 用于数据传输的机制

Input/Output  Stream   ：输入输出流

- InputStream：输入流
  - 是所有输入流的父类（抽象类）
  - 核心方法：`read()`
- OutputStream：输出流
  - 是所有输出流的父类（抽象类）
  - 核心方法：`write()`

**根据传输方向：**

- 输入流：向内存中传输数据
- 输出流：向硬盘中传输数据

**根据传输方式：**

- 字符流：底层以字符型式进行传输
- 字节流：底层以字节形式进行传输

**四大基本流：**

> 四大基本流都为抽象类
>
> 通过实现类去使用四大基本流

|            | 输入流                             | 输出流                             |
| ---------- | ---------------------------------- | ---------------------------------- |
| 字符流     | 字符输入流`Reader`                 | 字符输出流`Writer`                 |
| **实现类** | 文件字符输入流<br />`FileReader()` | 文件字符输出流<br />`FileWriter()` |
|            |                                    |                                    |
| 字节流     | 字节输入流`InputStream`            | 字节输出流`OutputStream`           |
| **实现类** | `FileInputStream()`                | `FileOutputStream()`               |

**根据传输的场景（数据存储/获取的位置）**

1. 硬盘
2. 内存
3. 网络
4. 外设设备

**flush()方法**

- OutputStream、Writer的子类都有该方法
- OutputStream中的`BufferedOutputStream`和`ObjectOutputStream`的`flush()`不是空方法
- Writer的直接子类一般都重写了`flush()`方法





# 字节流

> 底层以字节形式进行传输



## 字节输出流

> `FileOutputStream()`
>
> 没有缓冲区

### 构造方法

- `FileOutputStream(File file)` 
- `FileOutputStream(File file, boolean append)` ：==true可追加==
- `FileOutputStream(String name)` 
- `FileOutputStream(String name, boolean append)` ：==true可追加==

### 方法

#### `write()`写数据到文件

重载方法：

- `write(int b)` : 将指定的字节写入此文件
- `write(byte[] b)` : 将b.length个字节从指定字节数组写入此文件

`byte[] buffer = getBytes()` ：把字符串转换成byte类型的数组

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
>
> 编码问题通过getBytes()参数设置`output.write("hello,students!中".getBytes("UTF-8"));`

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



## 字节输入流

### 构造方法

- `FileInputStream(File file)`
- `FileInputStream(String name)`

### 方法

#### read()

> read()方法读到数据末尾会返回-1
>
> 将n的值强转为char，输出字符

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



#### `read(byte[] b)`  

> 可读汉字
>
> read(bytes)  返回每次读取到的字节个数

```java
public static void test7() throws IOException {
    File file = new File("D:/a.txt");
    FileInputStream inputStream = new FileInputStream(file);
    if(file.exists()) {
        byte[] bytes = new byte[5];
        int len = -1;
        //read(bytes)  返回每次读取到的字节个数
        while((len = inputStream.read(bytes))!=-1) {
            System.out.print(new String(bytes,0,len));
        }
    }
}
```



#### 综合

> `file.length()`、`read(byte[] b, int off, int len)` 
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



### 向文本中追加数据

> `FileOutputStream(String name,  boolean append)`

```java
public static void test1() throws IOException {
    //如果构造方法的第二个参数为true，表示追加文本数据
    //默认值为false，表示覆盖
    FileOutputStream output = new FileOutputStream("a.txt",true);
    output.write("Hello\n".getBytes());//自己换行
    output.close();
}
```



# 字符流

> 底层以字符形式进行传输
>

## 字符输出流

> `FileWriter()`

### 构造方法

> 使用一个参数的构造：
>
> - 创建对象时会检测路径是否存在，不存在会自动创建
> - 如果已经有文件存在，就会把之前的文件覆盖掉

- `FileWriter(File file)` 
- `FileWriter(File file, boolean append)` 
- `FileWriter(String fileName)` ：不存在文件会自动创建
- `FileWriter(String fileName, boolean append)`



### 方法

#### 写：`write(String str)`

> `FileWriter()`,第二个参数为true，会在原有数据上追加数据

```java
//字符流输出:写
public static void test1() throws IOException{
    //输出会自动创建文件
    FileWriter fw = new FileWriter("a2.txt");
    fw.write("Hello");
    fw.flush();//冲刷缓冲器
    fw.close();//关流
    fw = null;//将字符输出流对象回收

}
```

#### 冲刷缓冲器：`flush()`

> 规定只有缓冲区存满才能把数据传输过去；如果缓冲区没有存满，数据就会滞留再缓冲区，造成数据的丢失
>
> ==无论缓冲区是否存满都强制让缓冲区传输数据==

```java
代码同上
```

#### 关流`close()`

> 在关流动作中，缓冲区有可能有滞留数据，为了防止数据滞留在缓冲区中，会自动进行一次冲刷缓冲区的操作







## 字符输入流

> `FileReader()`

### 构造方法

- `FileReader(File file)` 
- `FileReader(String fileName)` 

### 方法

#### 读`read()`

> `read()`：每次返回读取到的字符的编码值，返回-1表示读取结束

```java
int i = -1;
while ((i = fileReader.read()) != -1) {
    System.out.println(i);
}
```

#### 读：`read(char[] arr)`

> `int:read(char[] arr)`：将读取的所有内容添加到arr数组中
>
> 返回值：每次读取到的字符个数
>
> ==常用方法二==

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


//写法二:一般用此方法
public static void test5() throws IOException {
    File f = new File("D:/a.txt");
    FileReader fileReader = new FileReader(f);
    char[] cs = new char[5];
    int len = -1;
    while ((len = fileReader.read(cs)) != -1) {
        System.out.print(new String(cs,0,len));
    }
}
```



## 练习：把a2.txt数据复制到a3.txt

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





# Io流的异常捕获

- 把对象在外进行声明赋值为null，在try块中进行真正的初始化
- 只有流对象真正的初始化才能进行关流
- 无论关流成功与否，都要把流对象置为null
- 关流的失败有可能发生在自动冲刷之前，有可能造成数据滞留缓冲区，时数据丢失

## 正常流程

```java
public static void test2() {
    //声明流对象，赋予初始值为null
    FileWriter fw = null;
    try {
        //对流对象初始化
        fw = new FileWriter("D:/c.txt");
        fw.write("abcd");
        fw.flush();

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fw != null) {
            try {
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                fw = null;//无论关流是否成功，都要将对象置为null
            }
        }
    }
}
```



## try with resources

> JDK1.7提供的 try with resources
>
> 会自动进行冲刷缓冲区以及关流

```java
public static void test3(){
    //JDK1.7提供的 try with resources
    //会自动进行冲刷缓冲区以及关流
    try(FileWriter fileWriter = new FileWriter("D:/a.txt")) {
        fileWriter.write("abc");

    } catch (IOException e) {
        e.printStackTrace();
    }
}
```









# 对象的输入输出



## 序列化&反序列化

> 序列化和反序列化，只要实现`Serializable`接口就可以
>
> - 序列化：把对象以及相关信息转成字节数组
>   - 相关信息不包含方法，包含属性
>
> - 反序列化：把字节数组转回成对象
>
> 持久化：把字节数组存储在硬盘（持久化必定会发生在序列化之后）
>
> 注意：
>
> - 序列化时也会把类的属性一起序列化过去
>
> - 被`static`/`transient`关键字修饰的属性不支持序列化
>
> - `serialVersionUID`：版本号
>- 当对象即将要进行序列化时会根据当时类的属性和方法一起计算出一个版本号。版本号计算出来之后会随着对象一起序列化
>   
>- 当反序列化时会再次根据类的属性和方法计算出一个版本号，与之前序列化的版本号比较，如果相等就能正常序列化，反之会报错
>   
>- 解决办法：==自己指定版本号：==
> 
> //指定版本号
>     `private static final long serialVersionUID = 2659989858582682143L;`
>    
>- 集合和映射对象都不能被序列化，只能一一遍历集合和映射，获取每个元素对象在进行序列化
> 
>
> 
> ==标识一个类是序列化的类，实现Serializable接口==
>
> 类的序列化由实现`java.io.Serializable`接口的类启用。 
>
> - 不实现此接口的类将不会使任何状态序列化或反序列化。
>- 可序列化类的所有子类型都是可序列化的。
> - 序列化接口没有方法或字段，仅用于标识可串行化的语义。



## 输出：写

> ==ObjectOutputStream(OutputStream out)==
>
> `writeObject(Object obj)` 
>
> - 对象输出不能向数据文件末尾累加数据，需要自己重新定义一个新的类来实现

```java
//Student类
import java.io.Serializable;
public class Student implements Serializable{...}


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
> `Object : readObject()`  

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

> 给其他流提供缓冲区
>
> ==读取速度最快，效率高，操作字符串方便==
>
> 字符缓冲流：
>
> - BufferedReader : readLine();------读数据，读一行
>
> - BufferedWriter：newLine();------换行
>
> 字节缓冲流：
>
> - BufferedInputStream
> - BufferedOutputStream
>
> 字节/字符输出缓冲流：
>
> - PrintWriter : println();------自动换行
>
> 



## `PrintWriter`写数据

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



## `BufferedWriter`写数据

> 给字符输出流提供更大的缓冲区；提供换行功能
>
> `newLine()`:换行
>
> - windows换行符 ：\r\n
> - Linux换行符：\n

```java
public static void test9() throws IOException {
    BufferedWriter bw = new BufferedWriter(new FileWriter("D:/123.txt"));
    bw.write("absdw时子ask了");
    bw.newLine();
    bw.write("s科技时代发来看是否kjashdkbasd");
    bw.close();
}
```





## `BufferedReader`读数据

> ==装饰者设计模式==：根据本类对象给本类对象新增功能、完善功能
>
> BufferedReader的构造方法
>
> - BufferedReader(Reader in)
>
> readLine()		
>
> ​	最后返回null

```java
//使用缓冲流BufferedReader读指定文件的数据到内存
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



# 转换流

> ==**只有转换流才能指定编码**==

## 字节流<——>字符流

> `OutputStreamWriter`：字符流-->字节流
>
> `InputStreamReader`：字节流-->字符流

1. `InputStreamReader`------输入时：把字节流转换成字符流

   - 要想追加数据，==应该在FileOutputStream后的第二个参数，加true==

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
   
   /**
   * 不使用缓冲流读取数据
   * @throws IOException
   */
   public static void test11() throws IOException {
       //在底层真正读取数据的是字节输入流，但是数据展示时，只能被字符数组接收
       //只能通过字符流读取
       //字节流转换成了字符流
       InputStreamReader isr = new InputStreamReader(new FileInputStream("D:/a.txt"));
       char[] cs = new char[3];
       int len = -1;
       //读取到了数据---是字符形式，被字符数组接收，用字符流读
       while ((len = isr.read(cs)) != -1) {
           System.out.println(new String(cs,0,len));
       }
       isr.close();
   }
   ```

   

2. `OutputStreamWriter`------输出时：把字符流转换成字节流

   ```java
   //使用PrintWriter
   public static void test() throws IOException {
       OutputStreamWriter ow = new OutputStreamWriter(new FileOutputStream("a.txt",true));
       PrintWriter pr = new PrintWriter(ow);
       String str = "hello!";
       pr.println(str);
       pr.close();
   
   }
   
   //不使用PrintWriter
   public static void test10() throws IOException {
       //底层真正往外写出数据的是字节输出流，但是提供数据是字符串，只能由字符流承接
       //字符流转成字节流
    OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("D:/a.txt"));
       //写数据-----提供往外写的数据是字符串，只能被字符流来承接
       osw.write("abc");
       osw.close();
   }
   ```
   
   



# 系统流

> out、err、in（底层都是字节流）
>
> 系统流都是静态的，使用系统流时不要关流

```java
static void test2(){
    Scanner sc1 = new Scanner(System.in);
    int num1 = sc1.nextInt();
    System.out.println(num1);
    //sc1.close();//关流
    //系统流是静态的被所有对象共享，所以只要有一个对象关流，其他都是关流状态

    Scanner sc2 = new Scanner(System.in);
    int num2 = sc2.nextInt();
    System.out.println(num2);
    sc2.close();//关流
}
```



# 打印流`PrintStream`

> 介绍：把需要打印的信息放到输出流之中进行数据传输
>
> 打印流可以打印到控制台；也可以打印到文件
>
> PrintStream、PrintWriter

## 向文件打印

```java
static void printStreamDemo() throws FileNotFoundException {
    //创建打印流对象
    PrintStream ps = new PrintStream("D:/1.txt");
    //打印信息
    ps.print("123");
    ps.println("123456");//自带换行
    ps.println("abcd");
    ps.close();
}
```







# IO流思维导图

<img src="E:\java笔记\API学习笔记\IO流.png"  >