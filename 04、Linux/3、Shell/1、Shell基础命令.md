[TOC]

# Shell基础

## 概念

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。即是shell程序，也是一门编程语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务.

简单的说就是用户和内核之间进行通信/沟通的翻译官

## Shell环境

Shell 编程跟 JavaScript、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。

1. 脚本解释器
2. 文本编辑工具

系统提供多种shell程序：`cat /etc/shells`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/shell程序.png)





## 初见Shell脚本

### 编写

> 打开文本编辑器(可以使用 vi/vim 命令来创建文件)，新建一个文件 `test.sh`
>
> 编辑`test.sh`，内容如下：

```shell
#!/bin/bash
echo "Hello World !"
```

### 运行

运行shell脚本的两种方法：

1. 方法一：
   - 使脚本具有执行权限：`chmod +x ./test.sh` 
   - 执行脚本：`./test.sh`
2. 方法二：`/bin/bash test.sh`

## Shell变量

### 定义变量

> `your_name="chenpingping"`

注意：

- 变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。
- 同时，变量名的命名须遵循如下规则：
  - 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
  - 中间不能有空格，可以使用下划线（_）
  - 不能使用标点符号

### 使用变量

> `echo $your_name`
> `echo ${your_name}`
>
> 变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界

例如：

```shell
boy="good"
echo "you are my $boyfriend" //$boyfriend值为空
```

改为：

```shell
echo "you are my ${boy}friend"
```



> 另外：已经定义的变量可以重新定义，但是如果是只读的则不可以
>
> `readonly +变量名` :表示这个变量只读，不能重新赋值

```shell
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```



### 删除变量

> `unset[空格] 变量名` （不能删除只读变量）





## Shell字符串

1. 单引号中的内容会被当作普通字符串(java的String)来处理。

   ```shell
   x='$LANG' 
   echo $x
   
   //结果：# $LANG
   ```

   

2. 双引号中的内容会按照其原本的属性进行输出

   ```shell
   x="$LANG"
   echo $x
   
   //结果：# zh_CN.UTF-8
   ```

   

3. 可用转义字符“\”将特殊符号（如$、\、！等）变为一般字符;

   ```shell
   your_name='chenzhe'
   str="Hello, I know you are \"$your_name\"! "
   echo -e $str
   
   //结果
   # hello,I konw you are "chenzhe"
   ```

   

4. 拼接字符串

   > `your_name="piaolaoshi"`

   - 使用双引号拼接

     ```shell
     greeting="hello, "$your_name
     greeting_1="hello, ${your_name}"
     echo $greeting  $greeting_1
     
     //结果
     hello,chenzhe
     hello,chenzhe
     ```

   - 使用单引号拼接

     ```shell
     greeting_2='hello, '$your_name
     greeting_3='hello, ${your_name} '
     echo $greeting_2  $greeting_3
     
     //结果
     hello,chenzhe
     hello,chenzhe
     ```

     

5. 获取字符串长度

   ```shell
   string="abcd"
   echo ${#string}
   
   //结果
   4
   ```

   

6. 获取子字符串

   ```shell
   string="mayun is a great man"
   echo ${string:1:4}
   
   //结果
   ayun
   ```

   



## Shell数组

### 定义数组

> `数组名=（值1 值2 值3 值4）`

```shell
array=(1 2 3 4)
echo ${array[1]}

//结果
2
```

### 获取所有元素

```shell
echo ${array[@]}

//结果
1 2 3 4
```

### 单独赋值

```shell
array[1]=89
```

### 获取数组长度

1. 获取数组元素的个数

   - `length=${#array[@]}`
   - 或者：`length=${#array[*]}`

2. 获取数组中某个元素的长度

   > `${#array[n]}`：下标为n的元素的长度

   ```shell
   array=(mayun mahuateng leijun liuqiangdong)
   length=${#array[1]}
   
   //结果
   9
   ```





# Shell传递参数

> 我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：`**$n**`。
>
> **n** 代表一个数字：
>
> - `$0`：为执行的文件名
> - `$1`： 为执行脚本的第一个参数，
> - `$2`： 为执行脚本的第二个参数
> - 以此类推……

1. 举例：向脚本传递三个参数，并分别输出

   - 脚本

     ```shell
     #!/bin/bash
     echo "Shell 传递参数实例！";
     echo "执行的文件名：$0";
     echo "第一个参数为：$1";
     echo "第二个参数为：$2";
     echo "第三个参数为：$3";
     ```

   - 执行命令

     > 执行前要设置执行权限

     ```shell
     [root@Demo02 ~]# chmod +x test1.sh 
     [root@Demo02 ~]#  ./test.sh 1 2 3
     ```


## 处理参数的特殊字符

| 字符 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| `$#` | ==传递到脚本的参数个数==                                     |
| `$*` | ==以一个单字符串显示所有向脚本传递的参数。  如"$*"用「"」括起来的情况、以"$1  $2 … $n"的形式输出所有参数。== |
| `$$` | 脚本运行的当前进程ID号                                       |
| `$!` | 后台运行的最后一个进程的ID号                                 |
| `$@` | ==与$*相同，但是使用时加引号，并在引号中返回每个参数。  如"$@"用「"」括起来的情况、以"$1"  "$2" … "$n" 的形式输出所有参数。== |
| `$?` | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

1. 脚本

   ```shell
   #!/bin/bash
   echo "Shell 传递参数实例！";
   echo "第一个参数为：$1";
   echo "参数个数为：$#";
   echo "传递的参数作为一个字符串显示：$*";
   ```

2. 执行命令

   ```shell
   [root@sxh shell]# ./test1.sh hehe haha heihei
   Shell 传递参数实例！
   第一个参数为：hehe
   参数个数为：3
   传递的参数作为一个字符串显示：hehe haha heihei
   ```

## `$*`和`$@`的区别

1. `$*`

   - 脚本

     ```shell
     #!/bin/bash
     echo "-- \$* 演示 ---"
     for i in "$*"; do
     	echo $i
     done
     ```

   - 执行命令

     ```shell
     [root@sxh shell]# ./test1.sh hehe haha heihei
     -- $* 演示 ---
     hehe haha heihei
     ```

2. `$@`

   - 脚本

     ```shell
     #!/bin/bash
     echo "-- \$@ 演示 ---"
     for i in "$@"; do
     	echo $i
     done
     ```

   - 执行命令

     ```shell
     [root@sxh shell]# ./test1.sh hehe haha heihei
     -- $@ 演示 ---
     hehe
     haha
     heihei
     ```

     