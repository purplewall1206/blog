---
title: typedef code pointer 代码指针
date: 2020-12-12 16:55:39
tags: [linux, kernel, c]
---

`typedef` 是把一个名字和一个类型联系起来。

```
  typedef int myinteger;
  typedef char *mystring;
  typedef void (*myfunc)();

  myinteger i;   // is equivalent to    int i;
  mystring s;    // is the same as      char *s;
  myfunc f;      // compile equally as  void (*f)();
```
<!-- more -->
注意第三个给函数做typedef，目的是简化代码阅读的难度，尤其是指针指向函数和structure。

```
typedef int (*t_somefunc)(int,int);

int product(int u, int v) {
  return u*v;
}

t_somefunc afunc = &product;
...
int x2 = (*afunc)(123, 456); // call product() to calculate 123*456
```

在汇编中这是典型的间接跳转 call %rax

```
│B+ 0x555555555154 <main+8>         lea    -0x22(%rip),%rax        #0x555555555139 <product>    
│
│   0x55555555515b <main+15>        mov    %rax,-0x8(%rbp)     
│
│  >0x55555555515f <main+19>        mov    -0x8(%rbp),%rax       
│              
│   0x555555555163 <main+23>        mov    $0x14,%esi   
│
│   0x555555555168 <main+28>        mov    $0xf,%edi  
│
│   0x55555555516d <main+33>        call   *%rax
```

