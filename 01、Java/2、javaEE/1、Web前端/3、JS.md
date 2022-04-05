[TOC]



# JS

## 概述

> js单独使用的话，可以单独写出页面小游戏（比如飞机大战等等）
>
> js与HTML结合使用，可以做出目前互联网网站的所有特效
>
> js是JavaScript的缩写，JavaScript是目前前端技术中非常流行
>
> js是一门变成语言（具有逻辑性，语法与Java有很多相似之处）
>
> js是浏览器端的技术，主要为HTML页面增加动态效果。
>
> js是为HTML这种页面布局进行提供动态支持
>
> 主要应用在客户端，在服务器端也有所应用（Node.js）

- js的特点：
  - 脚本语言没有编译过程
  - 基于对象
  - 弱类型语言
- js一般叫做：web前端的脚本语言
- 特性：
  - 交互性
  - 安全性
  - 跨平台性



## 常用写法：

- `<head>`标签中

  ```html
  <head>
      <script>
          /*打印语句*/
          alert(111);
      </script>
  </head>
  ```

- 外部文件引用

  ```html
  <script src="js01.js"></script>
  ```

- 页面的底部

  ```html
  <html>
  <body>
  </body>
      <!--书写方式三：js代码写在页面的底部-->
      <script>
          alert(333);
      </script>
  </html>
  ```

- 直接写在HTML标签中（不常用）

  ```javascript
  <a href="javascript:alert(123)">测试链接</a>
  ```

  

## JavaScript的输出语句

> js的输出方式一般有：4种
>
> `alert(123)`:弹窗打印
>
> `console.log()`：控制台打印
>
> `document.write("你好，JavaScript")`：向body中输出信息
>
> `document.getElementById("p1").innerText = "这是新的内容";`：对某一个标签做修改

```html
<script>
    /*1.弹窗打印：alert()*/
    alert(1);

    /*2.浏览器的控制台打印：主要是为了打桩测试
     *   console.log();
     **/
    console.log("haha");

    /*3.向浏览器的body中输出信息*/
    document.write("你好，JavaScript");

 	/*4.对HTML页面的某一个标签做修改*/
    document.getElementById("p1").innerText = "这是新的内容";

</script>
<body>
    <p id="p1">测试</p>
</body>
```



## JavaScript基本语法

> 1. 弱类型语言：所有类型统一使用var进行定义
> 2. 不写分号，语句正常运行：js解析器可以自动添加语句的分号
> 3. 单引号，双引号：单独成句时没有区别，都是字符串
>
>   - 混合使用时不能同时出现2个同类型引号

## 定义变量

> 一种弱类型的语言
>
> js定义变量时的类型只有一种声明方式：
>
> - `var x;`

### 定义

```javascript
<script>
    /*1.JavaScript是弱类型的语言，所有数据类型同意使用var声明*/
    var a = 10;
    var b = 10.1;
    var c = true;
    var d = "字符串";
    var e = '字符串哈哈哈哈哈哈哈哈哈哈哈哈';
    var f = '一';
    var g = new Object();
    console.log(a);
    console.log(b);
    console.log(c);
    console.log(d);
    console.log(e);
    console.log(f);
    console.log(g);

    /*2.JavaScript的语句结尾处可以不写分号
            *   因为在浏览器在解析时，js脚本中解析器会自动添加分号
            *   建议加分号*/
    console.log(a)
    console.log(b)

    /*3.单引号与双引号的区别
            *   单独成句时：没有区别。都是表示字符串
            *   单引号与双引号混合使用时：一定要成对出现，而且双引号里面不能再写双引号
            * */
    var x = "您好";
    var y = '您好';
    console.log(x == y);//结果为true，表示单引号与双引号没区别
    console.log(x === y);//值相等，证明两者完全一样

    var x1 = "'您好'";//输出：'您好'
    console.log(x1);
    var x2 = '"您好"';//输出："您好"
    console.log(x2);
    console.log(x1 == x2);//false
    console.log(x1 === x2);//false
</script>
```

