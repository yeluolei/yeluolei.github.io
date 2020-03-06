---
title: Ubuntu安装 Realtek R8125/R8168 驱动
date: 2020-03-07 00:31:28
tags: ubuntu, driver
---

如今网卡更新的速度已经超过操作系统的更新速度了，特别是开源操作系统，默认的网卡驱动动不动就跟不上新的主板，导致新的电脑无法识别到网卡，特别闹心。比如最近配置的`Z390 Phantom Gaming SLI/ac`这个主板，在ubuntu 18.04上，就是安装了`ubuntu-18.04-hwe-generic`，也没有找到对应的驱动程序，这个时候就需要手动安装了。在[官网](https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software)上下载好对应的驱动,注意我们是`2.5G Ethernet LINUX driver r8125 for kernel up to 4.15`的版本，不要下错了。如果是1G网卡，则对应`GBE Ethernet LINUX driver r8168 for kernel up to 4.15`版本。

下载好之后，当前版本是`r8125-9.003.02.tar.bz2`。里面有README文件，如果按照README安装，当时能够使用，但是重启或者升级内核版本之后，就失效了。所以还需要手动处理下。

* 准备编译环境
```
sudo apt-get install --reinstall linux-headers-$(uname -r) linux-headers-generic build-essential dkms
```
* 解压对应的源码到`/usr/src`
```
sudo tar xvf r8125-9.003.02.tar.bz2 -C /usr/src
```
* 添加一个`dkms.conf`到`/usr/src/8125-9.003.02/dkms.conf`，内容如下
```
PACKAGE_NAME=Realtek_r8125
PACKAGE_VERSION=9.003.02

DEST_MODULE_LOCATION=/updates/dkms
BUILT_MODULE_NAME=r8125
BUILT_MODULE_LOCATION=src/

MAKE="'make' -C src/ all"
CLEAN="'make' -C src/ clean"
AUTOINSTALL="yes"
```
* 编译DKMS
```
sudo dkms add -m r8125 -v 9.003.02
sudo dkms build -m r8125 -v 9.003.02
sudo dkms install -m r8125 -v 9.003.02
sudo depmod -a
sudo modprobe r8125
```

* 验证安装结果, 运行如下命令即可看到`enxxx`的有线网接口
```
ifconfig -a
```