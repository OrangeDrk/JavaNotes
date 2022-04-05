[TOC]



# 下载Git

Git最早只支持Linux平台，目前已经能够支持Linux、Unix、Windows、OS系统之上

下载地址：https://git-scm.com/



# 安装Git

## Linux上安装Git

1. 解压Linux版源码包
2. 执行命令：
   - `./config`
   - `make`
   - `sudo make install`

## Windows上安装Git

1. 下载安装包
2. 双击执行默认安装
3. 双击`Git Bash`即可启动git



# 初始配置Git

## 设置用户名，邮箱

因为Git是一款分布式的版本控制软件，多用户之间的互相通信需要确定身份，所以安装Git后需要先配置==当前用户的名称==和==邮箱==，才可以使用

```shell
#配置用户名
$ git config --global user.name "Your Name"

#配置邮箱
$ git config --global user.email "email@example.com"
```

> 案例

```shell
$ git config --global user.name "OrangeDrk"
$ git config --global user.email "531845606@qq.com"
```



## 查看用户名和邮箱

1. 查看用户名

   ```shell
   $ git config user.name
   ```

2. 查看邮箱

   ```shell
   $ git config user.email
   ```

   

## 修改用户名，密码

1. 修改用户名

   ```shell
   $ git config --global user.name "xxxx"
   ```

2. 修改密码

   ```shell
   $ git config --global user.email "xxxx"
   ```

   