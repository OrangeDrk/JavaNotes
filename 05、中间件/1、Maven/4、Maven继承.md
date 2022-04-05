[TOC]

# Maven工程的继承

## 父级工程

> 被继承的Maven项目中的Pom的部分定义是：

```xml
  <groupId>org.example</groupId>
  <artifactId>MavenTest</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>
```

> 父级工程的`packaging`必须是：`pom`

## 子级工程

> 继承的Maven项目中的POM的关键部分是：

```xml
    <parent>
        <artifactId>MavenTest</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
```

