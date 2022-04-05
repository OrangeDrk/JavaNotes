[TOC]

# 概述

1. MapReduce是Hadoop提供的一套用于==分布式计算==的框架

2. MapReduce是Doug根据Google的论文《The Google MapReduce》阐述的原来来实现的

3. MapReduce会将计算过程拆分成2个阶段：

   - Map（映射）阶段
   - Reduce（规约）阶段

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/MapReduce2个阶段.png)

# 案例

> 字符统计
>
> 数据文件：characters.txt
>
> **统计文件中每个字符出现的次数**



## Mapper类

### 继承Mapper类

> CharCountMapper，需要继承hadoop.mapreduce.Mapper
>
> ==MapReduce中要求被传输的数据必须能够被序列化==，所以4个泛型必须是支持序列化的

 - KEYIN - 输入的键的类型。

   > 在默认情况下，表示的是读取的这一行的字节偏移量。 `LongWritable`是MapReduce针对`long`类型提供的序列化形式
   >
   > ![](https://gitee.com/sxhDrk/images/raw/master/imgs/Windows配置Hadoop1.png)

 - VALUEIN - 输入的值的类型。

   > 在默认情况下，表示的就是读取的这一行数据。`Text`是MapReduce针对`String`类型提供的序列化形式

 - KEYOUT - 输出的键的类型。

   > 当前案例是计算每一个字符出现的次数，所以输出的键应该是字符

 - VALUEOUT - 输出的值的类型。

   > 当前案例是计算每一个字符出现的次数，所以输出的值应该是次数



### 重写`map()`方法

> MapTask的处理逻辑是放在map()方法中的。每读取一行，调用一次`map()`方法。`map()`方法参数：
>
> - `key`：字节偏移量
> - `value`：读取的这一行数据
> - `context`：环境参数，作用之一是将数据从Map写到Reduce



```java
//传入4个泛型
public class CharCountMapper extends Mapper<LongWritable, Text,Text,LongWritable> {

    //MapTask处理逻辑放在map()方法中的
    //每读取一行就会调用一次map()方法
    /*
        key     -   字节偏移量
        value   -   读取的这一行数据
        context -   环境参数，作用之一是将数据从Map写到Reduce
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //value表示的是一行数据
        //统计这一行中每一个字符出现的次数
        //现将这一行中的每一个字符拆分出来
        //hello -   {h,e,l,l,o}
        char[] cs = value.toString().toCharArray();
        for (char c : cs) {
            context.write(new Text(c+""),new LongWritable(1));
        }
    }
}
```



## Reducer类

### 继承Reducer

> 需要继承hadoop.mapreduce.Reducer

需要传入4个泛型

- `KEYIN`,`VALUEIN` - 输入的键值类型  -  Reduce的数据从Map传来的
- `KEYOUT`,`VALUEOUT` - 输出的键值类型 - 计算每一个字符出现的次数，所以输出的是字符:次数

### 重写`reduce()`方法

> 计算逻辑写在`reduce()`方法中，三个参数：
>
> - key：从Map传来的key值
> - values：这个键对应的是所有的值
> - context：环境参数，此处作用是将结果最终写会到HDFS上

```java
// KEYIN,VALUEIN - 输入的键值类型  - Reduce的数据从Map来的
// KEYOUT,VALUEOUT - 输出的键值类型 - 计算每一个字符出现的次数，所以输出的是字符:次数
public class CharCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {
    // key - 输入的键 - 当前案例中表示的是字符。每一个键调用一次reduce方法
    // values - 这个键所对应的所有的值
    // context - 环境参数。此处的作用是将结果最终写回到HDFS上
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        // 记录总次数
        long sum = 0;
        // key:h
        // values:1,1,1,1,1...
        // 迭代遍历这个values
        for (LongWritable val : values) {
            sum += val.get();
        }
        context.write(key, new LongWritable(sum));
    }
}
```



## Driver类

> 此类是入口类，main方法所在的类
>
> 1. 申请任务
> 2. 设置Mapper类
> 3. 设置Reducer类
> 4. 设置Mapper的输出类型（==如果输出类型和Reducer相同，只写Reducer的就行==）
> 5. 设置Reducer的输出类型
> 6. 设置输入路径
> 7. 设置输出路径
> 8. 提交任务

```java
public class CharCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 申请任务
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 设置入口类
        job.setJarByClass(CharCountDriver.class);
        // 设置Mapper类
        job.setMapperClass(CharCountMapper.class);
        // 设置Reducer类
        job.setReducerClass(CharCountReducer.class);

        // 设置Mapper的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        // 设置Reducer的输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 设置输入路径
        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/characters.txt"));
        // 设置输出路径
        // 要求输出路径在HDFS上不存在
        FileOutputFormat.setOutputPath(job, new Path("hdfs://hadoop01:9000/result/charcount"));

        // 提交任务
        job.waitForCompletion(true);
    }

}
```



## 结果

> 会在指定的路径下生成两个文件
>
> - `_SUCCESS`：此文件夹是空的
> - `part-r-00000`：数据统计的结果存在该文件中

```
 	1430
!	3
"	6
'	7
,	59
-	2
.	66
:	1
;	1
?	1
A	20
B	3
C	3
D	1
E	1
F	1
G	5
I	23
L	5
M	4
N	21
O	3
P	2
S	4
T	11
W	15
Y	2
a	480
b	99
c	164
d	226
e	760
f	180
g	137
h	348
i	476
j	19
k	46
l	282
m	148
n	388
o	520
p	77
q	6
r	343
s	370
t	588
u	163
v	71
w	132
x	5
y	109
z	6
```



# 练习

## 单词统计

> 源文件：words.txt

```
hello tom hello bob
hello joy
hello rose
hello joy
hello jerry
hello tom
hello rose
hello joy
```

### Mapper类

> `WordCountMapper`

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordCountMapper extends Mapper<LongWritable, Text,Text,LongWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        String[] s = value.toString().split(" ");
        for (String s1 : s) {
            context.write(new Text(s1),new LongWritable(1));
        }
    }
}
```



### Reducer类

> `WordCountReducer`

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordCountReducer extends Reducer<Text, LongWritable,Text,LongWritable> {

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long sum = 0;
        for (LongWritable value : values) {
            sum += value.get();
        }
        context.write(key,new LongWritable(sum));
    }
}
```



