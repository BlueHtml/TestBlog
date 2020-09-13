---
layout: post
title:  "IQueryable 和 IEnumerable 的区别"
categories: 编程
tags: csharp linq
author: 过千帆
excerpt: 在C#中使用 Linq to sql 时，经常搞混 IQueryable 和 IEnumerable 这两种类型，本文简单分析下它们之间的区别和使用场景。
---

* content
{:toc}

## 前言

不管是**Linq to object**，还是**Linq to sql**或**Linq to Entity**，`IQueryable`和`IEnumerable`都是**延迟执行**的，它们之间的区别仅仅在于**扩展方法的参数类型**不同。（迭代/枚举方式不同？作用对象不同？）

## IQueryable 和 IEnumerable 的区别

* `IQueryable`：扩展方法接受的是`Expression`

    对于**Linq to sql**或**Linq to Entity**，`Expression`必须要能转成sql，否则会报错。

* `IEnumerable`：扩展方法接受的是`Func`(`Func`是C#语法)。

    **`IEnumerable`跑的是Linq to Object，会强制从数据库中读取所有数据到内存里，所以可以使用C#语法。**
    
    对于**Linq to sql**或**Linq to Entity**，`Func`是无限制的，因为它是使用C#语法操作数据的。这也从侧面说明：数据已经读取到内存中了。
    
    `IEnumerable`的扩展方法是`Func`，不会转换为sql。转换为sql的是内部的`IQueryable`。所以要注意**条件最好限制在`IQueryable`里，否则`IQueryable`可能会读取大量数据，增加耗时和内存。**

## AsEnumerable() 和 ToList() 的区别

* `ToList()`：**立即**执行。会立即执行sql，取出数据到内存中。

* `AsEnumerable()`：**延迟**执行，真正使用时才执行sql读取数据。**此处有坑，一定要往下看**  

对`IQueryable`对象使用`AsEnumerable()`后，仍然是**延迟执行**，不过此时**对象本质已经变了**。  

前面已经说了 **`IEnumerable`的扩展方法接受的是`Func`(C#语法)**，当**ie对象(iq转变)** 真正使用时，会有2个步骤：  

1. 它会把**iq对象(转变之前的)** 的扩展方法翻译成sql语句，查询出数据加载到内存中，变为ie对象；  
2. 此时再把**ie对象(转变之后的)** 的扩展方法，使用C#求解，得到最终结果。 

例如： 
    
iq对象的Skip、Take方法，会被翻译成sql，在数据库里执行取出最终结果。  

而ie对象的Skip、Take方法，则会**取出全部数据到内存中，在内存中执行Skip、Take**，会耗费大量资源。

## 使用场景

- `IQueryable`：**使用EFCore动态拼接多个`where`条件时**使用。(**延迟查询，每次真正使用时都会重新读取数据。**)

- `IEnumerable`：当扩展方法无法转换为sql时，可以使用`AsEnumerable()`转换为`IEnumerable`。因为`IEnumerable`的扩展方法都是使用**C#语法**处理数据的。(**延迟查询，每次真正使用时都会重新读取数据。**)

- `ToList()`：当 where条件已经确定了，就可以使用`ToList()`从数据库中立即取出数据，后面重复使用这些数据就行。

不过为了省事，我一般都是使用`IQueryable`拼接好条件后，直接`ToList()`来使用了。。。

## 备注

### 异常：System.InvalidOperationException: 无法枚举查询结果多次

#### 异常出现条件

在**Linq to sql**或**EF**(非EFCore)中，**直接执行sql语句**来查询数据后，对数据集(`IEnumerable`)进行**多次枚举操作**就会引发这个异常。

经测试，多次`Count()`会引发此异常。其他的`Sum()`、`foreach`等等应该也是，有待验证。。。

我的理解是：~~数据集(`IEnumerable`)是使用枚举器来处理每项数据的，而枚举器只能走一次。~~ 搞不懂枚举器和迭代器了，需要研究下。。。

注意，**EFCore中不会出现这个异常**，原因请搜索`efcore执行sql`。

#### 解决方法

- 方法1：把查询结果`ToList()`，后续使用`List`来操作数据。
- 方法2【**推荐**】： 抛弃内置的，使用**Dapper**，因为Dapper的查询结果本质是`List`。（多结果集不是，更多信息搜索`Dapper`。）

#### 异常重现代码

下面的代码是**Linq to sql**的，网上说EF也会出现该异常，代码应该类似。

另外网上搜索该异常大部分都是**执行存储过程**时出现的，其实也是直接执行sql来查询数据，本质一样。

```cs
string sql = @"
select top 100 *
from [dbo].[BaseSupplier_OTAOnline]";

//return db.ExecuteQuery<T>(sql, parameters);
IEnumerable<BaseSupplier_OTAOnline> ieBs02 = bdb.QueryBySql<BaseSupplier_OTAOnline>(sql);//"exec Pro_BaseSpOtaOnline_Test01"

int count = ieBs02.Count();

ieBs02 = ieBs02.Skip(1).Take(2);

//****此处会引发异常****
int count02 = ieBs02.Count();
```

### 误区：对 iq对象 和 ie对象 使用`foreach`时，对于循环的每项都要查询数据库

**错误！**  

foreach针对的是**数据集整体对象**（迭代器？）。当使用foreach时，不管是iq对象还是ie对象，它们都是**查询数据库一次，然后开始循环，直至循环结束**。不过，当后续再次使用iq对象或ie对象的具体数据时，它们仍然会再次查询数据库。

注意：iq对象的结果是**数据集**。它只能把当前存储的表达式树转换为sql。它无法对其进行处理来做到一次一条的取出数据，因为根本就不可能！怎么能无中生有呢？

#### 反向验证

也可以这样想：如果是一条一条取数据的话，程序怎么知道每次应该取哪条数据？
- 使用`DataReader`？

    不行，效率太低下。因为取出每条数据后，还需要对数据进行一系列的操作(代码逻辑)，这需要耗费时间。而`DataReader`是需要在线保持数据库连接的，耗时太长会导致同一时间有很多数据库连接，很快就会达到数据库连接池上限。这种方法很不可取。
- 对生成的sql进行`top 1`处理？

    那要怎么知道每次取出哪条数据呢？使用上一条数据的信息作为`where`条件？不行，这么做太傻逼，网络数据传输增加；查询效率也低下；占用数据库连接池资源。种种缺点，简单问题复杂化。

由上面的反例可以看出，一条一条查数据可以实现，但是太二逼。完全不如一次性全部读取数据的好。

## 其他

### IQueryable和IEnumerable生成sql的测试代码

先说下结论：
- 只会把`IQueryable`的条件(`Expression`)翻译成sql，`IEnumerable`的条件(`Func`)不会被翻译成sql。代码中生成的sql可以验证。
- 二者都是延迟执行的，真正使用过的时候才会查询数据库。

#### NetFramework

测试环境：
- .NET Framework 4.5
- LINQ to SQL类（**不是**EntityFramework）

```cs
            BaseSpDB bdb = new BaseSpDB();
            //不查询数据库
            IQueryable<BaseSupplier_OTAOnline> iqBs = bdb.baseSpByAll().Where(p => p.ID < 10);
            //不查询数据库
            IEnumerable<BaseSupplier_OTAOnline> ieBs = iqBs.AsEnumerable();
            //不查询数据库
            ieBs = ieBs.Where(p => p.ID > 5);

            //执行sql
            //只执行iq的条件
            //查询数据库
//exec sp_executesql N'SELECT [t0].[ID], [t0].[OTAName], [t0].[OnlineSupplier], [t0].[PushUrl], [t0].[Note], [t0].[EditCode], [t0].[AddUser], [t0].[PushDate], [t0].[GetDate], [t0].[EditDate], [t0].[AddDate]
//FROM [dbo].[BaseSupplier_OTAOnline] AS [t0]
//WHERE [t0].[ID] < @p0',N'@p0 int',@p0=10
            List<BaseSupplier_OTAOnline> bsList = ieBs.ToList();

            //再次查询数据库
//exec sp_executesql N'SELECT [t0].[ID], [t0].[OTAName], [t0].[OnlineSupplier], [t0].[PushUrl], [t0].[Note], [t0].[EditCode], [t0].[AddUser], [t0].[PushDate], [t0].[GetDate], [t0].[EditDate], [t0].[AddDate]
//FROM [dbo].[BaseSupplier_OTAOnline] AS [t0]
//WHERE [t0].[ID] < @p0',N'@p0 int',@p0=10
            foreach (var item in ieBs)
            {
                
            }
```

#### NetCore

测试环境：
- AspNetCore 2.1
- EFCore

```cs
            //不查询数据库
            IQueryable<BaseSupplier_OTAOnline> iqOta = ctx.BaseSupplier_OTAOnline.Where(p => p.ID < 10);

            //不查询数据库
            IEnumerable<BaseSupplier_OTAOnline> ieOta = iqOta.AsEnumerable();

            //不查询数据库
            ieOta = ieOta.Where(p => p.ID > 5);

            //执行sql
            //只执行iq的条件
            //查询数据库
//SELECT [p].[ID], [p].[AddDate], [p].[AddUser], [p].[EditCode], [p].[EditDate], [p].[GetDate], [p].[Note], [p].[OTAName], [p].[OnlineSupplier], [p].[PushDate], [p].[PushUrl]
//FROM [BaseSupplier_OTAOnline] AS [p]
//WHERE [p].[ID] < 10
            List<BaseSupplier_OTAOnline> bsList = ieOta.ToList();

            //再次查询数据库
//SELECT [p].[ID], [p].[AddDate], [p].[AddUser], [p].[EditCode], [p].[EditDate], [p].[GetDate], [p].[Note], [p].[OTAName], [p].[OnlineSupplier], [p].[PushDate], [p].[PushUrl]
//FROM [BaseSupplier_OTAOnline] AS [p]
//WHERE [p].[ID] < 10
            foreach (var item in ieOta)
            {

            }
```

## 参考

1. [LINQ查询中的IEnumerable\<T\>和IQueryable\<T\>](https://www.cnblogs.com/long-gengyun/p/3929900.html)
2. [LINQ使用细节之.AsEnumerable()和.ToList()的区别](https://www.cnblogs.com/Mainz/archive/2011/04/08/2009485.html)
3. 建议29：区别LINQ查询中的IEnumerabl\<T\>和IQueryable\<T\> - 陆敏技《编写高质量代码改善C#程序的157个建议》
