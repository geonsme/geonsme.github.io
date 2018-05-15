Title: debian下crontab执行时间不准确
Category: Linux
Tags: crontab debian
Slug: debian-cron-time
Authors: geons
Summary: crontab和cmos时间统一
Date: 2017-11-19 16:19:31
Modified: 2017-11-19 16:19:36
## 遇到问题
重启了VPS后发现crontab定时任务执行时间不正确，并且与北京时间相差了8个小时，很明显是crontab定时任务依据的时间不是UTC时间，因此需要将BIOS时间和系统时间同步，

## 初步解决
```shell
#date
Mon Jun 25 14:34:18 CST 2012
#date -u
Mon Jun 25 06:34:29 UTC 2012
```


显示的本地时间和UTC的时间都是正常的，但是显示我那系统crontab执行的时间是按照UTC时间来了，于是我再次设置了下时区为国内的，记得我之前好像设置过了，不过还是尝试一下,删除原来的时区文件，又重新复制到本地

```shell
rm /etc/localtime
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```


然后vi /etc/localtime 查看了下时区文件，确实是CST-8,再向crontab里添加一个测试脚本，重启cron
```shell
/etc/init.d/cron restart
```


发现它依然没有按照国内的时间来执行，之前也没遇到过问题，而且同样的设置，目前另外的机子上也正常执行，最后才想起我这个用的是debian系统，**debian系统下面，仅是设置/etc/localtime是不够的，更加需要的是/etc/timezone这个文件**。

## 使用tzconfig

最后用了tzselect程序来设置时区，运行tzselect命令后，按照自己要的时间选择选项，最后选1保存确认即可。再次重启cron，添加测试任务，这次终于按照本地时间运行了，如果不行就重新登录一下或者重启下。

## 使用hwclock
使用**sudo hwclock**修改硬件时间，设置时间同步

## 相关资料

- [Debian下系统时间比正常时间快8小时的问题](https://www.tech1024.cn/reprint/1618.html)