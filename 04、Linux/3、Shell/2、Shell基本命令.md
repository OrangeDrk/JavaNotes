[TOC]



# Shell基础命令

## `find`

> 由于Linux系统大部分都是应用于服务器上，所以查询数据的时候并没有图形化界面查找数据的便捷性。

语法：`find [path] [选项] 参数`

选项：

| 选项      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| ==-name== | ==按文件名查询==                                             |
| -perm     | 按文件权限查询                                               |
| -size     | 按文件的大小查找                                             |
| -user     | 按用户(属主)查询                                             |
| -group    | 按用户组查询                                                 |
| -type     | 按文件的类型查询  <br />b ：块设备文件<br />d：目录<br />c：字符设备文件<br />p：管道文件<br />l  ：符号链接文件。 （这是个小写的L）<br />f  ：普通文件。 |

### 使用案例

1. 查找jdk安装的目录

   ```shell
   [root@sxh ~]# find / -name java
   
   /usr/lib/java
   /usr/lib/jvm/java-1.6.0-openjdk-1.6.0.0.x86_64/jre/bin/java
   /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.45.x86_64/jre/bin/java
   /usr/lib64/libreoffice/ure/share/java
   /usr/lib64/libreoffice/share/Scripts/java
   /usr/bin/java
   /usr/share/java
   /usr/share/doc/db4-devel-4.7.25/java
   /usr/share/doc/db4-devel-4.7.25/ref/java
   /var/lib/alternatives/java
   /etc/java
   /etc/alternatives/java
   /etc/pki/java
   /etc/pki/ca-trust/extracted/java
   ```

2. 查找当前系统中所有的.log后缀的文件

   ```shell
   [root@sxh ~]# find / -name *.log
   /root/install.log
   ```

3. 查找系统中`/home`目录下的非普通文件

   ```shell
   [root@sxh ~]# find /home ! -type f
   /home
   /home/newDemo.txt
   /home/shell
   /home/1
   /home/1/2
   /home/1/2/3
   /home/1/2/3/4
   /home/1/2/3/4/5
   ```

4. 查找当前用户`/home`目录下权限为700的文件

   ```shell
   [root@sxh ~]# find /home -perm 700
   ```

5. 查找`/dev`目录下的块设备文件

   ```shell
   [root@sxh ~]# find /dev -type b
   /dev/ram9
   /dev/ram8
   /dev/ram5
   /dev/ram7
   /dev/ram6
   /dev/ram3
   .....
   .....
   ```





## `sed`

> sed本身是一个逐行处理工具，会逐行处理到文件的结束。默认情况下不修改源文件，因为sed是将源文件内容逐行copy到一个临时缓冲区(模式空间)，对其进行编辑，行处理结束后，将其输出到屏幕上，也可以通过数据重定向将结果导入到新的文件中去。
>
> sed本身提供修改源文件的选项。但是如果修改源文件时，结果内容并不会发送到屏幕上。

**语法**：`sed [option] "[action]" [filename]`

**option**：

| -e   | 允许对输入数据应用多条sed命令进行编辑。 |
| ---- | --------------------------------------- |
| -i   | 表示直接操作源文件                      |

**action**：

| s    | 字符串匹配/查找 |
| ---- | --------------- |
| i    | 插入            |
| a    | 追加            |
| d    | 删除            |
| c    | 替换            |

> 注意：选项和动作的字母`i`不是同样的功能。

### 使用案例

> 源文件内容：
>
> hello  teduhadoop
>
> hello  hadoop
>
> hello  hdfs ，hi  sed

1. 将全文的h替换为H。

   ```shell
   [root@sxh shell]# sed "s/h/H/g" test2
   Hello  teduHadoop
   Hello  Hadoop
   Hello  Hdfs ，Hi  sed
   ```

2. 修改全文的h/H，第一个l/L

   - 写法一：

     ```shell
     [root@sxh shell]# sed -e "s/h/H/g" -e "s/l/L/1" test2
     HeLlo  teduHadoop
     HeLlo  Hadoop
     HeLlo  Hdfs ，Hi  sed
     ```

   - 写法二：

     ```shell
     [root@sxh shell]# sed "s/h/H/g;s/l/L/1" test2
     HeLlo  teduHadoop
     HeLlo  Hadoop
     HeLlo  Hdfs ，Hi  sed
     ```

3. 修改全文的第一个和第二个h/H

   > 修改完第一个`h`后会重新读取数据，那么第二个`h`在缓冲区中的位置是`1`

   ```shell
   [root@sxh shell]# sed "s/h/H/1;s/h/H/1" test2
   Hello  teduHadoop
   Hello  Hadoop
   Hello  Hdfs ，hi  sed
   ```

