---
layout: post
title: "Algorithm Tips"
date: 2020-12-20 21:00:00 +0800
tags: Algorithm Leetcode
---

![Algorithm_Tips](/assets/images/2020-12-20-Algorithm_Tips_1.png)
记录常用算法技巧

# 基本概念

- **O(Big O)**
  表示随问题元素数量增加，所需步骤/空间的增加速度。
  - 常见的有：
    - exponential(指数的)
      比如 dfs
    - polynomial(多项式的)
      如 O(n)、O(n^2)
    - linear(线性的)
      等价于 O(n)
    - 对数的(logn)
      比如 二分查找

# 做题流程

- 看清题意，理解问题，看懂给定的 case。提前准备纸笔，可以用来画图和模拟演化。
- 记录给定的 case、自己创建可能发生的情况的 case（如：数组为空、参数越界、前后顺序差异、重复变量...），存储在临时文本位置。如果有 case 不确定正常返回值，可以填写默认返回值，测试一下想到的 case，如果期待结果是报错的，说明不会有这个 case。
- 做题，将上面所有的 case 跑通。如果思路没问题，可以先将主要逻辑写出来，然后再考虑边界情况。如果发生错误，修改后，跑通当前 case 以及可能影响的 case。
- 确保没有问题后，修正格式、有意义的变量名、函数名、加上关键注释、删除 log 输出、删除无用注释，重新跑一遍所有 case。
- 最后提交代码。真正的线上笔试只能提交一次，提交一次就成功与提交很多次的难度不在一个数量级，所以要以一次提交成功为目标进行练习。

## 其他技巧

- 除非是题目留下的题目类本身的构造函数，不要自己去重新写构造函数，可能会有问题。
- leetcode 运行时报错，一般都是数组越界。有时也是空指针问题。
- leetcode 运行 Go 代码时，报"Runtime error"，一般是有死循环了。
- 有时"Run Code"和"Submit"检测结果不同，这时，很可能是某些变量没有初始化，在"Run Code"时由于是 Debug 模式，所以自动赋了初值，而"Submit"时没有初始化导致的。
- 在逻辑较为复杂时，可以先将逻辑嵌套写出来，然后再进行精简。一上来就精简可能会造成逻辑复杂，导致 case miss。
- 在 leetcode 中，可以通过 cout 输出 log。
- 看到题目中规模 n 的范围是 500~1000 数量级，说明题目不能用搜索可能性空间的 dfs 解法而要用 DP 来做，因为 500 对于 O(n2)<bf>来说太小了不够测，而对 O(nn)<dfs>来说足够超时了。
- 在 leetcode 中，认为红黑树的创建、遍历过程为`O(nlogn*) ≈ O(n)`，红黑树被广泛应用于 set、map 系列中。
- 在 double 型运算中，通常用 eps = 1e-6(10^-6)为期望(expects)测精度。
- leetcode 中经常要对 1000000007 取模，可以简写为 const int MOD = 1e9+7。

# 数据结构

## Array(数组)

## LinkedList(链表)

单向链表、双向链表、环状链表(Ring)

### 算法：快慢指针法

用于判断链表中是否存在回路，以及交叉点的位置。

对于一个有向单链表或类似的可形成环装的结构，为了判断是否存在回路(loop)，
可以分别用快、慢指针遍历，快指针每次走两步、慢指针每次走一步，如果快指针与慢指针在某个位置重叠了，
说明存在回路。例:"457. Circular Array Loop"、"141. Linked List Cycle"。
对于需要快速跳转到单链表结尾判断长度，并从中间开始操作的情况，也可以使用快慢指针法。例："234. Palindrome Linked List"。

### 技巧：添加辅助头节点

遍历处理链表时往往头节点的处理非常复杂，
可以 new 一个辅助 dummyHead 节点，设置合适的 Val 值，将 Next 设置为原 head，这样就可以无差别遍历了。

示例：
[`82. Remove Duplicates from Sorted List II` Golang]()

## Stack(栈)

符合 FILO(先进后出)，从栈顶入从栈顶出

### 算法：Monotone Stack(单调栈)

可以在遍历过程中保存到目前为止的一系列有序极值

示例：
[`456. 132 Pattern`]()

## Queue(队列)

符合 FIFO(先进先出)，一般从队尾入队，队首出队

### DQueue(double queue，双向队列、双端队列)

dq 可以同时提供两个 queue，一个 queue 的队首是另一个的队尾

