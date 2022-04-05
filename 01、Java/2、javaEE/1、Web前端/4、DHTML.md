[TOC]



# DHTML

> 根据DHTML的对象使用js操作HTML页面

DHTML：

- Dynamic HTML：动态的HTML页面的概念

- DHTML不是一门语言，更不是一门技术，而是一个集HTML、CSS、JS的技术集。是一种设计层面的概念
- DHTML：是将各个标签元素，以及CSS的各个属性转换成JS可以操作的对象
- DHTML种的各个对象，都可以被JS去操作，那么当找到其中一个节点时，该节点的树形结构的层级关系也就清晰了
- DHTML可以分为由BOM(Browser Object Model)和DOM(Document Objecect Model)两部分组成。

DHTML模型图解：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110727.png)



------



## BOM对象模型

> BOM：Browser Object Model（浏览器对象模型）
>
> BOM对象模型中，顶级的对象：window对象
>
> - window对象指的是：浏览器的窗口
> - ==共有9个对象==
>
> **BOM对象模型图：**
>
> ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110728.png)

### window窗口对象

#### `window.onload`

> 页面加载完毕后执行的代码，onload表示触发的事件

```javascript
window.onload = function () {
    //向div中输出一句话
    document.getElementById("d1").innerText = "123";
};
<div id="d1"></div>
```

#### `window.onblur`

> 鼠标失去焦点(鼠标移出界面)时触发

```javascript
window.onblur = function () {
    document.getElementById("d1").innerText = "页面失去焦点";
};
<div id="d1"></div>
```

#### `window.onfocus`

> 鼠标获得焦点(鼠标进入界面)时触发

```javascript
window.onfocus = function () {
    document.getElementById("d1").innerText = "页面获取焦点";
};
<div id="d1"></div>
```

#### `window.prompt()`

> 弹出对话框，带有提示信息和输入框的方法

```javascript
var password = window.prompt("请输入您的密码");//返回输入的值

if (password == "123") {
    alert("登陆成功");
}else {
    alert("登陆失败");
}
```

#### `window.confirm()`

> 确认对话框，带有提示信息和确认/取消按钮

```javascript
var check = window.confirm("确定要删除吗？")
if (check) {//true
    alert("删除成功")
}else {//false
    alert("取消删除");
}
```



### window子对象

#### `history`

> 历史对象
>
> - 前进`forward()`/`go(1)`
> - 后退`back()`/`go(-1)`

```javascript
function demo1() {
    //前进的方法:forward()
    //window.history.forward();
    //前进方法：go()
    window.history.go(1);//1:前进，-1：后退
}

function demo2() {
    //后退的方法:back()
    //window.history.back();
    //后退的方法：go()
    window.history.go(-1);//1:前进，-1：后退
}
```

#### `location`

> 包含了当前URL的所有信息
>
> `window.location.href`：获取当前url的地址
>
> `window.location.hostname`：获取当前url的主机名

```javascript
<script>
    console.log(window.location.href);
    console.log(window.location.hostname);//localhost
</script>
//输出信息：
/*http://localhost:63342/javawebCode/day03-js/
  DHTML-BOM02-window%E7%9A%84%E5%AD%90%E5%AF%B9%E8%B1%A1.html
  ?_ijt=l7c3r9j3hs4vluftmei2mqrt51
*/
```

#### `navigator`

> 包含web浏览器的信息
>
> `window.navigator.appVersion`：获取浏览器的版本

```javascript
<script>
	console.log(window.navigator.appVersion);
</script>
//结果：
/*5.0 (Windows NT 10.0; WOW64) AppleWebKit/
  537.36 (KHTML, like Gecko) Chrome/
  79.0.3928.4 Safari/537.36
*/
```

#### `screen`

> 屏幕对象：包含屏幕和渲染能力的对象

```javascript
<script>
	console.log(window.screen.width);//1920
	console.log(window.screen.height);//1080
</script>
```



------



## DOM对象模型

