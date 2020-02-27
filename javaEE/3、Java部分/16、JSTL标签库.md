[TOC]

# 介绍

## JSTL标签库

JSTL是apache对EL表达式的扩展（也就是说JSTL依赖EL），JSTL是标签语言！JSTL标签使用起来非常方便，它与JSP动作标签一样，只不过它不是JSP内置标签，需要我们自己导包，以及指定标签库

## 作用

提高在JSP中的逻辑代码的编写效率,使用标签



------



# ==JSTL的核心标签库(重点)==

> - 导入jar包：封装好的taglib函数库
>
> - 声明JSTL标签库的引入(核心标签库),不声明无法获取标签
>
>   `<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
>
> - (`prefix="c"`)默认的前缀是：c；可以自定义前缀：h、hello.....
>
> - ==尽量使用默认前缀==，方便自己和其他人阅读
>
> - 共12个标签
>



## 基本标签

### `<c:out>`：用于输出数据

> `<c:out value="数据" default="默认值"></c:out>`
>
> 作用:
>
> - 用于在JSP中显示数据，就像<%= ... > 
>
> 数据:可以为==常量值==,也可以是==EL表达式==

```java
<%
    request.setAttribute("str","今天天气很好");
%>

<c:out value="${str}" default="null"></c:out>
    
//结果:今天天气很好
```

> 转义输出：`<c:out value="数据" escapeXml="true/false"></c:out>`
>
> - `escapeXml`：true转义；false不转义
> - 如果value是HTML标签
>   - 转义：将标签==字符串==原本原样输出
>   - 不转义：作为==HTML标签==输出到浏览器

```java
//字符串输出：
<c:out value="<a href='http://www.bing.com'>Bing</a>"></c:out><br>
//结果：<a href='http://www.bing.com'>Bing</a>

//超链接：
<c:out value="<a href='https://www.bing.com'>Bing</a>" escapeXml="false"></c:out>
//结果：Bing  —————这是一个超链接
```



### `<c:set>`：用于保存数据

1. 保存到作用域中

   > `<c:set var="键名" value="键值" scope="作用域对象"></c:set>`
   >
   > - scope:默认page
   >   - page：域对象pageContext
   >   - request：域对象request
   >   - session：域对象session
   >   - application：域对象：application
   >
   > 作用:
   >
   > - 存储数据到作用域对象中

   ```java
   <c:set var="hello" value="hello pageContext" scope="page"></c:set>
   ${pageScope.hello}
       
   //结果:hello pageContext
   ```

2. 向map中增加、修改键值对

   > `<c:set property=“key” value=“value” target=“map对象”></c:set>`
   >
   > - `property`：要对map对象那个key值操作
   > - `value`：操作map中key对应的value值
   > - `target`：通过EL表达式获取map对象

   ```java
   //修改key为name的value 为  张飞
   <c:set property="name" value="张飞" target="${map}"></c:set>
       
   //添加键值对：son：张苞
   <c:set property="son" value="张苞" target="${map}"></c:set> 
   ```

3. 修改javaBean对象属性

   > `<c:set property=“对象属性” value=“属性值” target=“javaBean对象”></c:set>`
   >
   > - `property`：对象的属性
   > - `value`：对象属性的值
   > - `target`：要操作的对象

   ```java
   //把p1对象的name改为贾宝玉
   <c:set property="name" value="贾宝玉" target="${p1}"></c:set>
   ```

   



### `<c:remove>`：用于删除数据

> `<c:remove var="键名" scope="作用域对象"></c:remove>`
>
> scope：表示要删除的作用域（可选），==不写的话就是默认将四个作用域对象中符合要求的数据全部删除==

```java
<c:set var="hello" value="hello pageContext" scope="page"></c:set>
<c:set var="hello" value="hello request" scope="request"></c:set>

<c:remove var="hello" scope="page"></c:remove>
${hello}
    
//结果： hello request
```



### `<c:catch>`：用于处理异常（不常用）

> 专门用于捕获异常，能够自动识别异常的类型，并可以打印输出

```java
<c:catch var="e1">
	<%int j = 10/0;%>
</c:catch>
${e1}//java.lang.ArithmeticException: / by zero
```





## 逻辑标签

### `<c:if>`：与一般程序中的if一样

> ```jsp
> <c:if test="${表达式}">
> 	前端代码
> </c:if>
> ```
>
> 作用：
>
> - 进行逻辑判断，相当于Java代码的单分支判断
>
> 注意：
>
> - 逻辑判断便签需要依赖于EL的逻辑运算，EL表达式中涉及到的数据必须从作用域获取
>   - 作用域的数据可以通过`<c:set>`标签设置

```java
<c:set var="a" value="4"></c:set>
<c:if test="${a>3}">
    <b>今天天气很好哦</b>
</c:if>
  
