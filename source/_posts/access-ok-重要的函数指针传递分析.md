---
title: access_ok 重要的函数指针传递分析
date: 2020-12-15 09:45:55
tags: [linux, kernel]
---

/arch/x86/include/asm/uaccess.h

用于检查user space 的指针，是否超过用户地址空间的界限，读取或写入内核地址。

Checks if a pointer to a block of memory in user space is valid.

Note that, depending on architecture, this function probably just checks that the pointer is in the user space range

<!-- more -->

```
#define access_ok(addr, size)					\
({									\
	WARN_ON_IN_IRQ();						\
	likely(!__range_not_ok(addr, size, user_addr_max()));		\
})

#define __range_not_ok(addr, size, limit)				\
({									\
	__chk_user_ptr(addr);						\
	__chk_range_not_ok((unsigned long __force)(addr), size, limit); \
})

#define user_addr_max() (current->thread.addr_limit.seg)
# define __chk_user_ptr(x)	(void)0


/*
 * Test whether a block of memory is a valid user space address.
 * Returns 0 if the range is valid, nonzero otherwise.
 */
static inline bool __chk_range_not_ok(unsigned long addr, unsigned long size, unsigned long limit)
{
	/*
	 * If we have used "sizeof()" for the size,
	 * we know it won't overflow the limit (but
	 * it might overflow the 'addr', so it's
	 * important to subtract the size from the
	 * limit, not add it to the address).
	 */
	if (__builtin_constant_p(size))
		return unlikely(addr > limit - size);

	/* Arbitrary sizes? Be careful about overflow */
	addr += size;
	if (unlikely(addr < size))
		return true;
	return unlikely(addr > limit);
}

```


注意这里有一个__chk_user_ptr，编译器不会生成任何代码
