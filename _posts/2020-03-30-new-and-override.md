---
layout: post
title: '分清楚了算我输: new and override'
date: 2020-03-30
author: __yh__
color: rgb(255,210,96)
tags: C#
---
> 已经不知道第几次翻这篇笔记了……

# override

override针对父类中的virtual方法，当子类方法被指定为override，那么它的父类方法被覆盖。

覆盖可以这样理解：

```C#
var d = new Derived(); 
var b = new Derived() as Base; 
b.Hello(); 
d.Hello();
```

第3，4行调用的都是子类的Hello方法，无论是否向上转型，父类的Hello方法代码都不会被直接执行，看起来父类方法消失了，被子类方法完全覆盖(override)。所以被称为覆盖。

# new

new关键字，用于**成员隐藏**的作用。如果子类的Hello方法被它修饰，那么在3/4行，分别调用父类和子类的Hello函数代码。当对象的类型分别表现为子类和父类时具有不同的行为，从继承层级来看，子类的方法仅属于子类，所以看作new的方法。