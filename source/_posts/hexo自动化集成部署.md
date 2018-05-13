---
title: hexo+github+travis实现博客持续集成、发布
toc: true
---

之前网站是用Pelican+GithubPage，由于整体风格不是自己最喜欢的简约风，花了一个周末改成hexo，并集成Travis CI功能，实现一键发布，自动部署。本文主要介绍hexo安装部署，travis ci使用，githubPage由于网上资料很多，略过。本文环境是Debian8，Windows大同小异。

## Hexo安装

### 安装nodejs，npm，git

基于Debian安装node，按照官网安装即可。如果通过apt-get直接安装的话，版本比较低，后面安装会错误，建议安装最新版nodejs。

``` bash
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs
$ sudo apt-get install -y build-essential
```

More info: [nodejs官网](https://nodejs.org/en/download/package-manager/)

安装cnpm，由于国内网络原因，主题中的某些库需要淘宝镜像，采用cnpm进行安装。

```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

More info: [cnpm安装](https://npm.taobao.org/)

安装git

```bash
$ sudo apt-get update
$ sudo apt-get install git
```

安装hexo

```bash
$ mkdir web ; cd web
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo g # hexo generate简写，生成public目录
$ hexo s # hexo server会启动dev server
```

安装Maupassant主题和渲染器

```bash
$ git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
$ npm install hexo-renderer-pug --save
$ npm install hexo-renderer-sass --save
```

安装完成后，编辑Hexo目录下的 **_config.yml** ，将 theme的值修改为maupassant。

注：若npm install hexo-renderer-sass安装报错或者无进度，修改成cnpm安装。

关于主题的配置项，请参考maupassant作者博客进行配置修改

More info: [大道至简——Hexo简洁主题推荐](https://www.haomwei.com/technology/maupassant-hexo.html)

至此，在blog目录中执行 **hexo s**，通过IP加端口可以访问到调试博客页面。下一步就是将博客进行发布。

### Github Page
待续
