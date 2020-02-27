[TOC]

# JavaScript回顾

## 示例：通过script代码处理按钮点击

### HTML代码：

> 通过`onclick`属性设置函数`getData()`；函数名自己起：`onclick="getData()"`

```html
<body>
    <h3>欢迎访问英雄商店</h3>
    <hr>
    英雄名称:<input type="text" name="uname" value="" id="uname">
    <input type="button" value="搜索" onclick="getData()">
    <hr>
    <table border="1px" id="ta">

    </table>

</body>
```

### script代码：

```jsp
<head>
    <title>英雄商店</title>
    <script type="text/javascript">
        function getData() {
            //需要的功能…………
        }
    </script>

</head>
```



## 获取输入框输入的内容

```javascript
//获取用户请求信息
var name=document.getElementById("uname").value;
```



------



# 介绍

## Ajax

> ==异步刷新技术，用来在当前页面响应不同的请求内容==

- Ajax全称为“Asynchronous JavaScript and XML”（异步JavaScript和XML），是一种创建交互式网页应用的网页开发技术。
- 不是一种新技术，是如下几种技术的组合应用：
  - 基于web标准（standards=based presentation）XHTML+CSS的表示
  - 使用DOM（Document Object Model）进行动态显示及交互
  - 使用XML和XSLT进行数据交换及相关操作
  - ==使用XMLHttpRequest进行异步数据查询、检索==
  - 使用JavaScript将所有的东西绑定在一起
- Ajax本质上是一个浏览器端的技术
- 利用js发送请求，而且是局部的发送请求，异步的访问服务器

## 操作服务端返回的数据

- text纯文本（“用户名已被注册”）
- xml
- ==json==

## 同步&异步交互

1. 同步：
   - 向服务器发一个请求, 必须等待响应结束, 才能发送第二个请求, 在服务器处理期间, 浏览器不能做其他操作
   - 刷新范围：刷新整个页面
2. 异步：
   - 向服务器发一个请求, 不用等待响应结束, 就可以发送第二个请求, 在服务器处理期间, 浏览器可以做其他操作
   - 刷新范围：可以使用js接受服务器的响应, 再利用js局部刷新页面



## 为什么需要Ajax

- 需求：
  - 有时候我们需要将本次的响应结果和前面的响应结果内容在同一个页面中展现给用户
- 解决：
  - 在后台服务器端将多次响应的内容重新拼接成一个jsp页面，进行响应。（但是这样会造成很多响应内容被重复的响应，资源浪费）
  - 使用Ajax技术

## Ajax优缺点

1. 优点：
   -  异步交互, 提高了用户体验!
   -  服务器只响应部分数据, 而不是整个页面, 所以降低了服务器的压力!
2. 缺点：
   - ajax不能应用所有的场景
   - ajax会无端的增加访问服务器的次数, 给服务器带来了压力!!

## 应用：

1. 原生态的js
2. Jquery封装好的



------



# 使用：JavaScript实现AJAX

## Ajax访问原理

通过js发送Ajax对象，服务器接收到请求并相应，js接收到响应，把响应的页面扔到一个当前页面的标签中，实现在当前页面中显示另外的页面

## 开发思路

> 1. 确定js函数的触发时机：用户名文本框失去焦点时触发
> 2. ajax发送局部请求，服务器端需要对应一个servlet去处理请求
> 3. 服务器返回的结果，需要显示在前端HTML页面中

## 创建Ajax程序的基本流程

> - 导入ajax.js脚本到项目`/js`目录中
> - 创建Ajax引擎对象`var xmlHttp = new XMLHttpRequest();`
> - 在需要功能的jsp页面引入`ajax.js`脚本，并编写JavaScript脚本实现Ajax
> - 创建XMLHttpRequest对象（通过ajax.js中的函数获取）
> - 打开与服务器端的连接
> - 发送请求
> - 重写onreadystatement函数，处理响应结果
>   - 判断Ajax状态码
>     - 判断响应状态码
>       - 获取响应内容
>       - 处理响应内容（js操作文档结构）



### 创建XMLHttpRequest对象

```javascript
 //创建Ajax引擎对象
 var ajax;
 ajax = new XMLHttpRequest();

//通过ajax.js获取XMLHttpRequest对象
var xmlHttp = ajaxFunction();
```



### 打开与服务器的连接

> `xmlHttp.open(method, url, async);`
>
> - `method`：设置请求数据格式（get/post）
>
> - `url`：设置请求到哪个servlet
> - `async`:表示设置同步代码执行和是异步代码执行。
>   - true：异步——默认为true
>   - false：同步

```javascript
//发起请求
xmlHttp.open("get", "ajax");
```



### 发送请求

> 此处发送的是请求数据，get，post数据在此处发送

```javascript
xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xmlHttp.send("username=" + value);
```



### 监听response的状态，获取/处理服务器返回的数据

> ==一般写在open和send方法之前==
>
> - 获取数据：ajax.responseText
>
>   `var result = ajax.responseText;`

