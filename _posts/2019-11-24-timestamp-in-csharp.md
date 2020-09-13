---
layout: post
title:  "C#中的时间戳"
categories: 编程
tags: csharp
author: 过千帆
excerpt: 本文介绍了C#中时间和时间戳的转换方法，以及DateTimeOffset的简单使用。
---

* content
{:toc}

## 什么是时间戳

> 时间戳默认是**Unix时间戳**。

首先要清楚JavaScript与Unix的时间戳的区别：

JavaScript时间戳：是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的**总毫秒数**。

Unix时间戳：是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的**总秒数**。

可以看出JavaScript时间戳是**总毫秒数**，Unix时间戳是**总秒数**。

比如同样是的 2016/11/03 12:30:00 ，转换为JavaScript时间戳为 1478147400000；转换为Unix时间戳为 1478147400。

从上面也可以看出**时间戳与时区无关**。

## Unix时间戳相互转换

### C# DateTime转换为Unix时间戳

#### .NET 4.6新方法

只能在 .NET 4.6及更高版本里才能使用。

```cs
long timeStamp = DateTimeOffset.Now.ToUnixTimeSeconds(); // 相差秒数
Console.WriteLine(timeStamp);
```

#### 通用的老方法

```cs
System.DateTime startTime = TimeZone.CurrentTimeZone.ToLocalTime(new System.DateTime(1970, 1, 1)); // 当地时区
long timeStamp = (long)(DateTime.Now - startTime).TotalSeconds; // 相差秒数
System.Console.WriteLine(timeStamp);
```

### Unix时间戳转换为C# DateTime

#### .NET 4.6新方法

由时间戳转换的`DateTimeOffset`的时区默认是`+00:00`，此时我们需要**转为本地时区**，否则后续使用可能会有问题。

转为本地时区：`DateTimeOffset.LocalDateTime`。

示例代码如下：

```cs
//默认为UTC时间：{2019/11/14 1:53:26 +00:00}
DateTimeOffset dto = DateTimeOffset.FromUnixTimeMilliseconds(1573696406184);
//默认为UTC时间：{2019/11/14 1:53:26}
DateTime dt01 = dto.DateTime;
//转为本地时区：{2019/11/14 9:53:26}
DateTime dt02 = dto.LocalDateTime;
```

#### 通用的老方法

```cs
long unixTimeStamp = 1478162177;
System.DateTime startTime = TimeZone.CurrentTimeZone.ToLocalTime(new System.DateTime(1970, 1, 1)); // 当地时区
DateTime dt = startTime.AddSeconds(unixTimeStamp);
System.Console.WriteLine(dt.ToString("yyyy/MM/dd HH:mm:ss:ffff"));
```

## 备注

### DateTimeOffset使用Now还是UtcNow

对于`DateTimeOffset`，发现有2个获取当前时间的属性：`DateTimeOffset.Now`和`DateTimeOffset.UtcNow`。

如果只是**获取时间戳**，这2个使用哪个都可以，得到的值是一样的。

因为`DateTimeOffset`里面有时区信息，获取时间戳时会使用时区进行转换的，所以获得的时间戳一样。

而也是因为时区的原因，`DateTimeOffset`的其他操作可能会不一样。例如`DateTimeOffset.DateTime`就不一样，此时推荐使用`DateTimeOffset.LocalDateTime`来获得本地时区的时间。

测试代码如下：
```cs
//none：2019/6/14 15:17:43 +08:00
Console.WriteLine("none：{0}", DateTimeOffset.Now);
//utc：2019/6/14 7:17:43 +00:00
Console.WriteLine("utc：{0}", DateTimeOffset.UtcNow);

//none：1560496663
Console.WriteLine("none：{0}", DateTimeOffset.Now.ToUnixTimeSeconds());
//utc：1560496663
Console.WriteLine("utc：{0}", DateTimeOffset.UtcNow.ToUnixTimeSeconds());
```

### DateTime转换为DateTimeOffset

可以直接把`DateTime`赋值给`DateTimeOffset`，内部会自动进行隐式转换。这里涉及到时区，请往下看。

