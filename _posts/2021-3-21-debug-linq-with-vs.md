---
layout: post
title:  "在VS中调试LINQ(Lambda)"
categories: 编程
tags: csharp linq debug
excerpt: 本文介绍下怎么在VS中调试LINQ(Lambda)
---

* content
{:toc}

## 前言

Linq调试有3种方法，准确来说是2种，因为LinqPad算是复制代码段到外部了。。
1. VS自带调试：lambda表达式打断点
2. VS插件OzCode
3. LinqPad

## VS自带调试

在VS里，是可以对Linq调试的，不过一般打断点都会打在整个语句上，这时候我们要换个打法，把断点打在**lambda表达式**上。

### 注意和前提

1. Linq是**Linq to object**
2. 对于Linq to object，只有集合对象是 **`IEnumerable`** 时，才能命中到Linq里的lambda表达式，`IQueryable`是不行的。如果是`IQueryable`，此时就算在lambda表达式里打上断点，在代码执行时，断点会向上转移到整个语句上。

    如果是`IQueryable`，在lambda表达式里打上断点和设置**操作**，操作会输出错误：`order name: id=error CS0103: 当前上下文中不存在名称“p”, name=error CS0103: 当前上下文中不存在名称“p”`。
3. 对于Linq to object，当集合对象是 **`IEnumerable`** 时，是**延迟执行**的。**只有结果被用到时，才会进行迭代**。所以如果在实际执行前，集合数据发生改变会导致结果集和预期不符。
4. 对于Linq to object，当集合对象是 **`IEnumerable`** 时，对单个对象进行迭代的方式是：先把单个对象走完所有的Linq方法后，直到**最后**或者**执行到返回值不是`IEnumerable`的Linq方法(该方法会被执行)**，才会迭代下一个对象。

    如果Linq方法的返回值不是`IEnumerable`，单个对象的迭代会到该方法(含)为止，会立即进行下一个对象的迭代。所有的对象迭代完毕后，会有一个临时的结果集(非`IEnumerable`)，然后把这个结果集重复前面的步骤，直至结束。
    
    `OrderBy()`的返回值是`IOrderedEnumerable`，所以运行了`OrderBy()`后，单个对象的迭代就会结束，继续下一个对象的迭代。，然后把这个暂存结果集执行`OrderBy()`后面的Linq方法。
    
5. 在 **4** 的基础上，对于`IEnumerable`，如果有多个条件，我们可以写在同一个`Where()`里，也可以拆开写在多个`Where()`里，不会影响效率的，因为不会生成多个暂存结果集。
6. `IEnumerable.AsQueryable().AsEnumerable()`是没问题的，遵循`IEnumerable`的正常流程：断点不会转移，仍然是延迟执行。

### 操作步骤

有2种方法：

- **光标**移动到**箭头`=>`后面的lambda语句(方法体)内**，按`F9`，这个lambda语句的断点就打上了。其他的lambda语句操作类似。
- 右键单击其中一个lambda语句(方法体)内的任意位置，然后选择“断点 - >插入断点”。断点就打在这个lambda表达式上了。

上面的2种方法，都是要把位置选在**lambda语句内**，因为这个语句才是**方法体**，必须要定位到**方法体**内才行！否则还是打在外面了！

### 断点的高级用法

打断点后，我们可以对断点进行设置，可以达到2个目的：
1. 满足条件才触发断点（**条件断点**）
2. 触发断点后，输出当前的数据（**断点操作**）

#### 操作步骤

鼠标放在**断点的小红点**上，会出现浮动块，点击里面的**齿轮**，我们可以在里面设置**条件**和**操作**(可以同时勾选设置)。
- 条件：满足条件才触发断点
- 操作：触发断点后，输出当前的数据

##### 条件 (条件断点)

勾上**条件**，会出现设置框，有3个框。
前2个框可以点开看看一些选择项，第3个框可以输入一些代码，代码里可以使用**变量/方法**，会有智能提示的。