- **readyState值**：`xmlHttp.readyState`

| **readyState** | **含义**                                                     |
| -------------- | ------------------------------------------------------------ |
| 0              | 表示XMLHttpRequest已建立，但还未初始化，这时尚未调用open方法 |
| 1              | 表示open方法已经调用，但未调用send方法（已创建，未发送）     |
| 2              | 表示send方法已经调用，其他数据未知                           |
| 3              | 表示请求已经成功发送，正在接受数据                           |
| ==4==          | ==表示响应结束，数据已成功接收==                             |

- **Http状态码**`xmlHttp.status`

| **Http状态码** | **含义**       |
| -------------- | -------------- |
| 200            | 请求成功       |
| 404            | 请求资源未找到 |
| 500            | 内部服务器错误 |

```javascript
//重写onreadystatement函数
ajax.onreadystatechange=function() {
    //判断Ajax状态码
    if(xmlHttp.readyState==4){
        //判断响应状态码
        if(xmlHttp.status==200){
            //获取响应内容
            var result = xmlHttp.responseText;
            //获取元素对象
            var showdiv = document.getElementById("showdiv");
            showdiv.innerHTML=result;
        }else if (xmlHttp.status==404){
            //获取元素对象
            var showdiv = document.getElementById("showdiv");
            showdiv.innerHTML="请求资源不存在";
        }else if(xmlHttp.status==500){
            //获取元素对象
            var showdiv = document.getElementById("showdiv");
            showdiv.innerHTML="服务器繁忙";

        }
    }
}
```



------



## Ajax的异步和同步

> `ajax对象.open(method,url,async)`
>
> - `async`:表示设置同步代码执行和是异步代码执行。
>   - true：异步——默认为true
>   - false：同步

- 示例代码

  ```javascript
  <script type="text/javascript">
      function getData() {
      	//创建Ajax引擎对象
      	var ajax;
      	ajax = new XMLHttpRequest();
  
          ajax.onreadystatechange=function() {
              //重写onreadystatement函数
              var showdiv = document.getElementById("showdiv");
              //判断Ajax状态码
              if(ajax.readyState==4){
                  //判断响应状态码
                  if(ajax.status==200){
                      //获取响应内容
                      var result = ajax.responseText;
                      //获取元素对象
                      showdiv.innerHTML=result;
                  }else if (ajax.status==404){
                      //获取元素对象
                      showdiv.innerHTML="请求资源不存在";
                  }else if(ajax.status==500){
                      //获取元素对象
                      showdiv.innerHTML="服务器繁忙";
                  }
              }else {//请求加载动画
                  //获取元素对象
                  var showdiv = document.getElementById("showdiv");
                  showdiv.innerHTML="<img src='img/2.gif' width='229px' height='247px'>";
              }
          }
          //发起请求
          ajax.open("get", "ajax");
          ajax.send(null);
  	}
  
  </script>
  ```

  



------



## Ajax的两种传参方式

### Get方式传参

- 直接通过url传参

- 注意：get方式提交经常会遇到浏览器缓存问题，浏览器不对同样的url重复提交。这时可以在url后面增加参数：

  - ？rand = Math.random() 或者 rand = new Date()

- 代码

  > 参数根据情况需要用到字符串拼接

  ```javascript
  //get方式
  ajax.open("get","ajax?name=张三&pwd=123");
  ajax.send(null);
  ```

### Post方式传参

> `application/x-www-form-urlencoded`——==告诉服务器表单是以键值对形式发送的==
>
> 没有这句话即使表单有数据，但是后台无法截取到数据

```javascript
//post方式
ajax.open("post","ajax");
ajax.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajax.send("name=张三&pwd=123456")
```



------



## Ajax获取响应内容的格式

> - 普通字符串
> - json（重点）
> - XML数据

### ==json（重点）==

> 通过`responseText`获取数据
>
> 查询信息，要把信息响应在当前页面，要使用Ajax；
>
> 点击查询按钮会在script代码中执行请求，并接收响应数据，响应的数据为另一个页面，无法把对象直接传回来，所以需要==json的数据格式传输==

- Json格式

  > {key:value,key2:value2,.....}
  >
  > - key:可以有引号：`"name"`;==但一般没有引号==：`name`
  > - value:可以有有引号，也可以没有引号

  ```json
  { "name":"runoob", "alexa":10000, "site":null };
  ```

- 访问对象

  ```json
  var myObj = { "name":"runoob", "alexa":10000, "site":null };
  
  //结果———————————————————————————————————————————
  myObj.name:runoob
  myObj.alexa:10000
  myObj.site:null
  ```

- #### 通过for-in循环对象的==属性==

  ```json
  var myObj = { "name":"runoob", "alexa":10000, "site":null };
  for (x in myObj) {
      document.getElementById("demo").innerHTML += x + "<br>";
  }
  //结果————————————————————————————————————————————
  name
  alexa
  site
  ```