### Driver类

> `WordCountDriver`

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        //申请任务
        Job job = Job.getInstance(new Configuration());

        //设置启动类入口
        job.setJarByClass(WordCountDriver.class);
        //设置Mapper类
        job.setMapperClass(WordCountMapper.class);
        //设置Reducer类
        job.setReducerClass(WordCountReducer.class);

        //设置Mapper输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        //设置Reducer输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        //设置处理的文件
        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/words.txt"));
        //设置处理结果输出路径
        FileOutputFormat.setOutputPath(job,new Path("hdfs://hadoop01:9000/result/wordcount"));

        //提交任务
        job.waitForCompletion(true);
    }
}

```



### 结果

```
bob		1
hello	9
jerry	1
joy		3
rose	2
tom		2
```



## IP去重

> 源文件：`ip.txt`

```
10.9.80.16
10.9.132.111
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.133
10.9.129.86
10.9.132.111
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.65
10.9.162.13
10.9.132.111
10.9.130.83
10.9.21.119
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.1
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.33
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.1
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.8
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.33
10.9.152.66
10.9.80.16
10.9.152.65
```



### Mapper类

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

// 统计每一个IP出现的次数 --- 去重，不需要次数
public class IPMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // value - 表示IP
        context.write(value, NullWritable.get());
    }
}
```



### Reducer类

```java
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class IPReducer extends Reducer<Text, NullWritable, Text, NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        // key:IP
        // values:null,null,null...
        context.write(key, NullWritable.get());
    }
}
```



### Driver类

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class IPDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(IPDriver.class);
        job.setMapperClass(IPMapper.class);
        job.setReducerClass(IPReducer.class);

        // 如果Mapper和Reducer输出类型一致，那么可以只写一个
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/ip.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/ip"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
10.9.129.86
10.9.130.8
10.9.130.83
10.9.132.1
10.9.132.111
10.9.152.33
10.9.152.65
10.9.152.66
10.9.162.13
10.9.162.133
10.9.21.119
10.9.80.133
10.9.80.16
```



## 找出每个人的最高分

> 源文件：`score2.txt`

```
Bob 684
Alex 265
Grace 543
Henry 341
Adair 345
Chad 664
Colin 464
Eden 154
Grover 630
Bob 340
Alex 367
Grace 567
Henry 367
Adair 664
Chad 543
Colin 574
Eden 663
Grover 614
Bob 312
Alex 513
Grace 641
Henry 467
Adair 613
Chad 697
Colin 271
Eden 463
Grover 452
Bob 548
Alex 285
Grace 554
Henry 596
Adair 681
Chad 584
Colin 699
Eden 708
Grover 345
```

### Mapper类

```java
public class MaxScoreMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // Adair 345
        String[] arr = value.toString().split(" ");
        context.write(new Text(arr[0]), new IntWritable(Integer.parseInt(arr[1])));
    }
}
```



### Reducer类

```java
public class MaxScoreReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        IntWritable max = new IntWritable(0);
        // 在MapReduce为了节省内存空间，采取了地址复用机制
        // 在遍历迭代器的时候，迭代对象只创建一次
        // key:Bob
        // values:312 684 340 548
        // IntWritable val = new IntWritable();
        // val.set(312);
        // val.get() > max.get() -> 312 > 0 -> true
        // max = val; - val和max都是对象，所以赋值给的是地址 -> max和val的地址就一样了
        // val.set(684); - val和max的地址一样，那就意味着max中的值也变成了684
        // val.get() > max.get() -> 684 > 684 -> false
        // val.set(340); - max的值也跟着变成了340
        // 这么遍历下去，max的最终结果应该遍历的最后一个值
        for (IntWritable val : values) {
            if (val.get() > max.get())
                // max = val;
                max.set(val.get());
        }
        context.write(key, max);
    }
}

