[TOC]





# File介绍

> File代表文件或者文件夹（目录）的类

## 创建File对象

> 创建对象过程中不会检测路径是否存在；只是把路径名放到对象身上

1. `File(String pathname)`

   ```java
   public static void test1() {
   		//File(String pathname)
   		File file = new File("D:\\a.txt");
   		System.out.println(file);
   	}
   ```

   

2. `File(File parent, String child)` 

   ```java
   public static void test2() {
   		//File(File parent, String child) 
   		File fileParent = new File("D:\\a\\b");//表示路径
   		File file = new File(fileParent,"a.txt");//文件名
   		System.out.println(file);
   	}
   ```

   

3. `File(String parent, String child)` 

   ```java
   public static void test3() {
   		//File(String parent, String child) 
   		File file = new File("D:\\a\\b","a.txt");
   		System.out.println(file);
   	}
   ```

   

## 判断文件/路径是否存在

> `boolean : exists()`  

```java
public static void test1() {
		File file = new File("E:\\a");
		//判断文件/路径是否存在
		boolean b = file.exists();
		
	}
```



## 判断是文件还是文件夹

> `isFile()`：是文件，返回true
>
> `isDirectory()`：是目录，返回true

```java
public static void test2(){
    File file = new File("D:/a");
    File[] files = file.listFiles();
    for (File f : files) {
        if (f.isFile()) {
            System.out.println("文件");
        }else if(f.isDirectory()){
            System.out.println("目录");
        }
        System.out.println(f);
    }
}
```



## 创建单个/多个文件夹

> boolean : `mkdir()`:创建单个文件夹
>
> boolean : `mkdirs()`:创建多个文件夹

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

> boolean : `delete()`
>
> **只能删除空目录**
>
> **文件都可以删**
>
> 所有删除是彻底删除，不会放到回收站中

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



## getName()

> 返回文件名及后缀

```java
File file = new File("D:/a.txt");
System.out.println(file.getName());//a.txt
```



## getParent()

> 返回文件所在目录

```java
File file = new File("D:/a.txt");
System.out.println(file.getParent());//D:\
```



## lastModified()

> 返回文件最后修改时间



## setLastModified

> 设置文件修改时间
>
> 设置成功返回true

```java
boolean b = file.setLastModified(53446876445L);
```



## 创建文件

> boolean : `createNewFile()`
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



## 重命名-剪切操作

> `boolean:renameTo()`
>
> 传入文件类型的新对象，指定路径----指定新名称
>
> ==重命名是根据底层的剪切完成的==

```java
public static void test6(){
    File file = new File("D:/a.txt");
    //重命名
    boolean b = file.renameTo(new File("D:/b.txt"));
    //剪切并重命名---剪切到D:/a路径下并重命名为b.txt
    boolean b = file.renameTo(new File("D:/a/b.txt"));
    System.out.println(b);
}
```





## 在当前工程下创建文件(通常使用这种方式)

> `File file = new File("a.txt");`//默认在当前工程下
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



## 获取文件夹下的所有信息

> `listFiles()`
>
> `File[]:listFiles(FileFilter filter)`：过滤文件类型
>
> `File[]:listFiles(FilenameFilter filter)` ：过滤文件名
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

//结果
E:\$RECYCLE.BIN
E:\360downloads
E:\BaiduNetdiskDownload
E:\day04
E:\leigodAPapers
E:\leigodDownLoad
```





### 过滤出文件名称包含数字的信息

#### 方法一：

```java
public static void test4(){
    File file = new File("D:/");
    //展示目录下所有包含数字的信息
    
    //把过滤之后的内容放在数组中
    //--------------------------匿名类--------------------------------
    File[] files = file.listFiles(new FileFilter() {
        //重写抽象方法，指定过滤规则
        @Override
        public boolean accept(File pathname) {
            return pathname.getName().matches(".*[0-9]+.*");
        }
    });
    
    //--------------------------λ表达式--------------------------------
    File[] files = file.listFiles(str -> str.getName().matches(".*[0-9]+.*"));
    for (File f : files) {
        System.out.println(f);
    }

}
```

#### 方法二：

```java
public static void test5(){
    File file = new File("D:/");
    //展示目录下所有包含数字的信息
    //把过滤之后的内容放在数组中
    
    //--------------------------匿名类--------------------------------
    File[] files = file.listFiles(new FilenameFilter() {
            //指定过滤规则，重构抽象方法
            @Override
            public boolean accept(File dir, String name) {
                return name.matches(".*\\d.*");
            }
        });

    //--------------------------λ表达式--------------------------------
    File[] files = file.listFiles((str, name) -> name.matches(".*\\d.*"));
    for (File f : files) {
        System.out.println(f);
    }
}
```



## 获取文件夹下所有名字

> `list()`
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
//结果
$RECYCLE.BIN
360downloads
BaiduNetdiskDownload
day04
leigodAPapers
leigodDownLoad
others
qycache
System Volume Information
迅雷下载
```





