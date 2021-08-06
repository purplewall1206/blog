---
title: 内核符号kallsyms和只读测试
date: 2021-08-06 16:17:28
tags: kernel
---


# sudo cat /proc/kallsyms
```
ffffffff81140450 T put_old_timex32
ffffffff81140560 t __do_sys_adjtimex_time32
ffffffff811405e0 T __x64_sys_adjtimex_time32
...
ffffffff82e59660 D acpi_rs_convert_gpio
...
ffffffff826c0360 D kmalloc_caches
ffffffff826f8f5c r __ksymtab_kmalloc_caches
ffffffff827266f0 r __kstrtabns_kmalloc_caches
ffffffff82728043 r __kstrtab_kmalloc_caches
```

# 符号类型

* T 全局代码符号
* t local代码符号
* D 全局数据符号
* d 本地数据符号
* B/b 全局和本地bss符号（未初始化data区）
* R/r 只读数据符号（和D/d重合）

# kernel EXPORT_SYMBOL() 介绍

需要通过export_symbol 导出给用户使用

* `__kstrtab_<symbol_name>` - name of the symbol as a string
* `__ksymtab_<symbol_name>` - a structure with the information about the symbol: its address, address of `__kstrtab_<symbol_name>`, etc.
* `__kcrctab_<symbol_name>` - address of the control sum (CRC) of the symbol

# 小实验 ext4_file_operations 只读保护？

```
    fops read only

    unsigned long *ext4_fops = (unsigned long*)0xffffffff8224cb40;
    pr_info("%016lx\n", *ext4_fops);
    *ext4_fops = 0xdeadbeef0123abcd;
    pr_info("%016lx\n", *ext4_fops);// trigger page fault
```

触发了page fault，显然这些页是只读的。