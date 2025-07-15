---
layout: post
title: "（转）Java 编码规范(开发手册)"
date: 2025-03-27 22:00:00 +0800
tags: Java
---

![Java Develop Standard](/assets/images/2025-03-27-Java_Standard_1.webp)
记录 Java 开发中常见编码规范

# 编程规约

## 命名风格

- _强制_ 所有编程相关的命名均不能以**下划线或美元符号**开始，也不能以**下划线或美元符号**结束
  说明：任何情形，没有必要插入**多个空行**进行隔开。
- _强制_ 所有编程相关的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式。
  - 正确使用英文或拼音都可以，易于理解、避免歧义。另外，尽量避免拼音方式。
- _强制_ 代码和注释中都要避免使用任何人类语言中的种族歧视性或侮辱性词语。
  - 正例：blockList/allowList/secondary
  - 反例：blackList/whiteList/slave/SB/WTF
- _强制_ 类名使用 UpperCamelCase 风格，以下情形例外：DO/PO/DTO/BO/VO/UID 等。
- _强制_ 方法名、参数名、成员变量、局部变量都统一使用 lowerCamelCase 风格。
- _强制_ 常量命名应该全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。
- _强制_ 抽象类命名使用 Abstract 或 Base 开头；异常类命名使用 Exception 结尾，
  测试类命名以它要测试的类的名称开始，以 Test 结尾。
- _强制_ 类型与中括号紧挨来定义数组。
  - 正例：定义整形数组`int[] arrayDemo`。
  - 反例：在 main 参数中，使用`String args[]`来定义。
- _强制_ POJO 类中的任何布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误。
  - 反例：定义为布尔类型`Boolean isDeleted`的字段，它的 getter 方法也是 isDeleted()，
    部分框架在反向解析时，"误以为"对应的字段名称是 deleted，导致字段获取不到，得到意料之外的结果或抛出异常。
- _强制_ 包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用**单数**形式，
  但类名如果有复数含义，类名可以使用复数形式。
- _强制_ 避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名，使可理解性降低。
  - 说明：子类、父类成员变量名相同，即使是 public 也是能够通过编译，而局部变量在同一方法内的不同代码块中同名也是合法的，
    但是要避免使用。对于非 setter/getter 的参数名称也要避免与成员变量名称相同。
- _强制_ 杜绝完全不规范的英文缩写，避免望文不知义。
- 接口和实现类的命名有两套规则：
  - _强制_ 对于 Service 和 DAO 类，基于 SOA 的理念，暴露出来的服务一定是接口，内部的实现类用 Impl 的后缀与接口区别。
    - 正例：CacheServiceImpl 实现 CacheService 接口。
  - _推荐_ 如果是形容能力的接口名称，取对应的形容词为接口名（通常是 -able 结尾的形容词）。
    - 正例：AbstractTranslator 实现 Translatable。

## 常量定义

- _强制_ 不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。
- _强制_ long 或 Long 赋值时，数值后使用大写 L，不能是小写 l，小写容易跟数字混淆，造成误解。
- _强制_ 浮点数类型的数值后缀统一为大写的 D 或 F。

## 代码格式

- _强制_ 如果大括号内为空，简洁地写成`{}`即可，大括号中间无需换行和空格；如果是非空代码块，则：
  - 左大括号前不换行。
  - 左大括号后换行。
  - 右大括号前换行。
  - 右大括号后还有 else 等代码则不换行；表示终止的右大括号后必须换行。
- _强制_ 左小括号和右边相邻字符之间不需要空格；右小括号和左边相邻字符之间也不需要空格；而左大括号前需要加空格。
- _强制_ if/for/while/switch/do 等保留字与左右括号之间都必须加空格。
- _强制_ 任何二目、三目运算符的左右两边都需要加一个空格。
- _强制_ 采用 4 个空格缩进，禁止使用 Tab 字符。
- _强制_ 注释的双斜线与注释内容之间有且仅有一个空格。
- _强制_ 在进行类型强制转换时，右括号与强制转换值之间不需要任何空格隔开。
- _强制_ 单行字符数限制不超过 120 个，超出需要换行，换行时遵循如下原则：
  - 第二行相对第一行缩进 4 个空格，从第三行开始，不再继续缩进。
  - 运算符与下文一起换行。
  - 方法调用的点符号与下文一起换行。
  - 方法调用中的多个参数需要换行时，在逗号后换行。
  - 在括号前不要换行。
- _强制_ 方法的参数在定义和传入时，多个参数逗号后面必须加空格。
- _强制_ IDE 的 text file encoding 设置为 UTF-8；IDE 中文件的换行符使用 Unix 格式，不要使用 Windows 格式。
- _推荐_ 不同逻辑、不同语义、不同业务的代码之间插入一个空行，分隔开来以提升可读性
  - 说明：任何情形，没有必要插入**多个空行**进行隔开。

## OOP 规约

- _强制_ 避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用**类名**来访问即可。
- _强制_ 所有的覆写方法，必须加`@Override`注解。
  - 说明：`getObject()`与`get0bject()`的 typo，一个是字母 O，一个是数字 0，加`@Override`可以准确判断是否覆盖成功。
    另外，如果抽象类中对方法签名进行修改，其实现类会马上编译报错。
- _强制_ 相同参数类型，相同业务含义，才可以使用可变参数，参数类型避免定义为 Object。
  - 说明：可变参数必须放置在参数列表的最后。(建议开发者尽量不用可变参数编程)
  - 正例：`public List<User> listUsers(String type, Long... ids) {...}`
- _强制_ 外部正在调用的接口或者二方库依赖的接口，不允许修改**方法签名**，避免对接口调用方产生影响。
  接口过时必须加`@Deprecated`注解，并清晰地说明采用的新接口或者新服务是什么。
- _强制_ 不能使用过时的类或方法。
  - 说明：`java.net.URLDecoder`中的方法`decode(String encodeStr)`这个方法已经过时，应该使用双参数`decode(String source, String encode)`。
    接口提供方既然明确是过时接口，那么有义务同时提供新的接口；作为调用方来说，有义务去考证过时方法的新实现是什么。
