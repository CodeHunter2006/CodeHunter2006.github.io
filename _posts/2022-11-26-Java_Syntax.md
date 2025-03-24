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

## 移位

- `>>>`
  无符号右移。由于 Java 中没有`unsigned`类型，所以在做移位操作时始终需要考虑符号位的处理。
  当最左边是`1`(负数)时，进行`>>>`操作时，最左边的`1`不会变化(不影响符号位)，符号位右边会自动补`0`。

## 类

- `boolean ret = object instanceof class`
  判断某个对象是否属于某个类，或其子类，或者符合某个接口

- `obj1.getClass().equals(ClassA.class)`
  精确匹配类型
