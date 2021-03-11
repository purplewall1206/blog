---
title: Linux User-mode Driver
date: 2020-11-28 15:18:10
tags: [kernel, Driver, Linux]
---

## UIO userspace i/o

[20200702Make the user mode driver code a better citizen](https://lwn.net/Articles/825190/)   
[20040120User-space device drivers](https://lwn.net/Articles/66829/)   
[tldp userspace device driver](https://tldp.org/LDP/khg/HyperNews/get/devices/fake.html)   
[20161019Linux drivers in user space — a survey](https://lwn.net/Articles/703785/)  
[UIO: user-space drivers](https://lwn.net/Articles/232575/)   
[The Userspace I/O HOWTO](https://www.kernel.org/doc/html/latest/driver-api/uio-howto.html?highlight=uio)

<!-- more -->

UIO因为不支持DMA已经被vfio取代了,zd 2.6.22被引入

UIO的目的，对于有些设备来说创建Linux kernel driver有点多余了，这些设备只是需要处理中断和提供对device内存区域的访问，例如industrial I/O card。

UIO不完全排除内核空间代码的使用，需要一个小的模块负责device设置，PCI总线接口、注册interrupt handler等

相关数据结构在`linux/uio_driver.h`,在sysfs和/dev/uioX 导出用户接口，使用dev的接口进行设备控制，mmap映射设备的寄存器或者RAM，通过读dev/uioX处理中断（可以通过select），每个UIO设备可以创建一个或多个内存区域能够进行内存映射，因为一些industrial I/O card在driver中需要访问超过一个PCI内存区域

UIO驱动是char drivers，没有block和network drivers，不能在用户态设置DMA

![usermodedriver-UIO](usermodedriver-UIO.gif)

## VFIO
[vfio-kernel document](https://www.kernel.org/doc/Documentation/vfio.txt)   
[VFIO core framework](https://lwn.net/Articles/473234/)   
[VFIO](https://lwn.net/Articles/499240/)   
[vfio: expose virtual Shared Virtual Addressing to VMs](https://lwn.net/Articles/815745/)   
[Safe device assignment with VFIO](https://lwn.net/Articles/474088/)   
[IOMMU & iomap &VFIO & uio](https://www.cnblogs.com/yi-mu-xi/p/10515609.html)  
[VFIO - "Virtual Function I/O"](https://www.kernel.org/doc/Documentation/vfio.txt)

VFIO 把内核态的和IO相关的内存区域、IO port和DMA暴露给用户态，可以用KVM做设备直通，也可以构建用户态driver。也是通过 /dev/vfio暴露接口，通过IOCTL系统调用发送操作device的command

在VFIO中虚拟机直通的最小单元不再是单独的设备，而是同一group里面的所有设备，而仅仅通过IOMMU设置直通则无法处理多个有联系的PCI设备的相互访问。因此VFIO集成了IOMMU和UIO的优点。

![usermodedriver-vfio](usermodedriver-vfio.png)

## DPDK Data Plane Development Kit
[DPDK介绍](https://www.jianshu.com/p/86af81a10195)   
[Network acceleration with DPDK](https://lwn.net/Articles/725254/)   


是一种intel使用intel硬件提供的用户空间的网络驱动解决方案。用于快速数据包处理的函数库与驱动集合，可以极大提高数据处理性能和吞吐量，提高数据平面应用程序的工作效率。

DPDK使用了轮询(polling)而不是中断来处理数据包。在收到数据包时，经DPDK重载的网卡驱动不会通过中断通知CPU，而是直接将数据包存入内存，交付应用层软件通过DPDK提供的接口来直接处理，这样节省了大量的CPU中断时间和内存拷贝时间

 DPDK defines an execution environment which contains user-space network drivers.

具体优点是使用了HUGEPAGE和numa，CPU亲和等硬件技术，加快运行速度，还有使用了UIO或VFIO技术直接暴露设备内存通过zero copy的方式减少系统调用和复制数据开销。

![dpdk](usermodedriver-dpdk.PNG)

## SPDK  storage performance development kit
给NVME-ssd设计的高速工具包，intel提供的使用intel网络、处理、存取技术，充分发挥固态存储的优势，采用了UIO和polling，将驱动代码运行在内核态，多路复用取代阻塞I/O。

其中也涉及不少实现细节，但总之是当下速度比较快的一种I/O工具。实现思路和DPDK一样，也是取消get/put_user，mmap直接干。


## reference
[Linux drivers in user space — a survey](https://lwn.net/Articles/703785/)   
[User-space device drivers](https://tldp.org/LDP/khg/HyperNews/get/devices/fake.html)   
[User-space device drivers](https://lwn.net/Articles/66829/)   
[Make the user mode driver code a better citizen](https://lwn.net/Articles/825190/)
