---
layout: post
title: "Java 常用 package 记录"
date: 2022-10-13 22:00:00 +0800
tags: Java
---

![Java](/assets/images/2022-10-13-Java_package_1.gif)

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

## `com.google.common.base.Preconditions`

- `public static void checkArgument(boolean expression, @Nullable Object errorMessage)`
  检查 expression 是否为 true，如果为 false，则将 errorMessage 作为异常抛出。

## `java.lang`

- `String System.setProperty(String key, String value)`
  设置一个全局系统变量，如果有旧值则作为返回值。设置后可以在其他地方用`System.getProperty`获取
