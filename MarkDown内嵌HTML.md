[TOC]



Markdown是一种可以使用普通文本编辑器编写的标记语言，类似于HTML的标记语法，可以在普通文本内具有一定的格式。但是他本身不支持修改字体、字号和颜色，所以在Markdown中要写实现字体颜色、字号的设置，需要使用内嵌的HTML。
CSDN-markdown 编辑器是其衍生版本，支持基于 PageDown ( Stack Overflow）所使用的编辑器的扩展功能（如表格、脚注、内嵌HTML、内嵌 LaTeX 等等）。

# 一、更改字体、颜色、大小

## 1.1 内嵌HTML

### 实现如下：

```html
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>
```

### 效果如下：

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>



注：Size：规定文本的尺寸大小，取值从 1 到 7 ，浏览器默认值是 3.

## 1.2 LaTeX

### 实现如下：

不推荐用嵌入 LaTeX 设置字体颜色。
否则的话否则的话：`$\color{DarkTurquoise}{否则的话}$`

汉字格式汉字格式：`$\color{silver}{汉字格式}$`

就是这样就是这样：`$\color{HotPink}{就是这样}$`



# 二、设置文字背景色

由于 style 标签和标签的 style 属性不被支持，所以这里只能是借助 table, tr, td 等表格标签的 bgcolor 属性来实现背景色。故这里对于文字背景色的设置，只是将那一整行看作一个表格，更改了那个格子的背景色（bgcolor）

### 语法如下:

```html
<table><tr><td bgcolor=yellow>背景色yellow</td></tr></table>
```

### 效果如下：

<table><tr><td bgcolor=yellow>背景色yellow</td></tr></table>
# 三、设置图片大小

## 3.1 设置图片百分比

### 语法如下：

```html
<img src="https://img-blog.csdn.net/20180916113631403?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQwNzMyOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" width="50%" height="50%">
```

### 效果如下：

<img src="https://img-blog.csdn.net/20180916113631403?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQwNzMyOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" width="50%" height="50%">

## 3.2 设置图片大小

语法如下：

```html
<img src="http://wx3.sinaimg.cn/large/703b9b92gy1fhzundcyiyj20kk0uujuc.jpg" 
     width="251" 
     height="350" >
```



## 3.3 设置图片居中

语法如下：

```html
<div align=right>
    <img src="http://wx3.sinaimg.cn/large/703b9b92gy1fhzundcyiyj20kk0uujuc.jpg" 
         width="50%" 
         height="50%">
</div>
```


效果如下：

# 四、RGB颜色对照表

RGB颜色对照表

参考文献
https://blog.csdn.net/heimu24/article/details/81189700
https://blog.csdn.net/thither_shore/article/details/52181464