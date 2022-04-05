[TOC]

# Web概述

## B/S:Browser-Server

- 浏览器服务器模型 WEBQQ 网页游戏
  - 优点：不需要下载客户端程序, 使用浏览器可以直接访问. 程序的升级操作是在服务器端进行的. 浏览器只需要刷新页面就可以看到升级后的效果
  - 缺点：浏览器具有一定的局限性, 页面的展示能力仍然是很差. 所有的页面数据都需要从服务器实时的获取, 所以对网速的依赖很高

## C/S：Client-Server

- 客户端服务器模型 QQ LOL
  - 优点：客户端可以任意的设计, 页面的展示能力就可以很强. 由于大量的资源都已经保存在了客户端, 和服务器交互的仅仅是一些变化的数据, 所以对网速的依赖很低
  - 缺点：第一次使用时需要下载客户端程序, 一旦程序需要升级操作, 所有的客户端程序都需要升级. 在有些场景中是不能被接受的

## web概述图

<img src="https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110639.png" style="zoom: 200%;" />





# HTML

## HTML是什么

1. - HTML(Hyper Text Mark-up Language)超文本标记语言 最基础的网页语言        W3C
   - HTML不是一门编程语言 而是一门标记语言,没有逻辑存在，只有标签
   - HTML是用标记(标签/元素)来描述网页内容的
   - HTML是文档的一种
   - HTML的文件后缀：`.html`
   - HTML文件中所有的标签，需要遵循W3C的标准

## HTML的结构

> `<!DOCTYPE HTML>`用来指定当前页面所遵循的html的版本

```html
<!--注释   Shift+ctrl+/-->
<!--doctype指的是描述文档的类型
 html:该文档是HTML页面
-->
<!DOCTYPE html>
<html><!--根标签-->
    <head><!--head标签是：头部标签；里面存放了页面的基本属性-->
        <meta charset="UTF-8"/><!--meta标签：设置页面编码的字符集  HTML5.0的写法-->
        <title>demo01-HTML的基本结构</title><!--title标签是：页面的标题标签-->
    </head>
    <body><!--body标签是：身体标签； 主要是页面中的内容信息-->
        body主要作用：用于浏览器页面的展示
    </body>
</html>
```

- 头部分用来存放html页面的基本属性信息,优先被加载
- 体部分用来存放页面数据,是可见的页面内容

`<title></title>`指定网页的标题

`<meta http-equiv="Content-type" content="text/html; charset=UTF-8" />`其中的`charset`的值用来指定浏览器用什么编码解析当前页面

## HTML语法

- 结束标签,如果标签内没有修饰的内容, 开始标签和结束标签可以合并为一个自闭标签
  - `<br/>`  换行
  - `<hr/>`  在当前行画一条线

- 标签通常都可以具有属性, 属性与属性值用"="连接, 属性的值可以用双引号、单引号引起来或者不用引号, 一般会用双引号引起来。
  
- ==如果不生效查看页面代码使用的是否为英文双引号==
  
- html中对页面中代码需要做注释：`<!-- html的注释 -->`

- html中多个连续的空白字符(制表符,空格,换行)默认会合并为一个空格来显示。

  - 如果非要输入空格,可以用转义字符来替代 `&nbsp;`

  - 如果非要输入换行,可以用 `<br/>` 来代替

  - 转义字符

    ```html
    <	&lt;
    >	&gt;
    "	&quot;
    '	&apos;
    空格	&nbsp;
    ```

## 标签

> 块级元素：div  , p  , form,  ul, li , ol, dl,  form,  address, fieldset,  hr, menu,  table
>
>  行内元素：span,  strong,  em,  br,  img ,  input,  label,  select,  textarea,  cite, 
>
> 区别：
>
> 1. 块级元素会独占一行，其宽度自动填满其父元素宽度
>
>       行内元素不会独占一行，相邻的行内元素会排列在同一行里，知道一行排不下，才会换行，其宽度随元素的内容而变化
>
> 2. 块级元素可以设置 width, height属性，行内元素设置width,  height无效
>
>    -  【注意：块级元素即使设置了宽度，仍然是独占一行的】 
>
> 3. 块级元素可以设置margin 和 padding.  
>
>    - 行内元素的水平方向的padding-left,padding-right,margin-left,margin-right 都产生边距效果，但是竖直方向的padding-top,padding-bottom,margin-top,margin-bottom都不会产生边距效果。（水平方向有效，竖直方向无效）

### font标签

> 指定字体的大小、颜色

- 属性：
  - color：指定字体的颜色
    - 直接指定颜色的单词名称：red
    - 指定十六进制的像素：#123456，#AABBCC
    - rgb(0-255,0-255,0-255):使用三基色,IE浏览器使用
  - size：指定字体大小
    - 1-7 有效(如果值超过7，默认7)
- 实例代码:

```html
<font color="rgb(0,0,0)" size="88">字体标签font</font>
```



------



### 标题标签

> h1-h6标题标签:指的是标题的级别，从一级到六级

