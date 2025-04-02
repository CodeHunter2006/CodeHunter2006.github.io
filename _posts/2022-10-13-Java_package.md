---
layout: post
title: "Java 常用 package 记录"
date: 2022-10-13 22:00:00 +0800
tags: Java
---

![Java](/assets/images/2022-10-13-Java_package_1.gif)

# Java 官方包

## `java.util.Arrays`

提供常用字节数组操作

- `static byte[] copyOfRange(byte[] original, int from, int to)`
  取得子数组

## `java.util.Date`

提供时间相关

```java
import java.util.Date;

static main() {
    Date startTime = new Date();  // get now time
    int waitSeconds = 100;
    while ((new Date()).getTime() - startTime.getTime() < (waitSeconds * 1000)) {
        Thread.sleep(1000);
    }
}
```

## `java.util.Optional`

方便根据对象指针是否为 null 进行一系列函数式编程

## `java.lang`

- `String System.setProperty(String key, String value)`
  设置一个全局系统变量，如果有旧值则作为返回值。设置后可以在其他地方用`System.getProperty`获取

# 第三方

## `com.google.common.base.Preconditions`

- `public static void checkArgument(boolean expression, @Nullable Object errorMessage)`
  检查 expression 是否为 true，如果为 false，则将 errorMessage 作为异常抛出。

## `org.projectlombok:lombok`

- 自动生成 getter setter toString 构造函数等常用方法

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

### `lombok.extern.slf4j.Slf4j`

annotation 自动添加 log 对象

```java
import lombok.extern.slf4j.Slf4j;
@Slf4j
public class MyClass {
    // 可以直接使用log对象记录日志
    public void myMethod() {
        log.info("This is an info message");
    }
}
```

### `lombok.Builder`

自动为 POJO 类生成 builder 模式需要的方法

# AOP(Aspect - Oriented Programming) 面相切面编程

- 对 Java 类、方法的各个时间点(对象，创建前、创建后、析构前、析构后；方法，调用前、调用后、正常返回后、异常后)进行回调函数处理

- **Aspect(切面)**
  由一系列切点组成。对应一个 class
- **Pointcut(切点)**
  指定具体的构造函数执行、方法调用等。对应一个空方法
- **Advice(通知)**
  对指定的时机点设置回调函数。对应一个回调函数
- **JoinPoint(连接点)**
  收到通知时用于获取切点命中时的信息。对应通知回调函数的入参

```java
@Aspect
@Component
public class SecurityAspect {
    // 绑定切点
    @Pointcut("execution(* com.example.controller.*.*(..))")
    public void securityPointcut() {}

    // 针对切点设置通知
    @Before("securityPointcut()")
    public void checkSecurity() {
        // 进行安全检查的逻辑
    }
}
```

# Spring

### `org.springframework.lang.Nullable`

annotation，标记参数等可为 null

### org.springframework.boot

启动相关

### `org.springframework.boot.context.properties.ConfigurationProperties`

将外部配置文件与 config 类对象绑定

## org.springframework.context

上下文相关操作

### `org.springframework.context.ApplicationContextAware`

当一个类实现了这个接口，Spring 容器会在创建该类的实例时，自动将 ApplicationContext（应用上下文）注入到这个类中。这样，在这个类内部就可以方便地获取 Spring 容器中的其他 bean、资源等信息。

## org.springframework.web.server

### `org.springframework.web.server.WebFilter`

annotation

- 它可以在请求到达目标处理组件（如控制器方法）之前拦截请求。
- 可以对请求进行预处理操作，比如修改请求头、请求参数等。

### `org.springframework.web.server.ServerWebExchange`

class

获取本次请求的上下文，可以获取 request 和 response

## JPA(Java Persistance API)

简化 DAO 操作

### `org.springframework.stereotype.Repository`

annotation 标记一个接口为 DAO 类

### `org.springframework.transaction.annotation.Transactional`

annotation，标记一个方法内的所有操作为一个事务

### `org.springframework.data.jpa.repository.JpaRepository`

interface 可支持基本的 CRUD 操作

### `org.springframework.data.jpa.repository.JpaSpecificationExecutor`

interface 通过函数名规约语法实现 DML 操作

### `org.springframework.data.jpa.domain.Specification`

用于以一种灵活且类型安全的方式构建复杂的查询条件。

- 函数式接口，可以用 Lambda 函数赋值
