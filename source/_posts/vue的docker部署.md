---
title: 用Docker实现Vue前端持续集成开发（上）
toc: true
description: vue的部署流程之一就是webpack的打包，用daoCloud实现docker持续集成，可以快速安全构建最新和回滚版本，大幅提升了前端版本管理效率。
date: 2017-12-02
category: 
 - 前端
tag:
 - vue
---

## 持续集成
传统讲来，持续集成无非就是将业务随着提交不断地进行构建、集成并部署到测试环境或生产环境，过去都是用脚本代替人工进行构建，在回滚和事务隔离上是致命弱点。因此，有了基于Docker的持续集成概念，本文以现学现卖的VUE例子，抛弃传统的部署方式，用容器技术快速打造一个开发部署环境，最引人注目的是在传统的事务回滚上加上了环境回滚，将版控提升了一层，个人觉得很好用。

## 流程
- 代码在本地编写后提交到Github
- 触发分支构建
- 自动部署到服务器上
- 版本可视化管理，支持环境回滚

## 初始化Vue项目

### 使用vue-cli初始化项目

Vue-cli脚手架的具体用法请移步[Vue脚手架-手摸手系列](http://geons.me/fast-study-vue.html)

```bash
vue init webpack vue-docker-demo
```

### 在项目的根目录下编写Dockerfile
![dockerfile文件](http://oyc3sy7c4.bkt.clouddn.com/vue-docker-dockerfile.png)
```docker
# 使用Node精简版作为基础镜像
FROM node:6.10.3-slim

# 安装nginx
RUN apt-get update \    
    && apt-get install -y nginx

# 指定工作目录
WORKDIR /app

# 当前目录拷贝到工作目录下
COPY . /app/

# 声明运行时容器提供服务端口
EXPOSE 80

# 安装依赖
# build
# 拷贝到ng目录下
# 删除目录文件
RUN  npm install \
     && npm run build \     
     && cp -r dist/* /var/www/html \     
     && rm -rf /app

# 以前台形式启动ng
CMD ["nginx","-g","daemon off;"]
```

### 推送到github上
> 单独建立一个仓库，这样便于维护和部署
至此，我们的开发代码已经成功版本控制了，接下来是使用Devops流程

## 使用DaoCloud搭建Devops流程
> 这里只是举例使用DaoCloud，其他公有云平台也可以做到的，只是这个对个人独立开发者比较OK

### 注册一个DaoCloud并授权Github仓库
- 用户中心
- 代码托管
- 授权访问Github仓库
> 注册流程就不详细说了，关键一点是授权Github仓库，也就是你源码管理的仓库，这样就可以直接关联到你的GitHub。本人在授权过程中，忽略了一个问题，因为github之前授权给了很老的一个号，是不能同时授权的，所以幸好找回了老号码接触了绑定，希望大家不要犯这样的低级错误。

### 在Devops中新建一个项目
新建后选择对应的github项目，也就是vue-docker-demo所在的仓库。

### 构建镜像版本，准备好创建一个应用
- 选择“手动触发”
- 触发方式选择master分支
![创建镜像](https://user-gold-cdn.xitu.io/2017/11/21/15fdf12d8a14e5ea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 镜像构建成功后可在镜像仓库中查看
![镜像](https://user-gold-cdn.xitu.io/2017/11/21/15fdf152e156f10d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 部署到云端测试环境或者自己主机中
Daocloud提供了胶囊主机，可以做临时的开发测试使用，如果是正式开发环境建议购买主机，将主机接入集群中，进行统一管理。
官网提供了很完善的自有主机接入教程，这里不再详细展开
![主机](https://user-gold-cdn.xitu.io/2017/11/21/15fdf0e8b7f97804?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
接入主机后，可以在面板看到集群中已经有1台主机正常运行。
![主机](https://user-gold-cdn.xitu.io/2017/11/21/15fdf1d36fd5a210?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 创建第一个应用！
> 进入镜像仓库，选择刚刚构建出来的镜像进行部署，应用和镜像的关系可以查看Docker官方文档区分。

![](https://user-gold-cdn.xitu.io/2017/11/21/15fdf1c416ab9539?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/11/21/15fdf1f270b7eb89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 接下来的时间就是静候镜拉取，容器启动

### 外网访问我们的Vue应用！
可以看到刚刚部署的应用已经成功了！
点击容器卡的端口，选择外网访问的入口，即可外网访问到应用
![](http://oyc3sy7c4.bkt.clouddn.com/vue-docker-daocloud.png)

> 这里不仅可以很直观查看应用的状况，还能够对整个容器进行监控，还有云隧道、迁移、发布等选项，很方便的完成负责的环境部署等等。

## 自定义构建和发布
请移步[用Docker实现Vue前端持续集成开发（下）]()

## 总结
从整个流程来看，我们没有关心任何生产环境的配置等等，专注的编写好vue业务，然后就让外网用户访问到了，只花了半小时时间，足见docker进行开发部署的方便，当然，这是一个单点应用Demo，没有涉及到负责的nginx配置，负载均衡，日志Col，灰度发布等，在后面的文章中会介绍docker更多的用处。