- 属性：

  - align ：字体的排列方式

    | 属性值  | 作用   |
    | ------- | ------ |
    | left    | 靠左   |
    | center  | 居中   |
    | right   | 靠右   |
    | justify | 自适应 |

    - ==left/right/center 是常用的==
    - justify自适应，默认从左到右布局排列，不常用

- 实例代码：

```html
<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题</h6>
```



------



### 列表标签

#### 无序列表

> `<ul>  <li></li>  <ul>`

- 属性：
  - type
    - disc：默认属性值——实心圆
    - square：方块
    - circle:空心圆
- 实例代码

```html
    <ul type="square"><!--定义一个无序列表-->
        <li>华为</li><!--表示列表项，可以有多个-->
        <li>小米</li>
        <li>联想</li>
    </ul>
```

#### 有序列表

> `<ol>  <li></li>  <ol>`

- 实例代码

```html
    <ol>
        <li>a</li>
        <li>b</li>
        <li>c</li>
    </ol>
```



------



### 图片标签

> 单标签

- 属性：
  - 必要属性
    - src：图片的存储路径
    - alt：表示替换图片的文字，只有再图片不存在时显示
  - 可选属性
    - width：宽度  单位：px像素  %占用浏览器宽度的百分比
    - height：高度  单位：px像素  %占用浏览器高度的百分比
    - border：边框
- 实例代码

```html
<img src="image/1.gif" alt="此地无银三百两"
         width="50%" height="50%" border="10px"/>
```



------



### 超链接

> 用于指向当前位置以外的资源
>
> - 用于创建指向另外一个文档的超链接
> - 用于在当前页面的不同位置之间进行跳转，利用id或name属性进行跳转。
>   - 一般在本页面中使用，当网页内容过长，定位标记会比拖动滚动条方便快捷。
>   - ==定位标记要和超链接结合使用才有效。==

#### a标签

> 用于创建指向另外一个文档的超链接

- 属性：
  - href：访问资源的url路径
  - target:页面的跳转方式
    - _self:默认的当前窗口打开链接
    - _blank：在新窗口打开一个链接
- 实例代码

```html
<a href="https://www.tmooc.cn" target="_blank">达内教育</a>
<a href="demo02.html" target="_blank">本地文件demo02.html</a>
```



#### 超链接与锚点配合使用：指定位置跳转

> 用于在当前页面的不同位置之间进行跳转

- 属性
  - 锚点：
    - name：用于定位当前的位置
    - #name值：跳转地址
- 实例代码

```html
<a name="top" >小说目录</a>
<p>第一章</p>
<p>第一章</p>
<p>第一章</p>
<p>第一章</p>
	....
	....
<!--返回顶部的超链接-->
<a href="#top">返回顶部</a>
```



------



### 表格标签

> `<table></table>`

- 成员：
  - `tr`标签：行标签
  - `td`标签：列标签
  - `th`标签：列标签（标题）
-  table属性：
  -  `border`：边框
  - `cellspacing`：两个单元格之间的间距，外边距
  - `cellpadding`：一个单元格内填充的距离，内边距
  -  `bgcolor`：背景颜色
  - `bordercolor`：边框颜色
  -  `width`：宽度
  - `align`：对齐方式。center：整个表格相对于浏览器居中
-  tr属性：
  - `align`：对齐方式。center：整行所有的列居中
  - `bgcolor`：一整行的背景颜色
- td属性：
  - `align` 对齐方式
  - `bgcolor` 背景颜色
  - `width` 宽度
  - `height` 高度
  - `colspan` 可横跨的列数  col:column列
  - `rowspan` 可竖跨的行数  row:行
  -  `<caption> `定义表格的标题
- th标签：    
  - 自动字体加粗并剧中
- 实例代码

```html
<table border="2px" cellspacing="5px" cellpadding="5px"
       bgcolor="#faebd7" bordercolor="red" width="50%" align="center">

    <caption>这是一个表格标题</caption>

    <tr align="center" bgcolor="#00bfff"><!--一行-->
        <th>华为</th><!--一列-->
        <th>小米</th>
        <th>苹果</th>
    </tr>
    <tr><!--一行-->
        <td colspan="2">mate 30</td><!--一列-->
        <td rowspan="2">青春版</td>
        <td>iPhone 11</td>
    </tr>
    <tr><!--一行-->
        <td>mate 30 Pro</td><!--一列-->
        <td>红米</td>
        <td>iPhone 11 Pro Max</td>
    </tr>
</table>
```



------



### form表单

> 主要作用：给用户通过浏览器维护数据使用

- 浏览器向服务器发送数据的方式，有两种:
  1. 利用超链接向服务器发送数据 -- 请求参数
     - 在超链接的后面拼接上要发送的请求参数, 链接和请求参数之间用?分割, 参数名和参数值用 = 连接, 多个参数之间用 & 分割, 可以存在多个同名的参数
  2. 利用表单向服务器发送数据
     - 中的`<form>`标签以及一些表单项标签, 用户可以输入数据, 通过提交表单发送数据给服务器

#### form标签

- 必须存在的属性：

  - `action`：指定表单发送的目标URL地址

