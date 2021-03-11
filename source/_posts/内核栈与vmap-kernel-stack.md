---
title: 内核栈与vmap kernel stack
date: 2021-01-15 16:37:32
tags: [kernel, stack, linux]
---
# 参考
[Virtually mapped kernel stacks](https://lwn.net/Articles/692208/)    
[Virtually mapped stacks with guard pages (x86, core)](https://lwn.net/Articles/691631/)

kernel stack在64位系统上通常是8k或16k连续映射，但是很容易造成溢出，而且在内存被大量占用的情况下不太容易找到连续的2页或4页。

因此有开发者提出了解决办法，使用vmap解决这两个问题,使得内核栈在溢出时print一个错误信息。
<!-- more -->
# Virtually mapped stacks

作者 Andy Lutomirski 使用vmalloc分配内核栈，然而这面临着内存分配速度1.5us，比较慢的问题。

作者本来打算不管这套，并且想让vmalloc速度快点，但是linus建议为per_cpu分配几个cache，实际实现的过程中是为每个CPU分配了2个cache

```
#define NR_CACHED_STACKS 2
static DEFINE_PER_CPU(struct vm_struct *, cached_stacks[NR_CACHED_STACKS]);
```

如果其中一个能用就分配给申请的task，如果两个都不行就再申请个新的

```
for (i = 0; i < NR_CACHED_STACKS; i++) {
		struct vm_struct *s;

		s = this_cpu_xchg(cached_stacks[i], NULL);

		if (!s)
			continue;

		/* Clear the KASAN shadow of the stack. */
		kasan_unpoison_shadow(s->addr, THREAD_SIZE);

		/* Clear stale pointers from reused stack. */
		memset(s->addr, 0, THREAD_SIZE);

		tsk->stack_vm_area = s;
		tsk->stack = s->addr;
		return s->addr;
	}

	/*
	 * Allocated stacks are cached and later reused by new threads,
	 * so memcg accounting is performed manually on assigning/releasing
	 * stacks to tasks. Drop __GFP_ACCOUNT.
	 */
	stack = __vmalloc_node_range(THREAD_SIZE, THREAD_ALIGN,
				     VMALLOC_START, VMALLOC_END,
				     THREADINFO_GFP & ~__GFP_ACCOUNT,
				     PAGE_KERNEL,
				     0, node, __builtin_return_address(0));
```
## kconfig
- CONFIG_HAVE_ARCH_VMAP_STACK=y
- CONFIG_VMAP_STACK=y

# 原本的内核栈

直接申请物理页，赋予的地址在直接映射区
```
	struct page *page = alloc_pages_node(node, THREADINFO_GFP,
					     THREAD_SIZE_ORDER);

	if (likely(page)) {
		tsk->stack = page_address(page);
		return tsk->stack;
	}
	return NULL;
```