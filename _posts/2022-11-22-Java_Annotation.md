---
layout: post
title: "Java Annotation"
date: 2022-11-22 22:00:00 +0800
tags: Java
---

![annotation](/assets/images/2022-11-22-Java_Annotation_1.webp)
Java1.5 引入了 Annotation(注解)，Spring 框架的使用更是由各种注解组成，这里记录注解相关的基本概念和开发流程

# 基本概念

注解就是加在某种目标对象上的特殊注释，在反射时可以取得，用以对被注解对象进行功能扩展。

- 注解和注释的差异：
  注释完全不影响程序的运行，只是为开发者提供辅助阅读代码的文字；
  注解是目标对象的一部分**元信息**，和对象的类型、名称一样，可以用反射获取到并在编译和运行时被使用。
- **APT(Annotation Processing Tool)**
  注解对于目标来说就是一些"key:value"值，本身无法直接起作用，需要和处理注解的工具 APT 配合才能进行扩展处理。
- 注解可作用的目标：
  包、类、构造器、方法、成员变量(包括函数)、参数、局部变量的声明

# 常用注解

- `@Override`
  对成员函数进行修饰，说明该成员函数是覆盖父类函数，否则编译不过。
  这个注解是防止 typo 引起的错误可能会被忽略。
  例如父类实现了函数`public void Hello()`，而子类实现了`He11o`(数字 1 代替字母 l)或`Hell0`(数字 0 代替字母 o)，子类本想覆盖父类函数而未能真正覆盖引起运行时问题。如果加了`@Override`，则编译时报错，可以尽早发现问题。

- `@Deprecated`
  可以作用于类或方法上，表示该元素已经过时了，编译器会给出警告。

  - java9 提供了两个新属性：
    - `forRemoval: boolean` 说明该 API 在将来是否会被删除
    - `since: String` 说明该 API 从哪个版本开始被标记为过时

- `@SuppressWarnings`
  可以作用于类型、字段、方法等目标，表示抑制编译时的 Warning。
  (尽量不使用，尽量早报出 Warning。除非是明确知道后果才使用)

- `@FunctionInterface`
  可用于接口，表示该接口是**函数式接口**(即只有一个抽象方法的接口)，以确保该接口可以用 λ 函数重写。

- `@Getter`
  作用于类，为其中的成员变量生成 getter 方法。
  如成员`public String name;`会自动生成`public String getName() { return this.name }`
  如果已经自定义了 getter 函数，则不会覆盖。

- `@Setter`
  与`@Getter`类似，生成 setter 方法。

- `@Data`
  相当于`@Getter+@Setter`再加上`public boolean equals(Object o)`+`protected boolean canEqual(Object other)`+`public int hashCode()`+`public String toString()`方法

# 元注解

元注解就是修饰注解的注解。Java 中定义了 5 个元注解：

- `@Rentention`
  修饰注解定义，指定被修饰的注解可以保留多长时间(阶段)，它包含一个`@RetentionPolicy`类型的 value 成员变量。可选项如下：
  - `RetentionPolicy.SOURCE` 只保留在源代码中，编译器直接丢弃，和注释相同。此为默认值。
  - `RetentionPolicy.CLASS` 注解将记录在 class 文件中，编译时可用，运行时不可用。
  - `RetentionPolicy.RUNTIME` 将一直被保留，运行时可利用反射获取。
- `@Target`
  修饰注解定义，指定被修饰的注解能用于修饰哪些程序元素。该注解包含一个名为 value 的数组类型的成员变量，只有下面可选项：
  - `ElementType.ANNOTATION_TYPE` 可修饰注解
  - `ElementType.CONSTRUCTOR` 可修饰构造器
  - `ElementType.FIELD` 可修饰成员变量
  - `ElementType.LOCAL_VARIABLE` 可修饰局部变量
  - `ElementType.METHOD` 可修饰方法定义
  - `ElementType.PACKAGE` 可修饰包定义
  - `ElementType.PARAMETER` 可修饰参数
  - `ElementType.TYPE` 可修饰类、接口(包括注解类型)或枚举定义
- `@Documented`
  被修饰的注解将被 javadoc 工具提取为文档，如果定义注解类时使用了`@Documented`修饰，则所有使用该注解修饰的程序元素的 API 文档中将会包含该注解说明。
- `@Inherited`
  为被修饰的注解增加继承性。如果某个类使用了被修饰的注解，那么子类将自动被该注解修饰。
- `@Repeatable`
  在 Java1.8 中引入的注解，被修饰的注解可以被重复多次作用于一个元素并传入不同的参数，参数可以用数组的形式被取得。

# 语法

```Java
// 注解本质上是一个 interface，只要在定义时在`interface`前面加上`@`即可
public @interface MyAnnotation {
    // 以无参方法的形式定义注解的属性
    // 属性的类型只能是基本类型和String及其对应的数组，不能是其他类型(如 List)
    String name();
    int age() default 100;  // 可以通过 default 关键字添加默认值
}

// 使用注解时，直接在元素前加上即可，可以根据情况传参
@MyAnnotation(name = "test")
public class MyClass {
}
```

# 开发流程

1. 定义 annotation

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String name();
    int age() default 100;  // 可以通过 default 关键字添加默认值
}
```

2. 定义 APT

- 利用`java.lang.annotation.Annotation`包取得注解的 key-value 并执行动作

```Java
public class AnnotationTool {
    public static String getAnnotationName(Object obj) {
        Annotation[] annotations = obj.getClass().getAnnotations();
        for (Annotation annotation : annotations) {
            if (annotation.annotationType().equals(MyAnnotation.class)) {
                return ((MyAnnotation)annotation).name();
            }
        }
        return "not find name of annotation MyAnnotation";
    }
}
```

3. 在目标使用 annotation

```Java
@MyAnnotation(name="test name")
public class MyClass {
}
```

4. 在程序运行中执行 APT

```Java
public class Test {
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        System.out.println(AnnotationTool.getAnnotationName(obj));
    }
}
```

- 通常步骤 4 是通过框架代码实现的，会隐藏在启动流程中

# 其他要点

- java8 时新增了`getDeclaredAnnotations`等方法，可以只取得直接修饰的注解，排除通过`@Inherited`修饰的注解。
