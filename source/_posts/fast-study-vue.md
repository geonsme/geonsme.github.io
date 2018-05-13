---
title: 十分钟把Vue跑起来
toc: true
category: 
 - 前端
tag:
 - vue
---

## Vue新特性
Vue的特别之处在于抛弃了DOM操作思维，Vue.js是数据驱动的，任何适合都是根据数据来保持视图的更新，当然，在使用时候，完成一些简单的业务处理也是需要jQuery或者js。

本文主要是Vue-cli，俗称Vue的脚手架介绍，快速入门Vue开发。Vue-cli是vue工程开发的脚手架，可以自动生成vue模版，并且和webpack紧密结合一键式完成vue整个项目的开发环境部署。

![](http://oyc3sy7c4.bkt.clouddn.com/upload_vue_mvvm.png)

## vue-cli使用

- 安装nodejs环境
> [Nodejs下载地址](https://nodejs.org/en/)

- 全局安装vue-cli脚手架
> npm install vue-cli -g

- 使用webpack初始化项目
> vue init webpack my_project

- 进入项目文件夹，使用npm安装依赖
> cd my_project

- 安装依赖（根据package.json安装）
> npm install 

- 完成后的完整目录（具体的文件作用请阅读参考）

``` python
./my_project
 ./build ## 打包发布的参数配置
 ./config ## webpack打包参数配置
 ./src ## 开发源代码
 ./static ## 静态文件
 ./test ## 其他配置文件
 ./dst ## 打包好的文件目录，只有一个index和打包好的js
```

- 开发环境运行
> npm run dev
此时打开浏览器localhost:8080即可看到主页

- build到开发环境，生成的文件在dst目录里面
> npm run build

注意这里比较坑的是，默认的引用路径需要修改，进入config/index.js文件，修改为assetPublicPath:'./'，设定为当前目录

## 总结
整个vue-cli就基本上完成了，不过在实际开发中可能还会根据不同的情况调整不同的参数。

## 有用的资料
- [史上最简单的 Angular2教程](https://gold.xitu.io/post/5860eebe1b69e6006ce1395c)
- [webpack](https://doc.webpack-china.org/guides/)
- [Vue官网](https://cn.vuejs.org/v2/guide/comparison.html)
- [VueJs技术文档](http://www.cnblogs.com/keepfool/p/5619070.html)
- [webpack入门](http://blog.guowenfh.com/2016/03/24/vue-webpack-01-base/)
- [vue-cli脚手架使用](https://segmentfault.com/a/1190000009291545)
- [ECS6新特性](http://es6.ruanyifeng.com/#docs/intro)
- [前端编码规范](http://codeguide.bootcss.com/)
- [Python爬虫技术](http://cuiqingcai.com/1052.html)
- [Django源码阅读方法](https://www.the5fire.com/the-way-to-explore-django-source-code-by-the5fire-part1.html)
- [Django源码分析](https://github.com/daoluan/decode-Django)
