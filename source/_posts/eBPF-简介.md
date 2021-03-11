---
title: eBPF 简介
date: 2020-11-15 19:17:06
tags: eBPF
categories:
- eBPF
---

# eBPF

   * [eBPF](#ebpf)
      * [1. 简介](#1-简介)
      * [2. 参考材料](#2-参考材料)
      * [3. in-kernel virtual machine](#3-in-kernel-virtual-machine)
      * [4. JIT just-in-time compile](#4-jit-just-in-time-compile)
      * [5. kprobe 实现原理](#5-kprobe-实现原理)

[ebpf summit 2020峰会](https://ebpf.io/summit-2020/) 这个有时间应该看一下

## 1. 简介

eBPF由BPF（berkeley packet filter发展而来，BPF现在也叫classic-BPF由于区别BPF），到目前为止被引入了linux内核并且使用了JIT just in time comilper进行加速，类似于浏览器的javascript脚本执行语言。

eBPF定义了一个内核内运行的虚拟机

![ebpf-arch](ebpf-arch.png)

使用bcc进行bpf的开发，bcc和bpftrace项目的维护放在 **iovisor**
<!-- more -->
## 2. 参考材料

1. [A thorough introduction to eBPF](https://lwn.net/Articles/740157/)
2. [A JIT for packet filters](https://lwn.net/Articles/437981/)
3. [BPF: the universal in-kernel virtual machine](https://lwn.net/Articles/599755/)
4. [LECTURE BPF: Tracing and More](https://www.youtube.com/watch?v=JRFNIKUROPE)
5. [TUTORIAL Learn eBPF Tracing: Tutorial and Examples](http://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html)
6. [TUTORIAL The bpftrace One-Liner Tutorial](https://github.com/iovisor/bpftrace/blob/master/docs/tutorial_one_liners.md)
7. [TUTORIAL bpftrace Reference Guide](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md)
8. [Linux内核攻击面之eBPF模块](https://www.anquanke.com/post/id/220047)


## 3. in-kernel virtual machine

Things started to change in the 3.0 release, when Eric Dumazet added a **just-in-time compiler** to the **BPF interpreter**. In the 3.4 kernel, the **"secure computing" (seccomp)** facility was enhanced to support a user-supplied filter for system calls; that filter, too, is written in the BPF language.

## 4. JIT just-in-time compile

Eric Dumazet's patch is a fundamental change: it puts a just-in-time compiler into the kernel to **translate BPF code directly into the host system's assembly code.** The simplicity of the BPF machine makes the JIT translation relatively simple; every BPF instruction maps to a straightforward **x86 instruction sequence**. 

## 5. kprobe 实现原理

这里仅做猜测，应该是直接使用kprobe技术，int3 打断点到指定位置，截取控制流到相应的probe callback 函数。