- _强制_ Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。
  - 正例：`"test".equals(param);`
  - 反例：`param.equals("test");`
  - 说明：推荐使用 JDK7 引入的工具类`java.util.Objects#equals(Object a, Object b)`
- _强制_ 所有整型包装类对象之间**值的比较**，全部使用 equals 方法比较。
  - 说明：对于 Integer var = ? 在`-128 ~ 127`之间的赋值，Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，
    这个区间内 Integer 值可以直接用 == 进行判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，
    这是一个大坑，推荐使用 equals 方法进行判断。
- _强制_ 任何货币金额，均以**最小货币单位**且为整型类型进行存储。
  - 说明：浮点型计算很容易由于精度问题产生差异，造成原本应相等的两个变量不相等，所以在涉及精度较高的浮点运算时，就直接用整型替代浮点型。
- _强制_ 浮点数之间的等值判断，基本数据类型不能使用 == 进行比较，包装数据类型不能使用 equals 进行判断。
  - 说明：浮点数采用"尾数+阶码"的编码方式，类似于科学计数法的"有效数字+指数"的表示方式。二进制无法精确表示大部分的十进制小数，
    具体原理参考《码出高效》
- _强制_ BigDecimal 的等值比较应实用 compareTo() 方法，而不是 equals() 方法。
  - 说明：equals() 方法会比较值和精度(1.0 与 1.00 返回的结果为 false), 而 compareTo() 则会忽略精度。
- _强制_ 定义数据对象 DO 类时，属性类型要与数据库字段类型相匹配。
  - 正例：数据库字段的 bigint 必须与类属性的 Long 类型相对应。
  - 反例：某业务的数据库表 id 字段定义类型为 bigint unsigned, 实际类对象属性为 Integer，随着 id 越来越大，
    超过 Integer 的表示范围而溢出成为负数，此时数据库 id 不支持存入负数抛出异常产生线上故障。
- _强制_ 禁止使用构造方法 BigDecimal(double) 的方式把 double 值转化为 BigDecimal 对象。
  - 说明：BigDecimal(double)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。
    如：`BigDecimal g = new BigDecimal(0.1F);`实际的存储值为：`0.100000001490116119384765625`
  - 正例：优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了 Double 的 toString，
    而 Double 的 toString 按 Double 的实际能表达的精度对数尾进行了截断。
    - `BigDecimal recommend1 = new BigDecimal("0.1");`
    - `BigDecimal recommend2 = BigDecimal.valueOf(0.1);`
- 关于**基本数据类型与包装数据类型**的使用标准如下：
  - _强制_ 所有的 POJO 类属性必须使用包装数据类型
  - _强制_ RPC 方法的返回值和参数必须使用包装数据类型
  - _推荐_ 所有的局部变量使用基本数据类型
  - 说明：POJO 类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何 NPE 问题，或者入库检查，都由使用者来保证。
  - 正例：数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险
  - 反例：某业务的交易报表上显示成交总额涨跌情况，即正负`x%`，`x`为基本数据类型，调用的 RPC 服务，调用不成功时，返回的是默认值，
    页面显示为`0%`，这是不合理的，应该显示成中划线`-`。所以包装数据类型的 null 值，能够表示额外的信息，如：远程调用失败、异常退出。
- _强制_ 定义 DO/PO/DTO/VO 等 POJO 类时，不要设定任何属性**默认值**。
  - 反例：某业务的 DO 的 createTime 默认值为 new Date(); 但是这个属性在数据提取时并没有置入具体值，在更新其它字段时又附带更新了此字段，
    导致创建时间被修正成当前时间。
- _强制_ 序列化类新增属性时，请不要修改 serialVersionUID 字段，避免反序列失败；如果完全不兼容升级，避免反序列化混乱，那么请修改 serialVersionUID 值。
- _强制_ 构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在`init()`方法中。
- _强制_ POJO 类必须写 toString 方法。使用 IDE 中的工具 source > generate toString 时，如果继承了另一个 POJO 类，注意在前面加一下 super.toString()。
  - 说明：在方法执行抛出异常时，可直接调用 POJO 的 toString() 方法打印其属性值，便于排查问题。
- _强制_ 禁止在 POJO 类中，同时存在对应属性 xxx 的 isXxx() 和 getXxx() 方法。
  - 说明：框架在调用属性的提取方法时，并不能确定哪个方法一定是被优先调用到，神坑之一。

## 日期时间

- _强制_ 日期格式化时，传入 pattern 中表示年份统一使用小写的 y。
  - 说明：日期格式化时，yyyy 表示当天所在的年，而大写的 YYYY 代表 week in which year(JDK7 之后引入的概念)，意思是当天所在的周属于的年份，
    一周从周日开始，周六结束，只要本周跨年，返回的 YYYY 就是下一年。
  - 反例：某程序员因使用`YYYY/MM/dd`进行日期格式化，`2017/12/31`执行结果为`2018/12/31`，造成线上故障。
- _强制_ 在日期格式中分清楚大写的 M 和小写的 m，大写的 H 和小写的 h 分别指代的意义。
  - 说明：日期格式中的这两对字母表意如下：
    - M 表示月份
    - m 表示分钟
    - H 表示 24 小时制
    - h 表示 12 小时制
- _强制_ 获取当前毫秒数：`System.currentTimeMillis();`而不是`new Date().getTime();`
  - 说明：获取纳秒级时间，则使用`System.nanoTime()`的方式。在 JDK8 中，针对统计时间等场景，推荐使用 Instant 类。
- _强制_ 不允许在程序任何地方中使用`java.sql.Date/Time/Timestamp`
  - 说明：
    - `java.sql.Date`不记录时间，getHours() 抛出异常
    - `java.sql.Time`不记录日期，getYear() 抛出异常
    - `java.sql.Timestamp`在构造方法`super((time/1000)*1000)`，在 Timestamp 属性 fastTime 和 nanos 分别存储秒和纳秒信息，造成冗余。
  - 反例：`java.util.Date.after(Date)`进行时间比较时，当入参是`java.sql.Timestamp`时，会触发 JDK BUG(JDK9 已修复)，可能导致比较时的意外结果。
