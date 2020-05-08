---
layout: post
title: '类型关键字_引用和值类型：class/struct/enum'
date: 2019-12-22
author: __yh__
color: rgb(255,210,160)
tags: C#, 设计模式
---



C# Docs中是这么描述class和struct的：

> Each is essentially a data structure that encapsulates a set of data and behaviors that belong together as a logical unit.



相比于C中的.c文件所代表的（相对来说是）物理单元，C#中的class和struct毫无疑问是逻辑单元，这个逻辑单元中定义了一系列和其名称相关的field/property/method/event，来表现它们的数据和行为。编程其实本质上是数据的修改，虽然它表现为复杂的多层函数调用。

和Java不同，C#除了基本数据类型，还有struct专门用于值类型的定义。



### class 

**引用型**，class类型对象拷贝时，拷贝的是对同一对象的引用。多个引用指向同一块内存中的对象。

相比于struct，

1. 可以为static class
2. 可以继承和被继承
3. 可以为abstract class

class更适合于更多数据/更抽象/与其他数据结构有更强联系/更复杂行为的数据结构的类型。



### struct

**值类型**，适用于封装较少的数据。一般被设计为struct的数据结构，它的行为（方法）都比较少，其所有字段都被视为一个整体。struct有以下特点

1. 不可继承
2. 实现接口后，cast为该接口，会封箱装箱
3. 它分配在栈上（不知道struct类型的field是不是）
4. 构造时必须所有字段都被实例化，但不能在声明字段时就指定默认值
5. ……

复制struct时，实际产生了原实例的一个副本，返回了新实例。
新实例拷贝了原有对象的所有数据，某个field的类型为引用类型，只复制引用，所以新实例和原实例的该field都指向同一个对象。
如果在复制一个数据结构时，只希望它的副本是快照形式，而非实时表现它最新状态，那么它最好是struct。

对于struct来说，改变副本的值，并不能改变被复制的实例的值，**这个错误却容易犯。**



### enum

**值类型**，它的数据只有那些枚举常量，不包含任何方法。这一点和java中不同，java的enum可以定义自己的行为。



# 参考文献

1. [System.ValueType的派生](https://docs.microsoft.com/en-us/dotnet/api/system.valuetype?view=netcore-3.1)
2. [Struct结构设计](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/struct)
3. [枚举类型](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum)