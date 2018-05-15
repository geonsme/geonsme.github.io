Title: Redis Windows64位安装启动
Category: 后台
Tags: redis
Slug: redis-win
Authors: geons
Summary: 专注后端的前端工程师
Date: 2017-11-26 16:30:04
Modified: 2017-11-26 16:30:10

## Redis(Win64)下载地址
官方是不支持win版的，不过Microsoft Open Tech group 在Github上开发了一个win64Redis项目，地址是[https://github.com/MSOpenTech/redis](https://github.com/MSOpenTech/redis)

## Redis安装
msi格式安装，Redis会以service形式运行，整个目录结构是
```shell
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
```shell
redis-server redis.windows.conf
```

或者使用
```shell
redis-server --service-start
```

### 停止Redis服务
```shell
redis-server --service-stop
```

## 卸载Redis服务
```shell
redis-server --service-uninstall
```

## 指定Redis服务
```shell
redis-server --service-install -server-name myRedisName -port 10001

redis-server --service-start -server-name MyRedisName -port 10001

redis-server --service-stop -server-name MyRedisName -port 10001
```