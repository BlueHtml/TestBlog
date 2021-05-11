---
layout: post
title:  "Visual Studio 快捷键"
categories: IDE
tags: vs
excerpt: 整理了一下 Visual Studio 里的快捷键
---

* content
{:toc}

## 前言

VS官方：[Visual Studio 中的默认键盘快捷方式](https://docs.microsoft.com/zh-cn/visualstudio/ide/default-keyboard-shortcuts-in-visual-studio)，查看`全局快捷键`和`文本编辑器`这2块。（中文翻译，有的不明其意）

另外VS里的操作后面也有对应的快捷键。

下面列出一些常用的快捷键，已在**VS2017**里测试通过。

## 常用

- ctrl+K+C：注释选中的代码块（C#为`//`，C++别人说是`/*..*/`）
- ctrl+K+U：取消注释选中的代码块
- ctrl+K+D：代码整理/格式化（全局）
- ctrl+K+F：代码整理/格式化（只针对**选中的代码块**）
- Ctrl+D：向右复制选中内容。

    - 如果选中内容以换行符结尾，则是向下复制。
    - 如果未选中任何内容，会把**光标所在行**向下复制。
- Ctrl+L：快速剪切当前行。可以当“删除当前行”用
- Ctrl+j：调出智能提示
- Ctrl+M+M：展开或折叠当前的代码（展开或收缩方法，类等）
- Ctrl+Enter：在光标**上面**插入空行
- Ctrl+Shift+Enter：在光标**下面**插入空行
- Alt+Enter：快速操作。可以弹出命名空间选项，常用于引入命名空间。（说是【Ctrl+.】也行，我这不行）（= Alt+Shift+F10）
- Ctrl+M+O：折叠所有方法 （只针对方法）
- Ctro+M+L：展开所有方法（只针对方法）
- Ctrl+减号：回退到光标的上一次位置。（可以跨越文件）
- Ctrl+Shift+- ：向前导航（移动到光标的**下一次**位置）
- Ctrl+G：跳转到指定行（或者`双击底部状态栏中的行号`）
- Alt+↑/↓：当前行向上/向下移动
- F1: 打开帮助文档。会打开网页，然后跳转到MS Docs，不用再赋值粘贴搜索了
- F4：显示属性窗口
- F5：调试(启动)
- F10：逐过程调试（不会进入方法体内）
- F11：逐语句调试（一句一句执行，会进入方法体内）
- Ctrl+F10: 运行到光标处
- Ctrl+F5：运行（不调试）
- Shift+F5：停止调试
- Ctrl+Shift+F5：重启调试
- F12：转到定义，到变量或函数定义的地方，与`ctrl+-`配合使用非常方便
- Ctrl+F12：转到实现（子类的实现，配合`ctrl+-`）
- Shift+F12：查找所有的引用（变量/函数）

- Ctrl+C：复制选中内容。（如果未选中任何内容，则复制光标所在行。）
- Ctrl+X：剪切选中内容。（如果未选中任何内容，则剪切光标所在行。此时可以当“删除当前行”用）
- Ctrl+U：选中字符变**小写**
- Ctrl+Shift+U：选中字符变**大写**
- Shift+Alt+方向键(Alt+鼠标拖动)：块状选择文本。选中区域后输入内容，会在所有行同时输入 
- Ctrl+Shift+B：编译（生成**解决方案**，注意与**当前项目**区分）
- Ctrl+F：查找
- Ctrl+H：替换

## 不常用

- Shift+Alt+Enter：最大化代码编写区域（代码全屏模式），即去掉所有其它辅助窗口只留下代码编写窗口。再按一次返回到原来界面
- ctrl+k+S：对选中的部分快速调出代码段（可以添加`foreach`、`region`）（C#代码段）
- Ctrl+K+X：插入代码段（有3个文件夹可供选择，不常用）
- Ctrl+R+W：显示或者隐藏Tab标记（查看可见空白）
- Ctrl+]：匹配选中的括号（大括号/小括号），在多层循环+判断语句时非常方便。可以选中括号，也可以把光标放在括号的前面/后面。
- Ctrl+Shift+]：在匹配的括号内选中代码段(含两端括号)
- Ctrl+F3：搜索当前单词（选中或者光标位于当前单词）
- Ctrl+W：选择当前单词
- Ctrl+Delete：删除下一个单词（从光标删除至单词结尾处）
- Ctrl+Backspace：删除上一个单词（从光标删除至单词开始处）
- 移动或复制代码块：左键拖动代码可以**移动**代码；同时按Ctrl**复制**代码到目标位置
- Ctrl+PgUp：移动光标到当前视图顶部
- Ctrl+PgDn：移动光标到当前视图底部
- Ctrl+Shift+PgUp：选中到当前视图顶部
- Ctrl+Shift+PgDn：选中到当前视图底部
- 让智能感知透明：按住Ctrl键
- Ctrl+K+K：添加、删除书签
- Ctrl+K，Ctrl+N：跳转到下一书签
- Ctrl+K，Ctrl+P：跳转到上一书签
- Ctrl+K，Ctrl+L：删除所有书签（需要二次确认）
- Alt+F7：工具窗口导航器（左、右、下侧的工具窗口之间进行切换）
- Ctrl+Alt+J：打开对象浏览器
- F9：打/删除断点
- Ctrl+F9：启用或禁用断点
- Ctrl+Shift+F9：删除所有断点（需要二次确认）
- ctrl+I：递增搜索，与ctrl+F不同的是搜索期间不显示搜索对话框，且ctrl+F搜索下一个直接按Enter即可，而ctrl+I搜索下一个按ctrl+I或F3，Escape退出，连续按两次ctrl+I重复上次搜索
- Shift+Alt+C：添加新类
- Ctrl+左右箭头键：光标一次移动一个单词
- Ctrl+上下箭头键：滚动代码屏幕，但不移动光标位置。

