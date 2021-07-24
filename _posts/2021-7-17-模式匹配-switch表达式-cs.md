---
layout: post
title:  "C#中的模式匹配和switch表达式"
categories: 编程
tags: csharp cs基础
excerpt: 本文介绍下C#中的模式匹配和switch表达式
---

* content
{:toc}

## if

```
if(a is string str)

//等价于
var str = a as string;
if(str != null) //...
```

## switch表达式

### 语法

```cs
var output = input switch
{
    模式 1 => result 1,
    模式 2 => result 2,
    ...
    _ => result n
};
```

switch是**从上到下**顺序执行的，里面的关键符号含义见下：
- `=>`：类似于`return`，会**退出switch**
- `_`：类似于`default`，会接收任何值，任何值均会匹配上。

注意：
- 在switch表达式内部，会**省略**掉`输入参数`，直接写*判断*或*运算*即可。
- `switch`现在是**表达式**，不是语句块，所以最后大括号右边的**分号**不能少；
- 因为`switch`成了表达式，就不能用`case`子句了，所以直接用具体的内容来匹配；

### 基础-类似if

```cs
long nnn = 666;
//基础版本
SwitchPattern(nnn);
//高级版本
SwitchPattern2(nnn);
Console.ReadKey();

//基础版本
void SwitchPattern(object obj0)
{
    switch (obj0)
    {
        case string str1:
            Console.WriteLine($"string: {str1}");
            break;
        case int num1:
            Console.WriteLine($"int: {num1}");
            break;
        default:
            Console.WriteLine($"未知: {obj0}");
            break;
    }
}

//高级版本
void SwitchPattern2(object obj0)
{
    string result = obj0 switch
    {
        string str1 => $"string: {str1}",
        int num1 => $"int: {num1}",
        _ => $"未知: {obj0}"
    };
    Console.WriteLine(result);
}
```

### 高级-逻辑运算符

在 C# 9 中引入了逻辑运算符`and`/`or`/`not`使得模式匹配更为强大。

#### if转switch

来看一个判断是否是合法的 Base64 字符的一个方法的变化：

C# 9 之前的代码：
```cs
private static bool IsInvalid(char value)
{
    var intValue = (int)value;
    if (intValue >= 48 && intValue <= 57)
        return false;
    if (intValue >= 65 && intValue <= 90)
        return false;
    if (intValue >= 97 && intValue <= 122)
        return false;
    return intValue != 43 && intValue != 47;
}
```

使用 C# 9 **增强的模式匹配**之后的代码：
```cs
private static bool IsInvalid(char value)
{
    var intValue = (int)value;
    return intValue switch
    {
            >= 48 and <= 57 => false,
            >= 65 and <= 90 => false,
            >= 97 and <= 122 => false,
            _ => intValue != 43 && intValue != 47
    };
}
```

除了作为返回值，也可以赋值到变量上：
```cs
char value = '=';
var intValue = (int)value;
bool isBase64 = intValue switch
{
    >= 48 and <= 57 => false,
    >= 65 and <= 90 => false,
    >= 97 and <= 122 => false,
    _ => intValue != 43 && intValue != 47
};
```

### 进阶-Switch Expression

Switch Expression 是 C# 8 引入的新特性，C# 9 又结合模式匹配做了进一步的增强，使得其功能更加强大，来看示例吧：

修改前的代码是这样的：
```cs
var state = ReviewState.Rejected;
var stateString = string.Empty;
switch (state)
{
    case ReviewState.Rejected:
        stateString = "0";
        break;

    case ReviewState.Reviewed:
        stateString = "1";
        break;

    case ReviewState.UnReviewed:
        stateString = "-1";
        break;
}
```

使用 switch expression 之后的代码如下：
```cs
var state = ReviewState.Rejected;
var stateString = state switch
{
        ReviewState.Rejected => "0",
        ReviewState.Reviewed => "1",
        ReviewState.UnReviewed => "-1",
        _ => string.Empty
};
```

是不是看起来简洁了很多，还有进一步的增加优化，来看下一个示例：
```cs
(int code, string msg) result = (0, "");
var res = result switch
{
        (0, _) => "success",
        (-1, _) => "xx",
        (-2, "") => "yy",
        (_, _) => "error"
};
Console.WriteLine(res);
```

## 传入对象判断其属性

```cs
var output = obj switch
{
    { 属性1: 判断1 } => result 1,
    { 属性1: 判断1, 属性2: 判断2 } => result 2,
    ...
    _ => result n
};
```

**注意**：
1. **不为`null`** 的写法是`not null`，**不要**写成`!null`，因为很容易忽略掉`!`的！
2. 多个属性共同判断时，使用逗号(`,`)隔开，代表`判断1 && 判断2`，是**且**的关系

示例如下：
```cs
//常规写法：switch语句
switch (od)
{
    case { Qty: > 1000f }:
        Console.WriteLine("发财了，发财了");
        break;
    case { Qty: > 500f }:
        Console.WriteLine("好家伙，年度大订单");
        break;
    case { Qty: > 100f }:
        Console.WriteLine("订单量不错");
        break;
}

//高级写法：switch表达式
string message = od switch
{
    { Qty: > 1000f } => "发财了",
    { Qty: > 500f } => "年度大订单",
    { Qty: > 100f } => "订单量不错",
    _ => "未知"
};
Console.WriteLine(message);
```

### 写法分析

大括号`{}`是重点。linq里构造匿名类型，`new Class{}`构造指定对象，都用到了大括号`{}`。在这里`{}`就是构造某实例的成员值用的。

`case { Qty: > 1000f }:`其实是这样写的：`case Order{ Qty: > 1000f }:`。

`Order{ ... }`就是匹配一个`Order`对象实例，并且它的`Qty`属性要符合`...`条件。由于变量`od`始终就是`Order`类型，所以，`case`子句中的`Order`就省略了。

详细请参考[说说 C# 9 新特性的实际运用](https://www.cnblogs.com/tcjiaan/p/13947928.html)

## 参考

1. 使用现代化 C# 语法简化代码：<https://mp.weixin.qq.com/s?__biz=MzAwNTMxMzg1MA==&mid=2654083545&idx=3&sn=aa2d310c92ae2b6bc6524d84b97fdaaf>
2. 说说 C# 9 新特性的实际运用：<https://www.cnblogs.com/tcjiaan/p/13947928.html>
