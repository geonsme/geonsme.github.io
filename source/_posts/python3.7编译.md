---
title: Python3.7生产环境
toc: true
date: 2019-3-15
description: 项目一直推进Python3的使用，很多生产服务器上自带的Python2已经不能用了，需要通过源码安装Python3.7，编译安装过程有比较多隐藏的坑点，这里分享Debian7上安装Python3的过程。
category: 
 - python
tag:
 - 源码
---

## 查看Python版本

```bash
$ python --version # 查看当前使用的版本
$ whereis python
$ which python
```

## 基本流程

1. 安装编译需要的第三方包

2. Python官网下载Python3.7源码到tmp目录（任意目录）

3. 解压源码

4. 带参数配置configure

5. 执行make

6. 链接python3和pip3

## 第一步

安装Python3编译所需要的包

```bash
$ sudo apt-get install build-essential -y
$ sudo apt-get install libncurses5-dev libncursesw5-dev libreadline6-dev -y
$ sudo apt-get install libdb5.3-dev libgdbm-dev libsqlite3-dev libssl-dev -y
$ sudo apt-get install libbz2-dev libexpat1-dev liblzma-dev zlib1g-dev -y
$ sudo apt-get install libssl-dev openssl -y
```

注意：在安装openssl后通过`openssl version`查看openssl`版本`，python3.7版本没有带ssl库，对openssl的版本要求是1.0.2或者1.1.x，apt安装的版本过低，编译出的pip3是无法进行https访问下载包，因此，升级openssl。

```bash
# 1.下贼openssl
wget https://www.openssl.org/source/openssl-1.1.1a.tar.gz
cd openssl-1.1.1a
# 2.编译安装
./config --prefix=/usr/local/openssl no-zlib #不需要zlib
make
make install
# 3.备份原配置
mv /usr/bin/openssl /usr/bin/openssl.bak
mv /usr/include/openssl/ /usr/include/openssl.bak
# 4.新版配置
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/local/lib64/libssl.so
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
# 5.修改系统配置
## 写入openssl库文件的搜索路径
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
## 使修改后的/etc/ld.so.conf生效 
ldconfig -v
# 6.查看openssl版本
openssl version
```

## 第二步

下载Python源码并解压

```bash
$ wget --no-check-certificate https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tgz
$ tar xzvf Python-3.7.2.tgz # 解压到当前目录
$ cd Python-3.7.2
$ ./configure --prefix=/usr/local/python3.7  --with-openssl=/usr/local/openssl # openssl文件夹指向第一步安装的openssl目录
$ make all
$ sudo make install
```

`install`完成后，成功的话会看到以下输出：

```bash
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-7.1.2 setuptools-18.2
```

这里表示pip也已经装好，只是目前还在`/usr/local/python3.7/bin`目录下，需要软链到`/usr/bin`目录下。

## 第三步 为当前用户添加执行路径

```bash
$ vim ~/.bashrc
最后一行添加
export PATH=$PATH:/usr/local/python35/bin，如果有多个，请用英文冒号间隔
$ source ~/.bashrc # source生效
```

## 第四步 软链到系统默认目录

```bash
$ sudo ln -s /usr/local/python35/bin/python3.7 /usr/bin/python3 # 将系统命令python3指向刚刚安装的python
$ sudo ln -s /usr/local/python35/bin/pip3 /usr/bin/pip3 # 将pip3指向刚刚安装的pip
```

完成后，可以通过`python3`和`pip3 install requests`等测试。

## 第四步 多版本并存

使用`virtualenv .env -p /usr/bin/python3`指定python版本

```bash
$ pip install virtualenv
$ virtualenv .env -p [python版本]
```

整个编译过程比较坑的就是`make all`过程可能不会报错，由于某些第三方库不齐全导致编译出Python不是完整的，建议可以通过`make test`检查，并且确保`openssl`版本比较新，如果失败了，一定先执行`sudo make clean`，重新`make`一次。

参考资料

- [python3.7安装， 解决pip is configured with locations that require TLS/SSL问题](https://blog.csdn.net/lkgCSDN/article/details/84403329)

- [Debian7编译python3](https://www.jianshu.com/p/977b51cee0fa)




















