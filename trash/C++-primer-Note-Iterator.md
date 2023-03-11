---
title: C++ primer Note Iterator
date: 2023-03-10
tags:
- C++
categories: Note
---

All of the library containers have iterators, but only a few of them support the subscript operator.

> As with pointers, an iterator may be valid or invalid.

> A valid iterator either denotes an element or **denotes a position one past the last element in a container**.

# Using Iterators

Types that have iterators have members that return iterators. 

```cpp
// the compiler determines the type of b and e; see § 2.5.2 (p. 68)
// b denotes the first element and e denotes one past the last element in v
auto b = v.begin(), e = v.end(); // b and e have the same type
```

> If the container is empty, the iterators returned by begin and end are equal —they are both off-the-end iterators.

Iterator Operations

![](./C++-primer-Note-Iterator/IteOp.png)