### 重复变量名

> 如果两个变量重名，js中会：把上一个变量覆盖掉

```javascript
<script>
    var a = 0;
    var b = 1,c=2;
    /*如果变量没有定义时，
       直接使用：编写时不会报错，执行时会报错
     */
    // console.log(x);

    /*如果两个变量重名，js中会：把上一个变量覆盖掉*/
    var a = 1;
    console.log(a);
</script>
```





## 基本数据类型

> 1. Number类型：数值类型
> 2. String类型：字符串类型
> 3. Boolean类型：布尔类型
> 4. Null类型：空
> 5. Undefined类型：未定义
> 6. Symbol类型：唯一（不与其他的值相等）

### Number

- 基本属性

  ```javascript
  console.log(Number.MAX_VALUE);//最大值 1.7976931348623157e+308
  console.log(Number.MIN_VALUE);//最小值 5e-324
  
  console.log(Number.POSITIVE_INFINITY);//超出最大值是 无穷大 Infinity
  console.log(Number.NEGATIVE_INFINITY);//无穷小 -Infinity
  
  console.log(Number.NaN);//非数字
  ```

- Number精度问题

  ```java
  var a = 0.9;
  var b = 0.999999999999999999999;//1 精度丢失
  console.log(a);
  console.log(b);
  
  var a1 = 0.1;
  var a2 = 0.2;
  console.log(a1 + a2);//0.30000000000000004
  ```

- Number类型的数值，底层默认都是浮点型

  ```javascript
  var a3 = 10;//底层为：10.0
  var a4 = 10.0;
  console.log(typeof a3);//Number
  console.log(typeof a4);//Number
  ```

- Number类型的八进制和十六进制

  ```javascript
  var n1 = 0o377;//八进制，前缀是：0或者0o
  var n2 = 0xAEB;//十六进制，前缀是：0x
  ```

- 判断一个数值是否为数字

  > isNaN():是一个全局方法，调用时不应该写Number.NaN(),直接使用isNaN()即可

  ```javascript
  console.log(isNaN("abad"));//false:数字    true：非数字
  ```

### String

- String的属性

  ```javascript
  var s1 = "这是一个字符串";
  console.log(s1.length);
  ```

- String的常用方法：`indexOf()`

  ```javascript
  console.log(s1.indexOf("个"));//3
  
  //字符串的索引：s1[]      返回结果为：下标对应的某一个字符
  console.log(s1[3]);
  ```

- 字符串的匹配：`match()`

  > 如果匹配上，返回匹配的结果对象
  > 如果没有匹配上，返回匹配的结果为：null

  ```javascript
  console.log(s1.match("我"));
  console.log(s1.match("一"));
  ```

- 字符串的拆分：`split()`

  > 返回结果是一个数组

  ```javascript
  console.log(s1.split(""));
  ```

- 字符串的大小写转换

  > `toLowerCase()`:转为小写
  >
  > `toUpperCase()`:转为大写

  ```javascript
  var s3 = "aBsasMJKRJFgg";
  console.log(s3.toLowerCase());//转为小写
  console.log(s3.toUpperCase());//转为大写
  ```



### Boolean

> true/false

```javascript
var b1 = true;
var b2 = false;
console.log(b1 && b2);//false
console.log(b1 || b2);//true
console.log(b1 | b2);//1  1:表示true
console.log(b1 & b2);//0  0:表示false
```



### Null

> 空，什么都没有

```javascript
/*4.Null:空，什么都没有*/
var x1 = null;
/*这里的方法IsNull() 是官方提供的。如果为null，返回true；否则返回false*/
console.log(IsNull(x1));
```



### Undefined

> Undefined:未定义。声明变量时没有直接初始化

```javascript
var x2;
console.log(x2);
```



### Symbol

> Symbol类型：唯一性      
>
> 与其他任何数值都不相等  

```javascript
var m1 = Symbol(1);
var m2 = Symbol(1);
console.log(m1 === m2);//false
```



## 运算符

