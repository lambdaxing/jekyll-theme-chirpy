---
title: C++ - lambda and bind
author: lambdaxing
date: 2020-09-28 20:00:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

## lambda

&emsp;&emsp;lambda 是一种可调用对象，可对其使用调用运算符。一个 lambda 表达式表示一个可调用的代码单元，我们可以将其理解为一个未命名的内联函数。lambda 表达式具有如下形式：

```c++
    [capture list](parameter list) -> return type { function body }
```

&emsp;&emsp;可以忽略 `parameter list` 和 `return type` ，但必须包含 `capture list` 和 `function body` 。如果 lambda 的函数体只是一个 `return` 语句，则返回类型可从返回的表达式类型推断而来。如果函数体包含任何单一 `return` 语句之外的内容，且未指定返回类型，则返回 `void` 。  
&emsp;&emsp;lambda 不能有默认参数。若要使用 lambda 所在函数中定义的（非 `static` ）变量，可将局部变量包含在其 `capture list` 中来指出将会使用这些变量。一个 lambda 可以直接使用局部 `static` 变量和在它所在函数之外声明的名字。捕获分为值捕获和引用捕获，顾名思义，值捕获的前提是变量可以拷贝，且在 lambda 创建时拷贝。引用捕获需确保 lambda 创建时捕获的引用，在 lambda 执行的时候，被引用的对象是存在的。如果函数返回一个 lambda，由于函数不能返回一个局部变量的引用，此 lambda 不能包含引用捕获。当我们在捕获列表中写一个 & 或 = 时，& 告诉编译器采用捕获引用方式，= 则表示采用值捕获方式，此时局部变量被相应的方式隐式捕获。当我们混合使用显示捕获和隐式捕获，捕获列表中的第一个元素必须是一个 & 或 = ，显（隐）式捕获的变量必须使用与隐（显）式捕获不同的方式。如果我们要改变一个被捕获的变量的值，必须在参数列表尾加上关键字 `mutable` ，即：

```c++
    [capture list](parameter list) mutable -> return type { function body }
```

&emsp;&emsp;当定义一个 lambda 时，编译器生成一个与 lambda 对应的新的（未命名的）类类型。当向一个函数传递一个 lambda 时，同时定义了一个新类型和该类型的一个对象：传递的参数就是此编译器生成的类类型的未命名对象。当用 auto 定义一个用 lambda 初始化的变量时，定义了一个从 lambda 生成的类型的对象。默认情况下，从 lambda 生成的类都包含一个对应该 lambda 所捕获的变量的数据成员，在 lambda 对象创建时被初始化。

## 进一步说明 lambda

&emsp;&emsp;当我们编写了一个 lambda 后，编译器将该表达式翻译成一个未命名类的未命名对象。在 lambda 表达式产生的类中含有一个重载的函数调用运算符，它的形参列表和函数体与 lambda 表达式完全一样。  
&emsp;&emsp;当一个 lambda 表达式通过引用捕获变量是，将由程序负责确保 lambda 执行时引用所引的对象确实存在，编译器可以直接使用该引用而无需在 lambda 产生的类中将其存储为数据成员。相反，通过值捕获的变量被拷贝到 lambda 中，这种 lambda 产生的类必须为每个值捕获的变量建立对应的数据成员，同时创建构造函数，令其使用捕获的变量的值来初始化数据成员。  
&emsp;&emsp;默认情况下 lambda 不能改变它捕获的变量，因此默认情况下，由 lambda 产生的类当中的函数调用运算符是一个 `const` 成员函数。如果 lambd 被声明为可变（`mutable`）的，则调用运算符就不是 `const` 了。  
&emsp;&emsp;lambda 表达式产生的类不含默认构造函数、赋值运算符及默认析构函数；它是否含有默认的拷贝/移动构造函数则通常要视捕获的数据成员类型而定。使用带有参数列表的 lambda 时，必须提供相应实参才行。

## bind

&emsp;&emsp;标准库函数 `bind` 是一个通用的函数适配器，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。

```c++
    auto newCallable = bind(callable, arg_list);
```

&emsp;&emsp;`newCallable` 本身是一个可调用对象，`arg_list` 是一个逗号分隔的参数列表，对应给定的 `callable` 的参数，当我们调用 `newCallable` 时，`newCallable` 会调用 `callable`，并传递给它 `arg_list` 中的参数。`arg_list` 中从左到右的参数，对应 `callable` 的第1，2，...，n 个参数）  
&emsp;&emsp;`arg_list` 中的参数可能包含形如 `_n` 的“占位符”，表示 `newCallable` 的参数，它们占据了传递给 `newCallable` 的参数的“位置”。数值 `n` 表示生成的新的可调用对象中参数的位置：`_1` 为 `newCallable` 的第一个参数，`_2` 为第二个参数，以此类推。名字 `_n` 定义在命名空间 `std` 中的 `placeholders` 命名空间中：

```c++
    using namespace std::placeholders;
```

&emsp;&emsp;`bind` 拷贝其参数，当我们不能拷贝一个对象时（如 `ostream` ），希望传递给 `bind` 一个对象而又不拷贝它，就必须使用标准库 `ref` 函数。`ref` 返回一个对象，包含给定的引用，此对象可以是拷贝的。`cref` 函数生成一个保存 `const` 引用的类。`bind`、`ref` 和 `cref` 都定义在头文件 `functional` 中。

```c++
    auto newCallable = bind(callable, ref(std::cout), _2, _1, " ");
    // newCallable(_1,_2) 等价于 callable(ref(std::cout), _2, _1, " ")
```

---

## References

《 C++ Primer 5 》