> DOM：Document Object Model（文档对象模型）
>
> DOM文档对象的树形结构

### DOM模型图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110729.png)



### 获取文档对象及数据

> HTML代码

```html
    用户名称：<input type="text" name="username" id="username"/><br />
    用户密码：<input type="password" name="password" id="password" /><br />
    用户密码2：<input type="password" name="password" id="password2" /><br />
    
    <hr />
    <input type="button" value="通过ID获取节点的值" onclick="demo1()"/>
    <input type="button" value="通过NAME获取节点的值"  onclick="demo2()" />
    <input type="button" value="通过TAG获取节点的值" onclick="demo3()" />        
    
    <hr  />
    <p id="pid"><font color="red">获取P标签中的文字</font></p>
	<input type="button" value="获取P中文字" onclick="demo4()" />
```



> 3种（id,class,元素名）

- 方法：

  | 方法                             | 介绍                                       |
  | -------------------------------- | ------------------------------------------ |
  | `getElementById("id")`           | 通过id获取数据：id表示标签的唯一标识符     |
  | `getElementsByName("name")`      | 通过name获取数据：name表示标签中的name属性 |
  | `getElementsByTagName("标签名")` | 通过HTML的标签名获取数据                   |
  | `innerText`                      | 设置或获取位于对象起始或结束标签内的文本   |
  | `innerHtml`                      | 设置或获取位于对象起始和结束标签内的HTML   |

#### `getElementById("id")`

```javascript
function demo1() {
    //通过id获取数据：id表示标签的唯一标识符
    //1.获取input对象
    var username = document.getElementById("username");
    console.log(username);//对象
    console.log(username.value);//输入的值
}
```

#### `getElementsByName("name")`

```javascript
function demo2() {
    //通过name获取数据：name表示标签中的name属性
    //返回结果是：数组
    var pwds = document.getElementsByName("password");
    console.log(pwds);
    //获取pwds数组中的第一个对象：
    var p1 = pwds[0];
    console.log(p1);//-----------------获取对象
    //获取p1对象中的value值
    console.log(p1.value);//-----------获取对象的值
}
```

#### `getElementsByTagName("标签名")`

```javascript
function demo3() {
    /*通过标签的名字（也就是元素名称）去获取所有的统一名称的元素对象*/
    var inputs = document.getElementsByTagName("input");
    console.log(inputs);
    //使用数组下标获取value值
    for (let i = 0; i < inputs.length; i++) {
        console.log(inputs[i].value);
    }
}
```

**结果：**

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110730.PNG)



------



#### 获取文本内容/修改文本内容

> `innerText`：获取内部的内容
>
> `innerHTML`：获取内部的标签
>
> `innerHTML = "<font color='green'>哈哈哈哈😄<font>"`：修改内部标签

```javascript
<p id="pid"><font color="red">获取P标签中的文字</font></p>
<input type="button" value="获取P中文字" onclick="demo4()" />


function demo4() {
    /*1.    获取p段落中的字*/
    var p = document.getElementById("pid");
    console.log(p);// 打印p段落元素对象

    //2.    获取内部的文本----获取P标签中的文字
    console.log(p.innerText);

    //3.    获取内部的HTML标签----<font color="red">获取P标签中的文字</font>
    console.log(p.innerHTML);

    //4.    动态修改内部的HTML标签
    p.innerHTML = "<font color='green'>哈哈哈哈😄<font>";
}
```

**结果**

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110731.PNG)

![inner2](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110732.PNG)

![inner3](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110733.PNG)



------



### 对文本对象的增删改

> HTML代码

```html
<body>
    <div id="div_1"></div>
    <div id="div_2">div区域2</div>
    <div id="div_3">div区域3 </div>
    <div id="div_4">div区域4</div>
    <hr />
    <input type="button" value="创建并添加节点" onclick="addNode()" />
    <input type="button" value="删除节点" onclick="deleteNode()" />
    <input type="button" value="替换节点" onclick="updateNode()" />
    <input type="button" value="克隆节点" onclick="copyNode()" />

</body>
```