#### DateTime的时区信息(Kind属性)

`DateTime`的**时区信息**存放在`Kind`属性里。`Kind`属性的数据类型是`DateTimeKind`枚举，只有3个值：
- `Unspecified`：未指定/未规定
- `Utc`：`UTC`时间
- `Local`：本地时区

不同情况下得到的`DateTime`的`Kind`是不同的，具体如下：
- `DateTime.Now`：`DateTime.Kind`是 **`Local`(本地时区)**。
- `DateTime.UtcNow`：`DateTime.Kind`是 **`Utc`**。
- `DateTime.Parse()`：

    - 【**默认**】在**未指定时区**时，`DateTime.Kind`是 **`Unspecified`**
    - 指定时区：指定时区后`DateTime.Kind`就是相对应的值。
    
        指定时区有2种方式：
        - 【**默认+优先**】**待转换的字符串**里有时区信息。例如：`2019/11/24 17:40:32 +08:00`
        - 使用`DateTimeStyles`参数来指定时区。`DateTimeStyles`是枚举类型，更多信息自己查看定义，这里不再多说。

`Local`和`Utc`都会把相应的时区传递过去。对于 **`Unspecified`(未指定)**，会被当做**本地时区**来处理（结果已验证，[源码](https://source.dot.net/#System.Private.CoreLib/DateTimeOffset.cs)没看懂）。

#### 测试代码

```cs
//dtNow：2019/11/24 17:40:32(Kind：Local)
DateTime dtNow = DateTime.Now;
//dtUtcNow：2019/11/24 9:40:32(Kind：Utc)
DateTime dtUtcNow = DateTime.UtcNow;
//dtParse：2019/11/24 17:40:13(Kind：Unspecified)
DateTime dtParse = DateTime.Parse("2019-11-24 17:40:13");

//dtoNow：2019/11/24 17:40:32 +08:00
DateTimeOffset dtoNow = dtNow;
//dtoUtcNow：2019/11/24 9:40:32 +00:00
DateTimeOffset dtoUtcNow = dtUtcNow;
//dtoParse：2019/11/24 17:40:13 +08:00
DateTimeOffset dtoParse = dtParse;

Console.WriteLine("DateTime：");
Console.WriteLine("dtNow：{0}(Kind：{1})", dtNow, dtNow.Kind);
Console.WriteLine("dtUtcNow：{0}(Kind：{1})", dtUtcNow, dtUtcNow.Kind);
Console.WriteLine("dtParse：{0}(Kind：{1})", dtParse, dtParse.Kind);

Console.WriteLine();

Console.WriteLine("DateTimeOffset：");
Console.WriteLine("dtoNow：{0}", dtoNow);
Console.WriteLine("dtoUtcNow：{0}", dtoUtcNow);
Console.WriteLine("dtoParse：{0}", dtoParse);
```

输出结果如下：
```
DateTime：
dtNow：2019/11/24 17:40:32(Kind：Local)
dtUtcNow：2019/11/24 9:40:32(Kind：Utc)
dtParse：2019/11/24 17:40:13(Kind：Unspecified)

DateTimeOffset：
dtoNow：2019/11/24 17:40:32 +08:00
dtoUtcNow：2019/11/24 9:40:32 +00:00
dtoParse：2019/11/24 17:40:13 +08:00
```

### DateTimeOffset.Parse的默认时区

`DateTimeOffset.Parse`的默认时区是**当前时区**。

```cs
//parse：2019/6/14 15:38:49 +08:00
Console.WriteLine("parse：{0}", DateTimeOffset.Parse("2019-6-14 15:38:49"));
```

## 参考

1. C# DateTime与时间戳转换：<https://www.cnblogs.com/polk6/p/6024892.html>
2. 如何将Unix时间戳转换为DateTime，反之亦然？：<https://stackoverflow.com/questions/249760/how-can-i-convert-a-unix-timestamp-to-datetime-and-vice-versa>
3. DateTimeOffset源码：<https://source.dot.net/#System.Private.CoreLib/DateTimeOffset.cs>
