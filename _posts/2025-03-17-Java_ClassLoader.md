---
layout: post
title: "ClassLoader 机制"
date: 2025-03-17 22:00:00 +0800
tags: Java
---

![class_loader](/assets/images/2025-03-17-Java_ClassLoader_0.jpg)

# Java 程序的启动并运行的过程(Java 中的类都是动态加载的，是语言灵活性的根源)

- JVM 启动(Launcher)
- 加载 main 方法所在的类，`起始类，Initial class`
- 执行 main 方法, `public static void main(string [])`
- 加载所依赖的类、执行其他方法

  - 在这个过程中，可能会触发其它类的加载

- 类加载流程
  ![process](/assets/images/2025-03-17-Java_ClassLoader_1.png)

# Java8 内置的类加载器

```Java
// 可以利用 Class.getClassLoader() 函数取得类加载器
public class BuiltinClassLoaderTest {
    public static void main(String[] args) {
        System.out.println("类 BuiltinClassLoaderTest 的加载器是: "
                + BuiltinClassLoaderTest.class.getClassLoader())

        System.out.println("类 Logging 的类加载器是: "
                + Logging.class.getClassLoader());

        System.out.println("类 ArrayList 的加载器是: "
                + ArrayList.class.getClassLoader());
    }
}
// 输出：
// 类 BuiltinClassLoaderTest 的加载器是: sun.miso.Launcher$AppClassLoader@18b4aac2
// 类 Logging的加载器是: sun.misc.Launcher$ExtClassLoader@7f31245a
// 类 ArrayList 的加载器是: null
```

- 可以看到上面 ClassLoader 三剑客：
  - AppClassLoader(应用类加载器)
    - 默认的系统类加载器
    - 负责加载当前应用`classpath`中的类
      - 设置 classpath: `java -cp /users/admin/javaproject/classes MainClass`
      - 读取 classpath: `System.getProperty("java.class.path")`
  - ExtClassLoader(扩展类加载器)
    - 负责加载扩展目录中的类
      - 扩展目录通常是`<JAVA_HOME>/lib/ext`
      - 设置扩展目录: `java -Djava.ext.dirs="/xx/java/ext" MainClass`
      - 读取扩展目录: `System.getProperty("java.ext.dirs")`
    - 从 Java9 开始，扩展机制(Extension Machanism)已被移除，ExtClassLoader 也因此被 PlatformClassLoader 取代
  - BootstrapClassLoader(启动类加载器)
    - 负责加载 JDK 核心类库中的类
      - 例如 Java8 中`<JAVA_HOME>/jre/lib/rt.jar`
    - 在 JVM 中通常使用`C/C++`语言原生实现，因此打印出来为`null`

三者类关系如图:
![class](/assets/images/2025-03-17-Java_ClassLoader_2.png)

- 从 JVM 的角度来看，只有两种类加载器
  - 一种是 BootstrapClassLoader，它是 JVM 的一部分，使用`C/C++`原生实现
  - 另一种是用户自定义的 ClassLoader，包括 JDK 中内置的 AppClassLoader、ExtClassLoader
    以及用户自行实现的 ClassLoader。用户定义的 ClassLoader 都使用 Java 语言实现，并要求都继承自抽象类`java.lang.ClassLoader`
- 如图所示，三者间不是继承关系，而是组合关系

# 双亲委派模型(Parents Delegation Model)

- 这个翻译很差，可能引起"多继承"、"父类"的误解。翻译为"干爹委派模型"更贴切 :)
- 实际上没有继承，只有组合关系。除 Bootstrap ClassLoader，其它的类加载器有且仅有一个 parent。

- 双亲委派模型规则
  - 每个 Class 都有对应的 ClassLoader
  - 每个 ClassLoader 都有一个"父"类加载器(Parent ClassLoader)。
    Bootstrap ClassLoader 除外，它是最顶层的类加载器。
  - 对于一个类加载请求，总是**优先委派给"父"类加载器**来尝试加载
  - 对于用户自定义的类加载器，默认的"父"类加载器是 AppClassLoader

类加载过程如图:
![sequence](/assets/images/2025-03-17-Java_ClassLoader_3.png)

- 优先由"父"类加载器加载，如果"父"类加载器无法加载，则自己再尝试加载，最后都无法加载指定的类则抛出`ClassNotFoundException`
- 这样的好处是越顶层的类加载器对其可见的类总是会被优先加载。
  例如你自行定义与`rt.jar`中同名的类`java.lang.String`并将其放入`classpath`中，并自定义了类加载器 UserClassLoader，
  但最终都是委派给了最顶层的 BootstrapClassLoader 进行加载， 这样最终加载的是`rt.jar`中的类。
  这样可以**保证 Java 类型体系的稳定性**。
  反之，你定义了类型`app.ClassA`，只有上层类加载器都找不到对应的类时，才会被用户自定义的类加载器加载。

