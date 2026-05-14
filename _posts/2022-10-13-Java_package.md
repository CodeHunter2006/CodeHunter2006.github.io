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

## `java.lang.ThreadLocal`

一个线程局部变量工具类，用于为每个使用该变量的线程都创建一个独立的副本，每个线程都可以独立地改变自己的副本，而不会影响其他线程所对应的副本。
通过合理使用 ThreadLocal，可以高效地实现线程安全的局部变量管理，不必加锁，避免多线程环境下的数据竞争问题、提高执行效率。

- 注意事项
  - 内存泄漏风险：
    1. 若 ThreadLocal 变量为静态且持有大对象（如数据库连接），需在线程结束前调用 remove()释放资源。
    2. 在线程池环境中（如 Tomcat、Spring），线程会被复用，必须在请求处理完成后调用 remove()，否则可能导致数据混乱或内存泄漏。
  - 初始化方式：
    使用 withInitial()方法（Java 8+）替代重写 initialValue()，代码更简洁。
  - 继承性：
    使用 InheritableThreadLocal 可让子线程继承父线程的 ThreadLocal 值。
- 最佳实践:
  - 静态常量：
    通常将 ThreadLocal 定义为 private static final，确保全局唯一。
  - 泛型约束：
    使用泛型明确存储的数据类型，避免类型转换错误。
  - try-finally 包裹：
    在可能抛出异常的场景中，用 try-finally 确保 remove()被调用。

## `javax.annotation.PostConstruct`

- 用于注释方法，被注释的方法将在对象构造之后被自动调用

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

### `lombok.experimental.UtilityClass`

声明一个类为工具类，无法被继承，只能使用 static 方法。

### `lombok.Builder`

自动为 POJO 类生成 builder 模式需要的方法

## `com.github.benmanes.caffeine.cache`

Caffeine（com.github.benmanes.caffeine.cache）是 Java 生态中性能最优的本地缓存库，基于 W-TinyLFU 淘汰算法，在并发吞吐量、命中率、内存效率上全面超越 Guava Cache，是 Spring 5+ 默认本地缓存实现。

- 核心功能（特性）
  - 超高性能：O (1) 读写、并发无锁设计，高并发下吞吐量远高于 Guava/ConcurrentHashMap
  - 智能淘汰：W-TinyLFU 算法（频率 + 热度 + 时间窗口），接近理论最优命中率
  - 灵活过期
    expireAfterWrite：写入后固定时长过期
    expireAfterAccess：最后一次访问后过期
    expireAfter(Expiry)：动态自定义过期（每条不同 TTL）
    refreshAfterWrite：后台异步刷新（不阻塞读）
  - 容量控制
    maximumSize：最大条目数
    maximumWeight + weigher：按权重 / 内存大小淘汰
  - 自动加载
    LoadingCache：get 时自动加载（原子、防击穿）
    AsyncLoadingCache：异步加载（CompletableFuture）
  - 引用类型
    weakKeys / weakValues：弱引用（GC 时自动回收）
    softValues：软引用（内存不足时回收）
  - 监控统计
    recordStats()：命中率、加载时间、淘汰数、请求量
  - 事件监听
    removalListener：条目过期 / 淘汰 / 删除时回调
    线程安全：全并发安全，无锁 / 分段锁设计

## `io.github.bucket4j`

Bucket4j（io.github.bucket4j） 是 Java 生态中最主流、高性能的令牌桶算法（Token Bucket）限流库，用于精准控制请求速率、防止系统过载，支持单机与分布式场景，是 API 网关、微服务限流的首选方案。

- 核心功能
  - 令牌桶算法实现：经典流量控制模型，支持平滑限流 + 突发流量容忍。
  - 高精度：纯整数运算，无浮点数误差，纳秒级时间精度。
  - 高性能：无锁设计，高并发下吞吐量极高、低延迟。
  - 灵活限流规则
    - 多带宽叠加（如：每秒 100 次 + 每分钟 1000 次）
    - 多种令牌填充策略
      - greedy（贪婪，匀速平滑）
      - intervallyAligned（间隔对齐，整点批量填充）
    - 初始令牌数、容量控制
  - 丰富 API
    - tryConsume()：尝试消费（非阻塞）
    - asBlocking()：阻塞等待令牌
    - asAsync()：异步 CompletableFuture
    - 令牌回滚、动态修改配置
  - 分布式支持：基于 Redis、Hazelcast、Ignite 等实现跨节点统一限流。
  - 监控与监听：获取可用令牌数、消耗统计、移除监听器。

## `org.redisson.Redisson`

java 里 redis 相关操作的封装

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

## `org.springframework.stereotype`

Spring 框架基本分层注解

- `org.springframework.stereotype.Controller`
  负责处理 HTTP 请求，将请求转发给服务层进行业务处理，并将处理结果返回给客户端
- `org.springframework.stereotype.Service`
  服务层负责处理业务逻辑，协调不同数据访问层操作，封装业务规则。
  使用 @Service 注解后，Spring 容器会自动扫描并将带有该注解的类注册为一个 Bean，这样就可以在其他组件中通过依赖注入的方式使用该服务。
- `org.springframework.stereotype.Service`
  主要用于标识数据访问层（DAO）组件，负责与数据库进行交互。服务层组件（@Service）通常会调用数据访问层组件（@Repository）来完成业务逻辑。

### `org.springframework.lang.Nullable`

annotation，标记参数等可为 null

### org.springframework.boot

启动相关

### `org.springframework.boot.context.properties.ConfigurationProperties`

将外部配置文件与 config 类对象绑定

## org.springframework.context

上下文相关操作

### `org.springframework.context.annotation.ComponentScan`

设定扫描 Component 的根目录列表，通常作为 Application 的注解。

### `org.springframework.context.ApplicationContextAware`

当一个类实现了这个接口，Spring 容器会在创建该类的实例时，自动将 ApplicationContext（应用上下文）注入到这个类中。这样，在这个类内部就可以方便地获取 Spring 容器中的其他 bean、资源等信息。

## org.springframework.beans.factory.annotation

### `org.springframework.beans.factory.annotation.Value`

用于将外部配置值注入到 Bean 的字段、方法参数或构造函数参数中。它支持从多种配置源读取值，包括属性文件、环境变量、命令行参数等。

```java
@Component
public class MyService {
    @Value("${app.name}")
    private String appName; // 从 application.properties 读取 app.name 的值

    @Value("${server.port:8080}") // 默认值 8080
    private int serverPort;
}
```

## org.springframework.scheduling

定时任务

### `org.springframework.scheduling.annotation.Scheduled`

## org.springframework.web.server

### `org.springframework.web.server.WebFilter`

annotation

- 它可以在请求到达目标处理组件（如控制器方法）之前拦截请求。
- 可以对请求进行预处理操作，比如修改请求头、请求参数等。

### `org.springframework.web.server.ServerWebExchange`

class

获取本次请求的上下文，可以获取 request 和 response

## JPA(Jakarta Persistance API)

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

### `jakarta.persistence.Lob`

用于标注实体类中的大对象（Large Object）属性
