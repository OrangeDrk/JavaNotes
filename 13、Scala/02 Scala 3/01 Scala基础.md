[toc]

> 以下笔记是Scala 3 笔记

# 1. Hello World

HelloWorld.scala

```scala
object HelloWorld {
  @main def hello = println("Hello, world!")
}
```

在上面的代码中，`hello`是一个方法，使用`def`定义并且通过`@main`注解来声明是main方法。通过`println`来输出

```shell
# 编译
$ scalac HelloWorld.scala
# 运行
$ scala HelloWorld
```

编译运行和Java类似。编译生成class字节码文件，就可以在jvm上运行了

# 2. Variables and Data Type

## 2.1 变量类型的定义

Scala中定义变量，可以声明为可变的和不可变的

1. `val` 声明变量不可变，类似`final`
2. `var` 声明变量可变

```scala
// 不可变，final
val a = 0

// 可变
var b = 1
```

## 2.2 声明变量类型

Scala中定义变量，可以声明类型，也可以让编译器自动推断类型

```scala
var x: Int = 1 // 明确声明
var y = 2 // 自动推断
```

## 2.2 内置数据类型

### 数值类型

```scala
val b: Byte = 1
val s: Short = 1
val i: Int = 1
val l: Long = 1
val f: Float = 3.0
val d: Double = 2.0

val x = 1_000L // val x: Long=1000
val y = 2.2D // val y: Double=2.2
val z = 3.3F //val z: Float = 3.3
```

如果需要准确的大数值，可以使用`BigInt`,`BigDecimal`类型

```scala
var a = BigInt(1_234_567_890_987_654_321L)
var b = BigDecimal(123_456.789)
```

### 字符类型

Scala 也有`String` 和`Char`类型

```scala
val name = "Bill"   // String
val c = 'a'         // Char
```

Scala中的Strings和Java中的类似，但是提供了2个强大的额外功能

- 支持字符串插值
- 可以创建多行字符串

> 字符串插值

符串插值提供了一种非常易读的方式来使用字符串中的变量。例如，给定这三个变量：

```scala
val firstName = "John"
val mi = 'C'
val lastName = "Doe"
```

您可以将这些变量组合在一个字符串中，如下所示：

```scala
println(s"Name: $firstName $mi $lastName")   
// "Name: John C Doe"
```

只需在字符串前面加上字母`s`，然后在变量名之前添加`$`符号

如果字符串中有表达式，需要加`{}`

```scala
println(s"2 + 2 = ${2 + 2}")   // prints "2 + 2 = 4"

val x = -1
println(s"x.abs = ${x.abs}")   // prints "x.abs = 1"
```

> 多行字符串

多行字符串使用三个双引号定义

```scala
val quote = """The essence of Scala:
               Fusion of functional and object-oriented
               programming in a typed setting."""
```



# 3. 方法



# 4. 流



# 5. 对象



# 6. 集合



# 7. 