注意：**lambda表达式的参数没有提示，需要手动输入参数名和参数的属性/方法**。

设置好后，只有满足**设置的条件**，才会触发断点。

##### 操作 (断点操作)

勾上**操作**，会出现输入框和勾选框。

我们可以在输入框里输入一些字符串，字符串里可以使用**变量/有返回值的方法**，不过它们必须要放在 **`{}`** 里，会有智能提示的。

注意：**lambda表达式的参数没有提示，需要手动输入参数名和参数的属性/方法**。如果想输出`{}`，需要转义`\{`；如果想输出`\`，需要转义`\\`。

另外，还可以使用一些**特殊关键字**，具体的可以把鼠标放在输入框右侧的 **叹号`!`** 上，会有提示的。

接下来说一下**勾选框(继续执行)**，它默认是**勾选**的：
- 勾选：当触发断点并输出数据后，程序**不会**停下来，会继续执行后面的代码；并且断点的**小红点**会变成**菱形**
- 不勾选：当触发断点并输出数据后，程序**会**停下来

设置好后，当断点触发时，会在**输出窗口**里输出数据的。不过**输出窗口**里会有其他信息，此时需要我们慢慢找数据了。。。

如果集合是`IQueryable`，在lambda表达式里打上断点和设置**操作**，操作会输出错误：`order name: id=error CS0103: 当前上下文中不存在名称“p”, name=error CS0103: 当前上下文中不存在名称“p”`。

### 注意

1. **不能调试LINQ to SQL**，因为LINQ to SQL是翻译成sql语句了。具体见[单步执行和 LINQ](https://docs.microsoft.com/zh-cn/visualstudio/debugger/debugging-linq?view=vs-2019#BKMK_SteppingAndLinq)
2. 由于要对单个Linq语句打断点，建议**每个Linq语句都放在单独的一行**，这样也清晰易读。

	```cs
	Robot tmpRobot01 = robots
		.Where(p => p.Id == miku001.Id)
		.OrderBy(p => p.Name)
		.FirstOrDefault();
	```

### 疑问

如果Linq里没有lambda表达式，打断点就打在了整个语句上，而不是单个Linq上。是这个原因吗？

### 参考

1. 如何在C＃中调试LINQ查询：<https://michaelscodingspot.com/debug-linq-in-csharp/>
2. C#中的条件断点：<https://www.c-sharpcorner.com/UploadFile/b1df45/conditional-breakpoints-in-C-Sharp/>
3. 调试 LINQ：<https://docs.microsoft.com/zh-cn/visualstudio/debugger/debugging-linq?view=vs-2019>

## VS插件OzCode

VS插件OzCode的功能强大，简单易用，可是是收费的。不过OzCode对MVP和开源贡献者是免费的，这就需要努力了。

我没使用过，暂时放几个链接：
1. 2017年调试LINQ：LINQPad与OzCode：<https://oz-code.com/blog/debugging-linq-available-tool-comparison/>
2. 如何在C＃中调试LINQ查询：<https://michaelscodingspot.com/debug-linq-in-csharp/>
3. Vs 调试插件 —OzCode 特性讲解+破解工具和教程：<https://blog.csdn.net/sky__god/article/details/86153982>

## LinqPad

这个软件很强大，可以执行代码片段，当然也可以执行Linq了。我几乎不用，暂时放几个链接：

1. 2017年调试LINQ：LINQPad与OzCode：<https://oz-code.com/blog/debugging-linq-available-tool-comparison/>
2. 如何在C＃中调试LINQ查询：<https://michaelscodingspot.com/debug-linq-in-csharp/>

## 扩展

### 如何知道每一步链式调用的结果

如何知道每一步链式调用的结果？如何知道每一句Linq执行的结果？

有4种方法：
1. VS里使用【快速监视】
2. VS里使用断点设置里的【操作】
3. 使用OzCode
4. 使用LinqPad

#### VS里使用【快速监视】

首先在**整个语句**上设置断点，当程序运行到该断点时，在集合对象上`右键`->`快速监视`，然后把**想知道结果的整个代码**复制到`表达式文本框`里，点击右侧的`重新计算`，就能知道这步链式调用的结果了。

##### 注意

1. 只有把断点设在**整个语句**上才能监视到。不能设置在*lambda表达式*上。

    因为lambda表达式是被编译成了一个方法，断点在这个方法里。运行到该断点时，上下文是这个方法的上下文，只能访问到该方法内部变量，是不能访问到外部对象的！
2. 该方式只能适用于返回结果较少的情况，如果返回结果很多，估计会出问题。某人说：`vs没事儿给你抽个风，整个调试器都直接挂，必须重启调试才能继续`

##### 图示

![vs里使用快速监视](https://img.guoqianfan.com/note/2021/03/vs里使用快速监视.jpg)

#### VS里使用断点设置里的【操作】

这种方式里的断点是设置在**lambda表达式**上，和前面的**VS里使用【快速监视】** 里的断点位置**不一样**！！！

把断点设置在**lambda表达式**上，然后在断点设置里添加条件和操作。
- 条件**必须和lambda表达式一模一样**，否则数据就不同了，建议直接把lambda表达式复制进去。
- 操作里输出有用的简单的信息。

详细的操作步骤见前面的[断点的高级用法](#断点的高级用法)

##### 不填条件的偷懒法

由于每个断点设置里的条件都要把lambda表达式复制进去，十分麻烦，推荐一个简单的方法：

每个Linq语句的结果让**下一个**Linq语句输出，下一个Linq语句不要设置条件，只设置操作。（因为只有当前Linq语句满足条件后，才会进入下一个Linq语句。）

不过如果**只有一个Linq语句**或者**是最后一个Linq语句**，这种偷懒方式就不行了，这时候我们只有1种选择：再加一个Linq语句(`OrderBy...`)，让它来输出。

其实还有一种选择：**在断点里添加条件**。不过这种选择只适用于**只有一个Linq语句**的情况。**是最后一个Linq语句**时是**不行**的！因为**最后一个Linq语句**输出的是**上一条Linq语句**的信息的，如果添加了条件，输出的就是当前Linq语句的信息了，那**上一条Linq语句**的信息由谁来输出？

##### 注意

1. 该方式只能适用于返回结果较少的情况，如果返回结果很多，输出窗口估计能翻好几页吧，那就难受了。。

##### 图示

下图是**不填条件偷懒法**：*每个Linq语句的结果让下一个Linq语句输出，下一个Linq语句不要设置条件，只设置操作*。所以图中是 **`OrderBy`输出`Where`的执行结果**。

![vs里使用断点设置里的操作-不填条件偷懒法](https://img.guoqianfan.com/note/2021/03/vs里使用断点设置里的操作-不填条件偷懒法.png)

#### 使用OzCode

VS插件OzCode很强大，每一个Linq语句的执行结果都能统计并展示出来，详情参考：[如何在C#中调试LINQ查询](https://michaelscodingspot.com/debug-linq-in-csharp/) 和 [如何在C#中调试LINQ查询](https://michaelscodingspot.com/debug-linq-in-csharp/)

#### 使用LinqPad

LinqPad软件很强大，不过数据源是个问题，操作步骤参考：[如何在C#中调试LINQ查询](https://michaelscodingspot.com/debug-linq-in-csharp/) 和 [如何在C#中调试LINQ查询](https://michaelscodingspot.com/debug-linq-in-csharp/)

## 参考

1. 2017年调试LINQ：LINQPad与OzCode：<https://oz-code.com/blog/debugging-linq-available-tool-comparison/>
2. 如何在C#中调试LINQ查询：<https://michaelscodingspot.com/debug-linq-in-csharp/>
3. C#中的条件断点：<https://www.c-sharpcorner.com/UploadFile/b1df45/conditional-breakpoints-in-C-Sharp/>
3. 调试 LINQ：<https://docs.microsoft.com/zh-cn/visualstudio/debugger/debugging-linq?view=vs-2019>