> 算术运算符：+ - * / % -- ++
>
> 逻辑运算符：|| && ！
>
> 位运算符：| & 
>
> 关系运算符：> <  ==   ===   <=   >=   !=
>
> 赋值运算符：+= -+ *= /=  %=   
>
> 三目运算符：同java

### 关系运算符

> if中，一个`=`表示：==先进性赋值操作，再用赋值后的值判断==
>
> - 如果判断条件是0/null：false  非零/非null：true

```javascript
var a = 10;
//if(a=20):先进性赋值操作，a变成了20；
//if(a=20):相当于  if(20)
if (a = 20) {//js中，如果判断条件是0/null：false  非零/非null：true
    console.log(a);
} else {
    console.log("不相等");
}
```



## 分支与循环

### 分支结构

> 分支结构的语法 与 Java中的完全一样

```javascript
<script>
    /*分支结构的语法 与 Java中的完全一样*/
    var a = 10;
    if(a>10){
        alert("大于10");
    }else if (a == 10) {
        alert("等于10");
    }else {
        alert("小于10");
    }

    var b = "周一";
    switch (b) {
        case "周一":
            console.log("1");
            break;
        default:
            console.log("默认");
    }
</script>
```

### 循环结构

> js中没有foreach，有for/in循环结构
>
> `for (var i in arr){}`：==i是  数组中的index下标==

```javascript
    /*1.for循环*/
    for (var i = 0; i < 10; i++) {
        console.log(i);
    }

    /*2.while循环*/
    var a = 1;
    while (a < 10) {
        a++;
    }

    /*3.do-while循环*/
    do {
        a++;
    } while (a<20);

    /*4.for/in 循环：与Java中的foreach完全不同*/
    /*因为js是弱类型的，所以数组的定义时也是弱类型，其中的元素也是弱类型的*/
    var arr = [1, "2", true, false, 3.4, null];//js数组使用[]
    for (var i in arr) {
        /*特别注意：输出的i是  数组中的index下标*/
        console.log(i);
    }
```



## 数据类型转换

### 自动类型转换

> js内部的函数在执行时，会根据函数的参数以及函数的定义，自动尝试转换为所期望的类型
>
> isNaN()中自动类型转换：
>
> | 原有类型              | 转换后        |
> | --------------------- | ------------- |
> | “123”：数字字符串     | 123：对应数字 |
> | “123abc”:非数字字符串 | NaN           |
> | “ ”                   | 0             |
> | null                  | 0             |
> | true                  | 1             |

```javascript
<script>
    /*问题：（）中的参数如何自动转换的*/
    /*答案：js内部的函数在执行时，会根据函数的参数以及函数的定义，自动尝试
     		转换为所期望的类型
     */
    console.log(isNaN(1));//false：数字  true：NaN
    console.log(isNaN("1"));//false --- 发生自动转换
    console.log(isNaN("123abc"));//true --- 自动转换失败
    console.log(isNaN(""));//false：""转为0，再判断
    console.log(isNaN(null));//false：null转为0
    console.log(isNaN(true));//false：true转为1

</script>
```

### 强制类型转换

> 通过js函数进行强制类型转换
>
> 1. 数字转为非数字`String()`
>
> 2. 非数字转为数字: 主要使用`Number()`
>    
>    | 原始数据   | 转换后       |
>    | ---------- | ------------ |
>    | “    ”     | 0            |
>    | “123”      | 123          |
>    | “123abc”   | NaN：非数字  |
>    | true       | 1            |
>    | false      | 0            |
>    | new Date() | 日期的毫秒数 |

```javascript
/*1.数字转为非数字*/
var a1 = 10;
console.log(String(a1));
console.log(a1.toString());

/*2.非数字转为数字: 主要使用Number()*/
var a2 = " ";//多个空格转为数字时,都成了0
var a3 = "123";//如果时非数字的字符串（123qwe），会转为NaN,表示非数字
var a4 = true;//true:1   false:0
var a5 = new Date();
console.log(Number(a2));//0
console.log(Number(a3));//123
console.log(Number(a4));//1
console.log(Number(a5));//1575339378151 ：日期的毫秒数
```



