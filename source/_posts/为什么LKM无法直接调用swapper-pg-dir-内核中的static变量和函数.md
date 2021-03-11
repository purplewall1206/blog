---
title: 为什么LKM无法直接调用swapper_pg_dir--内核中的static变量和函数
date: 2021-03-01 16:09:53
tags: [kernel, linux ,c ,programming]
---

# kernel static functions 内核静态函数

在编写drivers想要调用一些敏感的内核函数时，往往会出现`xxx undefined`这种情况，并不能通过加入头文件的方式解决。

深入了解一下，以`__p4d_alloc`为例：

在`include/linux/mm.h`中声明`static inline int __p4d_alloc(struct mm_struct *mm, pgd_t *pgd, unsigned long address)` 为静态内联函数

那么为什么在driver中无法调用静态函数呢？这回归到了C语言的一个基础问题，static到底是做什么的？
<!-- more -->
# static

1. 可见性不同，全局变量，所有未加static的全局变量和全局函数都有全局可见性，如在文件1中定义的函数和变量可以在文件2中调用，但是加上了static在文件2中则不可见。

```
char a = 'A'; // global variable
void msg()
{
    printf("Hello\n");
}
--------------另一个文件-----------
int main(void)
{    
    extern char a;    // extern variable must be declared before use
    printf("%c ", a);
    (void)msg();
    return 0;
}
```

2. 生命周期是整个程序，在程序启动时初始化，全局变量和static变量（全局和局部）都存储在静态存储区，主要区别在于可见性不同。
3. 默认初始化为0,因为静态区默认初始化为0.

因此我们能够明白为什么在LKM中我们无法直接调用`__p4d_alloc` 和 `swapper_pg_dir` 这些符号，因为都是被static限定了适用范围。

# 头文件中的static inline声明

头文件中最好不要使用static声明函数，因为：

> 头文件中的 static 函数会在每个文件中生成一份代码，这造成代码冗余倒不是最大的问题，最大的问题是可能带来库文件与工程文件同一函数的代码的不一致性，这有风险。

但是有例外情况 比如内核中大量`static inline int pud_present(pud_t pud)`这类函数，定义为static inline

因为c语言中的inline 关键字只是建议内联，通常编译器不执行，但是如果使用 `static inline` 则会保证一定**内联**。



# 如何使用

目前看来唯一可行的办法是通过地址引用`struct mmstruct *INIT_MM = 0xaddress`。

另外记录两次失败案例：
- 将static 函数使用 `EXPORT_SYMBOL()`导出，结果是无法载入内核中
- 将函数改为非static，则无法通过编译。

这里编译浪费了好多时间，不要轻易尝试。

如果只想通过编译，可以使用obj-y，显然无法生成.ko
