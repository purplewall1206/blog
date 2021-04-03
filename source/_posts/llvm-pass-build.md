---
title: llvm-pass-build
date: 2021-04-03 17:02:18
tags: [llvm, compiler, llvm-pass, instrumentation]
---

# llvm-pass-build

llvm pass 编译有三种方式，直接上clang干，或者使用github skeleton项目给出的cmake，或者直接make

# 写在前面

这里有个非常诡异的问题， `opt -load mypass.so`显示文件格式错误，但是`opt -load ./mypass.so` 就可以正常使用

因此记得 **opt -load ./mypass.so**

# clang

```
clang `llvm-config --cxxflags` -Wl,-znodelete -fno-rtti -fPIC -shared Hello.cpp -o LLVMHello.so `llvm-config --ldflags`

opt -load ./LLVMHello.so -hello test.bc
```
<!-- more -->

# make 

```
LLVM_CONFIG?=llvm-config

SRC_DIR?=$(PWD)
LD_FLAGS+=$(shell $(LLVM_CONFIG) --ldflags)

COMMON_FLAGS=-Wall -Wextra
CXXFLAGS+=$(COMMON_FLAGS) $(shell $(LLVM_CONFIG) --cxxflags)
CXXFLAGS+=-fPIC
CPPFLAGS+=$(shell $(LLVM_CONFIG) --cppflags) -T$(SRC_DIR)
CPPFLAGS+=-fPIC

ifeq ($(shell uname), Darwin)
LOADABLE_MODULE_OPTIONS=-bundle -undefined dynamic_lookup
else
LOADABLE_MODULE_OPTIONS=-shared -Wl,-O1
endif

HELLOPASS=hellopass.so
HELLOPASS_OBJECTS=Hello.o

default: $(HELLOPASS)

%.o : $(SRC_DIR)/%.cpp
	@echo Compiling $.cpp
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $<

$(HELLOPASS) : $(HELLOPASS_OBJECTS)
	@echo Linking
	$(CXX) -o $@  $(LOADABLE_MODULE_OPTIONS) $(CXXFLAGS) $(LD_FLAGS) $^

clean:
	rm -f $(HELLOPASS) $(HELLOPASS_OBJECTS)
```
[llvm-practice/tutorial/5-pass-test/Makefile](https://github.com/purplewall1206/llvm-practice/blob/main/tutorial/5-pass-test/Makefile)

# cmake 

[github skeleton](https://github.com/sampsyo/llvm-pass-skeleton)

或者我的github

[llvm-practice/tutorial/6-instrumentation/](https://github.com/purplewall1206/llvm-practice/tree/main/tutorial/6-instrumentation)

