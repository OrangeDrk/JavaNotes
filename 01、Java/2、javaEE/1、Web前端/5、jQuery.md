[TOC]



# jQuery

## 介绍

- 介绍
  - 完全基于js（JavaScript）开发出来的另外一套脚本语言。本质上来说js与jQuery的功能、作用是完全等价的
  - jQuery是一个前端技术的脚本框架，内部完全封装了js脚本。
  - 封装：把js的代码以一种更加简单，更加容易学习，更加面向对象的思想去设计出来的
  - 特点：简单易学
- 使用
  - jQuery是第三方开发的，需要导入项目中使用
  - jQuery-1.4.2.js 
    - 特点：文件中有注释，代码是还行的，版面工整
    - 缺点：文件较大，一般用于学习和测试环境
  - jQuery-1.4.2.min.js
    - 特点：所有代码没有换行，没有注释
    - 缺点：阅读性较差，但是文件小，一般生产环境使用
  - 两个版本，本质上没有区别，功能相同
- jQuery的优势
  - 可以简化JavaScript代码
  - 可以像css那样获取元素
  - 可以修改css来控制页面效果
  - 可以兼容常用的浏览器



# jQuery核心用法

## 主要格式

> `$("#id值")`,`$(".class名")`,`$("元素名")`

- $("css选择器")选出的元素是jQuery对象，而不是普通的dom对象
- $("HTML元素")可以直接将字符串描述的html元素创建出来，构成一个jQuery对象。
- $(document).ready(function(){…….});
  - 可以设定在页面加载完成之后执行的函数，也可以简写成
  - `$(function(){...});`

## jQuery引入

> jQueyr类库其实就是一个普通的js文件, 和之前在html中引入js文件方式是一样的！！！ 

```javascript
<script src="../js/jquery-1.4.2.js"></script>
```





# jQuery语法

## $介绍

- $符号等价于jQuery，是jQuery单词的简写
- 例如：`$()`相当于调用jQuery()。该函数会返回一个jQuery对象，这个jQuery对象包含0个或多个html元素。
  - 比如：`$("div")`:通过jQuery中提供的方法来操作这些匹配的元素，比如：`$(“div”).remove();`

## 文档就绪事件

> 相当于`window.onload`

- 格式：

  > 该函数会在整个html文档加载完之后立即执行! 其作用相当于: 
  > 	`window.onload = function(){ //xxx  }`

  ```javascript
  $(document).ready(function(){
      //xxxx
  });
  ```

- ==简写格式：==

  ```javascript
  <script>
      //案例1：操作div，并给div中添加文本元素：您好
      /*window.onload = function () {
      	document.getElementById("d1").innerText = "xxx";
         }
        */
      //使用jQuer取代上一句页面加载完毕的代码
      $(function () {
      	alert(123);
      	$("#d1").text("您好");
  	});
  </script>
  ```

## DOM与jQuery的互转

> JS对象只能调用JS上提供的属性和方法, 不能调用jQuery提供的属性和方法, 如果非要使用, 必须将JS对象转化成jQuery对象！反之亦然。 

**HTML代码：**

```html
<div id="d1"></div>
<div></div>
<div></div>
<div></div>
```

### 获取jQuery对象

> 总结：jQuery对象是把js对象重新包装了一次
>
> - jQuery对象的第0号元素就是被包装的js对象

```javascript
//获取jQuery的一个对象
var $d1 = $("#d1");
console.log(d1);

//jQuery对象输出结果:
/*	init(1)
 *		0: div#d1
 *		length: 1
 *		context: document
 *		selector: "#d1"
 *		__proto__: Object(0)
 */
```

### DOM-->jQuery

```javascript
var dd = document.getElementById("d1");
var $obj = $(dd);
```

### jQuery-->DOM

```javascript
var $d1 = $("#d1");
var jsobj = $d1[0];//方式一
var jsobj = $d1.get(0);//方式二
```



## jQuery获取所有div元素

> 如果超出数组范围，返回undefined

```javascript
var $divs = $("div");
console.log($divs);
console.log($divs[0]);
console.log($divs[1]);
console.log($divs[2]);
console.log($divs[3]);
console.log($divs[4]);//如果超出数组范围，返回undefined
```