## 适用于所有文本编辑器

- Home键可以快速跳到这一行的首部
- End键可以快速跳到这一行的尾部
- Shift+Home可以快速选中这一行光标前的部分
- Shift+End可以快速选中这一行光标后的部分
- Ctrl+Home：定位到文档首(首行）
- Ctrl+End：定位到文档尾(尾行)
- Ctrl+Shift+Home：选中到文档首行
- Ctrl+Shift+End：选中到文档尾行

## 适用于多个软件

- Ctrl+Tab：软件内的导航器

## 其他

- Ctrl+M+L：折叠和展开整个文件（最后折叠为2个：`using`，`namespace`，感觉没啥用）
- Ctrl+Shift+H：在文件中替换（不懂用处，比`Ctrl+H`要强大些？能搜索到外部文件，能设置指定后缀名，能处理外部文件夹。参照[使用Visual Studio查找和替换目录中的文件](https://www.cnblogs.com/jaxu/archive/2010/09/05/1818676.html)和[在文件中查找](https://docs.microsoft.com/zh-cn/visualstudio/ide/find-in-files)）

另外还有<http://www.dofactory.com/reference/visual-studio-shortcuts>和<https://shortcutworld.com/Visual-Studio/win/Visual-Studio_2015_Shortcuts>，不过都是英文的。。。

## 参考

**下面列出的链接里的操作已全部验证，有些不起作用，请以本文为准。**（失效原因可能是其他程序占用了VS的快捷键）

1. Visual Studio 常用快捷键：<https://www.cnblogs.com/TankXiao/p/3164995.html>
2. Visual Studio 常用快捷键 (二)：<https://www.cnblogs.com/TankXiao/p/3468831.html>
3. 【转】Visual studio 快捷键大全：<https://www.cnblogs.com/xylc/p/3665483.html>
4. Visual Studio最好用的快捷键（你最喜欢哪个）：<https://www.cnblogs.com/lanxuezaipiao/p/3451943.html>
5. Visual Studio 常用快捷键：<https://www.jianshu.com/p/e60543e14e7e>
6. VS2017常用快快捷键：<https://www.cnblogs.com/lsgxeva/p/7944986.html>
