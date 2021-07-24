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

## Chapter 3. Moving to Modern C++

### Item 7: Distinguish between () and {} when creating objects

&emsp;&emsp;C++11 引入了统一初始化：单一的、至少从概念上可以用于一切场合、表达一切意思的初始化 —— 大括号初始化（braced initialization）。  
&emsp;&emsp;大括号初始化的新特性：

+ 直接指定一个 STL 容器在创建时持有一个特性集合的值。
+ 为非静态成员指定默认初始化值可以使用大括号和 “=” 的初始化语法，却不能使用小括号。
+ 不可复制对象（如 `std::atomic` 型别的对象），可以采用大括号和小括号来进行初始化，却不能使用 “=”。注：前三条揭露出大括号初始化的 “统一” 之名。
+ 禁止内建型别之间进行隐式窄化型别转换（narrowing conversion）。
+ C++ 规定：任何能够解析为声明的都要解析为声明，而这会带来副作用，一个令人苦恼的解析语法，程序员本想以默认方式构造一个对象，结果却一不小心声明了一个函数。例如，这个语句 `Widget w2();` 声明了一个名为 `w2` 、返回一个 `Widget` 型别对象的函数！由于函数声明不能使用大括号来指定形参列表，所以使用括号来完成对象的默认初始化没有上面这个问题：`Widget w3();` 。

&emsp;&emsp;大括号初始化的缺陷，源于大括号初始化物、`std::initializer_list` 以及构造函数重载决议之间的纠结关系：

+ Item 2 曾说过：使用大括号初始化物来初始化一个使用使用 `auto` 声明的变量，推导出来的型别会是 `std::initializer_list`。
+ 如果有一个或多个构造函数声明了任何一个具备 std::initializer_list 型别的形参，采用大括号初始化语法的调用语句会强烈地优先选用 std::initializer_list 型别形参的重载版本，哪怕是复制或移动的构造函数也是如此。换句话说，大括号初始化语法优先选用带有 std::initializer_list 型别形参的构造函数（即使其他重载版本有着貌似更加匹配的形参表），只有在找不到任何办法把大括号初始化物中的实参转换成 std::initializer_list 模板中的型别时，编译器才会退而去检查其他普通的重载决议。
+ 空大括号对表示的是 “没有实参”，而非 “空的 std::initializer_list”。即，一对空大括号初始化对象执行的是默认构造，而非以一个不含任何元素的 std::initializer_list 为基础执行构造。