------



## jQuery选择器



### 💡基本选择器

> 📌元素名选择器:`$("div")`
>
> 📌class选择器:`$(".mini")`
>
> 📌id选择器:`$(".mini")`
>
> 📌多元素选择器:`$("span,#two")`
>
> `*`选择器:`$("*")`

#### 元素选择器

```javascript
$("div").css("backgroundColor","#FF69B4");
```

#### id选择器

```javascript
$("#one").css("background", "#9ACD32");
```

#### class选择器

```javascript
$(".mini").css("background","#FF0033");
```

#### *匹配所有的元素

```javascript
$("*").css("background", "#FDF5E6");
```

#### 多元素选择器：任意元素类型

```javascript
$("span,#two").css("background", "#FF1493");
```



------



### 💡层级选择器

1. 父元素下的所有==后代==元素

   ```javascript
   $("body div").css("background","#F08080");
   ```

2. 父元素下的所有的==子元素==

   ```javascript
   $("body>div").css("background","#FF0033");
   ```

3. 该节点后==紧邻==的==兄弟==节点

   > 只改变与该节点同一等级的兄弟节点，不包括兄弟节点的子节点

   ```javascript
   $("#one+div").css("background","#0000FF");
   ```

4. 匹配该节点==后面==所有的指定==兄弟==元素 

   ```javascript
   $("#two~div").css("background", "#9ACD32");
   ```

5. 该节点==所有==的==同级兄弟==节点

   ```javascript
   $("#two").siblings("div").css("background", "#FF6347");
   ```



------



### 💡基本过滤选择器

1. 匹配所有div中的==第一个==

   ```javascript
   $("div:first").css("background","#FF6347");
   ```

2. 匹配所有div中的==最后一个==

   ```javascript
   $("div:last").css("background","#9ACD32");
   ```

3. 匹配所有div中class==不为one的==

   ```javascript
   $("div:not(.one)").css("background","#FF0033");
   ```

4. 匹配所有div中索引值==等于==3的

   > 索引值从0开始计数

   ```javascript
   /*索引值从0开始计数*/
   $("div:eq(3)").css("background","#006400");
   ```

5. 匹配所有div中索引值==小于==3的

   ```javascript
   $("div:lt(3)").css("background","#FF69B4");
   ```

6. 匹配所有div中索引值==大于3==的

   ```javascript
   $("div:gt(3)").css("background","#F08080");
   ```

7. 匹配所有div中索引值为==偶数==的

   ```javascript
   $("div:even").css("background","#FF6347");
   ```

8. 匹配所有div中索引值为==奇数==的

   ```javascript
   $("div:odd").css("background","#FF1493");
   ```



------



### 内容选择器

1. 📌匹配所有==包含xxx内容==的div

   ```javascript
   $("div:contains('xxx')").css("background", "#FF6347");
   ```

2. 匹配内容为==空==的div（不常用）

   ```javascript
   $("div:empty").css("background", "#9ACD32");
   ```

3. 📌匹配==包含x元素==的div

   ```javascript
   $("div:has(div)").css("background", "#FF0033");
   ```

4. 📌匹配==非空==的div

   ```javascript
   $("div:parent").css("background", "#006400");
   ```



------



### 可见选择器

1. 匹配==所有可见的==div

   > `:visible`：可见的

   ```javascript
   $("div:visible").css("background-color","#F08080");
   ```

2. 匹配==所有不可见的==div

   > `:hidden`：不可见的
   >
   > `.show()`：相当于把display设置为了block

   ```javascript
   $("div:hidden").css("background-color","red").show();
   //$("div:hidden").css("display","block");这样也可以
   ```



------



### 💡属性选择器

> `^`：以xx开头
>
> `&`：以xx结尾

1. 匹配==有id属性==的div元素

   ```javascript
   $("div[id]").css("background", "#FF6347");
   ```

2. 匹配==包含title属性==的div

   > 不包含该节点的子节点，子节点没有tittle

   ```javascript
   /*不包含该节点的子节点，子节点没有tittle*/
   $("div[title='aa']").css("background", "#9ACD32");
   ```

