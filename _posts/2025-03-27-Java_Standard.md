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
  - 因为 Set 存储的是不重复的对象，一句 hashCode 和 equals 进行判断，所以 Set 存储的对象必须覆写这两种方法。
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
- _强制_ 使用 Map 的方法 keySet()/values()/entrySet()

# 参考链接

- [Java 开发手册(黄山版)](<https://github.com/alibaba/p3c/blob/master/Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C(%E9%BB%84%E5%B1%B1%E7%89%88).pdf>)
