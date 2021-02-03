---
title: C++ - smart pointer
author: lambdaxing
date: 2020-09-28 20:55:00 +0800
categories: [Notes, Cpp]
tags: [Cpp]
---

&emsp;&emsp;静态内存用来保存局部 static 对象、类 static 数据成员以及定义在任何函数之外的变量。栈内存用来保存定义在函数内的非 static 对象。分配在静态内存或栈内存中的对象由编译器自动创建和销毁。除了静态内存和栈内存，每个程序还拥有一个内存池，这部分内存被称为**自由空间**（ free store ）或**堆**（ heap ）。程序用堆来存储**动态分配**（ dynamically allocate ）的对象。标准库提供了**智能指针**（ smart pointer ）类型来管理动态对象，负责自动释放所指向的对象，它们都定义在头文件 `memory` 中。

## `shared_ptr`

|operation|explanation（`shared_ptr`和`unique_ptr`都支持的操作）|
|:-----------------:|:----------------------------------------:|
|`shared_ptr<T> sp`| 空智能指针，可以指向类型为T的对象|
|`unique_ptr<T> up`| 同上|
|`p`|p作为条件，若p指向一个对象，则为true|
|`*P`|解引用p，获得它所指的对象|
|`p->mem`|等价于 `(*p).mem`|
|`p.get()`|返回p中保存的指针。要小心使用，若智能指针释放了其对象，返回的指针所指向的对象也就消失了。不能用返回的指针初始化另一个智能指针或为智能指针赋值，也不能delete此指针。|
|`swap(p,q)`|交换 `p` 和 `q` 的指针|
|`p.swap(q)`|同上|

|operation|explanation（`shared_ptr` 独有的操作）|
|:---------------------:|:-----------------------------------------:|
|`make_shared<T>(args)`|返回一个 `shared_ptr`,指向一个动态分配的类型为T的对象。使用 `args` 初始化此对象，`args` 为空进行值初始化。|
|`shared_ptr<T>p(q)`|`p` 是指向 `shared_ptr q` 的拷贝；此操作会递增 `q` 中的计数器。`q` 中的指针必须能转换为 `T*`。|
|`p = q`|`p` 和 `q` 都是 `shared_ptr`，所保存的指针必须能够相互转换。此操作会递减 `p` 的引用计数，递增 `q` 的引用计数；若 `p` 的引用计数变为0，则将其管理的原内存释放。|
|`p.unique()`|若 `p.use_count()` 为1，返回 `true` ;否则返回 `false` 。|
|`p.use_count()`|返回与 `p` 共享对象的智能指针数量；可能很慢，主要用于调试。|

&emsp;&emsp;每个 `shared_ptr` 都有一个关联的计数器，通常称其为**引用计数**（ reference count ）。无论何时我们拷贝一个 `shared_ptr`，计数器都会递增。一旦一个 `shared_ptr` 的计数器变为0, 它就会自动释放自己所管理的对象。`shared_ptr` 的析构函数会递减它所指向的对象的引用计数，如果引用计数变为0，`shared_ptr` 的析构函数就会销毁对象，并释放它占用的内存。  
&emsp;&emsp;我们可以用 `new` 返回的指针来初始化 `shared_ptr`，但是必须使用直接初始化的形式。`shared_ptr` 使用 `delete` 释放它所关联的对象，默认情况下，一个用来初始化智能指针的普通指针必须指向动态内存。我们可以提供自己的操作来替代 `delete`，使得智能指针绑定到一个指向其他类型的资源的指针上（可以是非动态内存资源）。

```c++
    shared_ptr<int> p1 = new int(1024);     // 错误：必须使用直接初始化形式
    shared_ptr<int> p2(new int(1024));      // 正确
```

|operation|explanation|
|:--------------------------:|:---------------------------------:|
|`shared_ptr<T> p(q)`|`p` 管理内置指针 `q` 所指向的对象；`q` 必须指向 `new` 分配的内存，且能够转换成 `T*` 类型。|
|`shared_ptr<T> p(u)`|`p` 从 `unique_ptr u` 那里接管了对象的所有权；将 `u` 置为空。|
|`shared_ptr<T> p(q, d)`|同上上，另：`p` 将使用可调用对象 `d` 来代替 `delete`。|
|`shared_ptr<T> p(p2, d)`|`p` 是 `shared_ptr p2` 的拷贝，`p` 将用可调用对象 `d` 来代替 `delete`。|
|`p.reset()`|`reset` 会更新引用计数，若 `p` 是唯一指向其对象的 `shared_ptr`，`reset` 会释放此对象，|
|`p.reset(q)`|若传递了可选的参数内置指针 `q`，会令 `p` 指向 `q` ,否则会将 `p` 置空，|
|`p.reset(q, d)`|若还传递了参数 `d` ，将会调用 `d` 而不是 `delete` 来释放 `q`。|

