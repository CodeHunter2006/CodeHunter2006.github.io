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

## `com.google.common.base.Preconditions`

- `public static void checkArgument(boolean expression, @Nullable Object errorMessage)`
  检查 expression 是否为 true，如果为 false，则将 errorMessage 作为异常抛出。