```



### Driver类

```java
public class MaxScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(MaxScoreDriver.class);
        job.setMapperClass(MaxScoreMapper.class);
        job.setReducerClass(MaxScoreReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                        new Path("hdfs://hadoop01:9000/txt/score2.txt"));
        FileOutputFormat.setOutputPath(job,
                        new Path("hdfs://hadoop01:9000/result/maxscore2"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
Adair	681
Alex	513
Bob		684
Chad	697
Colin	699
Eden	708
Grace	641
Grover	630
Henry	596
```



## 计算每个人的总分

> 源文件：一个文件夹`score2`
>
> - `chinese.txt`：每个人的语文成绩
> - `english.txt`：每个人的英语成绩
> - `math.txt`：每个人的数学成绩



### Mapper类

```java
public class TotalScoreMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split(" ");
        context.write(new Text(arr[0]), new IntWritable(Integer.parseInt(arr[1])));
    }
}
```



### Reducer类

```java
public class TotalScoreReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class TotalScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(TotalScoreDriver.class);
        job.setMapperClass(TotalScoreMapper.class);
        job.setReducerClass(TotalScoreReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/score2/"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/totalscore"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
Adair	171
Alex	211
Bob	200
Chad	247
Colin	182
Eden	185
Grace	139
Grover	159
Henry	194
```



# 出现错误的解决方案

## Windows配置hadoop环境

> Windows环境对Hadoop不兼容，在运行MapReduce代码时会报错，需要配置Windows的Hadoop环境

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Windows配置Hadoop2.png)

1. 将`Hadoop-alone.tar.gz`解压到一个目录下

2. 将`hadoopbin_for_hadoop2.7.1.zip`解压到Windows的`Hadoop/bin`目录下

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/偏移量.png)

3. 添加环境变量

   - 新建：`HADOOP_HOME	-> D:\software\hadoop-2.7.1\`
   - PATH添加：`%HADOOP_HOME%\bin`

4. 双击`/Hadoop/bin`目录下的`winutils.exe`。出现黑窗口一闪而过证明成功

5. 重启IDEA即可



## 双击`winutils.ext`出现确实dell错误

将`mscvr120.dll`复制到`C:\Windows\System`下



## 运行程序出现异常

> 出现`Could not locate executable null\bin\winutils.exe in the Hadoop binaries`

- 先检查环境变量是否配置正确 
- 如果环境变量正确，那么在Driver添加`System.setProperty("hadoop.home.dir", "Hadoop的安装路径"); `
- 可以将hadoop解压目录下的hadoop.dll复制到`C:\Windows\System32`，重启IDEA，重新运行 
- 如果上面三种方案，全部都没有解决，将`NativeIO.java`文件复制到IDEA中，根据要求建好包

> 如果出现：`NativeIO$Windows `

- 先检查环境变量是否配置正确 
- 如果环境变量配置正确，那么也是将`NativeIO.java`文件复制到IDEA中，根据要求建好包 

> 如果出现：`Permission denied`

检查环境变量：`HADOOP_USER_NAME`的值是不是`root`

> 如果出现：`WebException: Inner Exception: 由于目标计算机积极拒绝，无法连接。 127.0.0.1:50075/50070`

- 在Linux中通过`jps`查看Hadoop的进程是否齐全
- 如果不齐全，重启Hadoop
- 如果重启依然不齐
  - 删除tmp目录：`rm -rf /home/software/hadoop-2.7.1/tmp `
  - 格式化NameNode：`hadoop namenode -format `
  - 开启Hadoop：`start-all.sh `



## 连接工具

如果进程齐全，检查Linux的hosts文件