&emsp;&emsp;作为一个类的作者，需要有清醒的意识，了解自己撰写的一组重载构造函数中只要有一个或多个声明了任何一个具备 std::initializer_list 型别的形参，则使用可大括号初始化的客户代码可能会只发现那些具备 std::initializer_list 型别形参的重载版本。最好把构造函数设计成客户无论使用小括号还是大括号都不会影响调用的重载版本，因此，std::vector 的接口设计被视为败笔（作者在书中如此说道）。  
&emsp;&emsp;往一组重载函数中添加带有 std::initializer_list 型别形参的新版本构造函数时，可能使别的重载版本连露脸的机会都没有。在添加这样的重载版本时，一定要做到心中完全有数。  
&emsp;&emsp;作为开发类客户代码的程序员，创建对象时选用一对小括号还是大括号可要三思而后行。默认选用大括号的程序员是被其宽泛的应用语境、对隐式窄化型别转换的禁止，以及对最令人苦恼之解析语法的免疫所吸引。小括号的拥护者则可以免受 `auto` 型别推导意外错误之苦，在创建对象时也不会碰到带有 std::initializer_list 型别形参的构造函数设置的路障，但有些场合非用大括号不可。究竟选用哪一方更好，作者的建议是选用任一方并坚持下去。  
&emsp;&emsp;作为开发模板的程序员，在模板内部进行对象创建时，到底应该使用小括号还是大括号会成为一个棘手问题。这是因为，在模板内部创建对象时并不知道对象的型别（对象的型别由使用者在具现化模板时指定的嘛），自然也就不知道该型别的大括号和小括号表达些什么。使用大括号还是小括号，模板的作者不可能下这个判断，只有调用者才有决定权。标准库函数 `std::make_unique` 和 `std::make_shared` 就面临这样的问题，解决办法是在内部使用了小括号，并把这个决定以文档的形式广而告之，作为其接口的组成部分。书中注释中说，更弹性的设计，也就是允许调用者自行决定在从模板中生成的函数内使用小括号还是大括号的设计，是可以实现的。参阅[这里](https://isocpp.org/blog/2013/06/intuitive-interface-part-1-andrzej-krzemieski)。  

### Item8: Prefer nullptr to 0 and NULL

&emsp;&emsp;`0` 和 `NULL` 都不具备指针型别。字面常量 `0` 的型别是 `int`，而标准允许各个实现给予 `NULL` 非 `int` 的整形型别（如 `long`）。`nullptr` 不具备整形型别，也不具备指针型别，但可以把它想成一种任意型别的指针。`nullptr` 的实际型别是 `std::nullptr_t`，并且在一个漂亮的循环定义下，`std::nullptr_t` 的定义被指定为 `nullptr` 的型别。型别 `std::nullptr_t` 可以隐式转换到所有的裸指针型别，这就是为何 `nullptr` 可以扮演所有型别指针的原因。  
&emsp;&emsp;模板型别推导会将 `0` 和 `NULL` 推导成“错误”型别（即他们的真实型别，而给退而求其次的表示空指针这个意义），这个事实构成了应该在表示空指针时使用 `nullptr` 而非 `0` 或 `NULL` 的压倒性理由。同时，`nullptr` 不会造成 `0` 和 `NULL` 稍不留意就会遭遇的重载决议问题。不过，还是不要在指针型别和整形之间做重载，有些程序员还是会继续使用 `0` 和 `NULL`。  

### Item9: Prefer alias declarations to typedefs

 &emsp;&emsp;除了 `typedef` ，C++11 提供了别名声明（alias declaration）：

 ```c++
 using UptrMapSS = 
    std::unique_ptr<std::unordered_map<std::string, std::string>>;
 ```

&emsp;&emsp;别名声明除了在处理涉及函数指针的型别时，比较容易理解之外，还可以模板化（这种情况下它们被称为别名模板，alias template）。`typedef` 就不支持，程序员不得不用嵌套在模板化的 `struct` 里的 `typedef` 才能硬搞出这种机制。

```c++
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;

template<typename T>
struct MyAllocList {
    typedef std::list<T, MyAlloc<T>> type;
};

MyAllocList<Widget>::type lw;
```

&emsp;&emsp;比如，C++11 中的型别特征就是用嵌套在模板化的 `struct` 里的 `typedef` 实现的。型别特征是在头文件 `<type_traits>` 给出的一整套模板，对给定待变换型别T，其结果型别为 `std::transformation<T>::type` ，这些变换的应用都以 “::type” 结尾，将它们应用于模板内的型别形参时，每次都要在前面加上 `typename` 。到了 C++14，C++11 中的所有型别变换都加上了对应的别名模板：

```c++
    std::remove_const<T>::type          // C++11: const T -> T
    std::remove_const_t<T>              // C++14 中的等价物

    std::remove_reference<T>::type      // C++11: T&/T&& -> T
    std::remove_reference_t<T>          // C++14 中的等价物

    std::add_lvalue_reference<T>::type  // C++11: T -> T&
    std::add_lvalue_reference_t<T>      // C++14 中的等价物

// C++ 14 标准
template <class T>
using remove_const_t = typename remove_const<T>::type;
template <class T>
using remove_reference_t = typename remove_reference<T>::type;
template<class T>
using add_lvalue_reference_t =
    typename add_lvalue_reference<T>::type;
```

### Item 10: Prefer scoped enums to unscoped enums

&emsp;&emsp;不限范围的（unscoped）枚举型别：C++98 风格的枚举型别中定义的枚举量的名字会泄露到枚举型别所在的作用域。它们在 C++11 中的对等物，限定作用域的（scoped）枚举型别则不会以泄露名字，限定作用域的枚举型别通过 `enum class` 声明，因此也被称为枚举类。  

```c++
    enum Color {black, white, red}; // black,white,red 所在作用域
                                    // 和 Color 相同
    auto white = false;             // 错误！white 已在范围内被声明过了
```

```c++
    enum class Color {black, white, red};   // black,white,red 所在作用域
                                            // 被限定在 Color 内
    auto white = false;         // 没问题，范围内并无其他 “white”
    Color c = white;            // 错误！范围内并无名为“white”的枚举量
    auto c = Color::white;      // 没问题
```

&emsp;&emsp;限定作用域的枚举型别的枚举量是更强型别的（strongly typed），从限定作用域的枚举型别到任何其他型别都不存在隐式转换路径（强型别转换能够实施），而不限范围的枚举型别中的枚举量可以隐式转换到整数型别（并能够从此处进一步转换到浮点型别）。  
&emsp;&emsp;限定作用域的枚举型别可以进行前置声明，即其型别名字可以比其中的枚举量先声明。限定作用域的枚举型别的底层型别是已知的（默认是 `int`），而对于不限范围的枚举型别，为了使编译器在枚举型别使用前确认其底层型别选择哪一种，C++98 就只提供了枚举型别定义（即列出所有枚举量）的支持，枚举型别声明则不允许。对于不限范围和限定作用域的枚举型别，都可以指定其底层型别，这样做了后，不限范围的枚举型别也能够进行前置声明了：

```c++
    enum Color: std::uint8_t;   // 不限范围的枚举型别的前置声明
                                // 底层型别是 std::unit8_t 
```

&emsp;&emsp;当需要引用 C++11 中的 `std::tuple` 型别的各个域时，不限范围的枚举型别还是有用的。关于如何以这两种枚举型别引用 `std::tuple` 型别的各个域的细节，详见书上。  

### Item 11: Prefer deleted functions to private undefined ones

&emsp;&emsp;优先选用删除函数，而非 private 未定义函数。后者无法应用于类外部的函数，也不总是能够应用于类内部的函数（能应用也可能直到链接阶段才发挥作用）。而任何函数都可以删除，包括非成员函数（为函数调用中想要滤掉的型别创建删除重载版本）和模板具现（删除函数可以阻止那些不应该进行的模板具现）。  
&emsp;&emsp;习惯上，删除函数会被声明为 public，而非 private，这是因为当客户代码尝试使用某个成员函数时，C++ 会优先检验可访问性，后校验删除状态。当客户代码试图调用某个 private 删除函数时，有些编译器只会抱怨该函数为 private，尽管函数的可访问性并不影响其是否可用。  

### Item 12: Declare overrding functions override

&emsp;&emsp;“改写”（override）和“重载”（overload）读起来很像，却是两个毫不相关的概念。由于对于声明派生类中的改写，保证正确性很重要，而出错又很容易，C++11 提供了一种方法来显式地标明派生类中的函数是为了改写基类版本：为其加上 `override` 声明。无论何时，只要你在派生类中声明了一个函数，并且该函数意在改写基类中的一个虚函数，请确保你给该函数加上 `override` 声明。  

### Item 13: Prefer const_iterators to iterators

&emsp;&emsp;`const_iterator` 是 STL 中相当于指涉到 const 的指针的等价物，它们指涉到不可被修改的值，任何时候只要需要一个迭代器而其至涉到的内容没有修改必要，就应该使用 `const_iterator`。并不存在 `const_iterator` 到 `iterator` 的型别转换。  
&emsp;&emsp;C++11 仅添加了非成员函数版本的 `begin` 和 `end`。下面是非成员函数版本的 `cbegin` 的一个实现：

```c++
template <class C>
auto cbegin(const C& container)->decltype(std::begin(container))
{
    return std::begin(container);
}
```

&emsp;&emsp;这个 `cbegin` 模板接受一个形参 `C` ，实参型别可以是任何表示类似容器的数据结构，并通过其引用到 const 型别的形参 `container` 来访问该实参。调用非成员函数版本的 `begin` 函数并传入一个 const 容器会产生一个 `const_iterator`，而模板返回的正是这个迭代器。这样一来，这个非成员函数版本的 `cbegin` 也可用在那些仅支持 `begin` 的容器上了。该模板对内建数组同样适用，因为 C++11 的非成员函数版本的 begin 为数组提供了一个特化版本，而模板对数组的型别推导在 Item 1 中已有讨论。  
&emsp;&emsp;C++14 纠正了 C++11 的短视，添加了 `cbegin`、`cend`、`rbegin`、`rend`、`crbegin`、`crend`。  
&emsp;&emsp;注意，在最通用的代码中，优先选用非成员函数版本的 `begin`、`end`、`rbegin`等，而非其成员函数版本。因为，最通用化的代码会使用非成员函数，而不假定其成员函数版本的存在性。  

### Item 14: Declare functions noexcept if they won't emit exceptions

&emsp;&emsp;在 C++11 形成过程中，逐渐达成了一个共识，一个函数或者可能发生异常，或它保证自己不会。`noexcept` 就是为了不会发射异常的函数准备的，并且 `noexcept` 声明是函数接口的组成部分，事关接口设计，是客户方面关注的核心，调用方可能会对它有依赖。  
&emsp;&emsp;对不会发射异常的函数应用 `noexcept` 声明可以让编译器生成更好的目标代码。在带有 `noexcept` 声明的函数中，优化器可能不需要在异常传出函数的前提下，将执行期栈保持在可开解状态（在 C++98 异常规格下，调用栈会开解至函数的调用方），也不需要在异常逸出函数的前提下，保证所有其中的对象以其被构造顺序的逆序完成析构。  
&emsp;&emsp;noexcept 性质对于移动操作至关重要。在容器中，只有在保证移动操作不会产生异常时，才会使用移动，否则会使用复制。而一个函数（移动构造函数等）怎么能知道移动操作不会产生异常呢？答案是，通过校验看看容器内元素型别的移动操作是否带有 `noexcept` 声明。  
&emsp;&emsp;`swap` 函数是许多 STL 算法实现的核心组件。标准库中的 `swap` 是否带有 `noexcept` 声明，取决于用户定义的 `swap` 是否带有 `noexcept` 声明。高阶数据结构的 swap 行为要 noexcept 性质，一般地，仅当构建它的低阶数据结构具备 noexcept 性质时才成立。例如标准库为数组和 `std::pair` 准备的 `swap` 函数使用带条件式 `noexcept` 声明，它们到底是不是具备 noexcept 性质，取决于它的 `noexcept` 分句中的表达式是否结果为 `noexcept`，即取决于数组和 `std::pair` 的元素型别的 `swap` 行为是否为 noexcept。  
&emsp;&emsp;优化诚可贵，正确价更高。noexcept 乃是函数接口的组成部分，所以应该只在函数实现长期具有 noexcept 性质的前提下，才给予其 `noexcept` 声明。  
&emsp;&emsp;在 C++11 中，默认地，内存释放函数（`operator delete` 或 `operator delete[]`）和所有的析构函数（无论是用户定义的，还是编译器自动生成的）都隐式地具备 noexcept 性质。析构函数未隐式地具备 noexcept 性质的唯一场合，就是所在类中有数据成员（包括继承而来的成员，以及在其他数据成员中包含的数据成员）的型别显式地将其析构函数声明为可能发射异常的（即为其加上 “`noexcept(false)`” 声明）。  
&emsp;&emsp;大多数函数都是异常中立的，不具备 noexcept 性质。由于有着确实的理由使得带有 `noexcept` 声明的函数依赖于缺乏 noexcept 保证的代码，C++ 允许此类代码通过编译，并且编译器通常不会就此生成警告。  

### Item 15: Use constexpr whenever possible

&emsp;&emsp;constexpr 对象都具备 const 属性，并由编译期已知的值完成初始化。  
&emsp;&emsp;constexpr 函数在调用时若传入的是编译期常量，则产出编译期常量，若传入的是直至运行期才知晓的值，则产出运行期值。  

+ constexpr 函数用在要求编译期常量的语境（例如使用 constexpr 对象保存函数的返回结果）中时，传给一个 constexpr 函数的实参值是在编译期已知的，则结果也会在编译期计算出来。如果任何一个实参值在编译期为未知，则代码无法通过编译。
+ 在不要求编译期常量的语境中调用 constexpr 函数时，传入的值有一个或多个在编译期未知，则它的运作方式和普通函数无异，即它也是在运行期执行结果的计算。这意味着，执行同样操作的函数，仅仅应用语境一个是要求编译期常量的，一个是用于所有其他值的话，那就不必写两个函数。constexpr 函数就可以同时满足所有需求。

&emsp;&emsp;constexpr 函数仅限于传入和返回字面型别（literal type），意思就是这样的型别（在 C++11 中，除了 `void` 的所有内建型别均符合）能够持有编译期可以决议的值。用户自定义的型别同样可以是字面型别，因为它的构造函数和其他成员函数可能也是 constexpr 函数。C++14 进一步放宽了 `constexpr` 的诸多使用限制。  
&emsp;&emsp;只要有可能使用 `constexpr`，就使用它。比起非 constexpr 对象或 constexpr 函数而言，constexpr 对象或是 constexpr 函数可以用在一个作用域更广的语境中。也要注意，“只要有可能使用 `constexpr`，就使用它” 这句话中的 “只要有可能” 的含义就是你是否有一个长期的承诺，将由 `constexpr` 带来的种种限制施加于相关的函数和对象之上。  

### Item 16: Make const member functions thread safe

&emsp;&emsp;const 成员函数意味着它所代表的是一个读操作，多个线程在没有同步的条件下执行读操作是安全的，至少会被认为是安全的。因此，保证 const 成员函数的线程安全性，避免发生数据竞险（data race），除非可以确信它们不会用在并发语境中。  
&emsp;&emsp;运用 `std::atomic` 型别的变量会比运用互斥量提供更好的性能，但前者只适用对单个变量或内存区域的操作。  

### Item 17: Understand special memeber function generation

&emsp;&emsp;C++11 中，支配 special member function 的机制如下：

+ Default constructor：仅当类中不包含用户声明的构造函数时才生成。
+ Destructor：析构函数默认为 `noexcept`，仅当基类的析构函数为虚的，生成的派生类的析构函数才是虚的。
+ Copy-constructor：按成员进行非静态数据成员的复制构造。仅当类中不包含用户声明的复制构造函数时才生成。如果该类生成了移动操作，则复制构造函数将被删除。在已经存在复制赋值运算符或析构函数的条件下，仍然生成复制构造函数已经成为了被废弃的行为。
+ Copy-assignment operator：按成员进行非静态数据成员的复制赋值。仅当类中不包含用户声明的复制赋值运算符时才生成。如果该类生成了移动操作，则复制赋值运算符将被删除（中文原文这儿写的是复制构造函数将被删除）。在已经存在复制构造函数或析构函数的条件下，仍然生成复制赋值运算符已经成为了被废弃的行为。
+ Move-constructor/Move-assignment operator：都按成员进行非静态数据成员的移动操作。仅当类中不包含用户声明的复制操作、移动操作和析构函数时才生产。按成员移动由两部分组成，一部分是在支持移动操作的成员上执行的移动操作，另一部分是在不支持移动操作的成员上执行的复制操作。

&emsp;&emsp;C++11 可以通过 `=default`  显式地为这些操作生成编译器的默认行为。  
&emsp;&emsp;成员函数模板在任何情况下都不会抑制特种成员函数的生成。哪怕，成员函数模板具现出的是这些操作中的某一个，也是如此。  

&emsp;&emsp;