#### `createElement("元素名称")`：新建

> 1. 创建节点对象：`createElement("div")`
> 2. 获取父节点对象：`getElementsByTagName("body")[0]`
> 3. 将新节点对象插入到父节点对象中
>    - 向最后添加元素：`父节点对象.appendChild(新节点对象)`
>    - 向某一节点前添加元素：`父节点对象.insertBefore(新节点对象, 目标节点对象)`

```javascript
function addNode() {
    /*创建一个节点，并把新节点添加到页面中*/
    //1.    创建一个新的div节点:createElement("元素名称")
    var newdiv = document.createElement("div");
    newdiv.style.backgroundColor = "#00FFFF";//添加样式
    newdiv.innerText = "div区域1";

    //2.    获取父节点对象：body对象,返回的是一个数组，第1个元素就是body
    var body = document.getElementsByTagName("body")[0];

    //3.    将newdiv插入到body中。向最后添加元素
    //body.appendChild(newdiv);

    //4.1   先获取div_2的节点：在该节点前面插入
    var div2 = document.getElementById("div_2");
    //4.2   确定新节点依然是body的孩子节点
    body.insertBefore(newdiv, div2);
}
```

#### `removeChild(节点对象)`：删除

> 1. 获取父节点对象：`getElementsByTagName("body")[0]`
> 2. 获取子节点对象：`document.getElementById("div_2")`
> 3. 删除父节点对象中的子节点对象：`父节点对象.removeChild(子节点对象);`

```javascript
function deleteNode() {
    //1.    获取父节点对象：body
    var body = document.getElementsByTagName("body")[0];
    //2.    获取子节点对象：div_2
    var div_2 = document.getElementById("div_2");
    //3.    父节点删除其中一个子节点
    body.removeChild(div_2);
}
```

#### `replaceChild(新节点对象，被替换的节点对象)`：修改

> 1. 获取父类节点对象：`getElementsByTagName("body")[0]`
> 2. 获取被替换的对象：`getElementById("div_3")`
> 3. 新对象：`createElement("div")`
> 4. 替换：`父节点对象.replaceChild(新节点对象, 被替换的节点对象)`

```javascript
function updateNode() {
    //1.    获取父节点对象
    var body = document.getElementsByTagName("body")[0];
    //2.    获取要被替换的节点对象div_3
    var div_3 = document.getElementById("div_3");
    //3.    生成一个新节点
    var newdiv = document.createElement("div");
    //4.    把div_3替换成新节点
    body.replaceChild(newdiv, div_3);
}
```

#### `cloneNode(boolean b)`：克隆

> 1. 获取父节点对象：`getElementsByTagName("body")[0]`
> 2. 获取要克隆的对象：`getElementById("div_4")`
> 3. 克隆：`被克隆对象.cloneNode(true)`
>    - true：完全克隆（克隆样式+内容）
>    - false：只克隆样式，不克隆内内容（默认为false）
> 4. 添加：`父节点对象.appendChild(新节点对象)`

```javascript
function copyNode() {
    //1.    获取div_4对象
    var div_4 = document.getElementById("div_4");

    //2.    获取body对象
    var body = document.getElementsByTagName("body")[0];

    //3.    克隆div_4对象
    //不写参数默认为false：只克隆样式，不能克隆内容
    //true：完全克隆（克隆样式和内容）
    var newdiv = div_4.cloneNode(true);

    //4.    追加克隆的div对象
    body.appendChild(newdiv);
}
```



------



### 调整样式

> 1. `元素名.style.CSS标签 = ”CSS值“；`
> 2. `元素名.className = “css样式选择器的名字”`

#### 通过style属性修改样式

> `div.style.backgroudColor="#f00";`

```javascript
var newdiv = document.createElement("div");
newdiv.style.backgroundColor = "#00FFFF";//添加样式
```

#### 通过class属性修改样式

> `div.className = "css样式选择器的名字";`
>
> **案例**：通过点击【小字体/中字体/大字体】来切换内容的样式