- _强制_ 禁止在程序中写死一年为 365 天，避免在公历闰年时出现日期转换错误或程序逻辑错误。

  - 正例：

    ```java
    // 获取今年的天数
    int daysOfThisYear = LocalDate.now().lengthOfYear();
    // 获取指定某年的天数
    LocalDate.of(2011, 1, 1).lengthOfYear();
    ```

  - 反例：
    ```java
    // 第一种情况：在闰年 366 天时，出现数组越界异常
    int[] dayArray = new int[365];
    // 第二种情况： 一年有效期的会员制，2020 年 1 月 26 日注册，硬编码 365 返回的却是 2021 年 1 月 25 日
    Calendar calendar = Calendar.getInstance();
    calendar.set(2020, 1, 26);
    calendar.add(Calendar.DATE, 365);
    ```

## 集合处理

- _强制_ 关于 hashCode 和 equals 的处理，遵循如下规则：
  - 只要覆写 equals，就必须覆写 hashCode。
  - 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须覆写这两种方法。
  - 如果自定义对象作为 Map 的键，那么必须覆写 hashCode 和 equals。
- _强制_ 判断所有集合内部的元素是否为空，使用 isEmpty() 方法，而不是`size() == 0`的方式。
  - 说明：某些集合中，前者的时间复杂度为`O(1)`，而且可读性更好。
- _强制_ 在使用 java.util.stream.Collectors 类的 toMap() 方法转为集合时，一定要使用参数类型为 BinaryOperator，
  参数名为 mergeFunction 的方法，否则当出现相同 key 时会抛出 illegalStateException 异常。
  - 说明:参数 mergeFunction 的作用是当出现 key 重复时，自定义对 value 的处理策略。
  - 正例：
    ```java
    List<Pair<String, Double>> pairArrayList = new ArrayList<>(3);
    pairArrayList.add(new Pair<>("version", 12.10));
    pairArrayList.add(new Pair<>("version", 12.19));
    pairArrayList.add(new Pair<>("version", 6.28));
    // 生成的 map 集合中只有一个键值对：{"version"=6.28}
    Map<String, Double> map = pairArrayList.stream().collect(Collectors.toMap(Pair::getKey, Pair::getValue, (v1, v2) -> v2));
    ```
- _强制_ 在使用`java.util.stream.Collectors`类的`toMap()`方法转为 Map 集合时，一定要注意当 value 为 null 时会抛出 NPE 异常。
  - 说明：在`java.util.HashMap`的 merge 方法里会进行如下的判断：
    ```java
    if (value == null || remappingFunction == null)
      throw new NullPonterException();
    ```
- _强制_ ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常：
  `java.util.RandomAccessSubList cannot be cast to java.util.ArrayList`。
  - 说明：`subList()`返回的是 ArrayList 的内部类 SubList，并不是 ArrayList 本身，而是 ArrayList 的一个视图，
    对于 SubList 的所有操作最终会反映到原列表上。
- _强制_ 使用 Map 的方法 keySet()/values()/entrySet() 返回集合对象时，不可以对其进行添加元素操作，否则会抛出 UnsupportedOperationException 异常。
- _强制_ Collections 类返回的对象，如：emptyList()/singletonList() 等都是 immutable list，不可对其进行添加或者删除元素的操作。
  - 反例：如果查询无结果，返回 Collections.emptyList() 空集合对象，调用方一旦在返回的集合中进行了添加元素的操作，就会触发 UnsupportedOperationException 异常。
- _强制_ 在 subList 场景中，**高度注意**对父集合元素的增加或删除，均会导致子列表的遍历、增加、删除产生 ConcurrentModificationException 异常。
  - 说明：抽查表名，90%的程序员对此知识点都有错误的认知。
- _强制_ 使用集合转数组的方法，必须使用集合的`toArray(T[] array)`，传入的是类型完全一致、长度为 0 的空数组。
  - 反例：直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[] 类，若强转其它类型数组将出现 ClassCastException 错误。
  - 正例：
    ```Java
      List<String> list = new ArrayList<>(2);
      list.add("xxx");
      list.add("yyy");
      String[] array = list.toArray(new String[0]);
    ```
  - 说明：使用 toArray 带参方法，数组空间大小的 length:
    1. 等于 0，动态创建与 size 相同的数组，性能最好。
    2. 大于 0 但小于 size，重新创建大小等于 size 的数组，增加 GC 负担。
    3. 等于 size，在高并发情况下，数组创建完成之后，size 正在变大的情况下，负面影响与 2 相同。
    4. 大于 size，空间浪费，且在 size 处插入 null 值，存在 NPE 隐患。
- _强制_ 使用 Collection 接口任何实现类的 addAll() 方法时，要对输入的集合参数进行 NPE 判断。
  - 说明：在 ArrayList#addAll 方法的第一行代码即`Object[] a = c.toArray();`其中 c 为输入集合参数，如果为 null，则直接抛出异常。
- _强制_ 使用工具类 Arrays.asList() 把数组转换成集合时，不能使用其修改集合(长度)相关的方法，它的 add/remove/clear 方法会抛出
  UnsupportedOperationException 异常。
  - 说明：asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList 体现的是适配器模式，只是转换接口，后台的数据仍是数组。
    ```Java
    String[] str = new String[]{"a", "b"};
    List list = Arrays.asList(str);
    ```
    - `list.add("c");` 运行时异常。
    - `str[0] = "c";` list 中的元素也会随之修改，反之亦然。
- _强制_ 泛型通配符`<？ extends T>`来接收返回的数据，此写法的泛型集合不能使用 add 方法，而`<? supper T>`不能使用 get 方法，两者在接口调用赋值的场景中容易出错。
  - 说明：扩展说一下`PECS(Producer Extends, Consumer Super)`原则，即频繁往外读取内容的，适合用`<? extends T>`; 经常往里插入的，适合用`<? supper T>`