```Java
// 双亲委派模型的实现代码
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
  //获取类时加锁 保证线程安全
    synchronized (getClassLoadingLock(name)) {
        //首先会检查这个类是否被加载过 同一个类不能重复的加载
        Class<?> c = findLoadedClass(name);
        //如果没有被加载
     if (c == null) {
            long t0 = System.nanoTime();
            try {
                // 如果当前parent不为空 就委派给parent类加载器来尝试加载
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                 //如果parent为空 那就直接委派给顶层的加载器尝试加载
                   c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                //如果父类加载器为能成功 就通过findClass方法进行加载 用户自定义的类加载器通常需要覆盖findClass方法
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

# 关键类`java.lang.ClassLoader`中的几个关键方法

- `protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException`
  - 根据类的全限定名来加载并创建一个类对象的入口
  - 如果遵循"双亲委派模型"，那么 ClassLoader 的子类则尽量**不要覆盖**此方法
  - 如果要打破"双亲委派模型"，子类需要覆盖此方法
- `protected final Class<?> defineClass(String name, byte[] b, int off, int len) throws ClassFormatError`
  - 将字节码的字节流转换为一个 Class 对象
  - final 方法，意味着子类无法覆盖它
  - 最终是通过一个 native 原生方法(JVM 的一部分)，将字节流转换为 Class 对象
- `protected Class<?> findClass(String name) throws ClassNotFoundException`
  - 根据类的全限定名，获取字节码二进制流，并创建对应的 Class 对象
  - 如果遵循"双亲委派模型"，那么我们通常不会覆盖`loadClass(..)`方法，而是覆盖`findClass(..)`方法
  - `findClass(..)`的实现逻辑通常是：
    1. 首先根据参数 name，从指定的来源获取字节码的二进制流
    2. 然后调用`defineClass(..)`方法，创建一个 Class 对象
- `private Class<?> findBootsrapClassOrNull(String name)`
  - 根据类的全限定名，委派 Bootstrap ClassLoader 进行类加载
  - 注意这个方法是 private 的，这意味着，如果要将某个类加载的请求委派给 Bootstrap ClassLoader，
    那么必须简介调用类 ClassLoader 中的某个 public/protected 方法
- ```Java
    private final ClassLoader parent;
    @CallerSensitive
    public final ClassLoader getParent();
  ```
  - 获取当前 ClassLoader 的"父"类加载器
  - 注意 parent 字段是`private final`的，它只能在构造函数中初始化
  - `parent != null`时，调用`parent.loadClass(..)`将加载请求委派给 parent
  - `parent == null`时，调用`findBootstrapClassOrNull(..)`将请求委派给 Bootstrap ClassLoader

# 自定义 ClassLoader 示例

- 遵循双亲委派模型
- 从任意指定的某个目录读取字节码类文件创建对应的类

```Java
package cn.memset.sample.classloaders;

import java.io.*;

/**
 * 从任意指定的某个目录中读取字节码类文件，然后创建加载对应的类
 */
public class MyCommonClassLoader extends ClassLoader {

    static {
        // 表明当前的ClassLoader可并行加载不同的类
        registerAsParallelCapable();
    }

    /**
     * 指定的字节码类文件所在的本地目录
     */
    private final String commonPath;

    /**
     * 构造函数。默认的parent ClassLoader是 AppClassLoader
     *
     * @param commonPath 字节码类文件所在的本地目录
     */
    public MyCommonClassLoader(String commonPath) {
        if (!commonPath.isEmpty()
                && commonPath.charAt(commonPath.length() - 1) != File.separatorChar) {
            commonPath += File.separator;
        }
        this.commonPath = commonPath;
    }

    /**
     * 构造函数。指定了一个 parent ClassLoader 。
     *
     * @param commonPath 字节码类文件所在的本地目录
     * @param parent     指定的parent ClassLoader
     */
    public MyCommonClassLoader(String commonPath, ClassLoader parent) {
        super(parent);
        if (!commonPath.isEmpty()
                && commonPath.charAt(commonPath.length() - 1) != File.separatorChar) {
            commonPath += File.separator;
        }
        this.commonPath = commonPath;
    }