- JS

```javascript
function resize(obj) {
    //console.log(obj);//1.obj形参:点击事件传递的CSS样式名(min/newsstyle/max)
    //2.将样式赋值给对应标签的class属性
    document.getElementById("newstext").className = obj;
}
```

- HTML

```css
.max {....}
.min {....}

<body>
    <a href="javascript:void(0)" onclick="resize('min')">小字体</a> 
    <a href="javascript:void(0)" onclick="resize('newsstyle')">中字体</a> 
    <a href="javascript:void(0)" onclick="resize('max')">大字体</a> 
    <hr />
    <div id="newstext" class="newsstyle">
    演示接口.很多内容.怎么办呢?等等<br />
    演示接口.很多内容.怎么办呢?等等<br />
	......
    </div>
</body>
```



#### 通过display属性修改样式

> `div.style.display = "none"|"block";`
>
> ==扩展：`nextSibling` 获取对此对象的下一个兄弟对象的引用。==
>
> - `nextSibling`中，回车和空格也算一个对象



> **案例**：实现点击列表后，展开具体的好友列表；其他的还有列表关闭

- JS

```javascript
<script type="text/javascript">
    function openDiv(obj) {
        var div = obj.nextSibling.nextSibling;//获取:div好友列表
        // div.className = "open";//3.	给div添加 打开（.open） 样式
    
        //需要判断，如果好友列表原来是展开的，单击后就要关闭，否则，打开
        if (div.className == "open") {
            div.className = "close";
        } else {
            div.className = "open";
        }

        //当点击其中一个好友列表时，其他好友列表是关闭状态的
        var divs = document.getElementsByTagName("div");//获取所有div
        for (var i = 0; i < divs.length; i++) {
            if (divs[i] != div) {//判断是否是已经点击的div
                //除了点击的div,其他的div状态设置为关闭即可
                divs[i].className = "close";
            }
    	}
	}
</script>
```

- HTML

```css
.open {display: block; /*显示div*/}
.close {display: none; /*不现实div*/}

<body>
<table>
    <tr>
        <td>
            <a href="javascript:void(0)" onclick="openDiv(this)">君王好友</a>
            <div>
                秦始皇<br/>刘邦<br/>李世民<br/>康熙<br/>
            </div>
        </td>
    </tr>
    <tr>.....</tr>
    <tr>.....</tr>
    <tr>.....</tr>
</table>
</body>
```



------



## 二级联动菜单

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>二级联动菜单</title>
    <script>
		function selectCity(obj) {
			//1.	定义一个json对象，描述 省：市 之间的关系
			var data = {
				"北京市": ["海淀区", "朝阳区", "丰台区"],
				"河北省": ["石家庄", "唐山", "秦皇岛"],
				"辽宁省": ["沈阳", "大连", "鞍山"],
				"山东省": ["青岛", "济南", "烟台"]
			};

			//2.	获取第一级下拉框的内容
			var s1 = document.getElementById("province").value;
			console.log(s1);

			//3.	根据s1的值获取json数据中的数组
			//s1代表的是json的key，数组就是json中s1对应的value
			var citys = data[s1];
			console.log(citys);

			//4.	把citys数组循环遍历，插入到二级下拉框中
			var s2 = document.getElementById("city");
			//s2.innerHTML = "";//每一次选择时先清空原来的HTML
			//每一次选择时先清空原来的HTML,并初始化原有的option
			s2.innerHTML = "<option>--选择城市--</option>";
			for (var i = 0; i < citys.length; i++) {
				s2.innerHTML += "<option>"+citys[i]+"</option>";
			}
		}
    </script>
</head>

<body>
	<select id="province" onchange="selectCity(this)">
		<option>--选择省市--</option>
		<option>北京市</option>
		<option>河北省</option>
		<option>辽宁省</option>
		<option>山东省</option>
	</select>
	<select id="city">
		<option>--选择城市--</option>
	</select>
</body>
</html>
```

