---
layout: post
title:  "使用PHP空间和天翼云盘搭建私人云盘"
categories: 技术
tags: 云盘 php
excerpt: 本文主要介绍下怎么使用PHP空间和天翼云盘搭建私人云盘。
---

* content
{:toc}

> **更新**：天翼云盘的Key已失效，作者也删库了，默哀。。。

## 前言

天翼云盘有2种方式做私人云盘：
- 用户登录后提取Cookie
- OpenAPI(开放平台授权)

下面主要介绍下**OpenAPI**方式。

## 原理

**OpenAPI**是使用开放API来开发的，需要先进行用户授权获取到token，程序使用Token来访问用户数据并展示。

## 部署

- 项目地址：[TCShare](https://github.com/xytoki/TCShare)
- 部署指南：[TCShare：云盘目录列表，支持天翼云](https://xylog.cn/2020/04/19/tcshare.html)

官方推荐使用**虚拟主机、宝塔**等，因为需要执行命令来还原php依赖，而PHP空间通常是不支持执行命令的。

实际上php空间也是可以搭建的，只要**把依赖一同上传了就行**。下面就介绍下怎么部署到php空间。

## 部署到php空间

部署到php空间的关键是**把php依赖一同上传到空间**。

`还原php依赖`很简单，只需要执行官方给出的命令(`composer install`)就行。

可以在本地还原，但是为此安装php环境就很麻烦了。这里介绍下github action方式：使用github action还原依赖，并把生成后的文件使用FTP上传到php空间。

### 0. 部署准备

PHP环境推荐7.x，服务器为Apache

### 1. fork项目+添加Secrets

fork[TCShare项目](https://github.com/xytoki/TCShare)->`Settings`->`Secrets`，添加下面3项：
- `FTP_SERVER`：PHP空间的FTP地址(`ftp://xxx.xx`)
- `FTP_USERNAME`：FTP账户
- `FTP_PASSWORD`：FTP密码

### 2. 创建php.yml

创建并编写`php.yml`：` Actions`->`Set up a workflow yourself`或`New workflow`->在`Edit new file`文本输入框中`Ctrl+A`全选，删除掉默认代码，复制粘贴下面的yaml代码->右上角点击`Start commit`->`Commit new file`

yaml代码如下：
{% raw %}
```yml
name: PHP Composer

on:
  push:
    branches: [ master ]
  pull_request:
    types: [closed]
  issue_comment:
    types: [created, edited, deleted]
  watch:
    types: [started]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Install dependencies
      run: composer install

    - name: Create ftp-include file
      run: echo "!_app/" > .git-ftp-include
      
    - name: List output files
      run: ls

    - name: FTP-Deploy-Action
      uses: SamKirkland/FTP-Deploy-Action@3.0.0
      with:
        ftp-server: ${{ secrets.FTP_SERVER }}
        ftp-username: ${{ secrets.FTP_USERNAME }}
        ftp-password: ${{ secrets.FTP_PASSWORD }}
        git-ftp-args: --remote-root public_html
```
{% endraw %}

**要注意下面几点：**
- **触发条件**。已经把触发条件设为`push`/`pull request`/`issue comment`/`started`，最简单的触发方式是**started(`Star`)**。
- **FTP文件夹**。默认会上传到`public_html`文件夹下。可以通过yaml代码中的`git-ftp-args: --remote-root {上传文件夹}`来修改。

### 3. 还原依赖+上传文件

选择一种方式触发Action(推荐`Star`)，触发后会进行下面2种操作：
1. 还原依赖(执行命令`composer install`)
2. FTP上传到php空间

注意：如果设置了上传文件夹并且**文件夹不是网站根目录**，在Action执行完毕后，我们还需要手动把该文件夹下面的所有文件都移动到**网站根目录**下。

### 4. 创建env文件

`.env`文件是配置文件，配置好后需要**上传到网站根目录下**。

`.env`文件配置说明如下：
```
#   XS 是前缀
#   | -KEY 是配置种类，可选KEY，APP，SEC
#   | | - -ct是key的ID（类似config.php）
#   | | - | - something是配置名称
#   | | - | - | - - - - value在等号右边
#   XS_KEY_ct_something=value

    XS_KEY_ct=ctyun   #必填，值为ctyun
    XS_KEY_ct_FD=     #应用文件夹名
    XS_KEY_ct_AK=     #AK
    XS_KEY_ct_SK=     #SK

#   这里APP后面的可以是任意值，一般就123456下去
#          ↓
    XS_APP_1=/              #挂载路径
    XS_APP_1_NAME=TCShare   #网盘名称
    XS_APP_1_THEME=mdui     #界面主题
    XS_APP_1_BASE=/         #网盘内路径
    XS_APP_1_KEY=ct         #对应上面Key的ID
```

配置key的格式：`{前缀}_{配置种类}_{key的ID}_{配置名称}`。

配置说明如下：
- 应用文件夹名：【**必填**】必须和应用名一样！
- AK：【**必填**】App Key
- SK：【**必填**】App Secret
- 挂载路径：链接里的路径，默认是`/`(主域名)
- 网盘名称：展示在页面上的名称，默认是`TCShare`
- 界面主题：默认是`mdui`
- 网盘内路径：天翼云盘的应用文件夹名下的路径，默认是`/`(应用文件夹)
- 对应上面Key的ID：默认是`ct`(ct是key的ID)

如果需要多盘挂载，可以在上述配置之后继续追加，但是要修改`key的ID`，相关配置也要修改。示例参考：[TCShare：云盘目录列表，支持天翼云](https://xylog.cn/2020/04/19/tcshare.html)

`.env`配置文件编辑好后，需要**上传到网站根目录下**。

### 5. 天翼云盘创建文件夹

登录天翼云盘，在`我的应用`下面创建和**应用名**一样的同名文件夹，并向里面添加一些文件。

### 6. 访问链接进行授权

打开网站链接，点击`Click here to authorize`，会跳转到授权页面，同意授权，授权成功后就大功告成了。

注意：
- 如果在`.env`里修改了`挂载路径`，网站链接也要做对应修改，才能正确授权。
- 如果跳转到授权页面后返回`{"errormsg":"InvalidSignature","success":false}`，可能原因是`AK`或`SK`填写错误。
- 授权成功后，如果网页一片空白，可能是网站还没反应过来，等一会再刷新网页就好了...

### 7. 每月续期token

token有效期是1个月，到期后需要访问`http://xx.xxx/-renew`，重新授权。

注意：
1. 授权是需要手动登录天翼云的，**计划任务是没用的**。
2. 可以**提前**重新授权，授权有效期为1个月
3. 如果授权后出现错误，需要看网站是否使用了CDN，暂时取消CDN再进行授权。

### 8. OneDrive版

V3版本支持Onedrive 国际版和世纪互联，在配置上稍有不同，本人未做测试故不再说明。

### 9. 和彩云

和彩云按照[官方文档](https://github.com/xytoki/TCShare#%E5%92%8C%E5%BD%A9%E4%BA%91%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F)配置即可。这里主要说几个注意点：
1. Token可以访问**和彩云内的全部文件**，所以一定要设置好**网盘内文件夹`XS_APP_<id>_BASE=暴露文件夹`**！小心别流出私人文件了！
2. Token有效期较长，个人测试发现暂无办法使Token立即失效（退出不行；退出重新登录再获取新Token也不行）
2. 如果网盘内文件过多，获取目录的耗时就会变长，就可能会报错：`cURL error 28: Operation timed out after 5000 milliseconds with 245520 out of 267672 bytes received (see https://curl.haxx.se/libcurl/c/libcurl-errors.html) (0)`。

个人不推荐使用和彩云作为公共云盘，如果必须要使用的话，建议和彩云里不要存放任何私人文件，防止失误而被公开。

### 10. 更新

[TCShare项目](https://github.com/xytoki/TCShare)更新较快，同步更新有2种方式：
- 删除掉项目->重新fork->重新部署（本人常用）
- 把官方项目Pull requests到个人的fork项目上（未测试）

## 更多功能

### 展示markdown文件内容

- 支持展示`header.md`、`head.md`、`readme.md`文件，会被展示在*文件路径*(顶)和*文件列表*(底)之间。
- md文件展示后会被**隐藏**。

### WebDav服务器

支持作为WebDav服务器，可以挂载到本地（目前仅为只读）。不过都用了云盘，WebDav服务器就没啥必要了吧。。

## 尾声

演示站见下（使用的国外PHP空间，速度较慢，请耐心等待加载完成）:
- ~~天翼云~~
- ~~和彩云~~