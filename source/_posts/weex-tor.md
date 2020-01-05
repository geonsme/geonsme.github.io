---
title: weex入门踩坑指南
toc: true
description: 为什么要入门weex，因为我发现v2ex网站没有app，每次获取信息都要打开网页等等操作很不方便，因此先把后端接口用python写好，还缺个app，为了美观和开发效率，花了一天入门weex，由于文档的不齐全，踩了很多坑。
date: 2019-02-28
category: 
 - 前端
tag:
 - weex
 - webapp
---

## weex介绍

weex是阿里开源的基于通用平台web开发来构建Android、iOS和Web应用的客户端框架，通过集成WeexSDKh后可以使用主流的Web技术实现移动应用开发。

## 准备环境

- 安装Nodejs，直接官网安装最新版本

- 安装weex-toolkit

在安装weex时候，比较坑的是如果不是科学上网，可能在装modules时候失败，并且失败后第二次安装不会报任何错误，所以在安装前，一定要准备好上网环境，全局科学。

```bash
$ npm install weex-toolkit -g
```

- 安装weex其他模块

第一次运行weex命令会出现```a process is running```的错误提示，使用-f 强制进行安装

```bash
$ weex -f
```

## 开始运行

- 初始化

```bash
$ weex create v2ex-app
$ cd v2ex-app
$ npm install
$ npm start
```

工程运行起来后，本地会启一个8081端口（动态），通过打开```http://localhost:8081```查看页面在Web下的渲染效果，也可以通过```http://localhost:8081/preview.html```查看渲染效果，通过扫描二维码，手机上实时显示设备。

## 编译运行

- 目前只踩了Android平台的坑，如果App要编译打包到Android平台上，首先在项目文件夹中添加android项目目录。

## 安卓打包

- 准备Android SDK

最终demo打包出来是apk，因此，需要Android SDK和Java环境。这一步是非常多坑点的，包括SDK版本，环境变量等。

```bash
1. 下载android studio，链接https://developer.android.com/studio

2. 通过android studio安装SDK，SDK版本不建议过高，通过Tools-Android SDK Manager可以安装对应的SDK版本，一般与android真机的版本匹配即可。

3. 添加环境变量。打开“环境变量”，在“系统变量”中新建SDK_HOME变量，并将SDK目录添加进去。

4. 编辑Path变量。保证现有变量值后面有一个分号“;”，然后添加%SDK_HOME%\tools;%SDK_HOME%\platform-tools;

5. 测试下，在cmd中用android -h，应该有输出
```

- 准备Java环境

1. 下载jdk，链接：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

2. 配置环境变量。

```bash
1. 点击“系统变量”下面的”新建“选项，新建JAVA_HOME，“变量值”为JDK安装路径，笔者的路径是”D:\Program Files\Java\jdk1.8.0_91”

2. 在“系统变量”中找到“Path”，增加“%Java_Home%\bin;%Java_Home%\jre\bin;”

3. 在“系统变量”栏，“新建”，“变量名”为“CLASSPATH”，“变量值”为“.;%Java_Home%\bin;%Java_Home%\lib\dt.jar;%Java_Home%\lib\tools.jar”，“确定”

4. 检查java环境。在Cmd中输入java、javac、java -version
```

- 运行与打包

```bash
$ npm run android，安卓真机打开真机调试模式
```

打包的结果在```v2ex-app\platforms\android\app\build\output```






