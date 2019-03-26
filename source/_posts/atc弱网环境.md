---
title:  ATC弱网测试环境搭建
toc: true
date: 2019-03-25
description: ATC企业级弱网测试工具相比传统的Newt工具具有明显的灵活性和可配置性，适合大规模测试，针对现在移动应用APP引入弱网测试工具，这里详细介绍ATC搭建步骤，整体来说分为三部分，准备、开热点、开atc
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

## 安装ATC

### 安装ATC库

ATC依赖Python、Linux环境，Raspberry已经支持Python环境

```bash
sudo pip install atc_thrift atcd django-atc-api django-atc-demo-ui django-atc-profile-storage
sudo pip install django
```

### 创建atc django工程

```bash
django-admin startproject atcui
cd atcui
```

修改`settings.py`中的INSTALL_APPS

```bash
INSTALLED_APPS = (
    ...
    # Django ATC API
    'rest_framework',
    'atc_api',
    # Django ATC Demo UI
    'bootstrap_themes',
    'django_static_jquery',
    'atc_demo_ui',
    # Django ATC Profile Storage
    'atc_profile_storage',
)
```

编辑`url.py`

```bash
from django.views.generic.base import RedirectView
from django.conf.urls import include

urlpatterns = [
        url(r'^admin/',admin.site.urls),
        url(r'^api/v1/',include('atc_api.urls')),
        url(r'^atc_demo_ui/',include('atc_demo_ui.urls')),
        url(r'^api/v1/profiles/',include('atc_profile_storage.urls')),
        url(r'^$',RedirectView.as_view(url='/atc_demo_ui/', permanent=False)),
    ]
```

执行migrate，更新django数据库

`python manage.py migrate`

### 启动atc服务

```bash
sudo atcd --atcd-lan wlan0
# 进入atcui目录
python manage.py runserver 0.0.0.0:8000
```

### 使用facebook提供的网络配置文件

```bash
apt-get install git
git clone https://github.com/facebook/augmented-traffic-control.git
apt-get install curl
cd augmented-traffic-control
utils/restore-profiles.sh localhost:8000
```

配置后重启`server`，访问`http://host:8000`即可访问到ATC配置页面。

### 启动流程

```bash
sudo ifconfig wlan0 172.0.0.1
# 启动hostapd
sudo hostapd -B /etc/hostapd/hostapd.conf
sudo service hostapd restart
sudo service udhcpd restart	
sudo atcd --atcd-lan wlan1
nohup sudo python manage.py runserver 0.0.0.0:8000 > atcdj.log  2>&1 &
```

### 参考资料

[在树莓派上部署ATC网络模拟工具（Augmented Traffic Control）](https://www.jianshu.com/p/0a10ead567c3)

[FaceBook ATC 弱网测试工具环境搭建](https://www.jianshu.com/p/fb4824fd5bbc)