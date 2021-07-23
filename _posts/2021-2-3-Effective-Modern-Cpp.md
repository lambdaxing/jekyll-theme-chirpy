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

&emsp;&emsp;当数组和函数作为实参时，按值传递（`ParamType` 为 `T`），数组为实参会推导 `T` 为指向数组内部元素的指针，函数为实参会推导 `T` 为函数指针。按引用传递（`ParamType` 为 `T&`），数组为实参会推导 `T` 为数组类型（包含数组大小），函数为实参会推导 `T` 为函数类型。注意，后一种情况推导出的 `T` 并不是 `param` 的类型 `ParamType`，还要加上 `&`（即数组引用和函数引用） 。  

### Item 2: Understand auto type deduction

&emsp;&emsp;auto 类型推导就是模板类型推导。当某变量采用 `auto` 来声明时，`auto` 就扮演了模板中的 `T` 这个角色，而变量的类型修饰符则扮演的是 `ParamType` 的角色。从概念上来说，auto 类型推导相当于编译器**仿佛**对应于每个 `auto` 声明，生成了一个模板和一次使用对应的初始化表达式针对该模板的调用。所有的情况都一模一样，在 `auto i` 、`const auto i` 、`const auto& i` 中，`auto` 相当于 `T` ，`i` 前面的整个声明相当于 `ParamType`。  
&emsp;&emsp;不过，有一个例外。auto 类型推导会假定用大括号括起的初始化表达式代表一个 `std::initializer_list`，即在表达式 `auto x = {11, 23, 9};` 中，auto 类型推导得到 `x` 的类型是 `std::initializer_list<int>`。模板类型推导不会这么做，如果可以的话，编译器既要从初始值 `{11，23，9}` 中推导出 `std::initializer_list<T>` 中的 `T`为 `int` ，还要推导出模板中的 `T` 为 `std::initializer_list`，编译器才不会这么自作多情，但它却对 `auto` 情有独钟，在 `auto` 推导中先推导出 `std::initializer_list<T>` 中的 `T` ，然后再推导 `auto` 。为什么这样？作者说他也感到奇怪并且找不到一个有说服力的解释，但规则就是规则。记住这条规则。也记住，auto 推导和模板型别推导的类似只是概念上的类似，而不是具体的实现手法。  
&emsp;&emsp;特别注意，C++14 允许使用 `auto` 说明函数的返回类型需要推导，同时允许在 lambda 的形参声明中用到 `auto` 来实现 lambda 参数类型的自动推导。这些 `auto` 用法使用**模板类型推导**，而不是 auto 类型推导，要注意前面所说的两者的不同之处。  

### Item 3: Understand decltype

