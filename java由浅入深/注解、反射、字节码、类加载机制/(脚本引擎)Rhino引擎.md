[TOC]



# 脚本引擎执行JavaScript代码

## 脚本引擎介绍

> - 使得Java应用程序可以通过一套固定的接口与各种脚本引擎交互，从而达到在Java平台上调用各种脚本语言的目的
> - Java脚本API是连通Java平台和脚本语言的桥梁
> - 可以把一些复杂异变的业务逻辑交给脚本语言处理，大大提高了开发效率
> - ==eval函数（解析器）——对JavaScript代码进行解释==
>   - 语法：==string 必需==。要计算的字符串，其中==含有要计算的 JavaScript 表达式==或==要执行的语句==
>   - eval函数是强大的数码转换引擎,字符串经eval转换后得到一个javascript对象



# 具体使用

## 获得脚本引擎对象

> ```java
> ScriptEngineManager sem = new ScriptEngineManager();
> ScriptEngine engine = sem.getEngineByName("javascript");
> ```



## 使用上下文

> - 使用上下文，执行eval之后，js的变量和函数都存储于上下文，通过get和put可以获取和保存变量，Java和JavaScript都可以使用同一个上下文
>
> ```java
>  //使用上下文，执行eval之后，js的变量和函数都存储于上下文，通过get和put可以获取和保存变量，java和javascript都可以使用同一个上下文
>         engine.put("myame", "liuxg !!");
>         System.out.println(engine.get("myame"));
> 
>         engine.eval("var yourname = 'who are you ?'");
>         System.out.println(engine.get("yourname"));
> ```
>
> - 结果：**liuxg！！**
>
>   ​		**who are you ?**



## 执行变量

> - 代码：(相当于把JavaScript代码提取到了script的字符串中，执行该字符串)
>
>   ```java
>   String script = "var compo = {addr : \"广州\",tel : \"12345678901\"};";
>   script += "print(compo.addr + \" : \" + compo.tel)";
>   engine.eval(script);
>   ```
>
> - 结果：**广州 : 12345678901**



## 执行函数

> - 代码：
>
>   ```java
>   //定义函数1
>   String funscript = "function sum(a,b){return a + b ;}";
>   engine.eval(funscript);
>   //定义函数2(等同于1)
>   engine.eval("function sum(a,b){return a+b};");
>   
>   //获取调用接口
>   Invocable jsInvocable = (Invocable)engine;
>   //执行脚本中定义的方法
>   Object obj = jsInvocable.invokeFunction("sum", 1,2);
>   
>   //输出结果
>   System.out.println("sum = " + obj);
>   ```
>
> - 结果：**sum = 3.0**



## js和Java代码混合用

> - 代码：
>
>   ```java
>   //在JavaScript代码中用Java的包写代码
>   String jsCode = 
>       " var list=java.util.Arrays.asList([\"清华大学\",\"北京大学\",\"五道口大学\"])";
>   engine.eval(jsCode);
>   
>   List<String> list = (List<String>) engine.get("list");
>   for (String temp : list) {
>       System.out.println(temp);
>   }
>   ```
>
> - 结果：
>
>   **清华大学**
>   **北京大学**
>   **五道口大学**



## 执行js文件

> - 前提：项目目录下有js文件（例如：a.js）
>
> - 代码：
>
>   ```java
>   URL url = Demo01.class.getClassLoader().getResource("TestRhino/a.js");
>   FileReader fr = new FileReader(url.getPath());
>   engine.eval(fr);
>   fr.close();
>   ```
>
> - JavaScript文件代码：
>
>   ```javascript
>   //定义test方法
>   function test(){
>   	var a = 3;
>   	var b = 4;
>   	print("invoke js file:"+(a+b));
>   }
>   //调用test方法
>   test();
>   ```
>
> - 结果：**invoke js file:7**