    /**
     * 覆盖父类的 findClass(..) 方法。
     * 从指定的目录中查找字节码类文件，并创建加载对应的Class对象。
     *
     * @param name
     * @return
     * @throws ClassNotFoundException
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            // 读取字节码的二进制流
            byte[] b = loadClassFromFile(name);
            // 调用 defineClass(..) 方法创建 Class 对象
            Class<?> c = defineClass(name, b, 0, b.length);
            return c;
        } catch (IOException ex) {
            throw new ClassNotFoundException(name);
        }
    }

    private byte[] loadClassFromFile(String name) throws IOException {
        String fileName = name.replace('.', File.separatorChar) + ".class";
        String filePath = this.commonPath + fileName;

        try (InputStream inputStream = new FileInputStream(filePath);
             ByteArrayOutputStream byteStream = new ByteArrayOutputStream()
        ) {
            int nextValue;
            while ((nextValue = inputStream.read()) != -1) {
                byteStream.write(nextValue);
            }
            return byteStream.toByteArray();
        }
    }

    @Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        // System.out.println("准备使用 MyCommonClassLoader 加载类：" + name);
        return super.loadClass(name, resolve);
    }
}
```

# 类加载器的特性

- 用来确定类的**唯一性**

  - 假设有 N 个类的全限定名、L 个加载定义这些类的类加载器 ClassLoader，
    那么`<N,L>`二元组可用来确定类的唯一性
  - 要比较两个类是否相等时，要对这个二元组进行比较。
    这里的"相等"是指 Class 对象的`equals()`、`isAssignableFrom()`、`isInstance()`三个方法的返回结果，
    以及`instanceof`关键字的类型判定等情况。

- 类加载器的**传递性**

  - 假设类 C 是由加载器 L1 定义加载的，那么类 C 中所依赖的其它类也将会通过 L1 来进行加载(最底层的类是最终委派给 Bootstrap ClassLoader 加载的)。
  - 利用类加载器的传递性，从某个入口类开始，不断使用相同的类加载器展开加载同一个模块或应用中的其他类。
  - `initial ClassLoader`创建入口类的 ClassLoader
  - `define ClassLoader`是指经过委派后，最终创建类的 ClassLoader

- 类加载器的**可见性**
  - "类 B"对于"类 A"可见，指的是加载类 A 的加载器可以直接或通过委派间接加载到类 B。
  - 如"类 A"是通过 AppClassLoader 加载的、"类 B"是通过 ExtClassLoader 加载的，那么"类 B" 对"类 A" 可见；反过来"类 A"对"类 B"不可见。

# 打破"双亲委派"模型

- "双亲委派"模型并不是强制约束，而是推荐给开发者的一种类加载器的实现方式。但由于其局限性，有时需要打破它。

## Java SPI(Service Provider Interface) 机制打破双亲委派模型

- 在 Java SPI 机制中，双亲委派模型的局限性：
  - 类 ServiceLoader 是在包 java.util 中，由 Bootstrap ClassLoader 加载
  - 然而在 ServiceLoader 中，要实例化第三方厂商的、部署在 classpath 中的 Service Provider 类

线程上下文类加载器(Thread Context ClassLoader)

```java
public class Thread implements Runnable {
    // The context ClassLoader for this thread
    private ClassLoader contextClassLoader;

    public ClassLoader getContextClassLoader() {
        return contextClassLoader;
    }

    public void setContextClassLoader(ClassLoader contextClassLoader) {
        this.contextClassLoader = contextClassLoader;
    }
}

// 类 ServiceLoader 源码片段
public static <S> ServiceLoader<S> load(Class<S> servcie) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

- 线程上下文加载器不是一个具体的类，只是 Thread 类的属性。感觉是 Java 设计团队临时设计的 :)
- 通过`Thread.setContextClassLoader()`方法来设置。如果创建县城时还未设置，它将会从父线程中继承。默认是 AppClassLoader。
- 在 Bootstrap ClassLoader 加载的 ServiceLoader 类中反向调用 AppClassLoader 来加载第三方厂商类，逆转了类间的可见性。

## 热加载/热部署打破双亲委派模型

- **热加载/热部署**: 在不停机的前提下，部署新的应用或者模块
- 实现原理：基于 Java 的类加载器来实现，当模块或应用发生改变时(如监听文件变化)，使用新的 ClassLoader 来加载它们

Tomcat 热加载自定义类
![tomcat](/assets/images/2025-03-17-Java_ClassLoader_4.png)

# 数组类的加载

- 所有的数组实例都属于 Object，每个数组实例都有对应的 Class

```java
public class ArrayTest {
    public static void main(String[] args) {
        int[] ia = new int[3];
        System.out.println("数组类: " + ia.getClass());
        System.out.println("数组类的父类: " + ia.getClass().getSuperclass());
        for (Class<?> c : ia.getClass().getInterfaces())
            System.out.println("数组类的父接口: " + c)
    }
}
// output
// 数组类: class [I
// 数组的父类: class java.lang.Object
// 数组类的父接口: interface java.lang.Cloneable
// 数组类的父接口: interface java.io.Serializable
```

- 数组类并不通过类加载器来加载创建，而是 JVM 直接创建的，有专门的 JVM 指令`newarray`
- 数组类的元素类型，如果是引用类型，那最终要靠类加载器去创建
- 数组类的唯一性，依然需要"类加载器"来加以确定

- 与数组类关联的类加载器
  - 假设数组 A 的组件类型是 C，那么与数组类 A 关联的类加载器为
    - 如果组件类型 C 是引用类型，那么 JVM 会将数组类 A 和加载组件类型 C 的类加载器关联起来
    - 如果组件类型不是引用类型(如 int[])，JVM 会把数组类 A 标记为与引导类加载器(Bootstrap ClassLoader)关联

# 参考链接

- [B 站视频](https://www.bilibili.com/video/BV1ZY4y1n7tg/?spm_id_from=333.1365.top_right_bar_window_custom_collection.content.click&vd_source=4d46acc644c2a30b82c5b684d91ab373)