&emsp;&emsp;`decltype` —— declared type —— 得出名字的声明类型。即， `decltype` 会得出变量或表达式的类型而不作任何修改，注意与 `auto` 的不同，`decltype` 会保留引用和顶层 const 属性。不过，如果是比仅有名字更复杂的左值表达式，即一个左值表达式不仅是一个型别为 `T` 的名字，`decltype` 保证得出的类型总是左值引用。例如，一个 `T` 类型的变量 `x`（`x` 是左值），单纯的名字 `x` 得出 `T`，特殊的 `(x)` 也是左值，却会得出 `T&`。不过，绝大多数左值表达式都自带一个左值引用饰词。  
&emsp;&emsp;C++14 的 `decltype(auto)` 相当于遵循 `auto`的用法却以 `decltype` 的规则从初始化表达式推导出变量的类型。条款 2 提到 `auto` 使用模板类型推导规则，`decltype(auto)` 使用 `decltype` 的规则从初始化表达式中推导类型。  
&emsp;&emsp;[auto 和 decltype 在《 C++ Primer 》中的描述。](https://lambdaxing.github.io/posts/Cpp-auto-and-decltype/)下面这个例子来自书上：

```c++
tempalte<typename Container, typename Index>
decltype(auto)
authAndAccess(Container&& c, Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

&emsp;&emsp;`c` 的声明是个万能引用，`decltype(auto)` 确保了函数的返回类型与 `Container` 的 `[]` 返回类型保持一致。无论 `Container` 的 `[]` 返回引用或值，都没问题。

### Item 4: Know how to view deduced types

&emsp;&emsp;撰写代码阶段通过 IDE 编辑器（鼠标指针悬停等方式）可显示出某个程序实体的类型。这是因为，IDE 让 C++ 编译器（或至少也是其前端）在 IDE 内执行一轮。  
&emsp;&emsp;在编译阶段，使用想要显示的类型导致编译错误，该类型就会被报告错误的消息显示出。例如，以该类型具现一个仅有声明没有定义的类模板。  
&emsp;&emsp;运行时阶段，通过 `typeid` 和 `std::typeinfo::name` 显示类型信息会依编译器的不同产生不同的非人类可读的结果，并且标准规格上说，`std::type_info::name` 处理类型的方式就仿佛是向函数模板按值传递形参一样，因此，引用、const 等饰词将被忽略或移除。Boost 的 TypeIndex 库虽然可以胜任这份工作，但理解 C++ 型别推导规则也是必要的。  

## Chapter 2. auto

### Item 5: Prefer auto to explicit type declarations

&emsp;&emsp;`auto` 避免了潜在的未初始化风险，因为 `auto` 通过初始化物推导变量类型。`auto` 可用于声明 lambda 表达式，在 C++14 中，lambda 表达式的形参中也可以使用 `auto`。并且，使用 `auto` 声明的、存储着一个闭包的变量和该闭包是同一类型，从而它要求的内存量也和该闭包一样。而使用 `std::function` 声明的、存储着一个闭包的变量是 `std::function` 的一个实例，一般都会比 `auto` 声明的变量使用更多内存，且调用闭包的行为也来得慢。  
&emsp;&emsp;显式指定类型可能导致既不想要，也没想到的隐式类型转换。除非，你无时无刻都明确了解初始化物与声明中显式指定的类型相容或有正确的转换关系。使用 `auto`，无需担心声明变量的类型和它的初始化表达式的类型之间的不匹配等问题。  
&emsp;&emsp;`auto` 会带来源代码可读性问题吗？这依每个人的专业判断而不同。在其他语言中，`auto` 已不是什么新鲜事。软件开发社区已经积累了丰富的类型推导方面的经验，而这也说明此类技术并不会与撰写和维护大型的、工业强度的基础代码这样的工作产生冲突。在很多情况下，对于对象类型的抽象理解与了解它的精确类型同等有用，例如容器、计数器、智能指针，别忘了再取个好一点的变量名字。  
&emsp;&emsp;`auto` 随其初始化表达式的类型变化而自动随之改变，这意味着一些重构动作被顺手做掉了。  

### Item 6: Use the explicitly typed initializer idiom when auto deduces undesired types

&emsp;&emsp;标准库很多地方使用了代理类的设计：模拟或增广其他类型的类。代理类往往隐藏在背后，客户使用代理类如同使用代理类所代理的那个类型一般。“隐形”代理类和 `auto` 无法和平共处。代理类的对象往往会设计成仅仅维持到到单个语句之内，所以，无意中使用 `auto` 创建这种类的变量，往往就是违反了基本的库设计的假定前提。例如，在一个 `+` 与 `=` 的赋值语句中（`Matrix sum = m1 + m2 + m3;`），代理类在 `+` 操作中生产出来，在 `=` 操作中被隐式地当作代理的那个类型被赋值过去，语句结束代理类就被销毁。把 `Matrix` 换成 `auto` 得到的 `sum` 将是一个代理类对象，其内部可能并没有真正的值。  
&emsp;&emsp;如何发现问题（代理类）？

+ 使用代理类的库往往会在其文档中写明这一点。
+ 代理类大多数是由客户意欲调用的函数所返回的，函数签名往往会反应出它们的存在。例如，特化的 `std::vector<bool>` 的 `[]` 返回一个其内部定义 `std::vector<bool>::referenc` 模拟 `bool`。
+ 仔细观察你所使用的接口。

&emsp;&emsp;如何解决问题（用 or 不用 `auto`）？只要问题出在 `auto` 被决断成了代理类型，而非意欲代理的那个类型，解决方案都不必放弃 `auto`。`auto` 本身并不是问题，问题在于 `auto` 没有推导你想推导出的类型。解决方法应该是强制进行另一次类型转换 —— 带显式类型的初始化物习惯用法（the explicitly typed
initializer idiom）。即，`auto sum = static_cast<Matrix>(m1 + m2 + m2);`。这种习惯用法同样可以应用于想要强调意在创建一个类型有异于初始化表达式类型的变量的场合，让事情变得显而易见：

```c++
    // d 是一个 double
    int index = d * c.size();   // 强制转换的事实含含糊糊   
    auto index = static_cast<int>(d * c.size());    // 显而易见的做法
```
