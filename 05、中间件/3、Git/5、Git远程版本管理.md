[TOC]

# 远程仓库概念

为了方便版本的交换，通常会使用一个中心服务器，24小时连续运行，提供版本控制服务

这就有两种做法：

- 自己搭建中心服务器
- 使用类似GitHub的代码托管网站    

目前我们更多使用代码托管的方式进行开发工作



# 远程仓库的使用



## 远程仓库连接

由于GitHub是境外的网站，连接不稳定，我们使用==GitEE==码云替代

1. 注册GitEE账号，并配置仓库，会分配到仓库的地址
2. 地址有两种形式，分别是HTTPS和SSH两种方式，用哪个都可以，区别是HTTPS需要输入用户名密码，而SSH可以实现免密登录

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Git远程仓库连接.png)



## 将本地仓库关联到远程仓库

> 本地关联到远程仓库：
>
> 命令：`$ git remote add <服务器名称><远程仓库地址> `

```shell
$ git remote add origin https://gitee.com/pcj2619/big1807_test01.git
```



## 将本地仓库推送到远程仓库

> 把本地库的内容推送到远程
>
> 命令：`$ git push -u <服务器名称> <本地分支名称>`

```shell
$ git push -u origin master
```

当使用HTTPS连接时，会弹出框要求输入用户名密码，输入码云的用户名和密码即可

==第一次推送`master`分支时，加上了`-u`参数==，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/对称加密图解.png)



## 从远程仓库克隆

> 将远程仓库克隆到本地
>
> 命令：`$ git clone <远程仓库地址> <本地目录名>`

```shell
$ git clone https://gitee.com/pcj2619/big1807_test01.git D:\gitdemo02 
```



## 拉取远程仓库中的更改到本地仓库

> 拉取远程仓库中的更改到本地仓库
>
> 命令：`$ git pull <远程仓库地址> <远程分支名>:<本地分支名>`

```shell
$ git pull https://gitee.com/pcj2619/big1807_test01.git master:master 
```



## 删除本地仓库到远程仓库的关联

> 将本地仓库和远程仓库的关联关系删除
>
> 命令：`$git remote rm <服务器名称> `

```shell
$ git remote rm origin 
```



# 扩展：对称加密VS非对称加密

## 对称加密

原理：只有一个密钥，同时作为加密密钥和解密密钥使用

缺点：密钥传输过程中无法保证完全安全

图解：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/非对称加密实现免密登录.png)

## 非对称加密

**原理**：有一对密钥，称为公钥和私钥

- 公钥加密只能由私钥解密
- 私钥加密只能由公钥解密
- 可以在网络中只传输公钥进行加密，在接收端通过私钥进行解密

**优点**：这种方式解密用的私钥没有在网络中传输过，更加安全

**图解**：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Git将本地仓库推送到远程仓库.png)

**应用场景**：免密登录、数字签名等

**免密登录原理**：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/非对称加密.png)



# 使用SSH连接远程仓库

> 使用SSH连接远程仓库相对于HTTPS连接的优势在于：
>
> - 通过非对称加密保护的，更加安全
> - 可以实现免密登录

## 1、客户端生成公私钥

> 此命令会在用户主目录下生成`.ssh`目录里面有`id_rsa`和`id_rsa.pub`两个文件:
>
> - `id_rsa`是私钥，不能泄露出去
> - `id_rsa.pub`是公钥，可以放心的传播。

```shell
$ ssh-keygen -t rsa -C "531845606@qq.com"
```

## 2、在码云(GitEE)中配置公钥

在码云(GitEE)的[配置]—[SSH公钥]中将本机中生成的公钥内容粘贴进去。之后就可以使用SSH连接免密访问码云了

> 码云官网：https://gitee.com/