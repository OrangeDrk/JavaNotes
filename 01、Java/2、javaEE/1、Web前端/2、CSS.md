[TOC]



# CSS

> 1. 网页组织的两种常用方式
>    - 表格套表格方式定义网页结构：目前不是主流，只在一些结构简单的页面中有所使用
>    - div+css方式定义网页结构：目前主流的网页开发方式，可以非常灵活的定义网页
> 2. 容器标签
>    - 本身没有任何特殊的能力，最主要的功能是用来包含其他标签组成一个整体
>    - 常见的容器标签：div、span、p
>      - `<div>`是一个块级元素。这意味着它的内容自动的开始一个新行
>      - `<span>`是一个行内元素。多个行内元素不会要求独占浏览器一行
>      - `<p>`是一个块级元素。用来声明一个段落。会在当前段落前后多出额外的空白行
> 3. css概念：层叠样式表
>    - 实现了网页中数据和样式的分离，使网页结构更加明细，解决了样式重复定义的问题，提高了开发效率和后期代码的可维护性，另外还增强了网页的美化能力。
>    - css中的注释只有一种：`/* */`

## css的四种引入方式

### 方式一：内联(行内)样式表

- 缺点：

  - 所有样式都写在一个标签中，代码不够简洁
  - 样式只在一个标签中有效，其他标签无效

- 实例代码

  ```html
  <div style="background-color: cadetblue;color: burlywood">
      第一个div  
  </div>
  ```

### 方式二：内嵌样式表

> 只在当前的HTML中有效

- 特点：如果遇到方式一（行内样式）会优先是使用行内样式

- 实例代码

  ```css
  <head>
      <style>
          /*对所有的div添加背景颜色*/
          div {
              background-color: #d683fb;/*背景颜色*/
              color: #25fffc;/*字体颜色*/
          }
      </style>
  </head>
  ```

### 方式三：外部样式表

- 有一个单独的css文件

  ```css
  span {
      background-color: burlywood;
      color: fuchsia;
  }
  p{
      background-color: #bdde4f;
      color: #ff4400;
  
  }
  ```

- 在HTML中引入

  ```html
  <head>
      <meta charset="UTF-8">
      <title>demo06-CSS样式</title>
      <!--语法三：外部样式表-->
      <link rel="stylesheet" href="demo06.css">
  </head>
  ```

### 方式四：import导入外部文件

> 在方式二的基础上，import导入外部文件

- 实例代码

  ```css
  <style>
  
  /*CSS样式语法四：import导入外部文件*/
  @import "demo06.css";
  
  </style>
  ```




## CSS选择器

> 作用：选择HTML中的标签的，对标签做样式的装饰
>
> CSS选择器的分类
>
> - 基本选择器：3种
> - 扩展选择器：6种
>
> CSS选择器优先级： ==行内式（inline）->ID选择器->类选择器->标签选择器==

### 基本选择器

#### 元素选择器

> 根据标签名，选择同一类型的标签
>
> 语法格式：`标签名{}`

```css
div {
    color: coral;
    background-color: cornsilk;
}
```

#### id选择器

> 每一个HTML标签都有一个id属性。
>
> id：标签的唯一标识
>
> 语法格式：`#id属性值{}`

```css
#d1 {
    color: deepskyblue;
    background-color: brown;
}

<div id="d1">我是一个块级元素，测试元素选择器</div>
```

#### class选择器

> 又称：类选择器
>
> 根据标签中的class属性选择，可选多个
>
> 语法格式：`.class属性值{}`

```css
.c1 {
    color: deeppink;
}
<div class="c1">我是一个块级元素，测试元素选择器</div>
<div class="c1">我是一个块级元素，测试元素选择器</div>
<div class="c1">我是一个块级元素，测试元素选择器</div>
```



### 扩展选择器

#### 后代选择器

> HTML标签有多层嵌套关系，不止有子节点，还有孙子节点等等
>
> 语法格式：`父节点 子节点 {}`

```css
div span {
    color: deepskyblue;
}

<div id="d2">
    测试后代选择器
    <div>这是子节点div
    	<span>这是孙子节点span</span>
    	<p>这是孙子节点p</p>
    </div>
    <span>这是子节点span</span>
    <p>这是子节点p</p>
</div>
```

#### 子元素选择器

> 选择特定的标签的子节点，不包含孙子节点
>
> 语法格式：`父节点>子节点 {}`

```css
#d2>p {
    color: orangered;
}
<div id="d2">
	测试后代选择器
	<div>这是子节点div
		<span>这是孙子节点span</span>
		<p>这是孙子节点p</p>
	</div>
	<span>这是子节点span</span>
	<p>这是子节点p</p>
</div>
```

#### 分组选择器

> 可以同时选择多个不同的标签，分成一组设置样式
>
> 语法格式：`元素选择器,id选择器,class选择器{}`

```css
span,p,#d1,.c1 {
    color: green;
}
```

#### 属性选择器

> 包含所有属性type的标签
>
> 可以选择指定标签的某个属性
>
> 可以选择指定标签的某个 属性="值" 的形式
>
> 可以选择指定标签的 多个 属性键值对 的形式

- **HTML代码：**

```html
<input type="text" name="username" value="测试"/><br>
<input type="password" name="password" value="123"/><br>
<input type="email" name="email"/><br>
```

- 包含所有属性type的标签

  ```css
  *[type] {
      background-color:silver;
  }
  ```

- 可以选择指定标签的某个属性

  ```css
  input[value] {
      background-color: darksalmon;
  }
  ```

- 可以选择指定标签的某个 属性="值" 的形式

  ```css
  input[name="password"] {
      background-color: deepskyblue;
  }
  ```

- 可以选择指定标签的 多个 属性键值对 的形式

  ```css
  input[type][name][value] {
      background-color: mediumpurple;
  }
  ```

#### 伪元素选择器

> “伪”：HTML已经准备好的一些选择器
>
> `:link`:没有被点击
>
> `:visited`:被点击过
>
> `:hover`:鼠标停留
>
> `:active`:鼠标正在被点击

- HTML代码：

```html
<a href="#">测试伪元素选择器</a>
```

- 未被点击的状态

  ```css
  a:link{/*未被点击的状态*/
      background-color: wheat;
  }
  ```

- 被点击过的状态

  ```css
  a:visited{/*被点击过的状态*/
      background-color: tomato;
  }
  ```

- 鼠标停留的状态

  ```css
  a:hover{/*鼠标停留的状态*/
      background-color: green;
      font-size: 20px;
  }
  ```

- 被鼠标正在点击的状态

  ```css
  a:active{/*被鼠标正在点击的状态*/
      background-color: aquamarine;
  }
  ```



## CSS属性

1. margin：0 auto 左右居中
2. line-height: px  等于行高时，自动垂直居中
3. float：left/right  浮动
4. resize：none  禁止textarea拖拽