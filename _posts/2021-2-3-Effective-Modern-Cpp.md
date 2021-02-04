---
title: Effective Modern C++ - notes
author: lambdaxing
date: 2021-2-3 18:00:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

&emsp;&emsp;《Effecive Modern C++》（中文版）阅读笔记的风格同《More Effective C++》。以我自己的情况进行记录，并将个别术语换成了我理解的偏大陆词汇，比如，将型别换为类型。不过在看习惯之后，我发现型别这个词也挺不错的。  

## Chapter 1. Deducing Types

### Item 1: Understand template type deduction

&emsp;&emsp;函数模板的类型推导中，编译器通过 `expr` 推导两个类型：`T` 和 `ParamType`，`ParamType` 常包含一些饰词，如 `const` 或引用符号等限定词。  

```c++
template<typename T>
void f(ParamType param);
// 调用
f(expr);
```

+ 当 `ParamType` 是指针（`T*` 、`const T*`）或引用（`T&` 、`const T&`），类型推导会忽略 `expr` 的引用（如果有的话），然后对 `expr` 的类型和 `ParamType` 执行模式匹配，得出 `T`。
+ 当 `ParamType` 是个万能引用（`T&&`），左值的 `expr` 推导出 `T` 和 `ParamType` 都为左值引用，`param` 的类型也是左值引用。右值的 `expr` 同前一种情况，执行模式匹配。  
+ `ParamType` 非指针和引用（`T` 、`const T`），就是所谓的按值传递。`param` 将是一个 `expr` 的副本，`expr` 的引用属性、顶层 const 属性和 volatile 属性都会被忽略，底层 const 属性会被保留进推导中。例如，若 `expr` 的类型是 `const char* const` （指向 `const char` 的 `const` 指针），`ParamType` 为 `T` 和 `const T` 都推导出的 `T` 为 `const char*` （指向 `const char` 的指针）。  

&emsp;&emsp;当数组和函数作为实参时，按值传递（`ParamType` 为 `T`），数组会推导 `T` 为指向数组内部元素的指针，函数会推导 `T` 为函数指针。按引用传递（`ParamType` 为 `T&`），数组会推导 `T` 为数组类型（包含数组大小），函数会推导 `T` 为函数类型。注意，后一种情况推导出的 `T` 并不是 `param` 的类型 `ParamType`，还要加上 `&` 。  

### Item 2: Understand auto type deduction

&emsp;&emsp;auto 类型推导就是模板类型推导。当某变量采用 `auto` 来声明时，`auto` 就扮演了模板中的 `T` 这个角色，而变量的类型修饰符则扮演的是 `ParamType` 的角色。从概念上来说，auto 类型推导相当于编译器为每个 `auto` 声明，生成了一个模板和一次使用对应初始化表达式对该模板的调用。所有的情况都一模一样，在 `auto i` 、`const auto i` 、`const auto& i` 中，`auto` 相当于 `T` ，`i` 前面的整个声明相当于 `ParamType`。  
&emsp;&emsp;不过，有一个例外。auto 类型推导会假定用大括号括起的初始化表达式代表一个 `std::initializer_list`，即在表达式 `auto x = {11, 23, 9};` 中，auto 类型推导得到 `x` 的类型是 `std::initializer_list<int>`。模板类型推导不会这么做，如果可以的话，编译器既要从初始值 `{11，23，9}` 中推导出 `std::initializer_list<T>` 中的 `T`为 `int` ，还要推导出模板中的 `T` 为 `std::initializer_list`，编译器才不会这么自作多情，也不该这么自作多情，只不过它对 `auto` 却是情有独钟。记住这条规则，也记住，两者的类似只是概念上的类似，而不是具体的实现手法。  
&emsp;&emsp;特别注意，C++14 允许使用 `auto` 说明函数的返回类型需要推导，同时允许在 lambda 的形参声明中用到 `auto` 来实现 lambda 参数类型的自动推导。这些 `auto` 用法使用**模板类型推导**，而不是 auto 类型推导。  
