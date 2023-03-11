---
title: Effective C++ Note 1 Introduction
date: 2023-03-03
tags:
- C++
categories: Note
---

# Guidance

> Use C++ effectively.

> Make software comprehensible, maintainable, portable, extensible, efficient, and likely to behave as you expect.

- general design strategies;
- the nuts and bolts of specific language features.

> Only features in the official language standard.

**None of the Items is universally applicable.**

# Terminology

## declaration

A **declaration** tells compilers about the name and type of something, but it omits certain details.

- object declaration
- function declaration
- class declaration
- template declaration

> size_t, is just a typedef for some unsigned type that C++ uses when counting things.

## signature

Each function’s declaration reveals its signature, i.e., its parameter and return types.

## definition

A **definition** provides compilers with the details a declaration omits.

- object definition, set aside memory for the object;
- function or a function template definition, code body;
- template definition, list the members of the class or template.

```cpp
template<typename T>
class GraphNode {
public:
    GraphNode();
    ~GraphNode();
    ...
};
```

## initialization

**Initialization** is the process of giving an object its first value.For objects generated from structs and classes, initialization is performed by constructors.

Explicit, prevents constructors from being used to perform implicit type conversions.

> 只要没有特殊理由implicit类型转换，就explicit。

```cpp
class B {
public:
	explicit B(int x = 0, bool b = true);
};

void doSomething(B b);
// fine, uses the B constructor to explicitly convert the int to a B for this call.
doSomething(B(28));
// error! doSomething takes a B not an int,
// and there is no implicit conversion from int to B
doSomething(28);
```

> Constructors declared explicit are usually preferable to non-explicit ones, because they prevent compilers from performing unexpected (often unintended) type conversions.

## copy constructor & copy assignment operator

The **copy constructor** is used to initialize an object with a different object of the same type, and the **copy assignment operator** is used to copy the value from one object to another of the same type:

```cpp
Widget w1; // invoke default constructor
Widget w2(w1); // invoke copy constructor
Widget w3 = w2; // invoke copy constructor
w1 = w2; // invoke copy assignment operator
```

新对象，肯定是构造；没有新对象，是赋值。

```cpp
bool hasAcceptableQuality(Widget w);
...
Widget aWidget;
if (hasAcceptableQuality(aWidget)) ...
```

The parameter w is passed to hasAcceptableQuality by value, so in the call above, aWidget is copied into w. The copying is done by Widget’s copy constructor.

拷贝构造**定义参数值如何传递到新对象**，这个操作在拷贝构造中完成。Pass-by-value means “call the copy constructor.”，事实上，pass-by-value不是一个好选择，pass-by-reference-to-const会更好。

## STL

The **STL** is the Standard Template Library, the part of C++’s standard library.

> 六大组件详情见 HJ C++。

Much of that related functionality has to do with **function objects**: objects that act like functions. Such objects come from classes that overload operator(), the function call operator.

## undefined behavior

Programmers coming to C++ from languages like Java or C# may be surprised at the notion of **undefined behavior**.

It’s true: a program with undefined behavior could erase your hard drive. But it’s not probable. More likely is that the program will behave erratically, sometimes running normally, other times crashing, still other times producing incorrect results.

## interface

C++并没有接口的概念，但可以实现类似的效果；常常谈到的interface更类似于design idea。

## client

code user.

> 希望去make things easy for clients；在软件开发领域同理，对于函数、类、template、interface，都希望easy。