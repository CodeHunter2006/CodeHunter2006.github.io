---
layout: post
title: "Java 语法点"
date: 2022-11-26 22:00:00 +0800
tags: Java
---

![Java](/assets/images/2022-11-26-Java_Syntax_1.webp)
记录 Java 与 C++/Go 的不同语法点

## 基本概念

- 字节码

  - Java 编译时会为每个 Class 输出一个`xx.class`文件，没有把整套系统链接起来，这个文件需要在运行时加载(链接)后运行

- 包

  - Java 的基本组织形式是包，就是一个文件夹，其中的类相互可见
  - 包的命名按照域名相反的方式，这样可以让不同组织的同名类自然分开，如`com.sub.aa`其实对应的域名是`aa.sub.com`

- 类的**全限定名**

  - 形如`com.aa.A`这种带着完整包名的类名

- **JavaBean**
  就是可重用的 Java 类，要求符合下面要求：
  - 必须是 public 的
  - 有无参构造器
  - 通过 Set/Get 暴露成员属性

## 数据类型

### 修饰符

- **final**
  - 修饰变量，表示常量，只能初始化时赋值
  - 修饰方法，表示不能被重写，与 abstract 正好相反
  - 修饰类，表示不能被继承，比如 String, Interger 以及其他包装类

### 容器文本初始化(initialize with literals)

```java
import java.util.*;

public class A {
  private static final List<String> LIST = new ArrayList<String>(Arrays.asList("1", "2", "3"));
}
```

## 流程控制

### for each

```java
for (int item : items) {
  //...
}
```

## import

- 关于**别名**
  Java 不支持在`import`时定义包或类的别名，如果有同名类需要同时使用，可以直接用包含全包名的类名。由于 Java 的包名就是域名的反向文本，所以不会出现全包名的冲突。

## 数学运算

### 移位

- `>>>`
  无符号右移。由于 Java 中没有`unsigned`类型，所以在做移位操作时始终需要考虑符号位的处理。
  当最左边是`1`(负数)时，进行`>>>`操作时，最左边的`1`不会变化(不影响符号位)，符号位右边会自动补`0`。

## 类

- `boolean ret = object instanceof class`
  判断某个对象是否属于某个类，或其子类，或者符合某个接口

- `obj1.getClass().equals(ClassA.class)`
  精确匹配类型

- 在 priave 方法中调自己的 private 方法或变量时，可以**省略 this.**。除非存在歧义的情况才需要加 this

- 在 interface 中可以用 default 关键字修饰方法，相当于在 abstract 类中定义了默认方法，如：
  `default void preProcess(String id) {}`

## 异常机制

- 通过`try {...} catch (Exception e) {...} finally {...}`的方式捕获代码段抛出的异常
- catch 可以有多次，从精确到粗放

```java
java.lang.Object
    └── java.lang.Throwable
        ├── java.lang.Error       (系统级错误)
        ├── java.lang.Exception   (程序可处理的异常)
            └── java.lang.RuntimeException  (运行时异常)
```

- Throwable 是所有异常的根类
- Error 表示系统级或 JVM 无法恢复的错误，通常情况程序不用捕获
  - 涉及二方类加载时可能抛出 NoSuchMethodError，有些情况也得捕获
- Exception 表示程序可处理的异常，是业务代码中最常处理的类型

## 函数式编程

### lambda 函数

Java 8 引入的一种简洁的表示可传递给方法或存储在变量中的代码块的方式。它可以被看作是一种匿名函数，一种没有名称但有参数列表、函数体和返回值（在某些情况下可以自动推断）的函数。

- Lambda 表达式的基本语法形式为：`(parameters) -> expression` 或者`(parameters) -> { statements; }`

- 场景：
  - 可以用于函数式接口：`Runnable runnable = () -> System.out.println("Hello from Lambda");`
  - 可以用于集合操作`numbers.stream().filter(num -> num % 2==0).forEach(System.out::println);`
    - 这里通过`filter`接收一个条件 lambda 过滤出想要的元素
    - 然后通过`forEach`接收一个`Consumer`函数式接口，打印结果

### 函数式接口

java8 引入，只包含一个抽象方法的接口，可以用 Lambda 函数直接赋值而生成对象，用于函数式编程

例如 `org.springframework.data.jpa.domain.Specification`

```java
Specification<User> ageSpec = (root, query, builder) -> builder.greaterThan(root.get("age"), 30);
```
