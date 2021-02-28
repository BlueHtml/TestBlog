---
layout: post
title:  "消息推送服务-消息通知服务"
categories: 技术
tags: 消息推送
excerpt: 本文介绍下当前流行的消息推送服务：Server酱、Qmsg酱、WxPusher、企业微信api...
---

* content
{:toc}

## 前言

目前最流行和常用的是[Server酱](#server酱)（微信通知），如果需要qq通知的话可以使用[Qmsg酱](#qmsg酱)，其他的推送服务请自行摸索。

## Server酱

Server酱：<https://sc.ftqq.com/>

Github登录，然后绑定微信，调用接口发送消息，会发送到自己的微信。

C#使用示例见[这里](https://github.com/BlueHtml/Note163Checkin/blob/fb930a51d7f7577784803b6aae37214cfdd48334/Program.cs#L90)

- Title可以直接在卡片上显示，Body不显示，需要点击卡片才能查看。
- 成功调用返回结果：`{"errno":0,"errmsg":"success","dataset":"done"}`
- 失败调用返回结果：`{"errno":1024,"errmsg":"bad pushtoken"}`
- TalkAdmin：这个很有意思。
    - 上行命令：可以向公众号说话，服务端语音转文字，分析文字，执行预定义的命令（向某个地址发送请求之类的）。
    - 下行命令：原来的信息是只读的。而下行命令可以设置交互界面模板，它会自动把消息渲染到模板上，该模板可以进行交互。

## Qmsg酱

Qmsg酱：<https://qmsg.zendee.cn/>

调用接口，发送消息到qq或qq群

### 前置操作步骤

1. 自己添加[qq机器人](https://qmsg.zendee.cn/)为好友，有4个qq机器人
2. 在[后台](https://qmsg.zendee.cn/me)选择自己添加的机器人，然后添加**自己的qq号**或**qq群号**

    **群推送**功能需要申请开通，没有开通的话「Qmsg酱」小姐姐会拒绝进群的！可以在[这里](https://qmsg.zendee.cn/api)申请

### 调用接口

[API文档](https://qmsg.zendee.cn/api)

- qq私聊：`https://qmsg.zendee.cn:443/send/key?msg=tt`
- qq群：`https://qmsg.zendee.cn:443/group/key?msg=tt`

GET和POST均可。

**另有`qq`参数**：【**可选**】指定接收消息的QQ/QQ群，可以添加多个，以英文逗号分割（指定的QQ必须在您的**QQ号列表**中）

## WxPusher

WxPusher：<http://wxpusher.zjiecode.com/>

类似Server酱，但是WxPusher可以**群发**。

WxPusher必须要创建应用。也可以创建主题。
- 应用：【**单发/群发**】通过UID发送。支持多个UID
- 主题(Topic)：【**群发**】关注该主题的用户均可以收到消息

笔记如下：
- 任何接口都需要`appToken`。**`appToken`只会显示一次，忘了的话就只能重置了**
- 简单消息可以用GET，复杂消息需要使用POST。
    - 简单消息：只支持**纯文本**。就像**别人给你发一条文本消息**一样，**不是卡片**。
    - 复杂消息：内容支持文本、HTML、Markdown。
- 发送消息时，`uid`和`topicId`不能同时为空！
- **GET**发送消息时，其中content和url请进行urlEncode编码。例：`http://wxpusher.zjiecode.com/api/send/message/?appToken=abcd&topicId=660&content=TestMsg`
- 获取UID：`http://wxpusher.zjiecode.com/api/fun/wxuser?appToken=appToken`，详细参考[查询App的关注用户](http://wxpusher.dingliqc.com/docs/#/?id=%e6%9f%a5%e8%af%a2app%e7%9a%84%e5%85%b3%e6%b3%a8%e7%94%a8%e6%88%b7)
- 用户可以管理消息服务：`微信公众号`->`我的`->`我的订阅`，查看订阅，也可以进入某个订阅进行删除(退订)。
- 支持上行命令：`#{appID} {内容}`，例：`#97 test`。WxPusher会将指令消息回调给开发者（需要设置回调地址）

## 企业微信

管理后台：<https://work.weixin.qq.com/wework_admin/frame>

注册企业微信，创建应用，参考[微信实时推送验证码，真香！](https://post.m.smzdm.com/p/aoozp7pn/)。

注意：创建企业微信后，还需要邀请自己加入，然后在微信收到消息后，会自动加入的（仅限自己）！貌似延迟挺高的。。。

可以在网页上进行测试和操作：菜单栏`管理工具`->`素材库`：发消息、上传文件...

### 通用

- 基本概念(术语)介绍：<https://work.weixin.qq.com/api/doc/90000/90135/90665>
- **企业id**：主菜单栏`我的企业`->拉到最下面`企业ID`
- **应用id和Secret**：主菜单栏`应用管理`->拉到最下面`自建`->点击应用->`AgentId`和`Secret`

#### 访问频率限制

访问频率限制：<https://work.weixin.qq.com/api/doc/90000/90139/90312>

- 上传图片频率: 每个企业，每天最多可以上传**100**张永久图片
- 发送应用消息频率
    - 每企业不可超过帐号上限数*30人次/天（注：若调用api一次发给1000人，算1000人次；若企业帐号上限是500人，则每天可发送15000人次的消息）
    - 每应用对同一个成员不可超过**30条/分**，超过部分会被丢弃不下发
    - 发消息频率不计入基础频率

#### access_token

请求任何操作接口，都需要使用access_token。2小时有效期，需要定时刷新。

- 请求方式：GET（HTTPS）
- 请求URL：https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=企业id&corpsecret=应用Secret

#### 返回errcode

**企业微信所有接口，返回包里都有errcode、errmsg**。**调用成功时，errcode为0**。

开发者需根据errcode是否为0判断是否调用成功(errcode意义请见全局错误码)。而errmsg仅作参考，后续可能会有变动，因此不可作为是否调用成功的判据。

API支持：
- 发送消息/媒体文件给指定用户
- 接收用户在应用里的操作(消息)，把消息发送给外部地址，进行交互。

### 发送消息

支持消息的格式如下：
文本（含超链接）
音频/视频（媒体文件需要先上传）
图文
交互页面
卡片

#### 语法

- 请求方式：POST（HTTPS）
- 请求地址: https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=ACCESS_TOKEN

##### 请求体

- touser: 指定为”@all”，则向该企业应用的全部成员发送，此时将忽略 toparty, totag 参数
- **agentid: 企业应用id**

示例：
```json
{
   "touser" : "@all",
   "msgtype" : "text",
   "agentid" : 1000002,
   "text" : {
       "content" : "我就试一下"
   },
   "safe":0
}
```

### 接收消息

> 接收消息与事件: <https://work.weixin.qq.com/api/doc/90000/90135/90237>


开启接收消息模式后，用户在**应用里发送的消息**会推送给企业后台。此外，还可配置地理位置上报等事件消息，当事件触发时企业微信会把相应的数据推送到企业的后台。

### 上传文件

> 素材管理: <https://work.weixin.qq.com/api/doc/90000/90135/91054>

也可以上传临时素材，其media_id仅**三天内**有效

**上传的媒体文件限制**：

所有文件size必须大于5个字节
- 图片（image）：2MB，支持JPG,PNG格式
- 语音（voice） ：2MB，播放长度不超过60s，仅支持AMR格式
- 视频（video） ：10MB，支持MP4格式
- 普通文件（file）：20MB

## sre24

sre24：<https://sre24.com/>

可以在[日志](https://sre24.com/logstat)里看到消息记录

## Core

- 企业微信机器人 for .NET: <https://github.com/dotnet-campus/dotnetCampus.WeChatWork>
