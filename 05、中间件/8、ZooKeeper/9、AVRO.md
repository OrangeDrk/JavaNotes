[TOC]

# AVRO概述

1. AVRO是Apache提供的一套用于进行**序列化**和**RPC**的机制
2. 市面上常见的序列化：Google Protobuffer、Apache AVRO、Facebook thrift等



# 序列化

## 序列化概述

1. **数据序列化**就是将对象或者数据结构转化成特定的格式，使其可在网络中传输，或者可存储在内存或者文件中
2. **反序列化**则是相反的操作，将对象从序列化数据中还原出来
3. **数据序列化的重点**在于数据的交换和传输 

## 衡量标准

1. 序列化之后的数据大小。

   > 因为序列化的数据要通过网络进行传输或者是存储在内存或者文件中，所以数据量越小，则存储或者传输所用的时间就越少

2. 序列化以及反序列化的耗时及占用的CPU

3. 是否能够跨语言或者平台。

   > 因为现在的企业开发中，一个项目往往会使用到不同的语言来进行架构和实现。那么在异构的网络系统中，网络双方可能使用的是不同的语言或者是不同的操作系统，例如一端使用的是Java而另一端使用的C++；或者一端使用的是Windows系统而另一端使用的是Linux系统，那么这个时候就要求序列化的数据能够在不同的语言以及不同的平台之间进行解析传输



## Java原生序列化的问题

1. Java的原生序列化不能做到对象结构的复用，就导致==序列化多个对象的时候数据量较大==
2. Java的原生序列化在使用的时候，是按照Java指定的格式将对象进行解析，解析为字节码格式，那么此时其他的语言在接收到这个对象的时候，是无法解析或者解析较为困难。即==Java的原生序列化机制是没有做到跨语言或者跨平台传递使用==



## 使用

1. 创建maven工程，导入pom依赖

   ```xml
   <dependencies>
       <!--avro依赖-->
       <dependency>
           <groupId>org.apache.avro</groupId>
           <artifactId>avro</artifactId>
           <version>1.8.2</version>
       </dependency>
       
       <!--Junit依赖-->
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.13-rc-1</version>
           <scope>compile</scope>
       </dependency>
   </dependencies>
   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.avro</groupId>
               <artifactId>avro-maven-plugin</artifactId>
               <version>1.8.2</version>
               <executions>
                   <execution>
                       <phase>generate-sources</phase>
                       <goals>
                           <goal>schema</goal>
                       </goals>
                       <configuration>
                           <sourceDirectory>${project.basedir}/src/main/avro/</sourceDirectory>
                           <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
                       </configuration>
                   </execution>
               </executions>
           </plugin>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-compiler-plugin</artifactId>
               <configuration>
                   <source>1.6</source>
                   <target>1.6</target>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

2. 在指定的目录下编辑==avsc==文件，例如：`User.avsc`

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/avro项目结构.png)

   ```json
   // 为AVRO指定输出的json格式
   {
       "namespace":"cn.tedu.serial", // namespace等价于Java中package
       "type":"record", // record等价于Java中class
       "name":"User", // class User{}
       "fields":
       [
           {"name":"username","type":"string"}, // private String username;
           {"name":"age","type":"int"}, // private int age;
           {"name":"gender","type":"string"}, // private String gender
           {"name":"height","type":"double"}, // private double height
           {"name":"weight","type":"double"} // private double weight
       ]
   }
   ```

3. 在maven管理中，运行compile

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/运行avro产生Java文件.png)

4. 序列化

   ```java
   @Test
   public void write() throws Exception{
       User u = new User("Tom",20,"女",1.7,70.0);
       User u1 = new User("Jack",20,"女",1.7,70.0);
       User u3 = new User("Halen",20,"女",1.7,70.0);
   
       //创建序列化流
       DatumWriter<User> dw = new SpecificDatumWriter<User>(User.class);
       //将序列化之后的数据写到文件中，需要闯进一个文件流
       DataFileWriter<User> dfw = new DataFileWriter<User>(dw);
   
       //指定文件，
       // dfw.create(User.getClassSchema(), new File("E:\\a.txt"));
       // dfw.create(u1.getSchema(), new File("E:\\a.txt"));
       dfw.create(User.SCHEMA$, new File("E:\\a.txt"));
   
       //序列化对象
       dfw.append(u);
       dfw.append(u1);
       dfw.append(u3);
   
       //关流
       dfw.close();
   
   }
   ```

   

5. 反序列化

   ```java
   @Test
   public void deSerial() throws IOException {
   
       //创建反序列化流
       DatumReader<User> dr = new SpecificDatumReader<User>(User.class);
   
       //创建文件流读取数据
       DataFileReader<User> dfr = new DataFileReader<User>(new File("E:\\a.txt"),dr);
   
       //读取对象,DataFileReader将读取过程封装成了迭代器来使用
       while (dfr.hasNext()) {
           User u = dfr.next();
           System.out.println(u);
       }
   
       dfr.close();
   }
   ```

   



# RPC

1. RPC(Remote Procedure Call，远程过程调用)

   > 允许程序员在一个节点上去调用另一个节点上的方法而不需要显式的实现这个方法

2. 分布式架构中，都会使用RPC

3. RPC的特点：简洁、高效、通用

![](https://gitee.com/sxhDrk/images/raw/master/imgs/RPC.png)