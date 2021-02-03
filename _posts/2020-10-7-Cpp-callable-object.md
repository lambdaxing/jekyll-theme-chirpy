---
title: C++ - callable object
author: lambdaxing
date: 2020-10-7 10:38:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

## 简述

&emsp;&emsp;C++语言中有几种可调用对象：函数、函数指针、lambda 表达式、`bind` 创建的对象以及重载了函数调用运算符的类。  
&emsp;&emsp;可调用对象也有类型，例如：每个 lambda 有它自己唯一的（未命名）的类类型；函数及函数指针的类型则由其返回类型和实参类型决定，等等。  
&emsp;&emsp;两个不同类型的可调用对象却可能共享同一种调用形式（call signature）。调用形式指明了调用返回的类型以及传递给调用的实参类型。一种调用形式对应一个函数类型，例如：

```c++
    int(int, int);
```

## 标准库 function 类型

&emsp;&emsp;有时候，我们希望几个具有同一种调用形式的可调用对象能被看成具有相同的类型。例如，在实现函数表时，不同类型的可调用对象需要当做同一种类型处理（通常把它们存储在  `map` 中）。

&emsp;&emsp;使用一个名为 `function` 的标准库类型可以解决将不同类型的可调用对象做同一种类型处理的问题。`function` 是一个模板，需提供的模板信息为能够表示的对象的调用形式。它定义在头文件 `functional` 中，下面是它的操作：

operation|explanation
:-------:|:---------:
`function<T> f;`|`f` 是一个用来存储可调用对象的空 `function`,这些可调用对象的调用形式应该与函数类型 `T` 相同（即 `T` 是 `retType(args)`）。
`function<T> f(nullptr);`|显式地构造一个空 `function`。
`function<T> f(obj);`|在 `f` 中存储可调用对象 `obj` 的副本。
`f`|将 `f` 作为条件：当 `f` 含有一个可调用对象时为真；否则为假
`f(args)`|调用 `f` 中的对象，参数是 `args`。
定义为 `function<T>` 的成员类型|
`result_type`|该 `function` 类型的可调用对象返回的类型
`argument_type`|当 `T` 有一个或两个实参时定义的类型。如果 `T` 只有一个实参，则`argument_type` 是该类型的同义词；
`first_argument_type` 、`second_agument_type`|如果 `T` 有两个实参，则 `first_argument_type` 和 `second_argument_type` 分别代表两个实参的类型。

```c++
    function<int(int, int)>
```

&emsp;&emsp;这里声明了一个 `function` 类型，它可以表示接受两个 `int` 、返回一个 `int` 的可调用对象。  
&emsp;&emsp;注意：不能将重载函数的名字存入 `function` 类型的对象中，因为可能会产生二义性。解决途径是存储函数指针而非函数名字或使用 lambda 。

## 标准库定义的函数对象

&emsp;&emsp;标准库定义了一组表示算术运算符、关系运算符和逻辑运算符的类，每个类分别定义了一个执行命名操作的调用运算符。这些类都被定义成模板的形式，我们可以为其指定具体的应用类型，这里的类型即调用运算符的形参类型。它们都定义在头文件 `functional` 中：

算术|关系|逻辑
:-:|:-:|:-:
`plus<Type>` ( + )|`equal_to<Type>`|`logical_and<Type>`
`minus<Type>` ( - )|`not_equal_to<Type>`|`logical_or<Type>`
`multiplies<Type>` ( * )|`greater<Type>`|`logical_not<Type>`
`divides<Type>` ( / )|`greater_equal<Type>`|
`modulus<Type>` ( % )|`less<Type>`|
`negate<Type>` ( - )|`less_equal<Type>`|

&emsp;&emsp;表示运算符的函数对象类常用来替换算法中的默认运算符。例如，传递给 `sort` 函数一个 `greater` 类型的对象，`sort` 将执行待排序类型的大于运算，而不再是默认的 `<` 运算。  
&emsp;&emsp;特别地，标准库规定其函数对象对于指针同样适用。比较两个指针将产生未定义的行为，标准库函数对象却不会。关联容器使用 `less<key_type>` 对元素排序，因此我们可以定义一个指针的 `set` 或 `map` 而无需直接声明 `less`。

## 其他可调用对象

+ **[lambda and bind](https://lambdaxing.github.io/posts/Cpp-lambda-and-bind/)**

---

## References

《 C++ Primer 5 》
