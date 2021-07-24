---
layout: post
title:  "C#8中的Range和Index(范围和索引)"
categories: 编程
tags: csharp cs基础
excerpt: 本文介绍下C#8中的Range和Index(范围和索引)
---

* content
{:toc}

## Range-范围

### 范围运算符

- 范围运算符：`..`，它会生成一个`Range`对象。
- 语法：`{startIndex}..{endIndex}`：包含`startIndex`，**不包含**`endIndex`
- **限制**：0 <= `startIndex` <= `endIndex` <= `arr.Length`，其他均会报错！

    `startIndex`**等于**`endIndex`时，取不出元素，会生成一个**空数组**。

**注意**：
- 默认情况下，`..`的前面是`0`，后面是 **`arr.Length`(不含)**。`..`等价于`0..arr.Length`。
- 所以`..`的前面和后面**均可以为空**。
- 所以`..`也可以单独使用，代表整个索引范围(`0..arr.Length`)：`arr[..]`是`arr`的完整拷贝。

*深拷贝*还是*浅拷贝*需要看元素是`值类型`还是`引用类型`。`值类型`复制值，可以认为是*深拷贝*；`引用类型`复制引用，就是*浅拷贝*。

### Range

`Range`用来从集合中取出 **指定索引范围** 的元素来生成**新的集合**。 

创建`Range`：`Range range = 2..4;`

```cs
var arr = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
Range range = 2..4;
int[] slice = arr[range]; // 或 arr[2..4];
foreach (var number in slice)
{// [3, 4]
    Console.WriteLine(number);
}
```

`arr[2..4]`表示把`arr`这个序列，从索引为2的元素一直到索引为4（不含4）的元素提取出来组成新的序列。所以结果就是3，4。 

## Index-索引

`Index`用来指定**索引**。该索引可以用来从集合中取出**指定索引处**的元素，是**单个元素**。

乍一看，`Index`与`int类型的索引`没啥区别。嗯。。确实是这样，因为`Index`要与`^`操作符结合起来才能发挥更大的作用。对了，还有`Range`。。

### 末尾运算符

`^`是末尾运算符（Hat运算符），它会生成一个`Index`对象，用来从`末尾`开始**往前**取数据。

与*正向取数据*时索引从0开始不同，`^`取数据时是从**1**开始的，代表**倒数第一个元素**。
- `arr[^1]`等于`arr[arr.Length-1]`，最后一个元素
- `arr[^0]`等于`arr[arr.Length]`，常与`Range`组合使用

如果使用`arr[^0]`的话就会抛出`IndexOutOfRangeException`，`arr[^0]`和`arr[arr.Length]`是一个意思。 

这确实有点容易让人混淆，但其实其它语言也差不多是这样设计的，例如`-1`这个索引表示最后一个元素。 

## 组合使用 Range 和 Index 

### 完整拷贝数组的3种方式

```cs
int[] arr = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
int[] arr01 = arr[0..arr.Length];
int[] arr02 = arr[0..^0];
int[] arr03 = arr[..];
```

**注意**：
- Range的范围包含Start，**不包含**End。 
- 所以索引为0的元素包含，索引为10或者`^0`的元素**不包含**（尽管也不存在）。 

### 更多例子

```cs
int[] arr = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
int[] arr01 = arr[..]; //完整的复制，数组
int[] allButFirst = arr[1..]; //不包含第一个元素的数组
int[] empty = arr[^0..]; //空数组
int[] onlyLastItem = arr[^1..]; //只包含最后一个元素的数组
int[] last4Items = arr[^4..]; //只包含最后四个元素的数组
int lastItem = arr[^1]; //最后一个元素
```

## 单独使用Range或Index

```cs
int[] arr = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
Index middle = 4;
Index threeFromEnd = ^3;
Range range = middle..threeFromEnd;
int[] mySlice = arr[range]; //5, 6, 7
```

## 参考

1. C# 8 - Range 和 Index（范围和索引）：<https://mp.weixin.qq.com/s?__biz=MzAwNTMxMzg1MA==&mid=2654076610&idx=1&sn=8e04e2da923f44d04c642a0e0aabd7aa>，好文，强烈推荐！！
