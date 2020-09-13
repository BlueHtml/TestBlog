---
layout: post
title:  "SQL Server中LIKE和PATINDEX的用法"
categories: 数据库
tags: mssql
author: 过千帆
excerpt: 在SQL Server中，能使用通配符的只有2个：LIKE、PATINDEX。本文记录下LIKE、PATINDEX的用法，以及通配符的使用和转义。
---

* content
{:toc}

在SQL Server中，能使用通配符的只有2个：LIKE、PATINDEX。

不过**LIKE**支持2种通配符转义，无限制最全面；而**PATINDEX**只支持最简单的通配符转义(`[]`转义)，限制较多。

# LIKE

**LIKE** 是逻辑运算符，能使用通配符，并且支持2种方法来转义通配符。

## 语法

```
match_expression [ NOT ] LIKE pattern [ ESCAPE escape_character ]
```

`[ XX ]`代表是可选的，所以可以没有**转义表达式`ESCAPE escape_character`**。

## 参数

* match_expression

    任何有效的字符数据类型的表达式。

* pattern

    要在 match_expression 中搜索并且可以包括下列有效通配符的特定字符串。 pattern 的最大长度可达 8,000 字节。
    
    * `%`：包含零个或多个字符的任意字符串。

    * `_`（下划线）：任何单个字符。

    * `[ ]`：指定范围 ([a-f]) 或集合 ([abcdef]) 中的任何单个字符。**数字**同样支持*范围*或*集合*([0-9]或[0123456789])。
    
        > 在范围搜索中，范围包含的字符可能因排序规则的排序规则而异。

    * `[^]`：**不属于**指定范围 ([a-f]) 或集合 ([abcdef]) 的任何单个字符。数字同样支持。

* escape_character

    放在通配符之前用于指示通配符应当解释为常规字符而不是通配符的字符。 escape_character 是字符表达式，无默认值，并且计算结果必须仅为一个字符。

## 结果类型

Boolean

## 结果值

如果 match_expression 与指定的 pattern 相匹配，则 LIKE 返回 TRUE。

## 备注

### 何时忽略尾随空格

* pattern里**有空格**时必须匹配！
* pattern里**无空格**时会**忽略**match_expression的零个或多个**尾随**空格。
* **此时应该注意`char`、`nchar`类型，因为不足长度时会自动添加尾随空格，有可能出现问题**。

测试sql如下：

```sql
--pattern里有空格时必须匹配
select 1 where '123' like '123 ';
--无

--pattern里无空格时会忽略match_expression的【尾随】空格
select 1 where '123 ' like '123';
--1

--pattern里无空格时【不会】忽略match_expression的【前置】空格
select 1 where ' 123' like '123';
--无
```

### 否定的几种形式

#### 相等

`'xx' NOT LIKE 'yy'` **=** `NOT 'xx' LIKE 'yy'`

#### 不相等

`NOT LIKE 'dm%'` **!=** `LIKE '[^d][^m]%'`

* `NOT LIKE 'dm%'`：结果可以是：da、tm...
* `LIKE '[^d][^m]%'`：会排除掉**所有以 d 开始或第二个字母为 m 的名称**。

    > 这是因为用**反向通配符**匹配字符串是分步骤进行计算的，一次一个通配符。如果在计算过程中**任一环节**匹配失败，那么就会将其消除。

### 通配符的转义

通配符当做普通字符来使用，有2种方式。

#### 使用`[]`

`[]`这种方式只适合简单的情况，有很多限制，不推荐。

能转义的字符有：百分号 (%)、下划线 (_) 和左括号 ([) ，其他的转义不了，或者只能特殊情况下使用(例如短横线`-`只有在首位时才被视为普通字符)。如果强行使用`[]`来转义会导致结果错误！！推荐使用`ESCAPE`子句来进行转义。

#### 使用`ESCAPE`子句

`ESCAPE`子句本来就是为转义而生，推荐使用。

`ESCAPE`子句的转义字符是自定义的，定义为某字符，pattern里就使用这个转义字符来转义即可。

相关sql如下：

```sql
--转义【%】
select 1 where '%123' like '\%123' ESCAPE '\'
--1

--转义【_】
select 1 where '123_' like '123\_' ESCAPE '\'
--1

--转义【[^]】
select 1 where '123[^]' like '123\[\^\]' ESCAPE '\'
--1

--自定义其他转义字符：%
select 1 where '123[^]%_' like '123%[%^%]%%%_' ESCAPE '%'
--1
```

# PATINDEX

**PATINDEX** 属于字符串函数，支持通配符，**转义通配符只能使用`[]`**，所以限制颇多。

**PATINDEX** 返回模式在指定表达式中第一次出现的起始位置(**从1开始**)；如果在所有有效的文本和字符数据类型中都找不到该模式，则返回零。

## 语法

```
PATINDEX ( '%pattern%' , expression )
```

## 参数

* pattern

    包含要查找的序列的字符表达式(**和`LIKE`的pattern参数一样**)。可以使用通配符；但 pattern 之前和之后必须有 % 字符（搜索第一个或最后一个字符时除外）。 pattern 是字符串数据类型类别的表达式。 pattern 最多包含 8000 个字符。

* expression

    是一个表达式，通常是针对指定模式搜索的列。 expression 属于字符串数据类型类别。

## 返回类型

如果 expression 的数据类型为 varchar(max) 或 nvarchar(max)，则返回 bigint；否则返回 int。

## 结果值

返回模式在指定表达式中第一次出现的起始位置(**从1开始**)；如果在所有有效的文本和字符数据类型中都找不到该模式，则返回零。

## 备注

* 如果 pattern 或 expression 为 NULL，则 PATINDEX 返回 NULL。
* PATINDEX虽然支持通配符，但是**转义通配符只能使用`[]`**！所以限制颇多。
* 在 PATINDEX 中可以使用 `COLLATE` 函数显式指定要搜索的表达式的排序规则。示例sql如下：

    ```sql
    USE tempdb;
    GO
    SELECT PATINDEX ( '%ein%', 'Das ist ein Test'  COLLATE Latin1_General_BIN) ;
    GO
    ```

# 参考

1. LIKE (Transact-SQL)：<https://docs.microsoft.com/zh-cn/previous-versions/sql/sql-server-2012/ms179859%28v%3dsql.110%29>
2. PATINDEX (Transact-SQL)：<https://docs.microsoft.com/zh-cn/previous-versions/sql/sql-server-2012/ms188395(v%3dsql.110)>
