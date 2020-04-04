---
layout: post
title: "ES6新特性"
date: 2020-04-04 22:00:00 +0800
tags: Javascript
---

![ES6](/assets/images/2020-04-04-ECMAScript_6_1.jpg)
ECMAScript 6 是 ECMA 于 2015.06 发布的版本，感觉变得"正规"了许多，记录一下变更点。

## let

- 对应以前的 var 关键字，可以定义一个局部变量，该变量只在自己的作用域(大括号内)起作用。
- 在 ES6 之前，我们都是用 var 来声明变量，而且 JS 只有函数作用域和全局作用域，没有块级作用域，所以{}限定不了 var 声明变量的访问范围

```ES6
{
  var i = 9;
}
console.log(i); // 9
```

```ES6
{
  let i = 9; // i变量只在 花括号内有效！！！
}
console.log(i); // Uncaught ReferenceError: i is not defined
```

- 与 var 相比，let 不存在"声明提升"处理，let 之前的区域叫"暂时性死区 temporal dead zone，简称 TDZ"，表示该变量完全不可用
- let 在一个块(大括号)内不允许重复声明

```ES6
var a = 99; // 全局变量a
f(); // f是函数，虽然定义在调用的后面，但是函数声明会提升到作用域的顶部。
console.log(a); // a=>99,  此时是全局变量的a
function f() {
  console.log(a); // 当前的a变量是下面变量a声明提升后，默认值undefined
  var a = 10;
  console.log(a); // a => 10
}
/* 输出结果：
undefined
10
99
*/
```

- 由于 let 只在块内起作用，很适合在 for 循环使用

```ES6
for (var i = 0; i < 10; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
} // 输出: 10 次 10

for (let i = 0; i < 10; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
} // 输出: 0 1 2 ... 9
```

## const

- const 是新增的关键字，与 C++中的 const 一样，表示常量
- 与 let 类似，在块内起作用
- const 用来声明一个常量，声明时必须赋值，且一旦声明就不能改变
- const 只要求被 const 修饰的变量本身是常量，如果这个变量本身是引用，则二级引用(成员变量)不受限制

## symbol

在以前 Javascript 6 个基础类型`object，string，boolean，number，null，undefined`基础上增加了 symbol，表示独一无二的值，与 NaN 有点类似。

```ES6
var sy = Symbol('test');
var sy1 = Symbol('test');
console.log(typeof sy);//'symbol'
sy == sy1;//false
var sy2 = new Symbol('test');//error : Symbol is not a constructor
```

## Destructuring Assignment(解构赋值)

这是一个很有意思的语法糖，可以根据模式匹配，自动将`=`前的多个元素赋值

```ES6
let [a,b,c] = [1,2,3]; // 1,2,3
**************************
let [a,b,c] = [1,,3]; // 1,undefined,3
**************************
let [a,,b] = [1,2,3]; // 1,3
**************************
//...是剩余运算符，表示赋值运算符右边除第一个值外剩余的都赋值给b
let [a,..b] = [1,2,3]; // 1,[2,3]
**************************
// 可以解构字符串为字符数组
let arr = 'hello';
let [a,b,c,d,e] = arr; // 'h','e','l','l','o'
**************************
// 可以解构对象，将属性映射赋值到变量
let obj = {name:'ren',age:12,sex:'male'};
let {name,age,sex} = obj;
console.log(name,age,sex); // 'ren' 12 'male'
let {name:myName,age:myAge,sex:mySex} = obj; // 自定义变量名
console.log(myName,myAge,mySex); // 'ren' 12 'male'
```

## Set Map

ES6 增加了两个基于 hash 的常用容器

## literal(字面量) 创建对象

可以直接用变量、函数拼接出一个对象

```ES6
let name = 'ren';
let age = 12;
let myself = {
    name,
    age,
    say(){
        console.log(this.name);
    }
};
console.log(myself); // {name:'ren',age:12,say:fn}
myself.say(); // 'ren'
```

## `...`拷贝属性运算符

- 提供了一个简易的拷贝对象属性的方法`...`
- 这个操作仍然只能算是"浅拷贝"，因为只有第一层属性拷贝了
- 可以利用`JSON.stringify() JSON.parse()`进行深拷贝

```ES6
let obj = {name:'ren',age:12};
let person = {...obj};
console.log(person); // {name:'ren',age:12}
obj == person; // false
let another = {sex:'male'};
let someone = {...person,...another}; // 合并对象
console.log(someone); // {name:'ren',age:12,sex:'male'}
```

