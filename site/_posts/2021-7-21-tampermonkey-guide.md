---
layout: post
title:  "Tampermonkey脚本编写指南"
categories: 编程
tags: Tampermonkey js
excerpt: 本文介绍下Tampermonkey脚本的语法和要点
---

* content
{:toc}

## 前言

>- 来源：<https://my.oschina.net/u/2268567/blog/828528>
>- 官方文档：<https://www.tampermonkey.net/documentation.php>
>
> 可能还有其他用法暂未列出，多找几个脚本看看就明白了。

### iframe-框架中的坑

iframe中可以放一个完整的网页。

如果网页里有多个iframe，Tampermonkey脚本在每个iframe**都会被触发和运行**，但是顺序**不固定**（每个iframe里网页的加载顺序是不确定的）。

在每个iframe中执行时，默认的上下文是**当前iframe**的上下文。可以使用`window.parent`来获取`上级window`。

## Userscript Header

### @name

脚本的名称。

可以配置多语言：`@name:{语言}`。
```
// @name         Userscript+ : Show Site All UserJS
// @name:zh      Userscript+ : 显示当前网站所有可用的UserJS脚本 Jaeger
// @name:zh-CN   Userscript+ : 显示当前网站所有可用的UserJS脚本 Jaeger
// @name:zh-TW   Userscript+ : 顯示當前網站所有可用的UserJS腳本 Jaeger
// @name:ja   Userscript +：現在のサイトの利用可能なすべてのUserJSスクリプトを表示するJaeger
// @name:ru-RU   Userscript+ : Показать пользовательские скрипты (UserJS) для сайта. Jaeger
// @name:ru   Userscript+ : Показать пользовательские скрипты (UserJS) для сайта. Jaeger
```

### @namespace

该脚本的命名空间

### @version

版本号。当脚本未从 userscript.org安装时用于更新检查，或 Tampermonkey 有问题时检索脚本元数据的。

### @author

作者

### @description

简介.

可以配置多语言：`@description:{语言}`，示例参照 **`@name`** 的多语言。

### @homepage, @homepageURL, @website and @source

作者主页，用于在Tampermonkey选项页面上从脚本名称点击跳转。请注意，如果`@namespace`标记以`http://`开头，此处也要一样。

### @icon, @iconURL and @defaulticon

低分辨率图标

`@icon`支持base64，示例如下：
```
// @icon  data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH3ggEBCQHM3fXsAAAAVdJREFUOMudkz2qwkAUhc/goBaGJBgUtBCZyj0ILkpwAW7Bws4yO3AHLiCtEFD8KVREkoiFxZzX5A2KGfN4F04zMN+ce+5c4LMUgDmANYBnrnV+plBSi+FwyHq9TgA2LQpvCiEiABwMBtzv95RSfoNEHy8DYBzHrNVqVEr9BWKcqNFoxF6vx3a7zc1mYyC73a4MogBg7vs+z+czO50OW60Wt9stK5UKp9Mpj8cjq9WqDTBHnjAdxzGQZrPJw+HA31oulzbAWgLoA0CWZVBKIY5jzGYzdLtdE9DlcrFNrY98zobqOA6TJKHW2jg4nU5sNBpFDp6mhVe5rsvVasUwDHm9Xqm15u12o+/7Hy0gD8KatOd5vN/v1FozTVN6nkchxFuI6hsAAIMg4OPxMJCXdtTbR7JJCMEgCJhlGUlyPB4XfumozInrupxMJpRSRtZlKoNYl+m/6/wDuWAjtPfsQuwAAAAASUVORK5CYII=
```

### @icon64 and @icon64URL

此脚本图标的大小为64x64像素。即使使用这个标签，但`@icon`已经定义了，那么Tampermonkey选项页面的一些地方仍然使用`@icon`定义的图标。

### @updateURL

检查更新的网址。注意：需要定义`@version`

### @downloadURL