3. 匹配==type属性值不等于button==的input元素

   > 不包含该节点的子节点

   ```javascript
   /*不包含该节点的子节点*/
   $("input[type!='button']").css("background", "#FF0033");
   ```

   

------



### 子元素选择器

1. 匹配==div元素中==的==第二个子元素==

   > 子元素的下标从1开始

   ```javascript
   $("div:nth-child(2)").css("background-color","#006400");
   ```

2. 匹配==div元素中==的==第一个子元素==

   ```javascript
   $("div:first-child").css("background-color","#FF69B4");
   ```

3. 匹配==div元素中==的==最后一个子元素==

   ```javascript
   $("div:last-child").css("background-color","#FF69B4");
   ```



------



### 💡表单选择器

> 开头的格式是  ------>`:`
>
> | 代码                 | 介绍                                                         |
> | -------------------- | ------------------------------------------------------------ |
> | `$(":input")`        | 匹配所有的input文本框、密码框、单选框、复选框、select框、textarea、button |
> | `$(":password")`     | 匹配所有的==密码框==                                         |
> | `$(":radio")`        | 匹配所有的==单选框==                                         |
> | `$(":checkbox")`     | 匹配所有的==复选框==                                         |
> | `$(":checked")`      | 匹配所有的==被选中的==单选框/复选框/option                   |
> | `$("input:checked")` | 匹配所有的==被选中==的单选框/复选框                          |
> | `$(":selected")`     | 匹配所有==被选中的option选项==                               |

1. 匹配所有的input

   ```javascript
   $(":input").css("background","#F08080");
   ```

2. 匹配所有的password

   ```javascript
   $(":password").css("background","#9ACD32");
   ```

3. 匹配所有的radio

   > radio元素【单选框】的个数

   ```javascript
   alert($(":radio").length);
   ```

4. 匹配所有checked

   > 所有被选中的【单选/复选/下拉表】元素的value值

   ```javascript
   $(":checked").each(function () {
       /*$(this)  指的是当前遍历的数组中的一个元素对象*/
       alert($(this).val());
   });
   ```

5. 匹配所有的selected

   > 被选中的【复选框】中的value值

   ```javascript
   alert($(":selected").val());
   ```

   

# 案例（jQuery方法）

## QQ列表

> jQuery的方法：
>
> 1. `toggle()`：切换元素的可见状态, 如果元素显示则设置为隐藏, 如果元素隐藏则设置为可见.
> 2. `next("元素名")`：当前节点的下一个指定元素
> 3. `父元素.find("元素名")`：寻找父元素下的指定元素
> 4. `jq对象.hide()`：将jq对象隐藏
> 5. `parents("元素名")`：寻找该节点的指定父节点
> 6. `siblings("元素名")`：匹配该节点前后的兄弟节点

```javascript
function openDiv(thisobj){
    console.log(thisobj);//1.thisobj对象是：原生态js对象

    //2.转换成jQuery对象
    //toggle() ---> 如果div是关闭的就打开，反之关闭
    $(thisobj).next("div").toggle();
    //3.thisobj点击时，需要把其他的好友列表关闭
    $(thisobj).parents("tr").siblings("tr").find("div").hide();

}
```



## 二级联动下拉框

> jQuery方法
>
> 1. `val()`：获取该对象的value值
> 2. `html("html内容")`：向该对象中输出html语句
> 3. `text("文本内容")`：向该对象中输出文本语句
> 4. `append("html内容")`：向该对象下追加新的html内容，可以写成字符串
> 5. `appendChild(new Child)`：向该对象下追加新的HTML，但必须是new，新建的标签

```javascript
function selectCity(thisobj){
    //1.获取一级下拉框选择的value值
    var pro = $(thisobj).val();
    //2.根据value值去匹配json中的数组
    var citys = data[pro];
    //3.获取二级下拉框对象
    var city = $("#city");
    city.html("<option>--选择城市--</option>");
    //4.遍历第2步的数组，把元素添加到二级下拉框中
    for (var i = 0; i < citys.length; i++) {
        /*city对象是select标签对象
			* appendChile()函数，是添加option子对象
			* */
        city.append("<option>" + citys[i] + "</option>");
        /*append() 参数可以是字符串类型
			* appendChild() 参数是一个newChild，新的节点
			* */
    }
}
```