- _强制_ 在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行 instanceof 判断，避免抛出 ClassCastException 异常。
  说明：笔记泛型是在 JDK5 后才出现，考虑到向前兼容，编译器是允许非泛型集合与泛型集合相互赋值。
  反例：

  ```java
  List<String> generics = null;
  List notGenerics = new ArrayList(10);
  notGenerics.add(new Object());
  notGenerics.add(new Integer(1));
  generics = notGenerics;
  // 此处抛出 ClassCastException 异常
  String string = generics.get(0);
  ```

- _强制_ 不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 iterator 方式，如果并发操作，需要对 iterator 对象加锁。
  - 正例：
    ```java
    List<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
      String item = iterator.next();
      if (删除元素的条件) {
        iterator.remove();
      }
    }
    ```
  - 反例：
    ```java
    for (String item : list) {
      if ("1".equals(item)) {
        list.remove(item);
      }
    }
    ```
  - 说明：返利中的执行结果肯定会出乎大家的意料，那么试一下把 "1" 换成 "2" 会是同样的结果吗？
    - "1"可以正常删除；"2"会报错"java.util.ConcurrentModificationException"
- _强制_ 在 JDK7 版本及以上，Comparator 实现类要满足如下三个条件，不然 Arrays.sort, Collections.sort 会抛`IllegalArgumentException`异常。

  - 说明：三个条件如下：
    1. x, y 的比较结果和 y, x 的比较结果相反。
    2. x > y, y > z, 则 x > z。
    3. x = y, 则 x, z 比较结果和 y, z 比较结果相同。
  - 反例：下例中没有处理相等的情况，交换两个对象判断结果并不互反，不符合第一个条件，在实际使用中可能会出现异常。

    ```java
    new Comparator<Student>() {
      @Override
      public int compare(Student o1, Student o2) {
        return o1.getId() > o2.getId() ? 1 : -1;
      }
    }

    ```

## 并发处理

- _强制_ 获取单例对象需要保证线程安全，其中的方法也要保证线程安全。
  - 说明：资源驱动类、工具类、单例工厂类都需要注意。
- _强制_ 创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。
  - 正例：自定义线程工厂，并且根据外部特征进行分组，比如，来自同一机房的调用，把机房编号赋值给 whatFeatureOfGroup:
    ```java
    public class UserThreadFactory implements ThreadFactory {
      private final String namePrefix;
      private final AtomicInteger nextId = new AtomicInteger(1);
      // 定义线程组名称，在利用 jstack 来排查问题时，非常有帮助
      UserThreadFactory(String whatFeatureOfGroup) {
        namePrefix = "FromUserThreadFactory's" + whatFeatureOfGroup + "-Worker-";
      }
      @Override
      public Thread newThread(Runnable task) {
        String name = namePrefix + nextId.getAndIncrecement();
        Thread thread = new Thread(null, task, name, 0, false);
        System.out.println(thread.getName());
        return thread;
      }
    }
    ```
- _强制_ 线程资源必须通过线程池提供，不允许在应用中自行显示创建线程。
  - 说明：线程池的好处是减少在创建和销毁线程上所小号的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，
    有可能造成系统创建大量同类线程而导致消耗完内存，或者"过度切换"的问题。
- _强制_ 线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
  - 说明：Executors 返回的线程池对象的弊端如下：
    1. FixedThreadPool 和 SingleThreadPool
       允许的**请求队列长度**为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。
    2. CachedThreadPool:
       允许的**创建线程数量**为`Integer.MAX_VALUE`，可能会创建大量的请求，从而导致 OOM。
    3. ScheduledThreadPool:
       允许的**请求队列长度**为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。
- _强制_ SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为 static，必须加锁，或者使用 DateUtils 工具类。
  - 正例：注意线程安全，使用 DateUtils。亦推荐如下处理：
    ```java
    private static final ThreadLocal<DateFormat> dateStyle = new ThreadLocal<DateFormat>() {
      @Overide
      protected DateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd");
      }
    }
    ```
  - 说明：如果是 JDK8 的应用，可以使用 Instant 代替 Date, LocalDateTime 代替 Calendar, DateTimeFormatter 代替 SimpleDateFormat，
    官方给出的解释：simple beautiful strong immutable thread-safe。
- _强制_ 必须回收自定义的 TreadLocal 变量记录的当前线程的值，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成泄漏等问题。
  尽量在代码中使用 try-finally 块进行回收。
  - 正例：
    ```java
    objectThreadLocal.set(userInfo);
    try {
      // ...
    } finally {
      objectThreadLocal.remove();
    }
    ```
- _强制_ 高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁。
  - 说明：尽可能使用加锁的代码块工作量尽可能的小，避免在所代码块中调用 RPC 方法。
- _强制_ 对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁。
  - 说明：线程 1 需要对表 A、B、C 依次全部加锁后才可以进行更新操作，那么线程 2 的加锁顺序也必须是 A、B、C，否则可能出现死锁。
- _强制_ 在使用阻塞等待获取锁的方式中，必须在 try 代码块之外，并且在加锁方法与 try 代码块之间没有任何可能抛出异常的方法调用，避免加锁成功后，在 finnaly 中无法解锁。

  - 说明 1：在 lock 方法与 try 代码块之间的方法调用抛出异常，无法解锁，造成其它线程无法成功获取锁。
  - 说明 2：如果 lock 方法在 try 代码块内，可能由于其它方法抛出异常，导致在 finally 代码块中，unlock 对未加锁的对象解锁，它会调用 AQS 的 tryRelease 方法(取决于具体实现类)，抛出 illegalMonitorStateException 异常。
  - 说明 3：在 Lock 对象的 Lock 方法实现中可能抛出 unchecked 异常，产生的后果与说明 2 相同。
  - 正例：

    ```java
    Lock lock = new XxxLock();
    // ...
    lock.lock();
    try {
      doSomething();
      doOthers();
    } finally {
      lock.unlock();
    }
    ```

  - 反例：
    ```java
    Lock lock = new XxxLock();
    // ...
    try {
      // 如果此处抛出异常，则直接执行 finally 代码块
      doSomething();
      // 无论加锁是否成功，finally 代码块都会执行
      lock.lock();
      doOthers();
    } finally {
      lock.unlock();
    }
    ```

