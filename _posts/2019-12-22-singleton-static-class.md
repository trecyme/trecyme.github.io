---
layout: post
title: '单例继承问题: static class & singleton design model'
date: 2019-12-22
author: __yh__
color: rgb(255,210,128)
tags: C#, 设计模式
---

# 0. Background



>最近遇到一个问题，它具有以下约束时：
>
>1. A类和B类继承同一个类Base
>2. A/B在程序中都只能有一个实例
>
>A/B的实现能否不包含相同结构的单例代码？（即没有创建单例的那段冗余代码）



## 0.1 分析

背景的背景：

> 有个工具类SingletonClass\<T>，继承它就能快速让子类变成单例模式。好用，但严格地说，SingletonClass\<T>并不能让AB都变成单例，只是最大程度地减少了获取单例的代码冗余。AB的使用者调用new A()等构造函数，AB就不是单例了。AB的单例表象建立在开发者都遵循约定（只用A.Instance获取实例）的基础上，并且导致AB无法继承基类Base。



A/B都需要继承Base，那么A/B/Base都不能是静态类（特性见1），它们都只能为非静态类;

A/B继承Base，由于C#的单继承规则，所以SingletonClass<T>不能被A/B继承；

现有的单例模式中，只有登记式单例可以实现A,B对Base的继承，但运用了反射，受到争议。简单的非线程安全代码实现如下：

```C#

class Base
{
    private static Dictionary<Type, Base> _regDic = new Dictionary<Type, Base>();

    public static Base Instance => GetInstance<Base>();

    protected Base() { }

    public virtual void Print()
    {
        Console.WriteLine($"this is {GetType().Name}.Method1");
    }

    protected static T GetInstance<T>() where T:Base // 父类管理子类的单例
    {
        var typeofT = typeof(T);
        if (!_regDic.ContainsKey(typeofT))
        {
            var con = typeofT.GetConstructors(BindingFlags.NonPublic | BindingFlags.Instance)[0];
            var instance = con.Invoke(null) as T;
            _regDic[typeofT] = instance;
        }
        return _regDic[typeofT] as T;
    }
}

class A : Base
{
    public static new A Instance => GetInstance<A>(); // 返回A的单例

    private A() : base() { }
}

class B : Base
{
    public static new B Instance => GetInstance<B>(); // 返回B的单例

    private B() : base() { }
}
```

该方法能让AB继承Base的情况下，都实现为单例，且最大程度地减少返回单例的冗余代码段。

但是，

1. Base同时也会变成单例。

2. Base管理AB的实例时，用反射的方法实例化AB（为什么反射？因为构造函数不能为public），则这些子类的构造函数必须统一，只能有一个且参数数目类型一致。否则，父类管理子类单例时需要特殊处理。



## 0.2 结论

登记式单例勉强**算是**一种可行的实现。下面对前面提到的一些相关名词稍作解释。



# 1. 静态类
**静态类**是一种只包含静态成员和方法的类，在程序中静态类只初始化一次，生存周期为初始化完毕到程序退出。

它可以看作是一系列方法和变量的集合，有类将它们封装起来，只是为了成为一个整体。

一般只有面向对象的语言才会有静态类，因为其他过程化编程语言比如C，方法声明存在于一个文件（模块），include该文件，直接调用即可，并没有类的定义。而面向对象的语言，无法直接访问一个方法，必须通过类或者对象访问方法。

## 1.1 定义
```C#
public static class StaticClassDemo
{
		public static void StaticMethod1()
		{
		     // doing sth.
		}
		public static void StaticMethod2()
		{
		     // doing sth.
		}
}
```
## 1.2 使用
```C#
class MainClass
{
		static void Main()
		{
		     StaticClassDemo.StaticMethod1();
		     StaticClassDemo.StaticMethod2();
		}
}
```
如果目的是访问方法，不关心对象和数据，那么静态类是最好的选择。

## 1.3 不适用的原因

而静态类无法用于我的场景，为什么呢？

由它的特点决定的：

-  **静态类无法被继承**
   A/B必须继承Base，导致A/B/Base不能是静态类。

> 其他特点：
>
> - 只有静态field/property/method
> - 无法被new
> - 没有实例构造器
>
> 这些都不足以判断它不适合，静态成员也能为静态类代表的单个实例存储数据。最关键的一点还是无法被继承。



# 2. 单例
单例在程序中是一种需求：一个类在程序中只能有一个实例。由此衍生了单例的设计模式。

单例模式，有很多种写法【2】：

- 饿汉式（lazy loading）
- 懒汉式（static initialization）
- ……

单例模式算是设计模式中最简单和常用的，实际上也需要考虑到很多问题：

1. lazy loading
2. 线程安全
3. 继承

一旦扯到线程安全，问题就变得复杂：加锁，加锁后竞争的损耗……

即使枚举法【2】能简单/线程安全地实现单例，它却没考虑到单例类的继承。登记式单例【3】可以解决单例继承问题，但父类管理子类单例时也会产生一些问题如<u>0.1</u>提到的。



# 3. 参考文献

1. [java单例类继承问题--使用登记式单例的发现](https://blog.csdn.net/whlxjq520/article/details/5199767)
2. [Java 实现单例模式的 9 种方法](https://cloud.tencent.com/developer/article/1381907)
3. [Static Classes and Static Class Members (C# Programming Guide)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members)

