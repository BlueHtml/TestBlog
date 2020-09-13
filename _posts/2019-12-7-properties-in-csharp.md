---
title:  "C#中的属性"
categories: 编程
tags: csharp
excerpt: 本文介绍了C#中的属性，以及C#6和C#7中与属性相关的新特性。
---

* content
{:toc}

## 前言

> C#属性是字段的扩展，它配合C#中的字段使用，用以构造一个安全的应用程序。
>
> 属性提供了灵活的机制来读取、编写或计算私有字段的值，可以像使用公共数据成员一样使用属性，但实际上它们是称做“访问器”的特殊方法，其设计目的主要是为了实现面向对象(Object Oriented, OO)中的封装思想。
>
> 根据该思想，字段最好设为private,  一个设计完善的类最好不要直接把字段声明为公有或受保护的，以阻止客户端直接进行访问，其中一个主要原因是，客户端直接对公有字段进行读写，使得我们无法对字段的访问进行灵活的控制，比如控制字段只读或者只写将很难实现。
>
> —— 姜晓东《C# 4.0权威指南》-【*9.4.5 属性*】

## 声明和使用读/写属性(旧)

这种方式是C#中**最基础**的，也是**最早出现**的读写属性的方式。本文中我暂时称它为***老式的读/写属性***。

该方式允许我们在对属性读/写时，进行一些**操作/计算**。

### 声明属性

首先要声明一个**私有字段**，然后使用`get`访问器**读取**私有字段的值，使用`set`访问器为私有字段**赋值**。

示例代码如下：

```cs
class Person
{
    private string _name = "N/A";
    private int _age = 0;

    // Declare a Name property of type string:
    public string Name
    {
        get
        {
            return _name;
        }
        set
        {
            _name = value;
        }
    }

    public int Age
    {
        get
        {
            return _age;
        }
        set
        {
            _age = value > 120 ? 120 : value;
        }
    }
}
```

### 使用属性

属性的使用很简单。外部可以直接对属性进行读取或赋值，就像使用类的公共字段一样。（**注意：这里假设属性都是公共读写的，实际使用中要注意属性的可访问性**）

```cs
//==========外部访问属性==========
Person person = new Person();
person.Name = "Bob";//为属性赋值
person.Age = 18;//为属性赋值

//读取属性的值
int age = person.Age;
```

### 备注

1. 在属性的`set`方法中，`value`变量是很特殊的， 它代表用户指定的值。

## 自动实现的属性(新)

当属性访问器中**不需要任何其他逻辑**时，我们可以使用自动实现的属性，它会使属性声明更加简洁。 

自动实现的属性在**编译后**，也是生成了**老式的读/写属性**。

VS中使用快捷键`prop`可以快速生成自动实现属性。
```cs
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

## 其他

### 自动实现的属性的本质

> 自动实现的属性在**编译后**，也是生成了**老式的读/写属性**。

这个是编译器自动帮我们做的，可以通过查看编译后生成的*IL*代码(又称作*MSIL*或*CIL*)来验证。

不过本人能力有限，就不分析了。这里推荐大家阅读 **《C# 4.0权威指南》** 中的【*9.4.5 属性*】一节，该章节详细分析了*自动实现的属性*经过编译后生成的IL代码。

### 属性初始化器

在 C# 6 和更高版本中，你可以像字段一样**初始化**自动实现属性：
```cs
public string FirstName { get; set; } = "Jane";  
```

上述代码经过编译后，是在构造函数中，为属性赋值的。([来源](https://www.cnblogs.com/mushroom/p/4666113.html#one))

### 只读属性默认初始化

在 C# 6 中，可以**去掉`set`访问器**，使属性变为只读属性。

```cs
public string Name { get; } = "hello world";
```

上述代码经过编译后，生成的属性关联字段是`readonly`的，并且仍然是在构造函数中为属性赋值的：`private readonly string kBackingField;`。([来源](https://www.cnblogs.com/mushroom/p/4666113.html#two))

> 这种方式下，生成的属性是没有 setter 的（即使用反射，也无法设置值，setter 根本就不存在）。这个属性只能**在构造函数中**，或者**结合特性**赋值。([来源](https://www.cnblogs.com/sword-successful/p/5532964.html#3440113))

### 表达式体属性

> 自 C# 6 起，支持方法、运算符和**只读属性**的表达式主体定义。
>
> 自 C# 7.0 起，支持构造函数、终结器、**属性**和索引器访问器的表达式主体定义。

在 C# 6 中，可以把**只读属性**改写为表达式体的形式。

在 C# 7.0 中，可以把**某个访问器**改写为表达式体的形式。

**只读属性的表达式体形式**和**属性(访问器)的表达式体形式**是不冲突的，因为它们的使用场景不一样（写法也不一样）。

因为都是表达式体形式，它们具有相同的**限制**：要求**方法体能够改写为lambda表达式**（必须是**单行代码**）。

#### 只读属性的表达式体形式(C#6)

**只读属性**的表达式体形式有2个**限制**：
- 只包含 **`get`访问器**
- 要求 **`get`访问器的方法体能够改写为lambda表达式**（必须是**单行代码**）

示例代码如下：

```cs
//C# 5
public string FullName
{
    get
    {
        return FirstName + "" + LastName;
    }
}

//C# 6
public string FullName => FirstName + "" + LastName;
```

我们可以通过VS的智能提示看到：该属性只有`get`访问器。

#### 属性访问器的表达式体形式(C#7)

在 C# 7.0 中，对于**老式的读/写属性**，我们可以把`get`访问器或`set`访问器改写为表达式体(lambda)。

注意：要求**访问器的方法体能够改写为lambda表达式**（必须是**单行代码**）

示例代码如下：

```cs
//C# 5
private int _id;
public int Id
{
    get
    {
        return _id;
    }
    set
    {
        _id = value;
    }
}

//C# 7.0
private int _id;
//全部改写为表达式体
public int Id { get => _id; set => _id = value; }
//只改写set访问器
public int Id { get { return _id; } set => _id = value; }
//只改写get访问器
public int Id { get => _id; set { _id = value; } }
```

## 备注

下面的备注，对于*老式的读/写属性*和*自动实现的属性*都是**通用**的。

1. 可以给访问器设置**可访问性**。例如把`set`访问器设为`private`，不允许外部直接赋值。**通常是限制`set`访问器的可访问性**。更多请参考：[限制访问器可访问性（C# 编程指南）](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/restricting-accessor-accessibility)
2. 可以省略掉某个访问器。通常是**省略掉`set`访问器可使属性为只读。**

另外，**属性的本质是方法，所以接口中可以包含属性**。

## 参考

1. 如何：声明和使用读/写属性（C# 编程指南）：<https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/how-to-declare-and-use-read-write-properties>
2. 自动实现的属性（C# 编程指南）：<https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties>
3. 限制访问器可访问性（C# 编程指南）：<https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/restricting-accessor-accessibility>
4. => 运算符（C# 参考）：<https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/lambda-operator>
5. 探索C#之6.0语法糖剖析：<https://www.cnblogs.com/mushroom/p/4666113.html>
6. 姜晓东《C# 4.0权威指南》-【*9.4.5 属性*】
