---
title: C++ primer Note Compound Types
date: 2022-06-01
tags:
- C++
categories: Note
---

# References

> Compound Type，基于其他类型定义的类型。

> 一般的Reference值得是lvalue Reference，C++11中新增了Rvalue Reference，主要用于内置类。

引用即别名。一般初始化变量时，初始值会拷贝到新建的对象中；初始化引用时，是将引用和对象绑定（bind）在一起。

```cpp
int ival = 1024;
int &refVal = ival; // refVal refers to (is another name for) ival
int &refVal2; // error: a reference must be initialized
```

引用无法重定向，只能一直指向初始值。

对引用的所有操作都是在与之绑定的对象上操作。为引用赋值，实际上是把值赋给了和引用绑定的对象；获取引用的值，实际上是获取了与引用绑定的对象的值。

```cpp
refVal = 2; // assigns 2 to the object to which refVal refers, i.e., to ival
int ii = refVal; // same as ii = ival
```

```cpp
// ok: refVal3 is bound to the object to which refVal is bound, i.e., to ival
int &refVal3 = refVal;
// initializes i from the value in the object to which refVal is bound
int i = refVal; // ok: initializes i to the same value as ival
```

> 不能定义对引用的引用，因为引用本身不是一个对象。

着重看下引用的定义。允许一条语句中定义多个引用。

```cpp
int i = 1024, i2 = 2048; // i and i2 are both ints
int &r = i, r2 = i2; // r is a reference bound to i; r2 is an int
int i3 = 1024, &ri = i3; // i3 is an int; ri is a reference bound to i3
int &r3 = i3, &r4 = i2; // both r3 and r4 are references
```

**大部分情况下，引用的类型要求和与之绑定的对象严格匹配；引用只可以绑定在对象上，而不能与字面值或某个表达式绑定在一起。**

```cpp
int &refVal4 = 10; // error: initializer must be an object
double dval = 3.14;
int &refVal5 = dval; // error: initializer must be an int object
```

# Pointers

指向另一种对象的复合类型；与引用类似，指针也实现了对其他对象的间接访问。与引用的不同点：

- 指针本身是一个对象而引用不是，允许对指针赋值拷贝；
- 指针可以先后指向几个不同的对象；
- 指针不需要在定义时候初始化，但是引用必须初始化。

指针的定义。

```cpp
int *ip1, *ip2; // both ip1 and ip2 are pointers to int
double dp, *dp2; // dp2 is a pointer to double; dp is a double
```

取地址符&，获取地址。

```cpp
int ival = 42;
int *p = &ival; // p holds the address of ival; p is a pointer to ival
```

> 类似引用，大部分情况下，指针的类型要求和其指向的对象严格匹配。

```cpp
double dval;
double *pd = &dval; // ok: initializer is the address of a double
double *pd2 = pd; // ok: initializer is a pointer to double
int *pi = pd; // error: types of pi and pd differ
pi = &dval; // error: assigning the address of a double to a pointer to int
```

> 无效指针，即不清楚是否有效的指针；有效指针类型很广，比如知道是紧邻对象空间的下一个位置，知道是空指针，当然这些指针的使用也是受限的。

解引用符*，利用指针访问对象。

```cpp
int ival = 42;
int *p = &ival; // p holds the address of ival; p is a pointer to ival
cout << *p; // * yields the object to which p points; prints 42
```

> 注意某些符号的多重含义。
>
> ```cpp
> int i = 42;
> int &r = i; // & follows a type and is part of a declaration; r is a
> reference
> int *p; // * follows a type and is part of a declaration; p is a
> pointer
> p = &i; // & is used in an expression as the address-of operator
> *p = i; // * is used in an expression as the dereference operator
> int &r2 = *p; // & is part of the declaration; * is the dereference operator
> ```

> ```
> int *p; 
> int *&r = p;
> ```
>
> **从右向左读**，r是对指针p的引用。

空指针不指向任何对象。在试图使用一个指针之前，代码可以首先检查其是否为空。

```cpp
int *p1 = nullptr; // equivalent to int *p1 = 0;
int *p2 = 0; // directly initializes p2 from the literal constant 0
// must #include cstdlib
int *p3 = NULL; // equivalent to int *p3 = 0
```

> nullptr是一种特殊类型的字面值，可以被转换为任意其他的指针类型；不可以把int变量直接赋给指针，即使该int变量值为0。
>
> 建议初始化所有指针，使用未经初始化的指针式引发运行时错误的一大原因。如果实在不知道应该指向何处，用nullptr初始化。

指针可以重定向，其实只是被赋值了一个新地址。

```cpp
int i = 42;
int *pi = 0; // pi is initialized but addresses no object
int *pi2 = &i; // pi2 initialized to hold the address of i
int *pi3; // if pi3 is defined inside a block, pi3 is uninitialized
pi3 = pi2; // pi3 and pi2 address the same object, e.g., i
pi2 = 0; // pi2 now addresses no object
```

对于赋值语句是修改指针值，还是修改了指针指向对象的值，记住赋值永远改变的是等号左侧对象。

```cpp
pi = &ival; // value in pi is changed; pi now points to ival
```

```cpp
*pi = 0; // value in ival is changed; pi is unchanged
```

指针可以使用在条件表达式中。

```cpp
int ival = 1024;
int *pi = 0; // pi is a valid, null pointer
int *pi2 = &ival; // pi2 is a valid pointer that holds the address of ival
if (pi) // pi has value 0, so condition evaluates as false
 // ...
if (pi2) // pi2 points to ival, so it is not 0; the condition evaluates as true
 // ...
```

void* 指针和空指针不是一回事。void* 指针是特殊的指针类型，可以存放任意对象的地址。它的用处比较有限，因为不知道指向的对象类型，所以不能操作指针指向的对象。

```cpp
double obj = 3.14, *pd = &obj;
// ok: void* can hold the address value of any data pointer type
void *pv = &obj; // obj can be an object of any type
pv = pd; // pv can hold a pointer to any type
```

概括来说，以void* 的视角看内存空间也就仅仅是内存空间，没办法访问内存空间中所存的对象。

# Understanding Compound Type Declarations

> `int* p;`恰恰是不好的，容易误导。

统一规定代码风格，*和&紧贴变量。

```cpp
// i is an int; p is a pointer to int; r is a reference to int
int i = 1024, *p = &i, &r = i;
```

指向指针的指针。

```cpp
int ival = 1024;
int *pi = &ival; // pi points to an int
int **ppi = &pi; // ppi points to a pointer to an int
```

```cpp
cout << "The value of ival\n" 
     << "direct value: " << ival << "\n" 
     << "indirect value: " << *pi << "\n" 
     << "doubly indirect value: " << **ppi 
     << endl;
```

引用并不是一个对象，所以没有指向引用的指针；但是存在对指针的引用。

```cpp
int i = 42;
int *p; // p is a pointer to int
int *&r = p; // r is a reference to the pointer p
r = &i; // r refers to a pointer; assigning &i to r makes p point to i
*r = 0; // dereferencing r yields i, the object to which p points; changes i
to 0
```

