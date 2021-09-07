---
layout: post
title:  "C#中的Tuple"
categories: 编程
tags: csharp cs基础
excerpt: 本文介绍下C#中的Tuple(元组)和值元组
---

* content
{:toc}

## 前言

### 旧Tuple的问题

旧Tuple：System.Tuple

Tuple类有一些明显的问题:

- 您需要使用ItemX这样的形式来访问属性，这样的属性名可能对调用者来说没有什么含义，如果我们可以使用类似info.Name和info.Age这样的形式来访问会比info.Item1和info.Item2更好。
- 最多只能有8个属性。如果需要更多，最后一个类型参数必须是另一个元组。这使得语法非常难以理解。
- Tuple是一种引用类型，不像其他基本类型(它们是大多数值类型)，它分配在堆上，对于CPU密集型操作来说，它可能需要太多的对象创建/分配。

旧Tuple有很多问题，所以现在主要使用的是**Value Tuples(值元组)**。

### Value Tuples-值元组

Value Tuples：System.ValueTuple

本文中使用到的是Value Tuples(值元组)，它是Named Tuple，里面的属性可以看到**名字**。

- 值元组类型是值类型，没有继承或其他特性，这意味着值元组具有更好的性能。
- 值元组元素仅仅是**设计层面**的，使代码更具有可读性。它在编译时仍然会被编译为Item1、Item2格式。
- 由于值元组元素的名称**不是**运行时的，所以在使用一些类库(如Newtonsoft)进行序列化时，必须小心使用元组类型。除非你更新过支持新元数据(TupleElementNameAttribute等)的类库，否则你会遇到bug。

### 区别与混淆

`Value Tuples-值元组`与`匿名类型`很容易搞混。

**匿名类型**↓↓↓：
```cs
foreach (var item in Model.Select((value, i) => new { i, value }))
{
    var value = item.value;
    var index = item.i;
}
```
`new { i, value }`是创建一个新的匿名对象。

**Value Tuples-值元组**↓↓↓：
```cs
foreach (var item in Model.Select((value, i) => ( value, i )))
{
    var value = item.value;
    var index = item.i;
}
```
`( value, i )`是`Value Tuples-值元组`。用它可以**避免堆分配**。

参考自[如何获得foreach的当前迭代的索引？](https://stackoverflow.com/q/43021/9779472)

## 常用

### 方法返回值

```cs
(int id, string name) Test()
{
    int id = 1;
    string name = "bob";

    //方式一
    return (id, name);

    //方式二
    return (id: id, name: name);
}
```

### 接收返回值

```cs
//方式一：当做对象的字段来调用
var result = Test();
int id = result.id;
string name = result.name;

//方式二：直接解构-var
var (id, name) = Test();

//方式三：直接解构-显式声明类型
(int id, string name) = Test();

//多余数据可以使用 _ 丢弃掉
var (id, _) = Test();
(int id, _) = Test();
```

## 高级

### Deconstruct-解构对象+为多个变量赋值

与`Named Tuple`同时出现的，我们可以在类中声明一个`Deconstruct`用于解构该类。来看一个示例吧：

```cs
public class Point
{
    //为属性X，Y赋值
    public Point(double x, double y)
        => (X, Y) = (x, y);

    public double X { get; }
    public double Y { get; }

    //解构对象
    public void Deconstruct(out double x, out double y) =>
        (x, y) = (X, Y);
}

var p = new Point(3.14, 2.71);
//解构对象
(double x, double y) = p;

//P.X:3.14, P.Y:2.71
//x:3.14, y:2.71
```

**注意**：
1. 构造方法(`Constructor`)**不是**必须的。要想解构对象，只需要定义解构方法(`Deconstruct`)即可。
2. 解构方法(`Deconstruct`)是输出参数，参数必须是`out`方式。

## 从Value Tuples到Tuples

类型System.Tuple和System.ValueTuple提供了一些扩展方法来帮助它们之间相互转换。
```
var valueTuple = (id: 1, name: "Annie", age: 25, dob: DateTime.Parse("1/1/1993"));
var tuple = valueTuple.ToTuple();
```

## 参考

1. 使用现代化 C# 语法简化代码: <https://mp.weixin.qq.com/s?__biz=MzAwNTMxMzg1MA==&mid=2654083545&idx=3&sn=aa2d310c92ae2b6bc6524d84b97fdaaf>
2. [译]C# 7系列，Part 1: Value Tuples 值元组:<https://www.cnblogs.com/wenhx/p/csharp-7-series-part-1-value-tuples.html>