&emsp;&emsp; `shared_ptr` 可以协调对象的析构，仅限于其自身的拷贝之间。推荐使用 `make_shared` 而不是 `new` 来初始化 `shared_ptr`，避免将同一块内存绑定到多个独立创建的 `shared_ptr` 上，这些 `shared_ptr` 共享同一块内存，却不共享引用计数，无法协调对象的析构。特别是在函数参数的传递和返回时，应特别小心。当将一个 `shared_ptr` 绑定到一个普通指针时，我们就不应该再使用内置指针来访问 `shared_ptr` 所指向的内存了。使用智能指针可确保在异常发生后资源能被自动地正确释放，内置指针则无法做到。

### Note

+ 不使用相同的内置指针初始化（或 `reset` ) 多个智能指针。
+ 不 `delete`  `get()` 返回的指针。
+ 不适用 `get()` 初始化或 `reset` 另一个智能指针。
+ 如果你使用了 `get()` 返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效的。
+ 如果你使用智能指针管理的资源不是 `new` 分配的内存，记住传递给它一个删除器。

## `unique_ptr`

&emsp;&emsp;一个 `unique_ptr` “拥有” 它所指向的对象。某个时刻只能有一个 `unique_ptr` 指向一个给定对象。当 `unique_ptr` 被销毁时，它所指向的对象也被销毁。当我们定义一个 `unique_ptr` 时，需将其绑定到一个 `new` 返回的指针上，且必须采用直接初始化形式。`unique_ptr` 不支持普通的拷贝或赋值操作。

|operation|explanation|
|:-------------------------:|:------------------------------------------:|
|`unique_ptr<T> u1`|空 `unique_ptr`，可以指向类型为 `T` 的对象，`u1` 使用 `delete` 来释放它的指针。|
|`unique_ptr<T, D> u2`|同上，`u2` 使用一个类型为D的可调用对象来释放它的指针。|
|`unique_ptr<T,D> u(d)`|同上，用类型为 `D` 的对象 `d` 代替 `delete`。|
|`u = nullptr`|释放 `u` 指向的对象，将 `u` 置为空。|
|`u.realease()`|`u` 放弃对指针的控制权，返回指针，并将 `u` 置为空。|
|`u.reset()`|释放 `u` 指向的对象，|
|`u.reset(q)`|如果提供了内置指针 `q`，令 `u` 指向这个对象；否则将 `u` 置为空，|
|`u.reset(nullptr)`|同上。|

&emsp;&emsp;调用 `realease` 会切断 `unique_ptr` 和它原来管理的对象间的联系，`realease` 返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。如果我们不用智能指针来保存 `realease` 返回的指针，我们的程序就要负责资源的释放。如果我们不保存 `realease` 返回的指针，我们将会丢失掉指针，`realease` 也不会释放内存。

## `weak_ptr`

&emsp;&emsp;`weak_ptr` 是一种不控制所指向对象生存期的智能指针，它指向一个 `shared_ptr` 管理的对象。

|operation|explanation|
:--------------------:|:------------------------------------------:
`weak_ptr<T> w`|空 `weak_ptr` 可以指向类型为T的对象。
`weak_ptr<T> w(sp)`|与 `shared_ptr sp` 指向相同对象的 `weak_ptr`。`T` 必须能够转换为 `sp` 指向的类型。
`w = p`|`p` 可以是一个 `shared_ptr` 或 一个 `weak_ptr`。
`w.reset()`|将 `w` 置为空。
`w.use_count()`|与 `w` 共享对象的 `shared_ptr` 的数量。
`w.expired()`|若 `w.use_count()` 为 0，返回 `true`,否则返回 `false`。
`w.lock()`|如果 `w.expire()` 为 `true`，返回一个空 `shared_ptr`;否则返回一个指向 `w` 的对象的 `shared_ptr`

&emsp;&emsp;创建一个 `weak_ptr` 时，要用一个 `shared_ptr` 来初始化它。`weak_ptr`像是一种伴随指针，陪伴着 `shared_ptr`，不会改变 `shared_ptr` 的引用计数。由于对象可能不存在，我们不能直接使用 `weak_ptr` 访问对象，而必须调用 `lock`。

---

## References

《 C++ Primer 5 》
