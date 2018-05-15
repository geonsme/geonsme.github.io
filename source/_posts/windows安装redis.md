---
title:  Redis Windows64位安装
toc: true
description:  Redis官方并不提供win下的安装包，github上有win64Redis项目可以实现redis在win下的使用。
date: 2017-11-26
category: 
 - 后端
tag:
 - redis
---

## Redis(Win64)下载地址
官方是不支持win版的，不过Microsoft Open Tech group 在Github上开发了一个win64Redis项目，地址是[https://github.com/MSOpenTech/redis](https://github.com/MSOpenTech/redis)

## Redis安装
msi格式安装，Redis会以service形式运行，整个目录结构是
```bash
redis-benchmark.exe // 基准测试
redis-check-aof.exe // aof
redis-check-dump.exe // dump 
redis-cli.exe   // redis客户端
redis-server.exe // redis服务端
redis.windows.conf // 配置文件
```

## 基本命令
### Redis服务启动
从命令行以服务形式启动Redis
```bash
redis-server redis.windows.conf
```

或者使用
```bash
redis-server --service-start
```

### 停止Redis服务
```bash
redis-server --service-stop
```

## 卸载Redis服务
```bash
redis-server --service-uninstall
```

## 指定Redis服务
```bash
redis-server --service-install -server-name myRedisName -port 10001

redis-server --service-start -server-name MyRedisName -port 10001

redis-server --service-stop -server-name MyRedisName -port 10001
```