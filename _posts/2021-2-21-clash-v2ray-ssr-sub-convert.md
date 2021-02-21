---
layout: post
title:  "clash-v2ray-ssr订阅互转"
categories: 技术
tags: clash v2ray ssr 科学上网
excerpt: 本文介绍下clash-v2ray-ssr的订阅怎么互转，以及如何搭建自己的subconverter和sub-web。
---

* content
{:toc}

## 公开api

- subconverter: <https://subcc.herokuapp.com/version>，**自己搭建，不要使用别人的，会看到链接的**
- sub-web：<https://sub-web.opo.repl.co/>

## subconverter-后端

> 注意，订阅转换不代表协议转换，vemss节点不可能转成ssr，但是可以转成clash节点和ssr节点在一个软件中同时使用

- subconverter：<https://github.com/tindy2013/subconverter>
- 文档：<https://github.com/tindy2013/subconverter/blob/master/README-cn.md>

简单用法看[这里](https://github.com/tindy2013/subconverter/blob/master/README-cn.md#简易用法)，`target`参考[支持类型](https://github.com/tindy2013/subconverter/blob/master/README-cn.md#支持类型)的最后一列`参数`。注意：`原始订阅url`必须要进行**url转码**！

转换结果不太妙：
- clash -> ssr：失败（原因见后面）
- v2ray -> ssr：失败（原因见后面）
- ssr -> clash：成功

注意：
- v2ray -> ssr：失败！为什么？因为ssr客户端只支持ss和ssr，它是不能处理v2ray的！所以转换失败！
- clash -> ssr：失败！为什么？同样的原因！clash节点 大部分也都是 v2ray节点，ssr客户端是不能处理的！
- ssr -> clash：成功。为什么？因为clash是后来出的客户端，它支持ss、ssr、v2ray、clash，clash客户端能支持这些协议，所以能转换。

### 部署到heroku

subconverter-heroku：<https://github.com/BlueHtml/subconverter-heroku>

步骤如下：
1. fork[subconverter-heroku](https://github.com/BlueHtml/subconverter-heroku)项目
2. 添加Secret：`HEROKU_API_KEY`和`HEROKU_EMAIL`
3. 修改`heroku.yml`里的`heroku_app_name`的值
4. 点击`Actions`->点击`heroku`->点击`Run workflow`
    
部署后访问`/version`，如果出现`subconverter v版本号 backend`说明部署成功。

## sub-web-前端

- sub-web：<https://github.com/CareyWang/sub-web>

sub-web是VUE开发的一个网页，没有后台，不要担心隐私泄露（*subconverter后端*是在页面上手动输入的）

可以从[action的Artifacts](https://github.com/CareyWang/sub-web/actions)下载最新版，解压即可使用。

推荐使用[render](https://render.com/)来部署：创建*Static Sites*，输入github项目地址，设置生成文件目录为`dist`，自动部署后即可。

注意：绑定的链接必须是**根路径**，否则网页不展示！！
