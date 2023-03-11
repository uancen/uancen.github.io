---
title: HJ C++ OOP Note 1 Header
date: 2021-04-01
tags:
- C++
categories: Course
---

# C vs. C++

## C vs. C++

C面向过程而C++面向对象。

> C++面向对象的关键：类的引入；类的封装性、继承性、多态性简化了程序编写，提到了代码重用率。

C的目的：比汇编方便易用，同时不要损失汇编的表达能力；因此简单容易编译，灵活贴近底层。

C++的目的：提高编程人员的生产率，哪怕代价是增加编译器的复杂度；提高编程人员生产率的方法有如下几种：提高抽象层次、支持模块化编程、自动化代码生成。

> C++不仅仅是面向对象，其目的也在于支持泛型编程。

## Object Oriented

Object Based：面对的是单一的Class的设计；Object Oriented：面对的是Classes的设计，classes和classes之间的关系。

Classes的两个经典分类：Class without pointer members、Class with pointer member。

![](./HJ-C++-OOP-Note-1-Class-Without-Pointers/classes.png)

> String数据内部只有一个指针，采用动态分配内存，该指针就指向动态分配的内存。

## C++ programs

![](./HJ-C++-OOP-Note-1-Class-Without-Pointers/programs.png)

> 延伸文件名不一定是.h或.cpp，也可能是.hpp（头文件和主程序放在一个文件中实现）。

## Output

向左cout“丢”（<<）即可

# 头文件

## 防范式声明

防止此头文件被重复包含。

```
#ifdef _COMPLEX_
#define _COMPLEX_

#endif
```

## Header布局

![](./HJ-C++-OOP-Note-1-Class-Without-Pointers/headerlayout.png)

# 后续内容

## 模板

如果把私有数据的类型写死了，定义示例的时候，数据类型受到限制。所以需要写一个模板类（含模板的类）。T写成什么都可以。

```cpp
#pragma once
template <typename T>
class complex
{
public:
    complex(T r = 0, T i = 0) : re(r), im(i)
    {
    }
    complex &operator+=(const complex &);
    T real() const { return re; }
    T imag() const { return im; }

private:
    T re, im;

    friend complex &__doapl(complex *, const complex &);
};

{
    complex<double> c1(2.5, 1.5);
    complex<int> c2(2, 6)
}
```

> 使用时绑定类型。

## 内联函数

```cpp
class complex {
public:
    complex(double r = 0, double i = 0) : re(r), im(i) {}
    complex &operator+=(const complex &);
    complex &operator-=(const complex &);
    complex &operator*=(const complex &);
    complex &operator/=(const complex &);
    double real() const { return re; }
    double imag() const { return im; }

private:
    double re, im;

    friend complex &__doapl(complex *, const complex &);
    friend complex &__doami(complex *, const complex &);
    friend complex &__doaml(complex *, const complex &);
};
```

```cpp
inline double imag(const complex &x) { return x.imag(); }
```

**inline**；快（但是最后是否inline由编译器决定）；复杂时还是需要定义在外部。

函数在class body内完成，自动称为inline候选人。

## 访问级别

数据部分一般private；如果函数是要被外界调用的就放在public，若不打算被外界调用则放在private。即分为public给外界使用、private处理内部数据。

> 尽量使外界通过方法“拿”数据，而不是直接访问。
>
> ```cpp
> // bad
> cout << c1.re;
> cout << c1.im;
> // good
> cout << c1.real();
> cout << c1.imag();
> ```

