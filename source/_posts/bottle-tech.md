---
title:  Bottle—快速实现小型服务接口
toc: true
description:  工作中有时候会临时部署一些接口服务，Django虽然很强大，但是太大了，本身也不需要多app，flask虽然有不少轮子，但是很多东西还要第三方库，最后发现bottle麻雀虽小五脏俱全，不仅小（只有一个文件），而且像错误码、路由等都包含了，比较适合快速实现小型服务接口。
date: 2018-11-05
category: 
 - 后台
tag:
 - python
 - 网络编程
---

### Bottle

![](http://t1.aixinxi.net/o_1crfolsf7vsd11tb9c4119fgqpa.png-w.jpg)

Bottle是一个简单高效遵循WSGI的Python Web框架，整个框架源码非常小，只有一个文件`bottle.py`，不依赖第三方库，只依赖Python标准库。和Django相比，bottle显示是mini了很多，不用配置复杂的setting和多app配置，相比flask而言，因为不用像flask会依赖非常多的第三方库，而且bottle源码很简短，可以阅读源码学习到flask的同理设计，深入理解HTTP服务器设计思路。

### 安装

- `pip install bottle`
- [Guide](https://bottlepy.org/docs/dev/)

### 特性

- Routing：把请求映射到函数，建立动态简洁的URLs
- Template：同很多Web框架一样，采用内置模板引擎，还支持mako,jinjia2等第三方模板
- Utilities：便捷读取表单，上传文件，cookie，HTTP头信息和其他HTTP相关的元数据
- Server: 内建HTTP开发服务器,并且支持past和很多其他WSGI服务器

### Hello Word

```python
# coding:utf8
from bottle import route, run, template, error, \
                    static_file, request, response, \
                    abort, HTTPError, install
from bottle_errorsrest import ErrorsRestPlugin

@route('/')
def index(name="index"): # 默认提交参数
    # 抛出异常，直接返回自定义的错误码信息，这个功能非常实用，在Django中实现比较麻烦，需要在抽到中间件，bottle只需要返回HTTPError
    raise HTTPError(status=600, body={"err_code": 1500, "message": "服务端抽了"})
    return {'code': 12, 'list': [1,234,12]}

@route('/test') # 灵活的路由
def test():
    # 正常返回
    return {'code': 12, "data": ["code", "2143", {"data": 123}]}

@route('/static/<filename>')
def download(filename):
    # 处理静态文件
    return static_file(filename, root='./tmp/', download=True)

install(ErrorsRestPlugin()) # 错误码插件
run(host='0.0.0.0', port=8090, debug=True, reloader=True) # 打开调试开关，并且打开自动diff-reload
```

bottle基本实现了常用的业务场景，通过`route`灵活配置urls，还提供了url的默认参数，返回值可以是dict对象，静态文件可以通过`static_file`实现下载，`HTTPError`实现错误返回。

更多的细节可以看官方Tutorial，可以说是非常详细了，传送门：[bottle tutorial](https://bottlepy.org/docs/dev/index.html)

后面在项目中用起来再介绍下其他高级功能，bottle源码麻雀虽小五脏俱全，非常值得阅读。期待自己后面出源码分析的文章。
