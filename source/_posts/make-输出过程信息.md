---
title: make-输出过程信息
date: 2021-07-28 12:55:37
tags: [compiler, make]
---

```
make SHELL='sh -x' # 通用的

make VERBOSE=1 # 少数情况下有用

make CC=clang -j4 SHELL='sh -x' 2>compile.log
```