4. 通过匹配字符串修改内容

   > eth0文件内容：
   >
   > ```shell
   > DEVICE=eth0
   > HWADDR=00:0c:29:37:f6:95
   > TYPE=Ethernet
   > UUID=5c4fcaf0-7f4f-4ab1-8dee-a96aecb23823
   > ONBOOT=no
   > ....
   > ....
   > ```
   >
   > 将文件中的`ONBOOT=no`修改为`ONBOOT=yes`

   ```shell
   [root@sxh shell]# sed -i "s/ONBOOT=no/ONBOOT=yes/g" eth0
   ```

5. 在文件中进行插入新的内容

   - 在第一行插入内容"hello bigdata"

     ```shell
     [root@sxh shell]# sed "1 i hello bigdata" test2
     hello bigdata
     hello  teduhadoop
     hello  hadoop
     hello  hdfs ，hi  sed
     ```

   - 在第一行追加内容"`hello 小强`" 

     ```shell
     [root@sxh shell]# sed "1 a hello 小强" test2
     hello  teduhadoop
     hello 小强
     hello  hadoop
     hello  hdfs ，hi  sed
     ```

6. 删除匹配的行

   ```shell
   [root@sxh shell]# sed "/hdfs/d" test2
   hello  teduhadoop
   hello  hadoop
   ```



## `grep`

> 这是一款强大文本搜索工具选项

| 选项                  | 作用                                              |
| --------------------- | ------------------------------------------------- |
| ==-number==           | ==同时显示匹配行上下的n行==                       |
| -b，--byte-offset     | 印匹配行前面打印该行所在的块号码。                |
| -c,--count            | 只打印匹配的行数，不显示匹配的内容。              |
| ==-i，--ignore-case== | ==忽略大小写差别。==                              |
| -q，--quiet           | 取消显示，只返回退出状态。0则表示找到了匹配的行。 |
| --color               | 将匹配内容上色区分                                |
| ==-n，--line-number== | ==在匹配的行前面打印行号。==                      |
| -v，--revert-match    | 反检索，只显示不匹配的行。                        |

### 使用案例

> 案例在`/root`下搜索

1. 匹配上下两行

   > 会把查询到的结果的上下两行内容输出

   ```shell
   [root@sxh ~]# ls | grep -2 log
   anaconda-ks.cfg
   install.log
   install.log.syslog
   公共的
   模板
   ```

2. 匹配行号

   > 显示查询到的内容在第几行

   ```shell
   [root@sxh ~]# ls | grep -n log
   2:install.log
   3:install.log.syslog
   ```

3. 将查询关键字添加颜色

   ```shell
   [root@sxh ~]# ls | grep log --color
   install.log
   install.log.syslog
   ```

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/grep查询关键字添加颜色.png)

## `tail`

> 文本监控，通常情况下用于监视文件的增长。
>
> 场景：
>
> ​		大数据环境中，很多软件在启动时，不会将真正的启动/过程日志打印在屏幕上，因为内容繁多，会影响程序员观察启动过程中，哪个进程没有启动。像此种场景，我们就可以利用tail工具用来监视该软件启动日志文件的实际内容。

语法：`tail [选项] fileName`

选项：

| 选项   | 作用                           |
| ------ | ------------------------------ |
| ==-f== | ==用于监控文件的增长！！==     |
| -n     | 从指定的行中进行监控文件的内容 |



## `cut`

> cut命令在文件中负责剪切数据用的

语法：`cut [选项] [条件] filename`

选项：

| 选项 | 作用               |
| ---- | ------------------ |
| -b   | 字节               |
| -c   | 字符               |
| -f   | 提取第几列         |
| -d   | 按指定分隔符分割列 |

### 使用案例

> 源文件内容：
>
> 192.168.1.1
> 192.168.1.3
> 192.168.1.5
> 192.168.1.4

1. 截取第11个字节

   ```shell
   [root@sxh shell]# cut -b 11 test3.txt 
   1
   3
   5
   4
   ```

2. 截取第7-9的字节

   ```shell
   [root@sxh shell]# cut -b 7-9 test3.txt 
   8.1
   8.1
   8.1
   8.1
   ```

3. 截取最后一个字节进行排序

   ```shell
   [root@sxh shell]# cut -b 11 test3.txt | sort
   1
   3
   4
   5
   ```

4. 以点为 分隔符 获取第二个字段 

   ```shell
   [root@sxh shell]# cut -d . -f 2 test3.txt 
   168
   168
   168
   168
   ```



## `history`

> 该命令可以用来查看Linux系统中曾经执行过的命令(默认1000条)。

用法：

| 选项       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| !!         | 运行上一条命令                                               |
| !88        | 运行第88条命令                                               |
| !88  /test | 运行第88条命令并在命令后面加上/test   （可以用ls举例）       |
| !ls        | 运行上一个ls命令                                             |
| !ls:s/CF/l | 运行上一个ls命令，其中把`CF`替换成`l`                        |
| history -c | 表示清除历史命令 <br />学习阶段不要使用此命令，会清空历史命令，不利于学习 |