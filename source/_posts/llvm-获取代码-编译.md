---
title: llvm 获取代码+编译
date: 2021-04-03 15:39:08
tags: [llvm, compile]
---

# llvm 获取代码+编译


[swapfile](https://wiki.archlinux.org/index.php/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E4%BA%A4%E6%8D%A2%E6%96%87%E4%BB%B6)  
[llvm作者写的文档](http://www.aosabook.org/en/llvm.html)  
[llvm-doc](https://llvm.org/docs/GettingStarted.html#id17)  
[llvm-tutorial](https://llvm.org/docs/tutorial/index.html)  
[zhihu-编译llvm内存不够情况](https://zhuanlan.zhihu.com/p/36769900)  
[zhihu-编译llvm clang](https://zhuanlan.zhihu.com/p/67625228)  
[ClangBuiltLinux](https://clangbuiltlinux.github.io/)  
[]()  

# 获取源代码

```
git clone https://gitee.com/mirrors/llvm-project.git
git checkout llvmorg-12.0.0-rc2
<!-- 因为zzzq的原因master分支不让用了,本次编译了rc2 -->
```
github不加代理实在太慢了

编译出来结果至少有120GB，因此最好分配一个200GB的虚拟机

<!-- more -->

# 编译llvm

首先下载llvm源代码，然后进行编译，要注意llvm编译过程中耗费大量内存，因此需要额外进行处理。

1. 创建llvm-build文件夹，进入文件夹

2. cmake

根据官网文档，试试看另一种编译
```
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_BUILD_TYPE=Debug -DLLVM_TARGETS_TO_BUILD=X86  -DLLVM_USE_LINKER=gold -DLLVM_ENABLE_PROJECTS="clang;lld"   ../llvm/
```



这里
- DCMAKE_BUILD_TYPE 有Debug 和 Release因为我要后期开发，因此使用debug
- DCMAKE_INSTALL_PREFIX 表示安装的文件夹，这里显示是上层目录，即llvm-source-code目录下的install文件夹
- DLLVM_USE_LINKER=gold 使用gold链接器替换ld，据说能够节省大量内存
- DLLVM_TARGETS_TO_BUILD=X86  只编译x86相关的指令即可
- DLLVM_ENABLE_PROJECTS="clang;lld"  这里可以一起编译clang和lld

3. 创建一个10GB的swap分区

具体看[swapfile](https://wiki.archlinux.org/index.php/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E4%BA%A4%E6%8D%A2%E6%96%87%E4%BB%B6)  
```
fallocate -l 10G /swapfile
# dd if=/dev/zero of=/swapfile bs=1M count=10240（和上面那条指令等价）
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```
如果想保存swap分区可以编辑 `/etc/fstab` 增加这一条目
```
/swapfile none swap defaults 0 0
```

4. make
```
make ; make install
```

这里单线程，不要多线程，占用内存太多有崩溃风险



# 添加环境变量

```
vim ~/.bashrc

export PATH=$PATH:/home/x/Documents/llvm-project/install/bin
export LD_LIBRARY_PATH=/home/x/Documents/llvm-project/install/lib

source ~/.bashrc
```

目录中有 clang和llvm需要的一系列工具，但是没有ld.lld

# 编译linux内核

```
make LLVM=1 -j4

make CC=clang  -j4
```

注意这里不知道为什么没有ld.lld，可能需要单独安装一下。

如果想要使用llvm编译linux内核，需要查看一下**ClangBuiltLinux** 项目，这里面维护了几个所有编译都能通过的linux版本，其他版本则有这样或那样的可能导致编译失败。

[ClangBuiltLinux](https://clangbuiltlinux.github.io/)  

<!-- ```
cmake -DCMAKE_BUILD_TYPE=Debug --enable-optimized --enable-targets=host-only -DCMAKE_INSTALL_PREFIX=../install  -DLLVM_USE_LINKER=gold -G "Unix Makefiles" ../llvm

cmake -DCMAKE_BUILD_TYPE=Debug --enable-optimized -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=../install  -DLLVM_USE_LINKER=gold -G "Unix Makefiles" ../llvm
``` -->

<!-- # 编译clang

此处需要编译过llvm之后才能进行编译，因为clang的cmake需要依赖llvm的编译结果

1. 创建clang-build文件夹，进入文件夹

2. cmake
```
cmake -DCMAKE_BUILD_TYPE=Debug --enable-optimized --enable-targets=host-only -DCMAKE_INSTALL_PREFIX=../install  -DLLVM_USE_LINKER=gold -G "Unix Makefiles" ../clang

cmake -DCMAKE_BUILD_TYPE=Debug --enable-optimized -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=../install  -DLLVM_USE_LINKER=gold -G "Unix Makefiles" ../clang

```

3. make
```
make ; make install
```

如果资源不够不要多线程，这里被卡死过一次。 -->