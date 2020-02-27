[TOC]

# wget

> 用于从网络上下载资源，没有指定目录，下载资源默认存储到当前目录。
>
> 语法：`wget [参数] [URL地址]`
>
> - 支持断点下载功能
> - 同时支持FTP和HTTP下载方式
> - 支持代理服务器

## 下载单个文件

```shell
[root@Demo01 ~]# wget http://www.tedu.cn
```



## 下载并重命名文件

> 使用`wget -O`下载并以不同的文件名保存

```shell
[root@Demo01 ~]#  wget -O NewName.new  http://www.tedu.cn
```



## 限速下载

> 使用`wget --limit-rate`限速下载（单位，byte/秒）

```shell
[root@Demo01 ~]# wget --limit-rate=300k  http://www.tedu.cn
```



## 断点续传

> 使用`wget -c`断点续传

```shell
[root@Demo01 ~]# wget -c http://www.tedu.cn
```



## 后台下载

> 使用`wget -b`后台下载

```shell
[root@Demo01 ~]# wget -b http://www.tedu.cn
```



## 下载多个文件

> 编写一个文件(`urlfile.txt`)，内容是下载的地址：
>
> - `http://www.tedu.cn`
> - `http://big.tedu.cn/index.html`
>
> 使用`wget -i`下载多个文件

```shell
[root@Demo01 ~]# wget -i urlfile.txt 
```

