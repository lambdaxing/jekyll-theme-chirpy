---
title: C++ - const
author: lambdaxing
date: 2020-09-24 20:55:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

&emsp;&emsp;`const` 是 C++ 非常重要的一个特性，虽然平时使用得比较多，但是重新回去看书才发现一些概念性的东西很容易遗忘了，特整理记录一下。

## 1. `const`

&emsp;&emsp;有时候需要这样一种变量，它的值不能被改变，使用它的地方不少，当我们要对其进行调整时很容易修改，但也要随时警惕防止程序修改这个值。为满足这一要求，可用关键字 `const` 对变量的类型加以限定:

```c++
    const int bufSize = 512;        // 输入缓冲区大小
```

&emsp;&emsp;编译器将在编译过程中把用到变量`bufSize`的地方都替换成对应的值 `512` 。`const` 对象的值一经创建就不能再改变，所以必须初始化，同时只能在 `const` 类型的对象上执行不改变其内容的操作，常量特性仅在执行改变内容的操作时发挥作用。

```c++
    const int k;        // error
```

&emsp;&emsp;默认情况下，`const` 对象被设定为仅在文件内有效。当多个文件出现同名的 `const` 变量时，其实等同于在不同文件中分别定义了独立的变量。若要只在一个文件中定义     `const` , 而在其他多个文件中声明并使用它，则需对 `const` 变量不管是声明还是定义都添加 `extern` 关键字，这样只需定义一次就可以了:

```c++
    // file_1.cc 定义并初始化了一个常量，该常量能被其他文件访问
    extern const int bufSize = fcn();
    // file_1.h 头文件
    extern const int bufSize;       // 与 file_1.cc 中定义的 bufSize 是同一个
```

## 2. 指向`const`的指针和引用

&emsp;&emsp;将引用绑定到 `const` 对象上，称之为对常量的引用（reference to const)，习惯性叫做常量引用。对常量的引用不能被用作修改它所绑定的对象，允许为一个常量引用绑定非常量的对象、字面值，甚至是一般表达式：

```c++
    int i = 42;
    const int &r1 = i;
    const int &r2 = 42;
    const int &r3 = r1 * 2;
    int &r4 = r1 * 2;           // 错误: r4 是一个普通的非常量引用
```

&emsp;&emsp;常量引用仅对引用可参与的操作做出了限定，对于引用的对象本身是不是一个常量未作限定。因为对象也可能是个非常量，所以允许通过其他途径改变它的值。  
&emsp;&emsp;指向常量的指针（pointer to const） 不能用于改变其所指对象的值，要想存放常量对象的地址，只能使用指向常量的指针。允许令一个指向常量的指针指向一个非常量对象，同常量引用一样，这个非常量对象的值可通过其他途径改变。

## 3. 顶层/底层 `const`

&emsp;&emsp;允许把指针本身定为常量，常量指针必须初始化，且一旦初始化完成，则它的值（也就是存放在指针中的那个地址）就不能再改变了。把`*`放在`const`关键字之前用以说明指针是一个常量，不变的是指针本身的值而非指向的那个值:

```c++
    int errNumb = 0;
    int *const curErr = &errNumb;       // curErr 将一直指向 errNumb
    const double pi = 3.14159;
    const double *const pip = &pi;      // pip 是一个指向常量对象的常量指针
    // 要想弄清楚这些声明的含义，从右向左阅读。
```

&emsp;&emsp;指针本身是一个常量并不意味着不能通过指针修改的其所指的值，能否这样做完全依赖于所指对象的类型。  
&emsp;&emsp;指针本身是不是常量以及指针所指的是不是一个常量就是两个相互独立的问题。用名词**顶层`const`(top-level const)** 表示指针本身是个常量，而用名词**底层 `const` (low-level const)** 表示指针所指对象是一个常量。  
&emsp;&emsp; 更一般的，顶层`const` 可以表示任意的对象是常量，这一点对任何数据类型都适用，如算术类型、类、指针等。底层 `const` 则与指针和引用等复合类型的**基本类型部分**有关。指针比较特殊，两种皆可以是。  
&emsp;&emsp;当执行对象拷贝操作时，顶层`const` （拷出）不受影响（顶层`const`除了初始化外，不再存在拷入操作）。拷入和拷出的对象必须具有相同的底层`const`，或者说两个对象的数据类型必须能够转换，非常量可以转换成常量，反之不行。

## 4. constexpr

&emsp;&emsp;**常量表达式（ const expression ）** 是指值不会改变并且在编译过程就能得到计算结果的表达式。**字面值**属于常量表达式，用常量表达式初始化的`const`对象也是常量表达式。允许将变量声明为 `constexpr` 类型以便由编译器来验证变量的值是否是一个常量表达式。声明为`constexpr`的变量一定是一个常量，且必须用常量表达式初始化，允许用 `constexpr` 函数去初始化 `constexpr` 变量：

```c++
    constexpr int mf = 20;
    constexpr int limit = mf + 1;
    constexpr int sz = size();      // 只有当 size 是一个 constexpr 函数时才是一条正确的声明语句
```

&emsp;&emsp;一个 `constexpr` 指针的初始值必须是 `nullptr` 或者 `0` ,或者是存储于某个固定地址中的对象。注意，函数体内定义的变量**一般来说**并非存放在固定地址中，定义于所有函数体之外的对象其地址固定不变。限定符 `constexpr` 仅对指针有效，与指针所指的对象无关， `constexpr` 把它所定义的对象置为了顶层 `const` ,与其他常量指针类似，`constexpr` 指针既可以指向常量也可以指向一个非常量：

```c++
    const int *p = nullptr;     // p 是一个指向整型常量的指针
    constexpr int *q = nullptr; // q 是一个指向整数的常量指针
```

```c++
    int j = 0;
    constexpr int i = 42;
    // i 和 j 都必须定义在函数体之外
    constexpr const int *p = &i;    // p 是常量指针，指向整型常量
    constexpr int *p1 = &j;         // p1 是常量指针，指向整数 j
```

---

## References

《 C++ Primer 5 》
