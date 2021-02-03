---
title: C++ - auto and decltype
author: lambdaxing
date: 2020-09-25 20:55:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

## `auto`

&emsp;&emsp; `auto` 让编译器通过初始值来推算变量的类型，`auto` 在一条语句中声明多个变量时，该语句中所有变量的初始基本数据类型都必须一样。编译器会适当地改变初始值的结果类型使其更符合初始化规则：

+ 引用被用作初始值时，编译器以引用对象的类型作为 `auto` 的类型。
+ `auto` 一般会忽略掉顶层 `const` ，保留底层 `const` 。若要推断出顶层 `const` 需明确指出。
+ 设置一个类型为 `auto` 的引用时，初始值中的顶层常量属性仍然保留。

```c++
    int i = 0;
    const int ci = i, &cr = ci;
    auto b = ci;                // b是一个整数（ci 的顶层 const 特性被忽略了)
    auto c = cr;                // c是一个整数（cr 是 ci 的别名，ci本身是一个顶层const）
    auto d = &i;                // d是一个整型指针
    auto e = &ci;               // e是一个指向整数常量的指针（对常量对象取地址是一种底层 const )
    const auto f = ci;          // ci推演出int，明确指出 f 是 const int
    auto &g = ci;               // g是一个整型常量引用，绑定到 ci，保留的。
    const auto &j = 42;         // 42 推演出int，明确指出 const int , j 是对const int 的引用
```

## `decltype`

&emsp;&emsp;`decltype` 选择并返回操作数的数据类型。操作数可以是变量、表达式，编译器分析表达式并得到它的类型，却不实际计算表达式的值。

+ `decltype` 返回表达式结果对应的类型。  
+ 当操作数是表达式并且该表达式的结果为左值时，`decltype` 返回引用。
+ `decltype` 返回变量的类型（包括顶层 `const` 和引用在内）， 引用从来都作为其所指对象的同义词出现，只有用在 `decltype` 处是一个例外。
+ 变量是一种作为赋值语句左值的特殊表达式，加括号的变量被当作表达式，所以加括号的变量就会得到引用类型，`decltype((variable))` 的结果永远是引用。
  
```c++
    int i = 5, *p = &i;
    decltype(*p) rp = i;    // 解引用运算符生成左值，rp 是 int&
    decltype(&p) pp;        // 取地址运算符生成右值，pp 是 int**
```

## Note

+ 当使用数组作为一个 `auto` 变量的初始值时，推断得到的类型是指针而非数组。当使用 `decltype` 关键字时，返回的类型是数组。
+ `decltype` 作用于某个函数时，它返回函数类型而非指针类型。

---

## References

《 C++ Primer 5 》