- _强制_ 在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。
  锁的释放规则与锁的阻塞等待方式相同。
  - 说明：Lock 对象的 unlock 方法在执行时，它会调用 AQS 的 tryRelease 方法（取决于具体实现类），如果当前线程不持有锁，则抛出 IllegalMonitorStateException 异常。
  - 正例：
    ```java
    Lock lock = new XxxLock();
    // ...
    boolean isLocked = lock.tryLock();
    if (isLocked) {
      try {
        doSomething();
        doOthers();
      } finally {
        lock.unlock();
      }
    }
    ```
- _强制_ 并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加锁，要么在数据层使用乐观锁，使用 version 作为更新一句。
  - 说明：如果每次访问冲突小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于 3 次。
- _强制_ 多线程并行处理任务时，Timer 运行多个 TimeTask 时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用 ScheduledExecutorService 则没有这个问题。

## 控制语句

- _强制_ 在一个 switch 块内，每个 case 要么通过 continue/break/return 等来终止，要么注释说明程序将继续执行到哪一个 case 为止；在一个 switch 块内，都必须包含一个 default 语句并且放在最后，即使它什么代码也没有。
  - 说明：注意 break 是退出 switch 语句块，而 return 是退出方法体。
- _强制_ 当 switch 括号内的变量类型为 String 并且此变量为外部参数时，必须先进行 null 判断。
  - 反例：如下的代码输出是什么？
    ```java
    public class SwitchString {
      public static void main(String[] args) {
        method(null);
      }
      public static void method(String param) {
        switch(param) {
          // 肯定不是进入这里
          case "sth":
            System.out.println("it's sth");
            break;
          // 也不是进入这里
          case "null":
            System.out.println("it's null");
            break;
          // 也不是这里
          default:
            System.out.println("default");
        }
      }
    }
    ```
- _强制_ 在`if/else/for/while/do`语句中必须使用大括号。
  - 反例: `if (condition) statements;`
  - 说明: 即使只有一行代码，也要采用大括号的编码方式
- _强制_ 三目运算符`condition ? 表达式1 : 表达式2`中，高度注意表达式 1 和 2 在类型对齐时，可能抛出因自动拆箱导致的 NPE 异常。

  - 说明：以下两种场景会触发类型对齐的拆箱操作：
    1. 表达式 1 或 2 的值只要有一个是原始类型。
    2. 表达式 1 或 2 的值的类型不一致，会强制拆箱升级成表示范围更大的那个类型。
  - 反例：

    ```java
    Integer a = 1;
    Integer b = 2;
    Integer c = null;
    Boolean flag = false;
    // a*b 的结果是 int 类型，那么 c 会强制拆箱成 int 类型，抛出 NPE 异常
    Integer result = (flag ? a*b : c);
    ```

- _强制_ 在高并发场景中，避免使用"等于"判断作为中断或退出的条件。
  - 说明：如果并发控制没有处理好，容易产生等值判断被"击穿"的情况，使用大于或小于的区间判断条件来代替。
  - 反例：判断剩余奖品数量等于 0 时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，这样的话，活动无法终止。

## 注释规约

- _强制_ 类、类属性、类方法的注释必须使用 Javadoc 规范，使用`/** 内容 */`格式，不得使用`// xxx`方式。
  - 说明：在 IDE 编辑窗口中，Javadoc 方式会提示相关注释，生成 Javadoc 可以正确输出相应注释；在 IDE 中，工程调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率。
- _强制_ 所有的抽象方法(包括接口中的方法)必须要用 Javadoc 注释、除了返回值、参数异常说明外，还必须指出该方法做什么事情，实现什么功能。
  - 说明：对于子类的实现要求，或者调用注意事项，请一并说明。
- _强制_ 所有类都必须添加创建者和创建日期。
  - 说明：在设置模板时，注意 IDEA 的@author 为`${USER}`，而 eclipse 的@author 为`${user}`，大小有区别，而日期的设置统一为 yyyy/MM/dd 的格式。
  - 正例：
  ```java
  /**
    *
    * @author test
    * @date 20250604
    *
    **/
  ```
- _强制_ 方法内部单行注释，在被注释语句上方另起一行，使用 // 注释。方法内部多行注释用 /\* \*/ 注释，注意与代码对齐。
- _强制_ 所有的枚举类型字段必须要有注释，说明每个数据项的用途。

## 前后端端规约

- _强制_ 前后端交互的 API，需要明确协议、域名、路径、请求方法、请求内容、状态码、响应体。
  - 说明：
    1. 协议：生产环境必须使用 HTTPS。
    2. 路径：每一个 API 需对应一个路径，表示 API 具体的请求地址：
       1. 代表一种资源，只能为名词，推荐使用复数，不能为动词，请求方法已经表达动作意义。
       2. URL 路径不能使用大写，单词如果需要分隔，统一使用下划线。
       3. 路径禁止携带表示请求内容类型的后缀，比如".json"，".xml"，通过 accept 头表达即可。
    3. 请求方法：对具体操作的定义，常见的请求方法如下：
       1. GET: 从服务器取出资源
       2. POST: 在服务器新建一个资源。
       3. PUT: 在服务器更新资源。
       4. DELETE: 从服务器删除
    4. 请求内容：URL 带的参数必须无敏感信息或符合安全要求；body 里带参数时必须设置 Content-Type。
    5. 响应体：响应体 body 可放置多种数据类型，由 Content-Type 头来确定。
- _强制_ 前后端数据列表相关的接口返回，如果为空，则返回空数组`[]`或空集合`{}`。
  - 说明：此约定有利于数据层面上的协作更加高效，减少前端很多琐碎的 null 判断。