- C++ STL 的 deque 实现比较好，底层用多段数组实现
- 也可以用 linkList 代替，只要使用 linkList 操作接口子集就可以
- 用 C++的 vector 或 Go 的 slice 也可以实现，但是队首出队后空间会被浪费

#### 算法：Monotone Queue(单调队列)

单调队列用 dq 实现，队列中的元素始终保持有序，队首始终保持最值。
队首元素按顺序出队、队尾元素可以在遍历过程中入队或出队来维护队列元素值有序。

示例：
["1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit" SlidingWindow+MonotoneQueue Golang]()

# Skill

### **剪枝**

在做某种可能性空间搜索时，如果某一状态以前已经遍历过，可以直接取得这个状态的结果，避免重复计算。
这就好像遍历树的时候，对一些已知肯定没有结果的支杈进行剪枝一样。
通常可以用 map 把状态结果保存，可以快速判断和返回。

### inplace(原地)思想

- 利用数组位置
  有时题目给定的数组元素只需要遍历一遍，这时可以利用原数组位置存储遍历结果以便后面使用。这样可以节省空间，对逻辑也没有干扰。

  - 示例：leetcode "448. Find All Numbers Disappeared in an Array"

- 利用 bit 位
  有时题目给定的数组元素有效位并不多，比如元素是 int32 型而表示的数范围是 1byte 内，那么实际上有将近 3byte 是可以利用的空间，可以存储一些中间结果。
  - 好处：
    1. 减少空间复杂度，直接利用原有输入参数的空间
    2. 不需要考虑映射关系，直接在原地操作即可
  - 坏处：
    1. 如果原数据和新数据边界不好划定，容易出错

### 常数内尝试

有时可以利用题目条件中某个参数数量有限，可以穷举所有该参数也不会影响时间复杂度，这时可用这种思路解决。

- 示例：穷举所有不同字母数量。
  ["395. Longest Substring with At Least K Repeating Characters" SlidingWindow Golang]()

# Little Tips

### 数组中间元素下标

设数组的长度为 n，如果 n 为奇数，则中间元素下标是`n/2`，经过整数向下取整，正好为中间下标；n 为偶数，则中间下标是`(n/2)-1`。

可以用统一公式`(n-1)/2`表示中间位置。

### 乘/除 2

用`x>>1`代替`x /= 2`可以节省 CPU 周期提高效率

### 判断奇偶性

用`x & 1 == 1`代替`x % 2 != 0`。因为位操作 CPU 效率远远大于取模。

# Golang Tips

- LeetCode 可以用`fmt`输出调试，无法用`log`输出
- Golang 中缺少一些常用的常量定义，可以自己用位操作实现
  - `const UINT_MIN uint = 0` // 所有位都是 0
  - `const UINT_MAX = ^uint(0)` // 所有位都是 1
  - `const INT_MAX = int(^uint(0) >> 1)` // 第一位是 0，其余都是 1
  - `const INT_MIN = ^INT_MAX` // 第一位是 1，其余都是 0
- 可以自己实现常用的 swap 函数

  ```Go
  func swap(a, b *[]int)  {
  	*a, *b = *b, *a
  }
  ```

- 自己实现 max/min 函数

  ```Go
  func max(a, b int) int {
    if a > b {  // 注意这里不用 >= 符号，没有必要
       return a
    }
    return b
  }
  ```

- 自己实现 set
  一般用 map[int]struct{} 可以实现 set，但有时为了快速判断是否存在节省代码，可以用 map[int]bool

# C++ Tips

## C++语言特性