## 数组

特点：

- 可以存储不同数据类型的数据
- 长度可变

### 数组定义：

> `new Array()`：new数组
>
> `[1, 2, true, "qwer"]`：直接量定义数组

```javascript
/*数组的定义：4种*/
var a1 = new Array();//定义一个空的数组
var a2 = new Array(3);//长度为3的数组
var a3 = new Array("123", true, 3.14);//初始化赋值
var a4 = [1, 2, true, "qwer"];//直接量定义数组
```

### 数组的操作

> 数组自动扩容，扩容时不会出现数组越界
>
> 当下标超出长度时，不会报错，但是返回值是undefined

```javascript
a2[0] = "1";
a2[1] = "2";
a2[300] = "3";/*js数组自动扩容，扩容时不会出现数组越界*/
console.log(a2);
console.log(a2[300]);/*数组取数时，根据下标取值*/
console.log(a2[400]);//当下标超出长度时，不会报错，但是返回值是undefined
```

### 添加元素

> `push()`，向数组最后添加元素

```javascript
var a4 = [1, 2, true, "qwer"];//直接量定义数组 
a4.push("测试");
console.log(a4);//[1, 2, true, "qwer","测试"]
```

### 删除元素

> `pop()`：删除并返回数组最后一个元素

```javascript
var a4 = [1, 2, true, "qwer"];//直接量定义数组  
console.log(a4.pop());//删除并返回数组最后一个元素
console.log(a4);
```

> `shift()`:删除并返回数组的第一个元素

```javascript
console.log(a4.shift());//删除并返回数组的第一个元素
console.log(a4);
```

### 数组的排序

> `sort()`
>
> - 默认按照字符的字典顺序从小到大排列
> - 在`sort()`参数中重写`function`，`a-b`为冒泡升序

```javascript
var a5 = [1, 2, 3, 10, 100, 33, 22];
console.log(a5.sort());//默认按照字符的字典顺序从小到大排列
var a6 = a5.sort(function (a, b) {//内部为冒泡算法
    return a - b;//a-b结果，如果是负数，a,b不交换
    //a-b结果，如果是整数，a,b交换
});
console.log(a6);
```



## 函数：function

> 1. ==JS中的函数是一堆可执行代码的合集。==在需要的时候可以通过函数的名字调用其中的代码。函数可以理解为一种特殊的对象，其实==本质上就是一段可执行的字符串==
> 2. ==JS中函数调用时，实参的数量可以和形参的数量不一致。==
>    - 如果实参少于形参，则形参依次赋值，没有被赋值到的形参取值为undefined。
>    - 如果实参多余形参，则依次赋值，对于没有被赋值到的实参也不会丢失，可以在方法中通过arguments来获取
> 3. JS中的函数可以认为是一种特殊的对象，可以任意的赋值给不同的引用甚至通过方法来当做参数传递，唯一特殊的是，通过跟一对小括号可以执行函数中的代码。其实js是一门解释运行的语言，函数的本质就是一段js代码的字符串，来回赋值、来回传递都可以，一但跟一对小括号，就执行这段js代码字符串。

### 定义function

> 第一种，第三种必须掌握，第二种了解即可
>
> - 普通定义方式
>
> - 动态定义函数
>
> - 匿名函数定义方式

```javascript
//1.1   普通定义方式
function f1() {
    console.log("无参函数");
}
function f2(a,b,x) {
    console.log(x);
    return a - b;
}

//1.2   动态定义函数;
// ()中可以有N个参数，前N-1个是形参，最后一个是方法体
var f3 = new Function("a", "b", "return a-b");
console.log(f3);

//1.3   匿名函数定义方式
var f4 = function (a, b) {
    return a - b;
}
```

### 函数的调用

```javascript
/*2.    函数的调用*/
f1();
console.log(f2(1,2,3));
console.log(f3(4, 5));
console.log(f4(6, 7));
```