//结果:今天天气很好哦
```



### `<c:choose>`-`<c:when>`-`<c:otherwise>`：多分支if（if-else if--else/switch-case）

> ```jsp
> <c:choose>
>     <c:when test="${表达式}">显示内容</c:when>
>     <c:when test="${表达式}">显示内容</c:when>
>     …………
>     …………
>     <c:otherwise>显示内容</c:otherwise>
> </c:choose>
> ```
>
> 作用：
>
> - 用来进行多条件的逻辑判断，类似Java中的多分支语句
>
> 注意：
>
> - 条件成立只执行一次，都不成立则执行`otherwise`

```java
<c:set var="score" value="85"></c:set>
<c:choose>
    <c:when test="${score>=90}">
        <i>奖励吃鸡套装</i>
    </c:when>
    <c:when test="${score<90&&score>=70}">
        <i>奖励空投箱</i>
    </c:when>
    <c:when test="${score<70&&score>=60}">
        <i>没有奖励，没有惩罚</i>
    </c:when>
    <c:otherwise>
        <i>弹药减少，配件减少</i>
    </c:otherwise>
</c:choose>
    
//结果————————————————————————————————————
奖励空投箱
```





## 循环标签

### `<c:forEach>`：

1. 常量for循环

   > ```jsp
   > <c:forEach begin="1" end="4" step="2" varStatus="变量名">
   > 	循环体
   > </c:forEach>
   > ```
   >
   > 属性：
   >
   > - begin：声明循环开始位置
   > - end：声明循环结束位置
   > - step：设置步长（每次循环自增的长度）
   > - varStatus：声明变量记录每次循环的数据
   >   - 角标：`${vs.index}`
   >   - 次数：`${vs.count}`
   >   - 是否是第一次循环：`${vs.first}`
   >   - 是否是最后一次循环：`${vs.last}`

   ```java
   <c:forEach begin="0" end="4" step="1" varStatus="vs">
       111---角标：${vs.index}---次数：${vs.count}
       ---第一次：${vs.first}---最后一次：${vs.last}<br>
   </c:forEach>
       
   //结果————————————————————————————————————
   111---角标：0---次数：1 ---第一次：true---最后一次：false
   111---角标：1---次数：2 ---第一次：false---最后一次：false
   111---角标：2---次数：3 ---第一次：false---最后一次：false
   111---角标：3---次数：4 ---第一次：false---最后一次：false
   111---角标：4---次数：5 ---第一次：false---最后一次：true
   ```

2. 增强for循环

   > ```jsp
   > <c:forEach items="${list}" var="str">
   >  ${str}<br>
   > </c:forEach>
   > ```
   >属性：
   > 
   >- item：声明要遍历的对象。结合EL表达式获取对象
   > - var：声明变量记录每次循环的结果。结果存储在作用域中，需要使用EL表达式获取

   - 遍历list

   ```Java
   <%
       List<String> list = new ArrayList<>();
       list.add("a");
       list.add("b");
       list.add("c");
       request.setAttribute("list",list);
   %>
   <c:forEach items="${list}" var="str">
    ${str}<br>
   </c:forEach>
       
   //结果———————————————————————————————————————
   a
   b
   c
   ```

   - 遍历map

   ```java
   <%
       Map<String,String> map = new HashMap<>();
       map.put("a1", "哈哈哈");
       map.put("b1", "嘿嘿嘿");
       request.setAttribute("map",map);
   %>
   <c:forEach items="${map}" var="s">
       ${s.key}---${s.value}<br>
   </c:forEach>
       
   //结果—————————————————————————————————
   a1---哈哈哈
   b1---嘿嘿嘿
   ```

### `<c:forTokens>`：用于分割字符串

> 类似于java 代码中的`str.split("")`
>
> 区别：
>
> - `str.split("")`，参数为空：表示把每一个字符都拆分出来
> - JSTL中，`delims=""` ： 表示不做任何拆分
>
> `items`：要分割的对象
>
> `delims`：用什么分割
>
> `var`：存放分割后的数据

```java
<c:forTokens items="${str}" delims="," var="s">
	${s}<br>
</c:forTokens>

//结果：
/*1
admin
123
超级管理员
1@163.com*/
```







------



# JSTL的格式化标签库（讲解）

> - 导入jar包
>
> - 声明JSTL标签库的引入(核心标签库),不声明无法获取标签
>
>   ```jsp
>   <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
>   ```

## `<fmt:formatNumber>`：格式化数字

> 使用指定的格式或精度格式化数字

- 属性：

  | **属性**               | **描述**                           | **是否必要** | **默认值**    |
  | ---------------------- | ---------------------------------- | ------------ | ------------- |
  | value                  | 要显示的数字                       | 是           | 无            |
  | type                   | NUMBER，CURRENCY，PERCENT类型      | 否           | Number        |
  | pattern                | 指定一个自定义的格式化模式用于输出 | 否           | 无            |
  | var                    | 存储格式化数字的变量               | 否           | Print to page |
  | scope                  | var属性的作用域                    | 否           | page          |
  | max(min)IntegerDigits  | 整型数最大/小的位数                | 否           | 无            |
  | max(min)FractionDigits | 小数点后最大/小的位数              | 否           | 无            |

- 实例

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
      pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
  
  <html>
  <head>
    <title>JSTL fmt:formatNumber 标签</title>
  </head>
  <body>
  <h3>数字格式化:</h3>
  <c:set var="balance" value="120000.2309" />
  <p>格式化数字 (1): <fmt:formatNumber value="${balance}" type="currency"/></p>
  <p>格式化数字 (2): <fmt:formatNumber type="number" 
                                  maxFractionDigits="3" value="${balance}" /></p>
  <p>格式化数字 (3): <fmt:formatNumber type="percent" 
                                  minFractionDigits="10" value="${balance}" /></p>
  <p>美元 :
  <fmt:setLocale value="en_US"/>
  <fmt:formatNumber value="${balance}" type="currency"/></p>
  </body>
  </html>
  ```

