[TOC]

# 创建版本库

1. 通过`cd`命令进入目标文件夹 或 如果目标文件夹不存在可以通过`mkdir`命令创建文件夹

   ```shell
   $cd d:
   $pwd
   $mkdir gitdemo01
   $cd gitdemo01
   ```

   ![](https://note.youdao.com/yws/api/personal/file/1DECF63D3B5E4B528E11F76D761C2707?method=download&shareKey=bdba5ca47d875d7cee25e55a989d8e46)

2. 通过`git init`命令将该目录变为git所管理的仓库

   ```shell
   $ git init
   ```

   ![](https://note.youdao.com/yws/api/personal/file/2F1CB10164434827BB2A5FAA70007547?method=download&shareKey=8867ad3b60bd6d360e2ec28b1bccbecb)

3. 命令执行后，该目录下会多出一个`.git`的目录，这个目录是Git用来跟踪管理版本库的，**请勿手动修改**。

   ![](https://note.youdao.com/yws/api/personal/file/868296D0E3B2474282C4F18C26CEA524?method=download&shareKey=86e68e0e00c2ae1b2bcd1a7f444d3322)

# 向仓库提交新文件

## 创建新文件

> 在工作区中创建一个文本文件，并在文件中保存一些内容

```
1.txt
	Git is a version control system.
	Git is free software.
```



## 查看版本库状态

```shell
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/20E16810F73C471084AE37DD6FCE989A?method=download&shareKey=6a28099db5f99cf5b45f79cba70576bc)

### 图解

![](https://note.youdao.com/yws/api/personal/file/76E49333462749448E2130B428CDAAA6?method=download&shareKey=e2298745d31c68eb1ad0723215752725)



## 添加文件到暂存区

> 添加文件到==暂存区==，并查看版本库状态

```shell
$ git add 1.txt
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/8D3014933FC84990A55F3828D1BBF8C0?method=download&shareKey=ec6bd3f8e6ac6fe4ba60efb4cf5ff0a2)

### 图解：

![](https://note.youdao.com/yws/api/personal/file/40A1E5824F854FEA8EFB311FD13FD10E?method=download&shareKey=5c474d43ccb17c1d4b50316b3fdaaea2)



## 提交版本到分支

> `-m`后面输入的是本次提交的说明，可以输入任意内容
>
> 本例中只经历了一个修改就提交了，其实完全可以 多个修改后一次提交

```shell
$ git commit -m "add file 1.txt"
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/952CA2724609459A8543FCEE461950DB?method=download&shareKey=59ef283f5c10bb7485bdca890b7cd0ab)

### 图解：

![](https://note.youdao.com/yws/api/personal/file/95DB880FA73B441A8FE2A1FC16907E28?method=download&shareKey=166f34cb85ae3013c100812b57081078)



# 提交修改

## 修改工作区文件

> 修改已有`1.txt`文件,将`G`改为`g`

```
git is a version control system.
git is free software.
```



## 查看版本库状态

```shell
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/A8FFCCB2273241059D9F5FAD262C9FFA?method=download&shareKey=f5eacb8be0b2d12fc1b51e49cf56a11f)

### 图解

![](https://note.youdao.com/yws/api/personal/file/D7786B70FFAE4A6593AC133E13EF44CA?method=download&shareKey=0a5a369d01ba3d4aa32fae65e4a680c9)



## 查看文件具体修改

```shell
$ git diff 1.txt
```

![](https://note.youdao.com/yws/api/personal/file/D57CF06684ED4AF5BF76DA72ED68D769?method=download&shareKey=a3e08ca1449cec0c4c30518b05839f70)



## 添加文件到暂存区

> 将修改后的文件添加到暂存区，并查看状态

```shell
$ git add 1.txt
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/CE54FC6960124E879C66F39C33F1046E?method=download&shareKey=b77a70fd5fd9b60b85baecfdd89c134d)

### 图解

![](https://note.youdao.com/yws/api/personal/file/AE0C75911D09436D8CE4C17858110533?method=download&shareKey=e38f0f6b792fac978e40585b0c0156a8)



## 提交版本到分支

```shell
$ git commit -m "modify 1.txt"
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/9E746632C8D64528A3D5C7492E1C49DD?method=download&shareKey=97bf7c1953018cc3c92f2da9451f0821)

### 图解

![](https://note.youdao.com/yws/api/personal/file/286F816D273F4F039C56BC224171B5AE?method=download&shareKey=50c16e7c09d84a551c3d88d5119b5fa2)



# 查看历史版本

## 修改文件

> 修改工作区1.txt文件内容

```
git is a version control system.
git is free software.
git is good.
...
```

## 添加文件到暂存区

```shell
$ git add 1.txt
```



## 提交版本到分支

```shell
$ git commit -m "...some msg.."
```



## 查看历史版本

> `--pretty=oneline`：每一行输出一个版本信息

```shell
$ git log
或
$ git log --pretty=oneline
```

![](https://note.youdao.com/yws/api/personal/file/E10BD0D1FCD6405C8EF14054AFC5CAC9?method=download&shareKey=4f7a2819e20cdb6448e5eafdf336a036)





# 回滚版本

## 基于当前版本回滚

> 在Git中，用`HEAD`表示当前版本
>
> - 上一个版本就是`HEAD^`
> - 上上一个版本就是`HEAD^^`
> - 当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

```shell
$ git reset --hard HEAD^
```



## 基于指定版本号回滚

> 这种方式的好处是可以任意跳转到指定版本,而不必参考当前版本位置。

> 写全版本号

```shell
$ git reset --hard 6d7b9c833b859d8fc0200ac33306803e6867e9ba
```

> 简写版本号，写版本号前6-7位即可

```shell
$ git reset --hard 6d7b9c8
```



## 查看历史记录及对应执行后的版本号

> 此命令将会列出所有执行过的导致git版本变化的命令及其对应的版本号，可以配合git reset方便回滚到任意版本

```shell
$ git reflog
```

![](https://note.youdao.com/yws/api/personal/file/D33EAE5F767B4811AD79AE6F861B6CC5?method=download&shareKey=05f73398df50f31daa9db711c9e904d2)





# 撤销修改

## 未加入暂存区

> 在工作区中进行了修改，但尚未加入到暂存区

撤销修改 ：将工作区中的文件恢复到最近一次add 或 commit之前的状态

```shell
$ git checkout -- 1.txt
```



## 加入暂存区，但未提交到分支区

> 在工作区中进行了修改，并已经增加文件到暂存区，但尚未提交到分支区

撤销修改 :

- 将暂存区中对这个文件的记录删除掉

  ```shell
  $ git reset HEAD 1.txt
  ```

- 将工作区中的文件恢复到最近一次add 或 commit之前的状态

  ```shell
  $ git checkout -- 1.txt
  ```

  

## 已经提交到了分支区

> 在工作区中进行了修改，并且已经增加文件到暂存区，且已提交到分支

==可以通过之前所学的版本回退技术完成撤销操作。==

但这仅仅是将版本进行了回滚，git并没有真正的忘记这次被回滚的提交，后续仍然可以再通过回滚操作回到这个版本上，这就是之前所说的==git一旦提交就无法删除。==



# 删除文件

## 从工作区中删除文件

1. 手动删除

2. 命令删除

   ```shell
   $ rm 1.txt
   ```

## 查看版本库状态

```shell
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/6FB71FF50D1B472CA216C1D0AAB475C5?method=download&shareKey=005825013995bbcaf145ceb368dd6da7)



## 添加操作到暂存区

```shell
$ git rm 1.txt
```

## 查看添加到暂存区的状态

```shell
$ git status
```

![](https://note.youdao.com/yws/api/personal/file/FACD970533CD4A4088E222C00F573065?method=download&shareKey=b27532f9df2e29cbf255609a8ae8d0d9)



## 提交到分支

```shell
$ git commit -m "..some msg..s"
```

![](https://note.youdao.com/yws/api/personal/file/91D0B000F103472B8A8638184FEBB875?method=download&shareKey=29c1a0a8e6050fa3de6890751bb01adf)





# 细节：Git跟踪并管理的是修改，而非文件

> 实验

1. 修改文件1.txt

   ```
   1.txt
       git is a version control system.
       git is free software.
       git is good
       git is easy to use
   ```

   

2. 增加文件到暂存区

   ```shell
   $ git add 1.txt
   ```

   

3. 修改文件1.txt

   ```
   1.txt 
       git is a version control system.
       git is free software.
       git is good
       git is easy to use
       git is very useful
   ```

   

4. 提交版本到分支

   ```shell
   $ git commit -m "change 1.txt"
   ```

   

5. 查看状态

   ```shell
   $ git status
   ```

   > 发现第二次修改的`git is very usefull`并没有提交



# 命令总结

![](https://note.youdao.com/yws/api/personal/file/57E89FF85869427BA95B0732885048EE?method=download&shareKey=7a682761442a4a7a2a3f1067e2b44835)

# 通过`tag`指定版本签名

> git中的版本都有版本编号，对代码的提交和回滚都是基于版本编号进行的，但是git中的版本编号是一串随机串，不好记忆，此时可以使用标签机制为某个提交增加标签，方便以后查找。
>
> 打了标签之后所有需要用到版本编号的位置都可以用对应的标签替代。

## 打标签

1. 在当前分支版本上打标签

   ```shell
   $ git tag v1.0
   ```

2. 在指定历史版本上打标签

   ```shell
   $ git tag v0.9 f52c633
   ```

3. 创建带有说明的标签

   ```shell
   $ git tag -a v0.8 -m "version 0.8..." 1094adb
   ```

4. 向远程推送一个标签

   > 使用命令`git push origin <tagname>`

   ```shell
   $ git push origin v1.1
   ```

5. 一次性推送全部尚未推送到远程的本地标签

   ```shell
   $ git push origin --tags
   ```

   

## 查看标签

1. 查看标签信息

   ```shell
   $ git show v1.0
   ```

   

## 删除标签

1. 删除本地标签

   ```shell
   $ git tag -d v1.0
   ```

   

2. 删除远程标签

   - 先删除本地标签

     ```shell
     $ git tag -d v0.9
     ```

   - 再删除远程便签

     ```shell
     $ git push origin :refs/tags/v0.9
     ```

     

   

# 指定git忽略指定内容

> 可以在仓库中配置`.gitIgnore`文件，在其中配置哪些文件是不需要git管理，则git在处理此仓库时，会自动自动忽略声明的内容。

## `.gitIgnore`文件格式

1. ==空格==不匹配任意文件，可作为分隔符，可用反斜杠转义
2. 以“`＃`”开头的行都会被 Git 忽略。即`#`开头的文件标识注释，可以使用反斜杠进行转义。
3. 可以使用标准的==glob==模式匹配。所谓的glob模式是指shell所使用的简化了的正则表达式。
4. 以斜杠"`/`"开头表示目录；
   - "`/`"结束的模式只匹配文件夹以及在该文件夹路径下的内容，但是不匹配该文件；
   - "`/`"开始的模式匹配项目跟目录；
   - 如果一个模式不包含斜杠，则它匹配相对于当前 `.gitignore` 文件路径的内容，如果该模式不在  `.gitignore` 文件中，则相对于项目根目录。
5. 以星号"`*`"通配多个字符，即匹配多个任意字符；
   - 使用两个星号"`**`" 表示匹配任意中间目录
   - 比如`'a/**/z'`可以匹配 `a/z`, `a/b/z` 或 `a/b/c/z`等。
6. 以问号"`?`"通配单个字符，即匹配一个任意字符；
7. 以方括号"`[]`"包含单个字符的匹配列表，即匹配任何一个列在方括号中的字符。
   - 比如`[abc]`表示要么匹配一个a，要么匹配一个b，要么匹配一个c
   - 如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配。
     - 比如`[0-9]`表示匹配所有0到9的数字，`[a-z]`表示匹配任意的小写字母。  
8. 以叹号"`!`"表示不忽略(跟踪)匹配到的文件或目录，即要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。
   - 需要特别注意的是：==**如果文件的父目录已经被前面的规则排除掉了，那么对这个文件用"!"规则是不起作用的**。==也就是说"`!`"开头的模式表示否定，该文件将会再次被包含，如果排除了该文件的父级目录，则使用"!"也不会再次被包含。可以使用反斜杠进行转义。  

> 需要谨记：git对于`.ignore`配置文件是按行从上到下进行规则匹配的，意味着如果前面的规则匹配的范围更大，则后面的规则将不会生效；  



## 示例

> 被过滤掉的文件就不会出现在git仓库中（gitlab或github）了，当然本地库中还有，只是push的时候不会上传。

```shell
#               表示此为注释,将被Git忽略
*.a             表示忽略所有 .a 结尾的文件
!lib.a          表示但lib.a除外
/TODO      表示仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/          表示忽略 build/目录下的所有文件，过滤整个build文件夹；
doc/*.txt       表示会忽略doc/notes.txt但不包括 doc/server/arch.txt

bin/:           表示忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
/bin:           表示忽略根目录下的bin文件
/*.c:           表示忽略cat.c，不忽略 build/cat.c
debug/*.obj:    表示忽略debug/io.obj，不忽略 debug/common/io.obj和tools/debug/io.obj
**/foo:         表示忽略/foo,a/foo,a/b/foo等
a/**/b:         表示忽略a/b, a/x/b,a/x/y/b等
!/bin/run.sh    表示不忽略bin目录下的run.sh文件
*.log:          表示忽略所有 .log 文件
config.php:     表示忽略当前路径的 config.php 文件

/mtk/           表示过滤整个文件夹
*.zip           表示过滤所有.zip文件
/mtk/do.c       表示过滤某个具体文件

```

> 需要注意的是，gitignore还可以指定要将哪些文件添加到版本管理中

```shell
!*.zip
!/mtk/one.txt
```

> 唯一的区别就是规则开头多了一个感叹号，Git会将满足这类规则的文件添加到版本管理中。为什么要有两种规则呢？
>
> 想象一个场景：假如我们只需要管理`/mtk/`目录中的`one.txt`文件，这个目录中的其他文件都不需要管理，那么`.gitignore`规则应写为：

```shell
/mtk/*
!/mtk/one.txt
```

> 假设我们只有过滤规则，而没有添加规则，那么我们就需要把`/mtk/`目录下除了`one.txt`以外的所有文件都写出来！
>
> 注意上面的`/mtk/*`不能写为`/mtk/`，否则父目录被前面的规则排除掉了，`one.txt`文件虽然加了`!`过滤规则，也不会生效！

### 另外一些规则

1. `fd1/*`

   > 说明：忽略目录 `fd1` 下的全部内容；
   >
   > 注意，不管是根目录下的 `/fd1/` 目录，还是某个子目录 `/child/fd1/` 目录，都会被忽略；

2. `/fd1/*`

   > 说明：忽略根目录下的 `/fd1/` 目录的全部内容；

3. 忽略全部内容，但是不忽略 `.gitignore` 文件、根目录下的 `/fw/bin/` 和 `/fw/sf/` 目录

   > 注意：要先对`bin/`的父目录使用`!`规则，使其不被排除。

   ```shell
   /*
   !.gitignore
   !/fw/ 
   /fw/*
   !/fw/bin/
   !/fw/sf/
   ```

   

# IDEA使用Git

参考文章：https://blog.csdn.net/heartj24210/article/details/93138741