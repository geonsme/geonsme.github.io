---
title:  ab——一款太真实的压测工具
toc: true
description:  ab压测是一个低成本的，快速方便的测试
date: 2018-11-07
category: 
 - 工具
tag:
 - ab压测
---

## 基本术语

- 吞吐率（Requests per second）

服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下`单位时间内能处理的最大请求数`，称之为最大吞吐率。
计算公式：总请求数 / 处理完成这些请求数所花费的时间，即Request per second = Complete requests / Time taken for tests

- 并发连接数（The number of concurrent connections）

某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

- 并发用户数（The number of concurrent users，Concurrency Level）

要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

- 用户平均请求等待时间（Time per request）

处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

- 服务器平均请求等待时间（Time per request: across all concurrent requests）

处理完成所有请求数所花费的时间 / 总请求数，即Time taken for / testsComplete requests。也叫做`平响`。可以看到，它是吞吐率的倒数。因此，平响越低，吞吐率越高。同时，它也=用户平均请求等待时间/并发用户数，Time per request / Concurrency Level

## 测试HTTP接口 

选了一台闲置的测试机，作为压测服务器，向`https://codemart.com/codemart.com/api/developer/rank`接口

发起GET压测，分析压测数据。

1. 安装ab工具

```bash 
Ubuntu安装ab工具
$ apt-get install apache2-utils
```

2. ab压测命令

```bash
测试总次数为1000，并发数为100，相当于100个用户同时访问，总共访问次数是1000次
$ ab -n 1000 -c 100 'https://codemart.com/api/developer/rank'
```

3. 测试结果

```bash
This is ApacheBench, Version 2.3 <$Revision: 1757674 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking codemart.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests

# 服务器相关信息

Server Software:        Nginx
Server Hostname:        codemart.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256
TLS Server Name:        codemart.com

Document Path:          /api/developer/rank
Document Length:        710 bytes

Concurrency Level:      100 # 并发数
Time taken for tests:   14.978 seconds # 压测花销总时长
Complete requests:      1000 # 总并发数
Failed requests:        241 # 失败的请求
   (Connect: 0, Receive: 0, Length: 241, Exceptions: 0)
Non-2xx responses:      759
Total transferred:      2514523 bytes
HTML transferred:       2209743 bytes
Requests per second:    66.76 [#/sec] (mean) # 平均每秒发起的并发数
Time per request:       1497.824 [ms] (mean) # 所有并发用户都请求一次的平均时间，用户平均等待时间
Time per request:       14.978 [ms] (mean, across all concurrent requests) # 非常重要：每个请求响应时间，等于用户平均等待时间除请求总数，计算出单个请求响应时间
Transfer rate:          163.94 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       91  434 684.5    155    7402
Processing:    29  574 1304.6     34   14725
Waiting:       22  571 1305.4     32   14723
Total:        121 1008 1474.1    424   14821

Percentage of the requests served within a certain time (ms)
  50%    424
  66%    883
  75%   1161
  80%   1635
  90%   2794
  95%   3707
  98%   5052
  99%   7427
 100%  14821 (longest request)
```

4. 结果解读

- 在并发100情况下，平响为14.978ms（一定要说明并发量，没有并发量的QPS就是耍流氓）
- 事务失败率24.1%




