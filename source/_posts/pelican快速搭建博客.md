---
title: Pelican+GithubPage搭建静态博客
toc: true
description: 十分钟使用pelican搭建静态博客，pelican也是除了hexo外用的比较多的静态博客框架，由于是Python和jinja2写的，可维护性比较高
category: 
 - 前端
tag:
 - pelican
 - 静态博客
---



## 准备工作
&emsp;在国内，Pelican的知名度和没有国外那么高，因此也没有几篇像模像样的教程，本文介绍Pelican+GithubPage进行快速博客搭建的方法，Windows和Linux平台差别不大，文章以Windows平台。

## 安装Pelican
&emsp;在Windows下搭建pelican环境可以说完爆Jekyll,如果熟悉Python，甚至比Hexo还要简单，闲话不多说，直接开干。首先安装Python环境，我安装的是Python2.7，如果你需要Python3.6.1，也可以安装，没有任何问题。建议你先安装pip。
```python
    easy_install pip
```
&emsp;安装完Python后，使用pip很方便安装pelican和markdown。
```bash
$ pip install pelican
$ pip install markdown
```
Now，你的Pelican环境已经搭建好了，So Ez？

## 创建博客
&emsp;在一个空白的文件夹里面打开cmd，使用命令快速建立博客模板。
```python
    pelican-quickstart
```
&emsp;进入quickstart配置过程后，这些设置会生成对应的pelicanconf.py配置文件和Makefile文件，不过Windows下由于不能使用Makefile，所以这里即使填错选项也可以待会在修改。因此，这个你可以根据你的实际需要完成。完成后可见如下文件目录。
```python
    ./content # 博文目录
    ./output # html生成目录
    ./fabfile.py # Deploy设置相关
    ./Makefile # 自动构建目录
    ./pelicanconfig.py # 配置文件
    ./publishconf.py # 发布时配置文件
```
&emsp;注意的是publishconf.py完全导入pelicanconf.py文件，所以前者用于本地调试使用，后者发布时自动替换某些属性，例如SITEURL，不过由于我们不使用Makefile，所以直接使用pelicanconf.py文件即可，它的配置方法比较多，针对不同的主题模板有不同的配置方法，这里不能给出具体的配置，初次搭建，可以不用任何配置。

&emsp;fabfile.py里面包括各种Deploy相关的配置和操作，可以在里面修改默认的本地服务器端口（80），一般来说都不需要修改。你也可以在Deploy时候修改各类参数，都没有任何问题。

## 编写文章
&emsp;我们首先写一篇HelloPelican文章然后生成发布到本地瞧瞧是怎么个样子。上文说过content目录是用来存放博文等文件目录的，直接在目录里面新建一个文件test.md(本文使用MarkDown来编辑，pelican也支reStructuredText)。
```txt
Title: HelloPelican
Date: 2017-10-26
Category: test
Tags: test
Slug: blog/test
Author: geons

##First
Text
```
&emsp;注意：Title等字段用来声明博客的属性，Slug为对应生成html的相对路径，如果上述最终生成路径是blog目录下的hello.html,这些字段还可以自定义，这个进阶内容可以后面再说。出去属性声明外的部分都是正文内容，##First就是正文的开始了。你可以用markdown随心所欲编写你的内容。

## 生成静态HTML
&emsp;发布后的博客其实是一个单纯如少女的HTML静态文件，所以，这样很安全的，你不用担心数据库安全。使用如下命令进行HTML静态文件生成。生成好的文件在output目录，因此，output目录就是我们要发布的文件。
```python
    pelican content
```
&emsp;生成成功后，在当前目录下，你可以使用下面命令启动server，通过浏览器可以访问到你的页面，你可以自定义端口，也可以空缺，默认采用80端口访问。
```python
    pelican -m pelican.server 80
```
如果没有任何的报错信息，服务器已经启动成功，可以通过http://127.0.0.1:80进行访问。

## 静态资源文件
&emsp;通过上文，我们已经成功添加第一篇博客，但是很快会发现，如果你往content目录里面添加一个images文件夹存放博文的图片，你会发现`pelican content`并不会复制images文件夹到output目录下。这种不需要编译但又要用到的文件，我们称它为“静态文件”。pelican默认不会复制静态文件到output目录，需要我们在pelicanconf.py配置文件上面配置一下,添加一行：
```python
    STATIC_PATHS = ['images']
```
&emsp;这样就会生成output资源时就会自动把iamges文件夹拷贝到output目录了。另外使用EXTRA_PATH_METADATA也可以把某个目录的文件映射过去，例如favicon.ico放在content/extra目录下，最后需要生成到output的根目录，可以添加：
```python
    STATIC_PATHS = ['images', 'extra/favicon.ico']
    EXTRA_PATH_METADATA = {
        'extra/favicon.ico': {'path': 'favicon.ico'}
}
```
&emsp;说到这里，你可能会想，static目录岂不是会持续增量，当然，我也有一个不错的方案告诉你，用七牛云的数据存储空间，进行静态文件的存储，在访问速度和安全性上都是不错的，七牛云的服务我会在下篇文章介绍。

## 自动渲染
&emsp;由于Win下不能使用makefile，因此不能使用`make html`命令进行生成。因此，写个批处理文件，实现渲染和重启服务器。脚本代码如下。
```
@echo off
setlocal enabledelayedexpansion
for /f "delims=  tokens=1" %%i in ('netstat -aon ^| findstr "8000"') do (
    set a=%%i
    goto job
)
:job
taskkill /F /pid "!a:~71,5!"
pelican content
cd output
start cmd /c "python -m pelican.server 8000"
cd ..
```
&emsp;上面的代码内容是找到已经运行的服务器程序，结束它，然后重新生成Output文件，并且重新打开本地服务器。如果要修改端口可以替换bat文件中的端口号。有了这个脚本，以后需要更新预览，只需要在博客目录下打开cmd，输入auto-update.bat，敲一下回车就能自动完成了，或者你可以直接双击启动，很放ez吧。

## 开始发布
&emsp;关于GithubPage的搭建我不做详细介绍，因为资料很多，你只需要简单的通过GitHub Doc能够快速搭建自己的github.io，我们只需要将output目录下的文件提交到github.io即可，外网就可以通过访问xxx.gethub.io访问到你的博客主页。很简单，把Github.io项目拉下来，用Output目录里面的内容替换掉，push上去刷新就能看到了。不过这里需要注意的是是否配置了RELATIVE_URLS这个相对路径设置，SITEURL也要设置成Pages的地址，否者Feed的xml地址将显示不完全，编译的时候也会提示： "WARNING: Feeds generated without SITEURL set properly may not be valid"。所以这些都要手动检查清楚后再发布。

> 我猜，你已经想拥有自己的域名，那就更简单了，仍然按照gihub doc进行DNS解析，注意一定访问官方提供的方法，因为解析IP可能有变动。  

## 最后的话
通过以上的步骤，至少本地已经可以访问到博客了，因此，Pelican门槛是非常非常低的，非常便利。作者提供一些优化思路。

1. 七牛云静态资源存储
2. 博客目录和发布目录分两个仓库
3. 增加评论功能
4. 阅读主题模板源码，修改模板
5. 阅读Pelican源码，部署到Nginx上提升性能


## 参考文章
&emsp;[Pelican官方文档](http://docs.getpelican.com/en/3.7.1/index.html)



