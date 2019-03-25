---
title:  ATC弱网测试环境搭建
toc: true
date: 2019-03-25
category: 
 - 工具
tag:
 - 测试技术
---

## 准备工作

1. 树莓派3B，电源线，保护壳，散热片（显卡和CPU散热），风扇（可选），TF卡（32GB或者64GB），读卡器（烧录系统使用）

2. 下载debian系统[下载地址，下载Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/)、win32 Disk Imager烧录下载[下载地址](http://www.onlinedown.net/soft/110173.htm)

3. Linux驱动支持的AP，一定要选可用的网卡，否则无能为力。

## 第一步 安装树莓派系统

1. 通过win32 Disk Imager将raspberry系统烧录到SD卡，并且烧录后，打开SSH开关。开启方式是在系统目录中新建一个名叫ssh的空白文件，即可开启ssh服务。

2. raspberry通过xshell连接，端口为22，用户名为pi，密码是raspberry，第一次进入系统后请修改，并且设置root密码。

3. 设置密码完成后，设置使用整个SD，通过`raspi-config`的`Expand Filesystem`扩展SD卡上的可用空间。扩展后可通过`df -h`查看各分区大小。

4.  其他非必须的初始化配置可查看[这里](http://www.cnblogs.com/abel/p/3441175.html)

## 第二步 树莓派pi3设置wifi热点

### 安装dnsmasq hostapd

```bash
$ apt-get install dnsmasq hostapd
```

编辑`sudo vim /etc/dhcpcd.conf`文件，为wlan0固定内网IP，如果是外部的ap，则为wlan1.

```bash
interface=wlan0
static ip_address=172.0.0.1/24
```

### 编辑 dnsmasq配置文件

编辑sudo vim /etc/dnsmasq.confdnsmasq配置文件，有600+行的内容，我们可以shift+g滚动到最下面，然后加入下面的命令，用来控制IP地址的取值范围

```bash
interface=wlan0
dhcp-range=172.0.0.2,172.0.0.50,255.255.255.0,12h
```

### 添加hostapd配置文件

```bash
sudo vim /etc/hostapd/hostapd.conf
ssid=PI3                 #账号
wpa_passphrase=12345678  #密码
interface=wlan0
driver=nl80211
hw_mode=g
channel=10
macaddr_acl=0
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP
```

### 编辑sysctl.conf文件

设置ipv4路由转发，打开相应的开关。

```bash
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

### 更新iptables规则

注意wlan0其实不是固定的，如果使用wlan1发热wifi，这里应该是wlan1

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

编辑`sudo vim /etc/network/interfaces`，加入

```bash
up iptables-restore < /etc/iptables.ipv4.nat
```

### 重启发射热点

```bash
sudo service hostapd start 
sudo service dnsmasq start 
```

### 安装ATC

`稍后上架`

### 参考资料

[在树莓派上部署ATC网络模拟工具（Augmented Traffic Control）](https://www.jianshu.com/p/0a10ead567c3)