# 其他常用方法

1. `.attr(attributeName,value);`

   ```js
   $( "#greatPhoto" ).attr({
     alt: "Beijing Brush Seller",
     title: "photo by Kelly Clark"
   });
   ```

2. `toggle()`：切换元素的可见状态, 如果元素显示则设置为隐藏, 如果元素隐藏则设置为可见.

3. `next("元素名")`：当前节点的下一个指定元素

4. `父元素.find("元素名")`：寻找父元素下的指定元素

5. `jq对象.hide()`：将jq对象隐藏

6. `parents("元素名")`：寻找该节点的指定父节点

7. `siblings("元素名")`：匹配该节点前后的兄弟节点

8. `val()`：获取该对象的value值

9. `html("html内容")`：向该对象中输出html语句

10. `text("文本内容")`：向该对象中输出文本语句

11. `append("html内容")`：向该对象下追加新的html内容，可以写成字符串

12. `appendChild(new Child)`：向该对象下追加新的HTML，但必须是new，新建的标签

## 查找

```js
$("div").children()      //div中的每个子元素,第一层
$("div").find("span")    //div中的包含的所有span元素,子子孙孙

$("p").next()       　　　//紧邻p元素后的一个同辈元素
$("p").nextAll()         //p元素之后所有的同辈元素
$("#test").nextUntil("#test2")    //id为"#test"元素之后到id为'#test2'之间所有的同辈元素,掐头去尾

$("p").prev()            //紧邻p元素前的一个同辈元素
$("p").prevAll()         //p元素之前所有的同辈元素
$("#test").prevUntil("#test2")    //id为"#test"元素之前到id为'#test2'之间所有的同辈元素,掐头去尾

$("p").parent()          //每个p元素的父元素
$("p").parents()         //每个p元素的所有祖先元素,body,html
$("#test").parentsUntil("#test2")    //id为"#test"元素到id为'#test2'之间所有的父级元素,掐头去尾

$("div").siblings()      //所有的同辈元素,不包括自己
```



------



## 属性操作：

### 基本属性操作：

```js
$("img").attr("src");    　　　　　　 //返回文档中所有图像的src属性值
$("img").attr("src","test.jpg");    //设置所有图像的src属性
$("img").removeAttr("src");    　　　//将文档中图像的src属性删除

$("input[type='checkbox']").prop("checked", true);    //选中复选框
$("input[type='checkbox']").prop("checked", false);
$("img").removeProp("src");    　　 //删除img的src属性
```

### HTML代码/文本/值

```js
$('p').html();    　　　　　　　　　　 //返回p元素的html内容
$("p").html("Hello <b>nick</b>!");  //设置p元素的html内容
$('p').text();    　　　　　　　　　　 //返回p元素的文本内容
$("p").text("nick");    　　　　　　　//设置p元素的文本内容
$("input").val();    　　　　　　　　 //获取文本框中的值
$("input").val("nick");     　　　　 //设置文本框中的内容
```

## 事件

```js
$("p").click();    　　//单击事件
$("p").dblclick();    //双击事件
$("input[type=text]").focus()  //元素获得焦点时,触发 focus 事件
$("input[type=text]").blur()   //元素失去焦点时,触发 blur事件
$("button").mousedown()//当按下鼠标时触发事件
$("button").mouseup()  //元素上放松鼠标按钮时触发事件
$("p").mousemove()     //当鼠标指针在指定的元素中移动时触发事件
$("p").mouseover()     //当鼠标指针位于元素上方时触发事件
$("p").mouseout()    　//当鼠标指针从元素上移开时触发事件
$(window).keydown()    //当键盘或按钮被按下时触发事件
$(window).keypress()   //当键盘或按钮被按下时触发事件,每输入一个字符都触发一次
$("input").keyup()     //当按钮被松开时触发事件
$(window).scroll()     //当用户滚动时触发事件
$(window).resize()     //当调整浏览器窗口的大小时触发事件
$("input[type='text']").change()    //当元素的值发生改变时触发事件
$("input").select()    //当input 元素中的文本被选择时触发事件
$("form").submit()     //当提交表单时触发事件
$(window).unload()     //用户离开页面时
```