检测到更新时将下载脚本的网址

### @supportURL

报告问题的网址

### @include

脚本生效地址，每行一个。不支持URL hash（即网址中的`#`），详情点击<https://forum.tampermonkey.net/viewtopic.php?p=3094>

示例：
```
// @include http://tampermonkey.net/*
// @include http://*
// @include https://*
// @include *
```

### @match

等于`@include`标签（也支持多个），详情点击<http://code.google.com/chrome/extensions/match_patterns.html>。

注意："<all_urls>"语句尚不支持，scheme部分也接受`http*://`和`*://`。

好像：
1. host的**尾缀**不能使用`*`，否则不起作用。

    例如`https://www.google.com.hk*`就**不起作用**，只能使用`https://www.google.com.hk/*`才起作用

### @exclude

不生效页面，每行一个。语法同[@include](#@include)

### @require

脚本本身开始运行之前加载并执行的JavaScript文件，每行一个。注意：通过@require加载的脚本如果是严格模式("use strict")的，可能会影响本脚本的严格模式！

示例：
```
// @require https://code.jquery.com/jquery-2.1.4.min.js
// @require https://code.jquery.com/jquery-2.1.3.min.js#sha256=23456...
// @require https://code.jquery.com/jquery-2.1.2.min.js#md5=34567...,sha256=6789...
```
请阅读本文sub-resource integrity一节，以获取有关如何确保完整性的更多信息。

### @resource

预加载资源，可通过`GM_getResourceURL`和`GM_getResourceText`读取

Code:
```
// @resource icon1 http://tampermonkey.net/favicon.ico
// @resource icon2 /images/icon.png
// @resource html http://tampermonkey.net/index.html
// @resource xml http://tampermonkey.net/crx/tampermonkey.xml
// @resource SRIsecured1 http://tampermonkey.net/favicon.ico#md5=123434...
// @resource SRIsecured2 http://tampermonkey.net/favicon.ico#md5=123434...;sha256=234234...
```
请阅读本文sub-resource integrity一节，以获取有关如何确保完整性的更多信息。

### @connect

此标记定义除顶级域以外的，允许被`GM_xmlhttpRequest`访问的域名。每行一个。

Code:
```
// @connect <value>
```
`<value>`可以具有以下值：
- 域名，例如`tampermonkey.net`（这也将允许**所有子域**）
- 子域名，即`safari.tampermonkey.net`
- `self`，即脚本运行的网址
- `localhost`
- IP地址，例如`1.2.3.4`
- `*`
如果无法全部声明脚本可能连接到的所有域，那么这是下面是推荐做法：

**声明所有已知**的可能由脚本连接的域名，另外添加`// @connect *`。这样，大多数用户可以避免确认对话框，对于没有显式标明的域名，Tampermonkey仍然会询问用户是否允许下一次的连接，而且还提供"始终允许所有域"按钮。如果用户点击此按钮，则将自动允许未来的所有请求。用户还可以通过在脚本设置选项卡中将"*"添加到用户域白名单来将所有请求列入白名单。

### @run-at

定义脚本注入的时刻。

与其他脚本处理程序不同，`@ run-at`定义了脚本想要运行的第一个**可能**的时刻。这意味着，通过@require引入的脚本如果在获取时耗费过多时间，那么脚本可能在网页加载后执行。无论如何，在给定注入时刻后发生的所有DOMNodeInserted和DOMContentLoaded事件都会在注入时缓存并传递到用户脚本。

- `// @run-at document-start`：脚本将尽可能快地注入。
- `// @run-at document-body`：如果body元素存在，则脚本将被注入。
- `// @run-at document-end`：该脚本将在发生DOMContentLoaded事件时或之后注入。
- `// @run-at document-idle`：【**默认**】在发生DOMContentLoaded事件后注入脚本。

    当初始的HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完全加载。
- `// @run-at context-menu`：如果在浏览器上下文菜单（仅限桌面Chrome浏览器）中点击该脚本，则会注入该脚本。注意：如果使用此值，则将忽略所有@include和@exclude语句，但这可能会在将来更改。

注意：**有些网站的某些函数是延迟加载的，所以上面这些脚本注入时刻都会无用，此时只能手动延迟几秒，等待这些函数记载完毕再执行用户脚本。**（可以通过js判断函数是否存在吗？）

### @grant

`@grant`用于添加`GM`函数到白名单，例如unsafeWindow对象和一些强大的window函数。如果没有给出`@grant`标签Tampermonkey会猜测脚本需要。

Code:
```
// @grant GM_setValue
// @grant GM_getValue
// @grant GM_setClipboard
// @grant unsafeWindow
// @grant window.close
// @grant window.focus
// ...
```
另外window.close和window.focus必须显式声明。

如果使用`// @grant none`，沙箱将被禁用，脚本将直接在页面上下文中运行。在此模式下，没有`GM_`函数，但`GM_info`属性仍然可用。

Code:
```
// @grant none
```

### @noframes

此标记使脚本在主页面上运行，但不会在iframe上运行。

### @unwrap

此标记会被忽略，因为Google Chrome / Chromium不需要此标记。

### @nocompat

由于部分代码可能是专门为Firefox/Greasemonkey所写，通过此标记，Tampermonkey会禁用所有代码优化行为。

Code:
```
// @nocompat支持的浏览器
// @nocompat Chrome
```

## Application Programming Interface(GM函数)

### 注意

GM函数要与`@grant`配合使用，`@grant`用于添加GM函数到白名单。详细参考 **@grant**。
- 无`@grant`标签：Tampermonkey会猜测脚本是**需要**的。`window.close`和`window.focus`必须**显式声明**。
- `@grant none`：沙箱将被禁用，脚本将直接在页面上下文中运行。在此模式下，**不能**使用`GM_`函数，但`GM_info`属性仍然可用。（相当于直接在页面上写js）
- `@grant GM_xxx`：只允许运行添加的GM函数。可以用多行添加多个GM函数。

### unsafeWindow

unsafeWindow对象提供对页面javascript函数和变量的完全访问。

### Subresource Integrity

`@resource`和`@require`标记的URL的哈希部分可用于此目的。

Code:
```
// @resource SRIsecured1 http://tampermonkey.net/favicon1.ico#md5=ad34bb...
// @resource SRIsecured2 http://tampermonkey.net/favicon2.ico#md5=ac3434...,sha256=23fd34...
// @require https://code.jquery.com/jquery-2.1.1.min.js#md5=45eef...
// @require https://code.jquery.com/jquery-2.1.2.min.js#md5=ac56d...,sha256=6e789...
```

> 个人理解：针对`@resource`和`@require`的增强功能，可以为其添加hash后缀，hash值不匹配时则**禁止**使用这个资源。

Tampermonkey支持 MD5作为备用，其他（SHA-1，SHA-256，SHA-384和SHA-512）依赖于window.crypto。在多个hash（由逗号或分号分隔）的情况下，TM使用最后一个支持的hash。如果外部资源的内容不匹配所选的hash，则资源不会传递给userscript。

### GM_addStyle(css)

将给定的CSS添加到文档。

**步骤如下：**

1.首先添加GM函数到白名单：
```
// @grant        GM_addStyle
```

2.使用`GM_addStyle(css)`函数：
```js
GM_addStyle(`
.toc-sticky {
    position: -webkit-sticky;
    position: sticky;
    top: -40px;
    z-index: 1;
}`);
   
document.querySelector('.document-toc-container').classList.add('toc-sticky');
```

参考：[GM_addStyle equivalent in TamperMonkey](https://stackoverflow.com/q/23683439)

### GM_deleteValue(name)

从storage中删除"name"。

### GM_listValues()

列出storage的所有name。

### GM_addValueChangeListener

语法：
```
GM_addValueChangeListener(name, function(name, old_value, new_value, remote) {})
```

向storage添加一个change listener，并返回此监听器ID。

`name`是被观察变量的名称。

回调函数的`remote`参数表明此值的修改者是另一个网页(browser tabs)的实例（true），或是在此脚本实例（false）。

**因此此function可通过脚本在不同的网页选项卡互相通信。**

### GM_removeValueChangeListener(listener_id)

通过ID删除change listener。

### GM_setValue(name, value)

在storage中存储值。

### GM_getValue(name, defaultValue)

在storage中取得name的值，取不到则使用defaultValue。

### GM_log(message)

将消息记录到控制台。

### GM_getResourceText(name)

从script header获取`@resource`标记定义的内容。

### GM_getResourceURL(name)

从script header获取`@resource`标记定义的base64编码的URI。

### GM_registerMenuCommand(name, fn, accessKey)

在要运行脚本的网页上，在Tampermonkey中注册一个menu，并返回menu的命令 id。

### GM_unregisterMenuCommand(menuCmdId)

通过menu id反注册由GM_registerMenuCommand注册menu。

### GM_openInTab(url, options), GM_openInTab(url, loadInBackground)

使用url打开新tab。

options对象可以有两个属性：'active'决定新的选项卡是否在前台打开，'insert'决定是否在当前标签之后打开新标签页('insert' that inserts the new tab after the current one.)。否则情况都是仅仅添加新选项卡。

'loadInBackground'与'active'相反，并且被添加以实现Greasemonkey兼容性。如果既没有"active"也没有"loadInBackground"，那么选项卡不会在前台打开。这个函数返回一个对象，该对象具有close方法，onclosed监听器，名为"closed"的标志。

### GM_xmlhttpRequest(details)

详细参考 [GM_xmlhttpRequest(details)](https://www.tampermonkey.net/documentation.php#GM_xmlhttpRequest) 和 [示例](#发送请求-gm_xmlhttprequest)

创建一个xmlHttpRequest。（**需要`@connect`支持**）

详细属性：
- method - GET，HEAD，POST之一
- url - 目标网址
- headers - 即user-agent, referer, ... (Safari和Android浏览器不支持某些特殊headers
- data - 通过 POST 请求发送的字符串
- binary - 以二进制模式发送数据
- timeout - 超时时间（单位ms）
- context - 将添加到response对象的属性
- responseType - arraybuffer，blob，json之一
- overrideMimeType - request的MIME类型
- anonymous - request将不发送cookies，参见fetch
- fetch - (beta) 使用fetch而不是xhr，在Chrome这会导致xhr.abort，details.timeout和xhr.onprogress失效，并使xhr.onreadystatechange只接收readyState4事件。
- username 用于认证的用户名
- password 密码
- <events> - onload onerror, onreadystatechange, onprogress, onloadstart, ontimeout

    - `onloadstart`：如果请求**开始加载**，则执行onloadstart回调
    - `onload`：如果请求**已加载**，则执行onload回调。
    
        `onload`事件绑定到一个方法，方法参数为`响应体`，`响应体`的详细属性参考[GM_xmlhttpRequest(details)](https://www.tampermonkey.net/documentation.php#GM_xmlhttpRequest)

**注意：**
- details不支持synchronous标志？
- 如果遇到3XX跳转，内部会自动跳转
- 浏览器仍然会处理响应，并设置cookie什么的
- `onload`绑定的方法设置断点后，好像不能命中

### GM_download(details), GM_download(url, name)

从给定的 URL 下载。

details可以具有下列属性：
- url - 下载数据的 url 路径
- name - 文件名，因为安全原因，文件扩展名必须显式声明于Tampermonkey 白名单。
- headers - 参见GM_xmlhttpRequest。
- saveAs - 布尔值，是否显示保存对话框.
- onload - function() {} - 下载完成后调用的函数
- onerror - function(download) {} - 当出现错误时调用的函数

	- onerror参数可以具有以下属性：
	
		- error -错误原因
		- not_enabled -用户未启用下载功能
		- not_whitelisted -所请求的文件扩展名没有列入白名单
		- not_permitted - 用户启用了下载功能，但没有给予下载权限
		- not_supported - 浏览器不支持下载功能
		- not_succeeded -下载启动失败，details属性提供更多信息
	- details -关于该错误的详细信息

根据下载模式，GM_info提供了一个名为downloadMode的属性，该属性设置为以下值之一：native,，disabled，browser.

### GM_getTab(cb)

获取一个tab的持久化对象，只要此选项卡是打开的。

### GM_saveTab(tab)

保存tab对象以在页面unload后重新打开。

### GM_getTabs(cb)

获取所有tab对象的hash，以便与其他脚本实例进行通信。

### GM_notification(details, ondone), GM_notification(text, title, image, onclick)

显示一个 HTML5 桌面通知（Desktop notification）或高亮当前tab。

details可以具有以下属性：

- text - 通知的文本，如果设置了highlight
- title - 通知标题（可选）
- image - 图像（可选）
- highlight - 布尔值，发送通知时是否要高亮tab（可选）
- timeout - 多久后通知将会隐藏 (可选，0 = 禁用)
- ondone - 当关闭通知时调用（无论是由于超时或鼠标单击），或选项卡已经是高亮状态（可选）
- onclick - 用户单击通知时调用（可选）

### GM_setClipboard(data, info)

将数据复制到剪贴板。参数 ' info ' 可以是对象,例如"{ type: 'text', mimetype: 'text/plain'}"或仅仅是一个字符串表达式（text或者html类型）。

### GM_info

获取有关脚本和 TM 的一些信息。该对象可能如下所示︰

Code:
```
Object+
---> script: Object+
------> author: ""
------>copyright: "2012+, You"
------>description: "enter something useful"
------>excludes: Array[0]
------>homepage: null
------>icon: null
------>icon64: null
------>includes: Array[2]
------>lastUpdated: 1338465932430
------>matches: Array[2]
------>downloadMode: 'browser'
------>name: "Local File Test"
------>namespace: "http://use.i.E.your.homepage/"
------>options: Object+
--------->awareOfChrome: true
--------->compat_arrayleft: false
--------->compat_foreach: false
--------->compat_forvarin: false
--------->compat_metadata: false
--------->compat_prototypes: false
--------->compat_uW_gmonkey: false
--------->noframes: false
--------->override: Object+
------------>excludes: false
------------>includes: false
------------>orig_excludes: Array[0]
------------>orig_includes: Array[2]
------------>use_excludes: Array[0]
------------>use_includes: Array[0]
--------->run_at: "document-end"
------>position: 1
------>resources: Array[0]
------>run-at: "document-end"
------>system: false
------>unwrap: false
------>version: "0.1"
---> scriptMetaStr: undefined
---> scriptSource: "// ==UserScript==\n// @name       Local File Test\n ...."
---> scriptUpdateURL: undefined
---> scriptWillUpdate: false
---> scriptHandler: "Tampermonkey"
---> isIncognito: false
---> version: "4.0.25"
```

### <><![CDATA[your_text_here]]></>

Tampermonkey支持这种写法，会尝试自动检测脚本是否需要启用此兼容性选项。

## 公共方法

### sleep

```js
function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
}
```

**注意**：使用时，父方法必须是异步方法！父方法必须被其他方法调用，并且调用时不能进行等待！

## 示例

### 发送请求-GM_xmlhttpRequest

```js
GM_xmlhttpRequest({
   method: "POST", 
   url: "http://localhost:7777", 
   data: "testing123",
   headers:  {
         "Content-Type": "application/x-www-form-urlencoded"
             },
   onload: function(response) 
   {
      console.log(response);
   }
});
```