> 扩展
>
> 参数列表可以随意定义长度，参数的个数不确定
>
> - ==参数列表如果参数较少时，其他参数是undefined未定义的==
> - ==参数列表传入的参数较多时，其他多余参数不做计算==

```javascript
function f2(a,b,x) {
    console.log(x);
    return a - b;
}

/*3.    函数调用：扩展*/
f1(1, 2, 3, 4);/*参数列表可以随意定义长度，参数的个数不确定，多余参数不做计算*/
f2(1,2);/*参数列表如果参数较少时，其他参数是undefined未定义的*/
```



## 内置对象

> js已经提供好的对象
>
> String，Number，Boolean，Array，Function、Math(数学类)，Global(全局对象)，Date(日期对象)，RegExp(正则对象）

### Math

```javascript
/*1.    Math对象：常用属性和方法;其他方法参考JS的API文档*/
console.log(Math.PI);
console.log(Math.E);
console.log(Math.round(3.14));//四舍五入
console.log(Math.pow(2, 4));//2^4
console.log(Math.random());//[0,1)
```

### Global

```javascript
/*2.    Global全局对象：主要是提供全局方法，直接使用即可*/
console.log(isNaN(123));
console.log(parseFloat("3.14"));//把字符串转为浮点数
console.log(parseInt("123"));//把字符串转为整数，默认转换为十进制
```

### Date

```javascript
/*3.    Date日期对象：*/
var d1 = new Date();//日期的创建
var d2 = new Date("2019-01-01");//把字符串转为日期。注意：字符串的类型不能随意定义
//一般为：2019-01-01  或者 2019/01/01
console.log(d1.getDate());//返回当前月份的第几天
console.log(d1.getDay());//返回星期的第几天 0-星期天，1-星期一
console.log(d1.getFullYear());//返回本地时间的年份
```

### RegExp

> `^`：正则表达式开始
>
> `$`：正则表达式结束
>
> `\w`：[0-9a-zA-Z_]

```javascript
/*4.    RegExp：正则表达式：匹配特定格式的字符串*/
//比如：邮箱地址：test@qq.com
var r1 = /^\w+@\w+(\.\w+)+$/;
console.log(r1.test("123AFD@qq.com.cn"));
```



## 自定义对象

> 步骤：
>
> - 创建一个构造函数
> - 添加属性和方法
> - 使用new关键字创建对象

### 无参构造

```javascript
function Person() {
}

//创建对象
var p = new Person();
console.log(p);

//添加属性
p.name = "小明";
p.age = 19;

//添加方法
p.say = function () {
    alert("您好");
}

//调用属性和方法
console.log(p.age);
console.log(p["name"]);
p.say();

```

### 有参构造

```javascript
//2.    有参构造
function Person(name,age) {
    this.name = name;
    this.age = age;
    this.say = function () {
        alert("有参构造");
    }
}

var p = new Person("张三",18);
console.log(p.name);
console.log(p["age"]);
p.say();
```

### ==直接量定义对象==

> 语法格式：`{key1:value1,key2:value2,key3:value3,......}`

```javascript
//3.    直接量定义对象
//语法格式：{key1:value1,key2:value2,key3:value3,......}
//最常用
var p = {
    name:"🐱 ",
    age:28,
    say:function () {
        alert("我是🐱 ");
    }
};
console.log(p);
```



## JSON对象

> json本质上就是一段字符串，解析时特别方便  `{}` 的形式
>
> json格式的数据：是目前web数据传输格式的最流行的方式

```javascript
var data = {
    name: "李白",
    age: 10,
    wife: [//数组中每一个元素都是一个对象
        {name:"张飞1",age:10, job: ["打架1", "喝酒1"]},
        {name:"张飞2",age:11, job: ["打架2", "喝酒2"]},
        {name:"张飞3",age:12, job: ["打架3", "喝酒3"]},
    ]
};

//调用json对象，输出其中的某个字段
console.log(data.wife[0]);
console.log(data["wife"][1]["job"][0]);
```

