Title: Debian或者Ubuntu内核更新教程
Category: 后台
Tags: debian, kernel, ubuntu
Slug: linux-update-kernel
Authors: geons
Summary: 专注后端的前端工程师
Date: 2017-12-09 23:50:40
Modified: 2017-12-09 23:50:50

## 为什么更换内核
由于Linux内核版本更新快，可能向后兼容的问题经常出现，需要更换底层内核来实现环境安装，比如Docker的内核要4.0以上，本文将Debian/Ubuntu的更换内核方法做详细讲解

## Debian的准备工作
添加软件包源并更新列表
```shell
echo -e "\ndeb http://ftp.debian.org/debian/ wheezy-backports main" >> /etc/apt/sources.list
apt-get update
```
使用root权限操作
```shell
su root
```

## 安装内核

### 查询可用内核列表

```shell
aptitude search linux-image | awk '{print $2}'
```

执行代码后可以看到很多内核版本，Debian 7 x64 系统选择 linux-image-3.2.0-4-amd64 内核，这个内核大部分 Debian 7 系统都是使用。而Ubuntu 14.04 则选择 linux-image-3.13.0-32-generic 内核。

Debian和Ubuntu唯一的区别就是这里提示的内核名称不一样，和下面安装内核用的命令略有区别，其他都通用！

### Debian安装内核
```shell
apt-get -t wheezy-backports install linux-image-3.2.0-4-amd64 -y
```

### Ubuntu安装内核
```shell
apt-get install linux-image-3.13.0-32-generic -y
```

## 卸载内核
### 查看当前系统内核
```shell
dpkg -l|grep linux-image | awk '{print $2}'
```

```shell
# VPS提示示例/64位 #
# Debian 7 和 8 可能不一样，还有 64位和32位 内容也不一样。
 
root@debian:~# dpkg -l|grep linux-image | awk '{print $2}'
linux-image-3.2.0-4-amd64
linux-image-4.10.1-041001-generic
 
# Ubuntu 14.04 提示内容 #
linux-image-3.13.0-32-generic
linux-image-4.10.1-041001-generic
```

### 卸载其余内核
```shell
apt-get purge 其余内核名字 -y
# 示例 #
apt-get purge linux-image-4.10.1-041001-generic -y
# 示例 #
```

## 更新grub系统引导文件并重启
```shell
update-grub
reboot
```

## 注意
在更换内核前，需要检查自己的VPS是否支持内核更新，KVM or Openvz，对于Openvz类型的不要手贱随意，可能更新后机器就只有重装解决了，具体请咨询你的主机服务商。

## 参考文章
[逗比根据地](https://doub.io/linux-jc6/)

[Dbian7更换内核版本](https://www.cmsky.com/debian-upgrade-kernel/)