- _强制_ 服务端发生错误时，返回给前端的响应信息必须包含 HTTP 状态码，errorCode、errorMessage、用户提示信息四个部分。
  - 说明：四个部分的涉众对象分别是浏览器、前端开发、错误排查人员、用户。其中输出给用户的提
    - 示信息要求：简短清晰、提示友好，引导用户进行下一步操作或解释错误原因，提示信息可以包括错误原因、上下文环境、推荐操作等。
    - errorCode: 参考"附表 3"。
    - errorMessage：简要描述后端出错原因，便于错误排查人员快速定位问题，注意不要包含敏感数据信息。
  - 正例：常见的 HTTP 状态码如下
    1. `200 OK`: 表名该请求被成功地完成，所请求的资源发送到客户端。
    2. `401 Unauthorized`: 请求要求身份验证，常见对于需要登录而用户未登录的情况。
    3. `403 Forbidden`: 服务拒绝请求，常见于机密信息或复制其它登录用户链接访问服务器的情况。
    4. `404 NotFound`: 服务器无法取得所请求的网页，请求资源不存在。
    5. `500 InternalServerError`: 服务器内部错误。
- _强制_ 在前后端交互的 JSON 格式数据中，所有的 key 必须为小写字母开始的 lowerCamelCase 风格，符合英文表达习惯，且表意完整。
  - 正例：errorCode, errorMessage, assetStatus, menuList, orderList, configFlag
  - 反例：ERRORCODE, ERROR_CODE, error_message, error-message, errormessage
- _强制_ errorMessage 是前后端错误追踪机制的体现，可以在前端输出到 type="hidden" 文字类控件中，或者用户端的日志中，帮助我们快速地定位出问题。
- _强制_ 对于需要使用超大整数的场景，服务端一律使用 String 字符串类型返回，禁止使用 Long 类型。
  - 说明：Java 服务端如果直接返回 Long 整型数据给前端，Javascript 会自动转换为 Number 类型（注：此类型为双精度浮点数，表示原理与取值范围等于 Java 中的 Double）。Long 类型能表示的最大值是`2^(63-1)`，在取值范围之内，超过`2^53`(9007199254740992)的数值转化为 Javascript 的 Number 时，有些数值会产生精度损失。扩展说明，在 Long 取值范围内，任何 2 的指数次的整数都是绝对不会存在精度损失的，所以说精度损失十一个概率问题。若浮点数尾数与指数位空间不限，则可以精确表示任何整数，但很不幸，双精度浮点数的数尾位只有 52 位。
  - 反例：通常在订单号或交易号大于 16 位，大概率会出现前后端订单数据不一致的情况。
    比如，后端传输的`"orderId": 362909601374617692`，前端拿到的值却是：`362909601374617660`
- _强制_ HTTP 请求通过 URL 传递参数时，不能超过 2048 字节。
  说明：不同浏览器对于 URL 的最大长度限制略有不同，并且对超出最大长度的处理逻辑也有差异，2048 字节是取所有浏览器的最小值。
  反例：某业务将退货的商品 id 列表放在 URL 中作为参数传递，当一次退货商品数量过多时，URL 参数超长，传递到后端的参数被截断，导致部分商品未能正确退货。
- _强制_ HTTP 请求通过 body 传递内容时，必须控制长度，超出最大长度后，后端解析会出错。
  说明：nginx 默认限制是 1MB，tomcat 默认限制为 2MB，当确实有业务需要传较大内容时，可以调大服务器端的限制。
- _强制_ 在翻页场景中，用户输入参数的小于 1，则前端返回第一页参数给后端；后端发现用户输入的参数大于总页数，直接返回最后一页。
- _强制_ 服务器内部重定向必须使用 forward; 外部重定向地址必须使用 URL 统一代理模块生成，否则会因线上采用 HTTPS 协议导致浏览器提示"不安全"，并且还会带来 URL 维护不一致的问题。

## 其它

- _强制_ 在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度。
  - 说明：不要在方法体内定义：`Pattern pattern = Pattern.compile("规则");`
- _强制_ 避免用 ApacheBeanUtils 进行属性的 copy。
  - 说明：ApacheBeanUtils 性能较差，可以使用其它方案比如 SpringBeanUtils, CglibBeanCopier，注意均是浅拷贝。
- _强制_ velocity 调用 POJO 类型的属性时，直接使用属性名取值即可，模板引擎会自动按规范调用 POJO 的 getXxx(), 如果是 boolean 基本数据类型变量(boolean 命名不需要加 is 前缀)，会自动调用 isXxx() 方法。
  - 说明：注意如果是 Boolean 包装类对象，优先调用 getXxx() 的方法。
- _强制_ 后台输送给页面的变量必须加`$!{var}`--中间的感叹号。
  - 说明：如果 var 等于 null 或者不存在，那么`${var}`会直接显示在页面上。
- _强制_ 注意 Math.random() 这个方法返回的是 double 类型，注意取值的范围`0 <= x < 1`(能够取到零值，注意除零异常)，如果想获取证书类型的随机数，不要将 x 放大 10 的若干倍然后取整，直接使用 Random 对象的 nextInt 或者 nextLong 方法。
- _强制_ 枚举 enum(括号内) 的属性字段必须是私有且不可变。

# 异常日志

## 错误码

- _强制_ 错误码的制定原则：快速溯源、沟通标准化。
  - 说明：错误码想的过于完美和复杂，就像康熙字典的生僻字一样，用词似乎精准，但是字典不容易随身携带且简单易懂。
  - 正例：错误码回答的问题是谁的错？错在哪？
    1. 错误码必须能够快速知晓错误来源，可快速判断是谁的问题。
    2. 错误码必须能够进行清晰地对比(代码中容易 equals)。
    3. 错误码有利于团队快速对错误原因达到一致认知。
- _强制_ 错误码不体现版本号和错误等级信息。
  - 说明：错误码以不断追加的方式进行兼容。错误等级由日志和错误码本身的释义来决定。
- _强制_ 全部正常，但不得不填充错误码时，返回五个零：00000
- _强制_ 错误码为字符串类型，共 5 位，分成两个部分：错误产生来源 + 四位数字编号。
  - 说明：错误产生来源分为 A/B/C, A 表示错误来源于用户，比如参数错误，用户安装版本过低，用户支付超时等问题；
    B 表示错误来源于当前系统，往往是业务逻辑出错，或程序健壮性差等问题；
    C 表示错误来源于第三方服务，比如 CDN 服务出错，消息投递超时等问题；
    四位数字编号从 0001 到 9999，大类之间的步长间距预留 100，参考文末《附表 3》。
