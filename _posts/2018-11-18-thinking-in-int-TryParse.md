---
layout: post
title:  "int.TryParse非预期执行引发的思考"
categories: 编程
tags: csharp
author: 过千帆
excerpt: 在C#中做类型转换时有时会使用到TryParse方法，该方法如果转换失败，out参数的值是什么呢？我就在这里犯傻了，特意写文记录下。
---

* content
{:toc}

## 问题出现

这天在写一个页面，想谨慎些就用了`int.TryParse`，结果出问题了。

代码如下：

``` c#
int id = 1000;
//Request.QueryString["id"] = null
int.TryParse( Request.QueryString["id"], out id );
//使用 id 进行其他操作...
```

因为`Request.QueryString["id"] = null`，所以我的预期是`id=1000`。可是我错了，实际结果是**id=0**。测试多次都是这样。我感觉要出事了。

事实上我对TryParse一直存在这么个认知（以上面代码举例）：

* 如果转换成功，id=转换后的值；
* 如果转换失败，不进行任何操作，id仍然是1000。

可是现在我知道我错了，更严重的是我按照这样的思维写了不少代码。。。还好我确保输入正确使其都能转化成功，至今没出什么幺蛾子。~~出幺蛾子的话我早就滚蛋了吧。~~

不过现在不是考虑这些的时候，工作要紧，就赶紧改了代码，先把新功能上线了再说...

## 问题分析

几天后，有空了，就开始研究这个问题，总结如下：  
**`TryParse`转换失败时，`out参数`返回的是什么？**

网上搜了下，在[stackoverflow](https://stackoverflow.com/questions/1078512/why-does-integer-tryparse-set-result-to-zero-on-failure?noredirect=1&lq=1)上看到了这么一段话（谷歌翻译）：
> 你是对的，如果失败，TryParse使用0。（MSDN非常清楚地说明了这一点）但你可以检查paseSuccess并返回你的默认值，如果这是你想要的。

~~现在发现当时看的是VB.NET...还好此时此刻这货特性和C#是一样的...不然又坑了...~~

既然提到了MSDN，那就去看看。果然，[MSDN](https://docs.microsoft.com/zh-cn/dotnet/api/system.int32.tryparse?view=netframework-4.7.2)上在`result`处写着这么一段话：

> `result` Int32  
当此方法返回时，如果转换成功，则包含与`s`中所包含的数字等效的 32 位无符号整数值；如果转换失败，则包含零。 如果`s`参数为`null`或为 Empty、格式不正确，或者表示小于 MinValue 或大于 MaxValue 的数，则转换失败。 此参数未经初始化即进行传递；最初在`result`中提供的任何值都会被覆盖。

有这么几处重点：

* 当此方法返回时，如果转换成功，则包含与`s`中所包含的数字等效的 32 位无符号整数值；**如果转换失败，则包含零。**
*  此参数未经初始化即进行传递；**最初在`result`中提供的任何值都会被覆盖**。

”out参数“、“未经初始化即进行传递”，看到这些，我突然想到了`out`参数的特性：“`out`参数好像是不需要初始化的“。如果传入时不需要初始化，那么在TryParse方法内部，当转换成功时可以赋值（转换后的值）；当转换失败时，也必须赋值，要赋值就必定是默认值。

到此我的疑惑已经解开了。搞了大半天，竟然是`out关键字`的性质。恍然大悟的同时，又感觉到自己的C#基础的薄弱。。。

## 总结

`TryParse`使用时，传入的`out`参数的原始值会被**覆盖**掉，具体如下：

* 如果转换**成功**，使用**转换成功后的值**覆盖
* 如果转换**失败**，使用**该类型的默认值**覆盖

## 其他

### 转换失败时不使用默认值覆盖原始值的2种方法

既然已经了解了本质，当然也不能忘了咱们的初衷，那就是**TryParse转换失败时，怎么不使用默认值覆盖我们设定的原始值？**

下面分享一下在stackoverflow上看到的2种方法

``` c#
//方法1、使用out参数的性质
int i = int.TryParse(s, out i) ? i : 42;

//方法2、扩展方法
public class Extensions
{
    public static int? TryParse(string this Source)
    {
        if(int.tryparse .... 
    }
}
//使用
v = "234".TryParse() ?? 0
```

### `out`关键字和`ref`关键字的区别

说到`out`关键字，就不得不提`ref`关键字，他们之间的区别是什么呢？

额，这个稍后我会再写一篇博文的，到时候此处会贴上链接，敬请期待...

## 参考

1. SF：<https://stackoverflow.com/questions/1078512/why-does-integer-tryparse-set-result-to-zero-on-failure?noredirect=1&lq=1>
2. SF：<https://stackoverflow.com/questions/10693231/elegant-tryparse>
3. MSDN：[Int32.TryParse Method](https://docs.microsoft.com/zh-cn/dotnet/api/system.int32.tryparse?view=netframework-4.7.2)




