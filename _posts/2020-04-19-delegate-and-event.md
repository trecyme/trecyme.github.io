---
layout: post
title: '傻傻分不清的委托和事件: delegate and event'
date: 2020-04-19
author: __yh__
color: rgb(255,210,32)
tags: C#
---
> 之前一直分不清delegate和event，event简直多此一举，最近用的多了发现两者根本不是一个层级的概念啊T_T

# delegate
委托delegate将某种函数签名定义为一个具有命名的类型。
```
public delegate void MyDelegate(int x);
public delegate void MyDelegate2(int x);
```
MyDelegate为返回值void和具有一个int形参的函数类型，声明类似于C/C++中的函数指针，但形式更简单灵活且功能强大：
1. 将函数签名定义为一种委托类型，那么它可以被实例化多次，可以作形参/返回值/域变量，使用范围更广泛
2. 相比于函数指针理解更容易
3. 一种函数签名并非只能对应一个委托类型，比如MyDelegate和MyDelegate2。不同的场景下，名字不同则意义不同

在C#中，委托可以看作是一种特殊的类型，与class/struct/enum处于一个层级。
既能在一个类中定义，作为内部类型，访问时名称为ClassName.DelegateName；也能定义在全局，名为DelegateName。

声明了这种函数类型后，还可以定义任意多个函数变量。

```
MyDelegate completeCallback;
completeCallback = RefreshUI;
completeCallback += RfreshData;  // +=/-=为委托的多播，这样执行一个委托可以同时调用多个函数
completeCallback += Clear;            
completeCallback -= Clear;
completeCallback.Invoke(1);
completeCallback(2);
```

以上为委托的一般用法，+=/-=为委托多播，事件event略微不同。

# event
简单来说，event是受限的委托变量，两者不是同一层级的概念。

```
delegate void HelloDelegate();
event HelloDelegate helloEvent;
helloEvent += SayHello;
helloEvent -= SayHello;
helloEvent?.Invoke();
```
看这段代码可以发现，它立足于特定的委托上。

在语法层面，如果与class对象概念类比，delegate相当于class关键字，HelloDelegate相当于一种类，event则是HelloDelegate对象的修饰符。

除了在定义事件的类中，其他类中只能有+=/-=，没有=。

其他类中，事件不能被随意覆盖，相比于多播的委托更安全：+=/-=决定了一个函数是否被加入到这个委托，两种操作无法影响其他的函数。

而=一旦执行，覆盖原有所有的函数，则影响之前被委托的多个函数。这种更大的权限意味着风险，尤其是多人协作项目中，一个不小心写下的=，可能就会给系统埋下隐患。

**为什么只有定义事件的类中可以用=呢？不与安全性相悖吗？**个人理解，这个规则相当于给定义事件的类开了管理员权限，对于自己的事件有一定的掌控，当某些条件下可以覆盖。如果没有这条规则，该类无法控制事件，实际使用中也会带来困扰。

event变量可以被static修饰，但同时也只能被委托static函数。delegate不能定义static的函数，上面说过delegate和class一个层级，class有无static修饰表现出不同的用途，而delegate如果能被static修饰，区分有无static反而削弱了它的功能，我猜测可能是这样的原因，C#并没有设计让delegate被static修饰。

> NOTE:
1. 委托变量-=不能多于+=，否则会空指针
2. +=重复应用于一个函数，该函数会执行多次