- 通过for-in循环对象属性时， 使用中括号（[]）来==访问属性的值==：

  ```json
  var myObj = { "name":"runoob", "alexa":10000, "site":null };
  for (x in myObj) {
      document.getElementById("demo").innerHTML += myObj[x] + "<br>";
  }
  //结果——————————————————————————————————————————————
  runoob
  10000
  null
  ```

- ==在script中将json数据转换成对象==

  > Json数据：`{uid=1, uname='盖伦', price='1350', loc='上', desc='是个肉'}`

  ```javascript
  //获取响应数据
  var result = ajax.responseText;
  // alert(result);
  eval("var u="+result);//将json的数据转成对象
  
  //测试结果——————————————————————————————————————————
  u.uid:1
  u.uname:盖伦
  u.price:1350
  u.loc:上
  u.desc:是个肉
  ```


### XML（了解）

> 通过`responseXML`获取数据，返回document对象
>
> 通过document对象将数据从xml中获取出来

### servlet代码：响应Json数据

> 可以通过Gson.jar包快速的将数据转换成json格式

```java
//获取请求信息
String name = req.getParameter("name");
System.out.println("用户请求信息为："+name);
//调用service层服务
User user = us.showUserInfo(name);
System.out.println(user);

//响应处理结果
//将查询到的数据以json的数据格式响应
resp.getWriter().write(new Gson().toJson(user));

```

### JSP中的script代码，接收Json格式的数据

```javascript
//获取响应数据
var result = ajax.responseText;
// alert(result);
eval("var u="+result);//将json的数据转成对象
```



# 使用：jQuery实现AJAX

> Jquery已经封装好了Ajax的函数

## 开发步骤

1. 导入jQuery.js到项目web/js文件中
2. 在对应的jsp页面中写jQuery代码



## 方法一：load

> `$(selector).load(url,data,callback);`
>
> - `selector` -- 选择器, 将从服务器获取到的数据加载到指定的元素中
> - `url` -- 发送请求的URL地址
> - `data` -- 可选, 向服务器发送的数据 key/value数据 如:`{"username":"张飞","psw":"123"}`
> - `callback` -- 可选, load方法完成后所执行的函数

```java
<script>
    /*使用jQuery实现Ajax*/
    $(function () {
        $("input[name='username']").blur(function () {
            var value = $("input[name='username']").val();
            console.log(value);
            //jQuery实现Ajax：方式一：load
            $("#uid_msg").load("/ajaxCheckUsername",{"username":value});
        })
    });
</script>
    
    
<input type="text" name="username" value="<%=name==null?"":name%>" />
<span id="uid_msg" style="color: red;font-size: 20px"></span>
```



## 方法二：get

> `$.get(url, [data], [callback]);`
>
> - url -- 发送请求的URL地址
> - data -- 可选, 向服务器发送的数据
> - callback -- 可选, 请求成功后所执行的函数

```js
//jQuery实现Ajax：方式二：get方式发送请求
$.get("/ajaxCheckUsername",{"username":value},function (result) {
    console.log(result);
    //把result结果显示到span元素中
    $("#uid_msg").text(result);
});


$.get("/ajaxCheckUsername?username="+value,function (result) {
    console.log(result);
    //把result结果显示到span元素中
    $("#uid_msg").text(result);
});
```



## 方法三：post

> `$.post(url, [data], [callback]);`
>
> - url -- 发送请求的URL地址
> - data -- 可选, 向服务器发送的数据
> - callback -- 可选, 请求成功后所执行的函数

```js
//jQuery实现Ajax：方式三：post方式发送请求
$.post("/ajaxCheckUsername",{"username":value},function (result) {
    console.log(result);
    //把result结果显示到span元素中
    $("#uid_msg").text(result);
});
```



## 方法四：（==经常使用的==）ajax

> `$.ajax(url, [data], [async], [callback]);`
>
> - url -- 发送请求的URL地址
> - data -- 可选, 发送至服务器的key/value数据
> - async -- 可选, 默认为true, 表示异步交互(==可以省路==)
> - type -- 可选, 请求方式 , ==默认为"GET"==。
> - dataType -- 指定data的数据格式，一般为json（==可以省略==）
> - success -- 可选, 请求成功后执行的函数, 函数参数:
>   - result -- 服务器返回的数据
> - error -- 可选，请求失败后执行的函数，函数参数:
>   - result -- 服务器返回的数据

```javascript
//jQuery实现Ajax：方式四：可以自定义参数内容，不如指定：get/post
$.ajax({
    url:"/ajaxCheckUsername",
    data:{"username":value},
    type:"post",/*请求的方法*/
    async:true,/*异步请求，可以省略*/
    //dataType:json,/*指定data的数据格式，可以省略*/
    success:function (result) {/*成功后响应结果*/
        $("#uid_msg").text(result);
    },
    error:function () {//url资源路径不存在
        $("#uid_msg").text("服务器404错误");
    }
});
```

