---
title: Raspberry Pi 内核编译
date: 2020-11-16 20:06:10
tags: [kernel, raspberryPi]
categories:
- kernel
- raspberryPi
---

**我的版本 : 树莓派3**



# 1.内核获取
## 1.1.软件下载
`sudo apt install git bc bison flex libssl-dev make`

<!-- more -->

### 关于libssl-dev的小问题
```
sudo apt-get install aptitude

dpkg -l *libssl*

sudo aptitude install libssl-dev
```


## 1.2.源码获取
`git clone --depth=1 https://github.com/raspberrypi/linux`



# 2.内核编译

预设环境

```
cd linux
KERNEL=kernel7
make bcm2709_defconfig
或者 zcat /proc/config.gz > .config   #从现在原有版本的内核中复制配置文件
```

编译
```
make -j4 zImage modules dtbs
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

# 3.更换内核

修改 `/boot/config.txt` 文件中内容

`kernel=kernel-myconfig.img`

myconfig的版本在`/lib/modules/` 文件夹中。

# 4.参考文件

[DOCUMENTATION > LINUX > KERNEL > BUILDING](https://www.raspberrypi.org/documentation/linux/kernel/building.md "building")