- 可选的属性：

  - `method`：指定以何种方式发送表单

- http协议指定了7种提交方式，其中5种使用的极少，多数只用GET提交和POST提交

- 只有使用表单并且明确指定提交方式为post时（也就是设置`method="post"`）才是post提交，其他提交都是get提交

- get和post提交的区别

  > 主要区别体现在数据传输方式的不同

  - get：
    - 请求参数会赋值在地址栏后进行传输
    - 这种方式发送的数据量有限，最大不超过1kb（或4kb）
    - 数据显示在地址栏，安全性差
  - post：
    - 请求参数在底层流中传输
    - 这种方式发送的数据量无限制
    - 地址栏上看不到数据，比较安全

- 表单中的项

  - 表单中可以有多个输入项，输入项必须有`name`属性才可以被提交，如果输入项没有`name`属性，则表单在提交时会忽略它

#### input：输入文本框

- 重要属性：type= 

  | **属性值** | **作用**   | **介绍**                                                     |
  | ---------- | ---------- | ------------------------------------------------------------ |
  | text       | 文本框     | 输入的文本信息直接显示在框中                                 |
  | password   | 密码框     | 输入的文本以圆点或者星号的形式显示                           |
  | radio      | 单选框     | 进行单项的选择 如性别选择 多个radio的name属性相同会被当作一组来使用 必须用value为选项指定提交的值 |
  | checkbox   | 复选框     | 进行多项选择，爱好的选择。 多个checkbox具有相同的name属性时会被当作一组来使用 必须用value为选项指定提交的值 |
  | hidden     | 隐藏字段   | 如果有一些信息，不希望用户看见，又希望表单能够提交，就可以用隐藏字段隐含在表单中 |
  | submit     | 提交按钮   | 实现表单提交操作的按钮 可以通过value属性指定按钮显示的文字   |
  | reset      | 重置按钮   | 重置表单到初始状态                                           |
  | button     | 按钮       | 普通按钮, 没有任何功能 需要配合JavaScript为按钮指定具体的行为。可以用value属性指定按钮显示的文字。 |
  | file       | 文件上传项 | 提供选择文件进行上传的功能                                   |
  | image      | 图像       | 利用一张图片替代提交按钮的功能, 不常用                       |

- name属性:

  - 表单中可以有多个输入项，输入项必须有name属性才可以被提交，如果输入项没有name属性，则表单在提交时会忽略它。另外name属性的值是可以重复的

- value属性：可以给input输入框设置一个初始值

- readonly属性：使当前输入项变为只读，不能修改，但是提交时仍会被提交

- disabled属性：使当前输入项不可用，不能修改值，也不会被提交

- size属性：指定当前输入框的宽度

- checked属性：指定单选框/复选框被选中

实例代码：

```html
<form action="http://www.tmooc.cn" method="post">
    <!--1.input输入框-->
    用户名：<input type="text" name="username"/><br/>

    <!--2.input密码框-->
    密码：<input type="password" name="password" size="10"/><br/>

    <!--3.input单选框
    name属性:单选框的内容划分成一组，一组中只能选中一个-->
    性别：<input type="radio" name="gender" value="male" checked="checked"/>男
    <input type="radio" name="gender" value="female"/>女<br>

    <!--4.input复选框-->
    爱好：
    <input type="checkbox" name="hobby" value="sing" checked="checked"/>唱
    <input type="checkbox" name="hobby" value="jump"/>跳
    <input type="checkbox" name="hobby" value="basketball"/>打篮球
    <br>

    <!--5.input隐藏框
    value属性：文本框中的默认值-->
    评价：<input type="hidden" value="此人应该很有才"/>
    <br>

    <!--6.input文件上传-->
    上传：<input type="file"/><br>

    <!--7.input图片按钮-->
    点我：<input type="image" src="image/1.gif" height="100px"><br>

    <!--8.input提交按钮-->
    <input type="submit" value="注册">

    <!--9.input重置按钮-->
    <input type="reset">

    <!--10.input日期按钮-->
    <input type="date"><br>
</form>
```



#### `textarea`文本域

- 属性：

  - rows：指定文本域的行数（高度）
  - cols：指定文本域的列数（宽度）
  - readonly：只读
  - disabled：禁用，不可操作文本域

- 实例代码

  ```html
  input there：
  <textarea name="" id="" cols="10" rows="5" disabled="disabled" readonly="readonly">
      123
  </textarea><br>
  ```

  

#### `<select><option>`

> select:提供下拉选择功能
>
> option：下拉选框中的选项
>
> - 可以用value属性指定提交的值，如果不指定，将会提交标签内的文本

- 重要属性：

  - name:下拉列表的名称
  - disabled：禁用下拉选框

- 其他属性：

  - size：设置下拉选项中可见选项的个数
  - multipe：是否支持多选

- 实例代码：

  ```html
  <!--下拉框：select+option组合标签-->
  <select ><!--定义一个下拉框 -->
      <option>北京</option>
      <option>上海</option>
      <option>广州</option>
      <option>深圳</option>
      <option>杭州</option>
  </select>
  ```