## Object.assign

浅拷贝属性到目标对象

```ES6
let source = {a:{ b: 1},b: 2};
let target = {c: 3};
Object.assign(target, source);
console.log(target);//{c: 3, a: {b:1}, b: 2}
source.a.b = 2;
console.log(target.a.b);//2
// 由于数组对象的属性值是下标，所以相同下标内容会被覆盖
Object.assign([1,2,3],[11,22,33,44]);//[11,22,33,44]
```

## Object.is

与`===`符号类似，用于判断是否全等，但是结果有些不同

```ES6
Object.is(1,1); // true
Object.is(1,true); // false
Object.is([],[]); // false
Object.is(+0,-0); // false
Object.is(NaN,NaN); // true
```

## 参数默认值

- ES6 的参数默认值规则和 C++不同，不必要必须右边的参数先有默认值，因为参数允许传 undefined

```ES6
function add(a=1,b=2){
    return a + b;
}
add(); // 3
add(2); // 4
add(undefined, 4); // 5
```

## variadic parameter 可变参数

```ES6
function add(...num){
    return num.reduce(function(result,value){
        return result + value;
    });
}
add(1,2,3,4); // 10
```

## lambda expression 箭头函数

箭头函数可以替代匿名函数、闭包，以更优雅的语法实现相同的功能。

```ES6
let add = (a,b) => {
    return a+b;
}
let print = () => {
    console.log('hi');
}
let fn = a => a * a;
```

- 箭头函数可以直接捕获定义时环境中的变量
- 如果只有一个参数或没有参数可以省略`=>`左边的括号
- 如果直接返回结果，可以没有大括号，但是可读性很差，不建议采用
- 箭头函数与闭包最大的区别是`this`指向的对象不同，闭包的`this`指向后期闭包的调用者，而箭头函数的`this`与声明时的 this 指向一致，这样就不需要重命名`this`为参数了
- 箭头函数没有 arguments 属性，可以利用新增的可变参数语法实现相同功能

## class constructor static extends

通过一些类相关关键字实现面向对象，但 class 的本质依然是一个函数，只是增加了一些辅助关键字方便使用。

```ES6
class Person {//关键字声明方式
    constructor(name, age){
        this.name = name;
        this.age = age
        this.say = () => {  // 这里定义实例的私有成员
            console.log(this.name + ":" + this.age);
        }
    }
    methods(){  // 此方法会被添加到类的 prototype 属性上
        console.log('hello ' + this.name);
    }
    // 无法在类中直接定义类的属性需要在类外添加到 prototype
    // property = "test";
    static sp = 123;    // 定义类的静态属性
    static sm = () => { // 定义类的静态函数
        console.log(this.sp);
    };
}
//let ex = class{}  字面量方式
var person = new Person('ren', 10);
person.say(); // 'ren:10'
person.methods(); // 'hello ren'
Person.sm(); // 123

class Student extends Person{
    constructor (name,age,sex){
        super(name,age);
        this.sex = sex;
    }
}
var student = new Student('ren',12,'male');
student.name; // 'ren'
student.sex; // 'male'
student.say(); // 'ren:12'
```

- 类中各成员间无须`,`
- 用`super`可以引用父类的`this`，且必须放在子类`constructor`的`this`调用之前

## export import

通过导入导出关键字，可以指定文件向外导出的对象

```ES6
let a = 'a';
let b = 1;
let c = {name:'c'};
let d = 'd';
export {a, b};
export defalut c;
export {d}
************************************
import 'url'
************************************
import x from 'url'
************************************
import c,{a,b} from 'url'
************************************
import {a as a1,b as b1} from 'url'
```

- 只有 export 的对象才可以用 import 接收
- export default 在一个文件中只能出现一次，表示默认导出的一个对象，可以用`import x from 'xxx'`导入
- export 可以出现多次，分别导出不同的对象，导出的对象要用大括号包住
- import 时 url 可以是绝对路径，也可以是相对路径
- import 时可以不指定变量名，表示只是导入一次文件并不关心导出变量，通常是用于预处理或引入样式
- import 时的对象名要与导出的一致，也可以在导入同时重命名，可以只导入所需的对象
- import 默认导出对象时，可以用任意名字接收，因为这个名字不会造成歧义，无须区分名字