- _强制_ 编号不与公司业务架构，更不与组织架构挂钩，以先到先得的原则在统一平台上进行，审批生效，编号即被永久固定。
- _强制_ 错误码使用者避免随意定义新的错误码。
  - 说明：尽可能在原有错误码附表中找到语义相同或相近的错误码在代码中使用即可。
- _强制_ 错误码不能直接输出给用户作为提示信息使用。
  - 说明：对战(stack_trace)、错误信息(error_message)、错误码(error_code)、提示信息(user_tip)十一个有效关联并互相转义的协调整体，但请勿互相越俎代庖。

## 异常处理

- _强制_ Java 类库中定义的可以通过预检查方式规避的 RuntimeException 异常不应该通过 catch 的方式来处理，比如：NullPointerException, IndexOutOfBoundsException 等等。
  - 说明：无法通过与检查的异常除外，比如，在解析字符串形式的数字时，可能存在数字格式错误，不得不通过 catch NumberFormatException 来实现。
  - 正例：`if (obj != null) {...}`
  - 反例：`try { obj.method(); } catch (NullPointerException e) {...}`
- _强制_ 异常捕获后不要用来做流程控制，条件控制。
  - 说明：异常涉及的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多。
- _强制_ catch 时请分清稳定代码和非稳定代码，稳定代码指的是无论如何不会出错的代码。对于非稳定代码的 catch 尽可能进行区分异常类型，再做对应的异常处理。
  - 说明：对大段代码进行 try-catch, 使程序无法根据不同的异常做出正确的应激反应，也不利于定位问题，这是一种不负责任的表现。
  - 正例：用户注册的场景中，如果用户输入非法字符，或用户名称已经存在，或用户输入密码过于简单，在程序上做出分门别类的判断，并提示给用户。
- _强制_ 捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。
- _强制_ 事务场景中，抛出异常被 catch 后，如果需要回滚，一定要注意手动回滚事务。
- _强制_ finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。
- _强制_ 不要在 finally 块中使用 return。
  - 说明：try 块中的 return 语句执行成功后，并不马上返回，而是继续执行 finally 块中的语句，如果此处存在 return 语句，则会在此直接返回，无情丢弃掉 try 块中的返回点。
    反例：
    ```java
    private int x = 0;
    public int checkReturn() {
      try {
        // x 等于 1，此处不返回
        return ++x;
      } finally {
        // 返回的结果是 2
        return ++x;
      }
    }
    ```
- _强制_ 捕获异常与抛异常，必须是完全匹配，或者捕获异常是抛异常的父类。
  - 说明：如果预期对方抛的是绣球，实际接到的是铅球，就会产生意外情况。
- _强制_ 在调用 RPC、二方包、或动态生成类的相关方法时，捕捉异常使用 Throwable 类进行拦截。
  - 说明：通过反射机制来调用方法，如果找不到方法，抛出 NoSuchMethodException。什么情况下会抛出 NoSuchMethodError 呢？二方包在类冲突时，仲裁机制可能导致引入非预期的版本使类的方法签名不匹配，或者在字节码修改框架(比如：ASM)动态创建或修改类时，修改了响应的方法签名。这些情况，即使代码编译期是正确的，但在代码运行期时，会抛出 NoSuchMethodError。
  - 反例：足迹服务引入了高版本的 spring，导致运行到某段核心逻辑时，抛出 NoSuchMethodError 错误，catch 用的类却是 Exception，堆栈向上抛，影响到上层业务。这是一个非核心功能点影响到核心应用的典型反例。

## 日志规约

- _强制_ 应用中不可直接使用日志系统(Log4j、Logback)中的 API，而应该依赖使用日志框架(SLF4J、JCL--Jakarta Commons Logging)中的 API，使用门面模式的日志框架，有利于维护各个类的日志处理方式统一。

  - 说明：日志处理框架(SLF4J、JCL--Jakarta Commons Logging)的使用方式(推荐使用 SLF4J)

    - 使用 SLF4J:

    ```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    private static final Logger logger = LoggerFactory.getLogger(Test.class);
    ```

    - 使用 JCL:

    ```java
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    private static final Log log = LogFactory.getLog(Test.class);
    ```

- _强制_ 日志文件至少保存 15 天，因为有些异常具备以"周"为频次发生的特点。对于当前日志，以"应用名.log"来保存，保存在`/{同一目录}/{应用名}/logs/`目录下，过往日志格式为：`{logname}.log.{保存日期}`，日期格式：`yyy-MM-dd`
  - 正例：以 mppserver 应用为例，日志保存`/home/admin/mppserver/logs/mppserver.log`，历史日志名称为`mppserver.log.2025-06-16`
- _强制_ 根据国家法律，网络运行状态、网络安全事件、个人敏感信息操作等相关记录，留存的日志不少于六个月，并且进行网络多机备份。
- _强制_ 应用中的扩展日志（如打点、临时监控、访问日志）命名方式：
  appName_logType_logName.log。logType: 日志类型，如 stats/monitor/access 等；
  logName: 日志描述。这种命名的好处：通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有有利于归类查找。
  - 说明：推荐对日志进行分类，将错误日志和业务日志分开放，便于开发人员查看，可便于通过日志对系统进行及时监控。
  - 正例：mppserver 应用中单独监控时区转换异常，如：mppserver_monitor_timeZoneConvert.log
- _强制_ 在日志输出时，字符串变量之间的拼接使用占位符的方式。
  - 说明：因为 String 字符串的拼接会使用 StringBuilder 的 append() 方式，有一定的性能损耗。使用占位符仅是替换动作，可以有效提升性能。
  - 正例：`logger.debug("Processing trade with id: {} and symbol: {}", id, symbol)`
