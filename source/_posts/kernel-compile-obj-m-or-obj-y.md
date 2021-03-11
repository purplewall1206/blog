---
title: 'kernel compile: obj-m or obj-y'
date: 2021-03-01 16:14:14
tags: [kernel, linux, compile]
---

# obj-m/obj-y
使用obj-m可以将文件编译成单独的.ko文件

使用obj-y可以将文件编译到内核的zImage中

使用obj-n表示不编译

内核中常见的还有 obj-$(SYM) 这种类型的，一般$(SYM)可以取值为y，m
