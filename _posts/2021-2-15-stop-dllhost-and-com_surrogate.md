---
layout: post
title:  "强制停止dllhost和COM Surrogate"
categories: 电脑
tags: win 任务管理器
---

* content
{:toc}

这天打开任务管理器，发现内存占用排第一位的是`dllhost.exe`。？？没见过这家伙哎，是net core搞出来的吗？没听说过这方面的传闻额。。

那就搜一下。结果还不少，大部分都是与其内存占用/CPU占用过高有关。找了半天，说是直接结束进程就行。




我很犹豫，因为是服务器，怕出问题。但是内存不足的话也会导致网站出问题，最后决定直接结束进程。

在`详细信息`栏，选中`dllhost.exe`，右键`结束任务`，然后看了下网站，好像没啥问题。

然后转移到`进程`栏，咦？这个内存占用第一位的`COM Surrogate`又是什么鬼？刚才网上看到的说它也与`dllhost.exe`有关，既然直接结束`dllhost.exe`没啥问题，那就把它也终止了吧。

在`进程`栏，选中`COM Surrogate`，右键`结束任务`，然后看了下网站，好像也没啥问题。

经过上面2个操作，内存占用又降低了，功德无量。

**注意：**
1. [这里](http://www.windows764.org/news/13050.html)说是终止`dllhost.exe`时要选择`结束进程树`，这样也能一并终止`COM Surrogate`了？下次可以试试
2. `dllhost.exe`(`COM Surrogate`)可能与文件管理器（文件预览/关联）有关，我也没搞清楚。

    详细推荐阅读[What Is “COM Surrogate” (dllhost.exe) and Why Is It Running on My PC?](https://www.howtogeek.com/326462/what-is-com-surrogate-dllhost.exe-and-why-is-it-running-on-my-pc/)，里面说可以使用Process Explorer来查看它托管的COM对象或DLL文件。
