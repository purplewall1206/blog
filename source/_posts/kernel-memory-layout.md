---
title: kernel memory layout
date: 2020-12-11 21:20:41
tags: [linux, kernel]
---


[latest mm documentation](https://www.kernel.org/doc/html/latest/x86/x86_64/mm.html)
[latest boot documentatino](https://www.kernel.org/doc/html/latest/x86/boot.html?highlight=layout)
[x86: fix return from kernel to userspace with 16-bit stack](https://lwn.net/Articles/338009/)

/Documentation/x86/x86_64/mm.txt

/Documentation/x86/boot.txt
<!-- more -->
随着芯片技术升级现在已经出现5级页表了，但是估计还是服务器上面居多，这里重点记录一下4级页表的布局

4页使用48位的虚拟地址，

```
========================================================================================================================
    Start addr    |   Offset   |     End addr     |  Size   | VM area description
========================================================================================================================
                  |            |                  |         |
 0000000000000000 |    0       | 00007fffffffffff |  128 TB | user-space virtual memory, different per mm
__________________|____________|__________________|_________|___________________________________________________________
                  |            |                  |         |
 0000800000000000 | +128    TB | ffff7fffffffffff | ~16M TB | ... huge, almost 64 bits wide hole of non-canonical
                  |            |                  |         |     virtual memory addresses up to the -128 TB
                  |            |                  |         |     starting offset of kernel mappings.
__________________|____________|__________________|_________|___________________________________________________________
                                                            |
                                                            | Kernel-space virtual memory, shared between all processes:
____________________________________________________________|___________________________________________________________
                  |            |                  |         |
 ffff800000000000 | -128    TB | ffff87ffffffffff |    8 TB | ... guard hole, also reserved for hypervisor
 ffff880000000000 | -120    TB | ffff887fffffffff |  0.5 TB | LDT remap for PTI
 ffff888000000000 | -119.5  TB | ffffc87fffffffff |   64 TB | direct mapping of all physical memory (page_offset_base)
 ffffc88000000000 |  -55.5  TB | ffffc8ffffffffff |  0.5 TB | ... unused hole
 ffffc90000000000 |  -55    TB | ffffe8ffffffffff |   32 TB | vmalloc/ioremap space (vmalloc_base)
 ffffe90000000000 |  -23    TB | ffffe9ffffffffff |    1 TB | ... unused hole
 ffffea0000000000 |  -22    TB | ffffeaffffffffff |    1 TB | virtual memory map (vmemmap_base)
 ffffeb0000000000 |  -21    TB | ffffebffffffffff |    1 TB | ... unused hole
 ffffec0000000000 |  -20    TB | fffffbffffffffff |   16 TB | KASAN shadow memory
__________________|____________|__________________|_________|____________________________________________________________
                                                            |
                                                            | Identical layout to the 56-bit one from here on:
____________________________________________________________|____________________________________________________________
                  |            |                  |         |
 fffffc0000000000 |   -4    TB | fffffdffffffffff |    2 TB | ... unused hole
                  |            |                  |         | vaddr_end for KASLR
 fffffe0000000000 |   -2    TB | fffffe7fffffffff |  0.5 TB | cpu_entry_area mapping
 fffffe8000000000 |   -1.5  TB | fffffeffffffffff |  0.5 TB | ... unused hole
 ffffff0000000000 |   -1    TB | ffffff7fffffffff |  0.5 TB | %esp fixup stacks
 ffffff8000000000 | -512    GB | ffffffeeffffffff |  444 GB | ... unused hole
 ffffffef00000000 |  -68    GB | fffffffeffffffff |   64 GB | EFI region mapping space
 ffffffff00000000 |   -4    GB | ffffffff7fffffff |    2 GB | ... unused hole
 ffffffff80000000 |   -2    GB | ffffffff9fffffff |  512 MB | kernel text mapping, mapped to physical address 0
 ffffffff80000000 |-2048    MB |                  |         |
 ffffffffa0000000 |-1536    MB | fffffffffeffffff | 1520 MB | module mapping space
 ffffffffff000000 |  -16    MB |                  |         |
    FIXADDR_START | ~-11    MB | ffffffffff5fffff | ~0.5 MB | kernel-internal fixmap range, variable size and offset
 ffffffffff600000 |  -10    MB | ffffffffff600fff |    4 kB | legacy vsyscall ABI
 ffffffffffe00000 |   -2    MB | ffffffffffffffff |    2 MB | ... unused hole
__________________|____________|__________________|_________|___________________________________________________________
```

> Negative addresses such as “-23 TB” are absolute addresses in bytes, counted down from the top of the 64-bit address space. It’s easier to understand the layout when seen both in absolute addresses and in distance-from-top notation.   
>For example 0xffffe90000000000 == -23 TB, it’s 23 TB lower than the top of the 64-bit address space (ffffffffffffffff).


现代的操作系统 bzImage kernel使用boot 协定 version >= 2.02 的内存布局

```
              ~                        ~
              |  Protected-mode kernel |
      100000  +------------------------+
              |  I/O memory hole       |
      0A0000  +------------------------+
              |  Reserved for BIOS     |      Leave as much as possible unused
              ~                        ~
              |  Command line          |      (Can also be below the X+10000 mark)
      X+10000 +------------------------+
              |  Stack/heap            |      For use by the kernel real-mode code.
      X+08000 +------------------------+
              |  Kernel setup          |      The kernel real-mode code.
              |  Kernel boot sector    |      The kernel legacy boot sector.
      X       +------------------------+
              |  Boot loader           |      <- Boot sector entry point 0000:7C00
      001000  +------------------------+
              |  Reserved for MBR/BIOS |
      000800  +------------------------+
              |  Typically used by MBR |
      000600  +------------------------+
              |  BIOS use only         |
      000000  +------------------------+

... where the address X is as low as the design of the boot loader permits.
```

解决一下 %esp fixup stacks