- _强制_ 对于 trace/debug/info 级别的日志输出，必须进行日志级别的开关判断。
  - 说明：虽然在 debug(参数) 的方法体内第一行代码 isDisabled(Level.DEBUG_INT) 为真时(Slf4j 的常见实现 Log4j 和 Logback)，就直接 return，但是参数可能会进行字符串拼接运算。此外，如果 debug(getName) 这种参数内有 getName() 方法调用，无谓浪费方法调用的开销。
  - 正例：
  ```java
  // 如果判断为真，那么可以输出 trace 和 debug 级别的日志
  if (logger.isDebugEnabled()) {
    logger.debug("Current ID is: {} and name is: {}", id, getName());
  }
  ```
- _强制_ 避免重复打印日志，浪费磁盘空间，务必在日志配置文件中设置 additivity=false
  - 正例：`<logger name="com.taobao.dubbo.config" additivity="false">`
- _强制_ 生产环境禁止使用 System.out 或 System.err 输出或使用 e.printStackTrace() 打印异常堆栈。
  - 说明：标准日志输出与标准错误输出文件每次 Jboss 重启时才滚动，如果大量输出送往这两个文件，容易造成文件大小超过系统大小限制。
- _强制_ 异常信息应该包括两类信息：案发现场信息和异常堆栈。如果不处理，那么通过关键字 throw 往上抛出。
  - 正例：`logger.error("inputParams: {} and errorMessage: {}", 各类参数或对象 toString(), e.getMessage(), e);`
- _强制_ 日志打印时禁止直接用 JSON 工具将对象转换成 String。
  - 说明：如果对象里某些 get 方法被覆写，存在抛出异常的情况，则可能会因为打印日志而影响正常业务流程的执行。
  - 正例：打印日志时，仅打印出业务相关属性值或调用其对象的 toString() 方法。

## 单元测试

- _强制_ 好的单元测试必须遵守 AIR 原则。
  - 说明：单元测试在线上运行时，像空气(AIR)一样感觉不到，但在测试质量的保障上，却是非常关键的。好的单元测试宏观上来说，具有：
    - A(Automatic) 自动化
    - I(Idenpendent) 独立性
    - R(Repeatable) 可重复
- _强制_ 单元测试应该是全自动执行的，并且非交互式的。测试用例通常是被定期执行的，执行过程必须完全自动化才有意义。输出结果需要人工检查的测试不是一个好的单元测试。不允许使用 System.out 来进行人肉验证，但愿测试必须使用 assert 来验证。
- _强制_ 保持单元测试的独立性。为了保证单元测试稳定可靠且便于维护，单元测试用例之间决不能相互调用，也不能依赖执行的先后次序。
  - 反例： method2 需要依赖 method1 的执行，将执行结果作为 method2 的输入。
- _强制_ 单元测试是可以重复执行的，不能受到外界环境的影响。
  - 说明：单元测试通常会被放到持续集成中，每次有代码 push 时单元测试都会被执行。如果单测对外部环境(网络、服务、中间件等)有依赖，容易导致持续集成机制的不可用。
  - 正例：为了不受外界环境影响，要求设计代码时就把 SUT(Software under test)的依赖改成注入，在测试时用 Spring 这样的 DI 框架注入一个本地(内存)实现或者 Mock 实现。
- _强制_ 对于单元测试，要保证测试粒度足够小，有助于精确定位问题。单测粒度至多是类级别，一般是方法级别。
  - 说明：测试粒度小才能在出错时尽快定位到出错的位置。单元测试不负责检查跨类或者跨系统的交互逻辑，那是集成测试的领域。
- _强制_ 核心业务、核心应用、核心模块的增量代码确保单元测试通过。
  - 说明：新增代码及时补充单元测试，如果新增代码影响了单元测试，请及时修正。
- _强制_ 但愿测试代码必须写在如下工程目录：src/test/java，不允许写在业务代码目录下。
  - 说明：源码编译时会跳过此目录，而单元测试框架默认是扫描此目录。

## 安全规约

- _强制_ 隶属于用户个人的页面或者功能必须进行权限控制校验。
  - 说明：放置没有做水平权限校验就可随意访问、修改、删除别人的数据，比如查看他人的私信内容。
- _强制_ 用户敏感数据禁止直接展示，必须对展示数据进行脱敏。
  - 正例：中国大陆个人手机号码显示：`139****1219`，隐藏中间 4 位，防止隐私泄露。
- _强制_ 用户输入的 SQL 参数严格使用参数绑定或者 METADATA 字段值限定，防止 SQL 注入，禁止字符串拼接 SQL 访问数据库。
  - 反例：某系统签名大量被恶意修改，即是因为对于危险字符`#--`没有进行转义，导致数据库更新时，where 后边的信息被注释掉，对全库进行更新。
- _强制_ 用户请求传入的任何参数必须做有效性验证。
  - 说明：忽略参数校验可能导致：
    - 页面 page size 过大导致内存溢出
    - 恶意 order by 导致数据库慢查询
    - 缓存击穿
    - SSRF
    - 任意重定向
    - SQL 注入，Shell 注入, 反序列化注入
    - 正则输入源串拒绝服务 ReDoS
      - 扩展：Java 代码用正则来验证客户端的输入，有些正则写法验证普通用户输入没问题，但如果攻击人员使用的是特殊构造的字符串来验证，有可能导致死循环的结果。
- _强制_ 禁止向 HTML 页面输出未经安全过滤或未正确转义的用户数据。
  - 说明：XSS 跨站脚本攻击。它指的是恶意攻击者往 Web 页面里插入恶意 html 代码，当用户浏览时，迁入其中 Web 里面的 html 代码会被执行，造成获取用户 cookie、钓鱼、获取用户页面数据、蠕虫、挂马等危害。
- _强制_ 表单、AJAX 提交必须执行 CSRF 安全验证。
  - 说明：CSRF(Cross-site request forgery)跨站请求伪造是一类常见编程漏洞。对于存在 CSRF 漏洞的应用/网站，攻击者可以实现构造好 URL，只要受害者用户一访问，后台便在用户不知情的情况下对数据库中用户参数进行相应修改。

# 参考链接

- [Java 开发手册(黄山版)](<https://github.com/alibaba/p3c/blob/master/Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C(%E9%BB%84%E5%B1%B1%E7%89%88).pdf>)