- 有时可通过类内重命名，避免重复定义代码。"typedef std::unordered_map<std::string,std::string> stringmap;"
- C++11 新特性：对于空指针，使用 C++11 的 nullptr 更好。执行 delete nullptr 时，什么都不会做。所以 delete 的指针要及时设置 nullptr。
- 跨函数的变量，可以考虑用类的成员变量的方式传递，比传参更符合面向对象设计。
- 在对 map 元素操作时，可以直接对未知元素进行自增、自减运算，map[x]--，如果该空间不存在，自动创建时值默认为 0。
- 如果没有必要对统计计数，可以用 unordered_set 代替 unordered_map 以节省容量。
- 如果想实现一个较大距离的位操作，可以用 bool[n]实现。
- 使用 hash 函数时一定要搞懂具体函数原理，可能会发生排序与 hash 结果无关的情况，导致重复。
- 可以利用 map 的有序搜索二叉树原理，lower_bound 查找最小 key 值以及对应的 value。在构造 map 时使用 greater<T>就可以查找较大值。
- C++中的封装类 string，其实本质还是 char\*，s[s.length()] == '\0' 并不会越界。
- 数组作为参数传递时，第一维的长度是不重要的，可以空着，相当于传数组的引用。如：bool flags[][9][9]
- C++14 新特性，返回任何空容器"return {};"，不需要构造一个空容器再返回。在调用 push_back 等函数时，如果是 pair 等集合类型，可以直接用"{v1,v2}"代替构造函数传入 push_back。注意，必须在上下文完全确定类型的时候才能使用，不是任何地方都能使用。
- C++11 新关键字"decltype(value)"，宏函数以变量类型替换位置。
- C++11 新特性，在类的拷贝构造函数、赋值构造函数后加"= delete;"，"SegmentNode (const SegmentNode &) = delete;"表示禁止生成对应的默认构造函数，这里函数只要参数类型不要参数名称，因为不会有具体实现。对应的加上"= default;"表示生成默认的构造函数。以前是需要自己都创建一下表示考虑到了，现在有这两个语法也是相同的作用。
- "for (auto p : dict)"一般这里的 p 都是引用类型，可直接进行赋值等操作。
- 查找容器中是否存在时，使用 count(value)比 find 会精炼一些。
- C++中想跳出多重循环，有两种方法：1 使用 goto 语句"targetLine:\n goto targetLine;"。2 在外层循环添加判断条件变量，在内层用 break。
- C++中运算符的耗时情况'/' = '%' >> '+' = '-' = '\*' >> '>>' = '='。数量级对比：'/' ≈ 1000; '+' ≈ 50; '=' ≈ 1;
- double 溢出后不会直接抛出异常，而是保持最大值不再变化。
- 在进行 for 循环时，如果涉及到字符串长度与下标比较的情况，不要使用"i < str.size() - 2"这种形式，应该用"i + 2 < str.size()"，因为 size()函数返回的是 size_t 类型是 uint32 类型的，如果 size()返回的是 0，"0-2"就会被看做一个超大的 uint32 数，就可能造成无限循环。还可以在函数一开始就计算一个 int N = str.size()，这样后面使用"i < N - 2"就不会出问题了。
- vector::back()、stack::top()等函数，返回的是引用，可以直接修改值。
- map<pair<int,int>,int>正常编译，而 unordered_map<pair<int,int>,int>会无法编译，大概因为 unordered 无法支持复杂结构的 hash。
- queue、priority_queue、stack 没有赋值构造函数，只能逐步构造。大概是因为这些结构没有迭代器，无法进行批量元素拷贝。同样的，这类数据结构也没有 clear 函数，不能直接清空。

## C++基本语法

- new 数组：int\* pVisited = new int[num]; delete 数组：delete[] pVisited;
- 在 C++中“乘方”应该用函数 pow(x,y)实现(包含 math.h)，而'^'符号是异或运算。
- C++类中，类型声明要有顺序关系。所以，做题时，一般声明都要放在前面，定义要放在声明之后。一般顺序是：私有声明、私有定义、公有声明、公有定义。
- '.' 、'&'、'!=' 符号顺序要注意括号，避免逻辑错误。按位操作的优先级要低于比较操作符，例如"M[x][y]&0b10 == 0"，"0b10 == 0"会被优先运算。
- 子类调用父类同名函数或变量"ClassName::FunctionName()";
- λ 表达式(匿名函数)：返回类型自动判断，使用 auto 类型时，λ 函数不需要标明返回值类型。λ 表达式可以直接替代 Compare 对象。auto add= [](int a, int b){ return a + b; }; int (\*add)(int a, int b) = [](int a, int b)->int{ return a + b; };
- 二进制表示"0b101",十六进制"0xaf"，八进制"017"。
- double 型表示科学计数法，"2.3e14.3" => "2.314.3"
- C++中强制转换一般为"double(x)"，而 C 中一般为"(double)x"，尽量使用 C++的方式。
- 成员函数声明时右边加 const 表示该成员函数不会改变任何类内变量。

# 计算机基本原理

- ascii 的范围是 0~127，一共是 128 个字符。
- 对于可能 int32 溢出的情况，可以在运算时使用 long long 类型，避免溢出。
- 从 C 语言继承来，在常量数字后面可以加上类型符号，避免被自动转换。如：10LL、10UL、10u、0.3f。例如：1000000000000 已经超过了 int 范围，long a = 100000000000000;在有些编译器会出错，写成 100000000000LL 就不会有问题了。