- 结果

  ```
  数字格式化:
  格式化数字 (1): ￥120,000.23
  格式化数字 (2): 120,000.231
  格式化数字 (3): 12,000,023.0900000000%
  美元 : $120,000.23
  ```

## `<fmt:formatDate>`：格式化日期

>  使用指定的风格或模式格式化日期和时间 
>
> 常用的模式：yyyy-MM-dd
>
> ​				  hh-mm-ss

- 属性

  | **属性**  | **描述**                           | **是否有必要** | **默认值** |
  | --------- | ---------------------------------- | -------------- | ---------- |
  | value     | 要显示的日期                       | 是             | 无         |
  | type      | date,time,both                     | 否             | date       |
  | dateStyle | full，long，medium，short，default | 否             | default    |
  | timeStyle | full，long，medium，short，default | 否             | default    |
  | pattern   | 自定义格式模式                     | 否             | 无         |
  | var       | 存储格式化日期的变量名             | 否             | 显示在页面 |
  | scope     | 存储格式化日志变量的范围           | 否             | 页面       |

- 实例

  > ==主要记（6），（7）==

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
      pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
  
  <html>
  <head>
    <title>JSTL fmt:dateNumber 标签</title>
  </head>
  <body>
  <h3>日期格式化:</h3>
  <c:set var="now" value="<%=new java.util.Date()%>" />
  
  <p>日期格式化 (1): <fmt:formatDate type="time" 
              value="${now}" /></p>
  <p>日期格式化 (2): <fmt:formatDate type="date" 
              value="${now}" /></p>
  <p>日期格式化 (3): <fmt:formatDate type="both" 
              value="${now}" /></p>
  <p>日期格式化 (4): <fmt:formatDate type="both" 
              dateStyle="short" timeStyle="short" 
              value="${now}" /></p>
  <p>日期格式化 (5): <fmt:formatDate type="both" 
              dateStyle="medium" timeStyle="medium" 
              value="${now}" /></p>
  <p>日期格式化 (6): <fmt:formatDate type="both" 
              dateStyle="long" timeStyle="long" 
              value="${now}" /></p>
  <p>日期格式化 (7): <fmt:formatDate pattern="yyyy-MM-dd" 
              value="${now}" /></p>
  
  </body>
  </html>
  ```

- 结果

  ```
  日期格式化:
  
  日期格式化 (1): 11:19:43
  
  日期格式化 (2): 2016-6-26
  
  日期格式化 (3): 2016-6-26 11:19:43
  
  日期格式化 (4): 16-6-26 上午11:19
  
  日期格式化 (5): 2016-6-26 11:19:43
  
  日期格式化 (6): 2016年6月26日 上午11时19分43秒
  
  日期格式化 (7): 2016-06-26
  ```

## `<fmt:parseDate>`：解析日期

>  解析一个代表着日期或时间的字符串 

- 属性

  | **属性** | **描述**               | **是否必要** | **默认值** |
  | -------- | ---------------------- | ------------ | ---------- |
  | value    | 要显示的日期           | 是           | 无         |
  | type     | date，time，both       | 否           | date       |
  | pattern  | 自定义格式模式         | 否           | 无         |
  | var      | 存储格式化日期的变量名 | 否           | 显示在页面 |

- 实例

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
      pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
  
  <html>
  <head>
    <title>JSTL fmt:parseDate 标签</title>
  </head>
  <body>
  <h3>日期解析:</h3>
  <c:set var="now" value="20-10-2015" />
  
  <fmt:parseDate value="${now}" var="parsedEmpDate" pattern="dd-MM-yyyy" />
  <p>解析后的日期为: <c:out value="${parsedEmpDate}" /></p>
  
  </body>
  </html>
  ```

- 结果

  ```
  日期解析:
  
  解析后的日期为: Tue Oct 20 00:00:00 CST 2015
  ```

## `<fmt:requestEncoding>`：设置请求编码格式

>  设置request的字符编码 

- 实例：`<fmt:requestEncoding value="UTF-8" />`



------



# JSTL的SQL标签库（了解）

# JSTL的函数标签库（了解）

# JSTL的XML标签库（了解）