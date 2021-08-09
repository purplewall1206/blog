---
title: arm memory tagging extension
date: 2021-08-09 15:42:16
tags: [arm, hardware]
---


[MTE技术在Android上的应用](https://zhuanlan.zhihu.com/p/353807709)  
[kernel - AArch64 TAGGED ADDRESS ABI](https://www.kernel.org/doc/html/latest/arm64/tagged-address-abi.html)  
[Armv8.5-A Memory Tagging Extension white paper](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/enhancing-memory-safety)  

<!-- more -->

![arm-mte](./arm-mte.png)

之前我只把这玩意当做kasan的硬件实现，其实不对，因为他提供了一个lock + key的匹配

比如给一个地址分配可读写权限，这意味着

1. 相应的地址，`addr = 0x-fff800000000000`,的开头四位("-")记录了key
2. 对应的shadow memory  `tagaddr =（addr >> 4）+ offset` 标记了lock（未来可能就不需要了，直接MTE提供shadow）

lock和key必须匹配才能读写，否则触发异常，这个异常可以配置成同步的，也可以是异步的。

忽略开头的四位地址的技术在arm中叫 TBI（top byte ignoring）

相应的技术有kasan  asan hwasan这类的sanitizer，以及android tagged pointers等

类似的还可以参考loki（用fpga模拟的mte/tbi），如果你想做论文的话，需要优化loki这类工作，或者绕开这类工作。

此外内核已经提供了MTE接口，可以研究一下。
