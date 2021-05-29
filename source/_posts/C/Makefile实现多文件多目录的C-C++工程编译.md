---
title:  Makefile实现多文件多目录的C/C++工程编译
date: 2020-08-30 12:30:03
abbrlink: C-CPP-0000-0004
author: XiaoPb
img: 
coverImg: /medias/banner/10.jpg
top: false
cover: false
toc: true
mathjax: false
summary: 
  VSCode中利用Makefile实现多文件多目录的C/C++语言工程编译运行，一个模板实现多工程自动适应编译运行。
tags:
  - C/C++
  - Makefile
  - make
  - vscode
categories:
  - C/C++
password:
---

# Makefile实现多文件多目录的C/C++工程编译

## 一、前期准备

1. 安装[VSCode](https://code.visualstudio.com/)；

2. 安装[MinGw](https://sourceforge.net/projects/mingw-w64/files/)；

3. 配置环境变量；

4. [配置C/C++环境](https://www.cnblogs.com/czlhxm/p/11794743.html)；

5. 安装make环境

   1. 下载[RT-Thread](https://www.rt-thread.org/page/download.html) 的ENV工具

   2. 解压到你想安装的位置

   3. 将ENV路径下的`.\env\tools\bin`添加到系统环境变量中

   4. 打开`powerShell`，运行`make -v`，出现

      ``` powershell
      GNU Make 3.81
      Copyright (C) 2006  Free Software Foundation, Inc.
      This is free software; see the source for copying conditions.
      There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
      
      This program built for i686-pc-msys 
      ```
      即是安装成功。

## 二、建立Makefile模板

```makefile
CC      = gcc
CXX     = g++
LINK    = g++
CFLAGS  = -g -Wall -O2
TARGET  = mk0
SRCS    = $(wildcard *.cpp)
SRCS    += $(wildcard *.c)

CXX_OBJS    = $(patsubst %.cpp, %.o, $(wildcard *.cpp))
C_OBJS  = $(patsubst %.c, %.o, $(wildcard *.c))

all:$(TARGET)

$(TARGET):$(CXX_OBJS) $(C_OBJS)
        $(LINK) $(CFLAGS) -o $@ $^
%.o:%.cpp
        $(CXX) $(CFLAGS) -c -o $@ $<
%.o:%.c
        $(CC) $(CFLAGS) -c -o $@ $<
.PHONY:clean
clean:
        rm -rf *.o $(TARGET) $(CXX_OBJS) $(C_OBJS)
```

1. C语言采用`gcc`编译，`C++`采用`g++`编译，使用`g++`进行链接，`CFLAGS`是用于编译器的选项，`TARGET`是工程名字，`SRCS`是当前目录下的所有`.cpp`和`.c`文件，`CXX_OBJS`和`C_OBJS`是编译产生的所有`.o`文件。

2. 使用`make`命令时，首先执行`all`；要想删除所有`.o`文件，就执行`make clean`。

3. 为了使工程名称跟随工程目录，不需要自己专门去更改`TARGET  = mk0`，则修改为

   ```makefile
   cd_path = $(abspath $(lastword $(MAKEFILE_LIST)))
   cd_path := $(patsubst %/Makefile,%,$(cd_path))
   cd_path := $(notdir $(cd_path))
   TARGET  = $(cd_path)
   ```


## 三、修改优化后的Makefile模板

```makefile
CC      = gcc
CXX     = g++
LINK    = g++
CFLAGS  = -g -Wall -O2
DEBUG   = ./debug
C1_PATH = ./*/src
C2_PATH = ./*/*/src
tg_path = $(abspath $(lastword $(MAKEFILE_LIST)))
tg_path := $(patsubst %/Makefile,%,$(tg_path))
tg_path := $(notdir $(tg_path))
TARGET  = $(tg_path)

RUN     = run

SRCXXS  = $(wildcard *.cpp)
SRCXXS  += $(wildcard ./*/src/*.cpp)
SRCXXS  += $(wildcard ./*/*/src/*.cpp)

SRCS    = $(wildcard *.c)
SRCS    += $(wildcard ./*/src/*.c)
SRCS    += $(wildcard ./*/*/src/*.c)

INC_PATH = $(subst ./,-I ./,$(wildcard ./*/inc))
INC_PATH += $(subst ./,-I ./,$(wildcard ./*/*/inc))

CXX_OBJS = $(patsubst %.cpp,$(DEBUG)/%.o,$(notdir $(SRCXXS)))
C_OBJS   = $(patsubst %.c,$(DEBUG)/%.o,$(notdir $(SRCS)))

all: $(DEBUG) $(TARGET) $(RUN)
$(DEBUG):
ifneq ($(DEBUG), $(wildcard $(DEBUG)))
		mkdir -p debug
endif
$(TARGET):$(CXX_OBJS) $(C_OBJS)
		$(LINK) $(CFLAGS) -o $@ $^
$(RUN):
		./$(TARGET).exe

$(DEBUG)/%.o:$(C1_PATH)/%.cpp
		$(CXX) $(CFLAGS) $(INC_PATH) -c -o $@ $<
$(DEBUG)/%.o:$(C2_PATH)/%.cpp
		$(CXX) $(CFLAGS) $(INC_PATH) -c -o $@ $<

$(DEBUG)/%.o: $(C1_PATH)/%.c
		$(CC) $(CFLAGS) $(INC_PATH) -c -o $@ $<
$(DEBUG)/%.o: $(C2_PATH)/%.c
		$(CC) $(CFLAGS) $(INC_PATH) -c -o $@ $<

.PHONY:clean
clean:
		rm -rf *.o $(TARGET) $(CXX_OBJS) $(C_OBJS)
```

要求工程目录格式如下

1. 文件夹名字随意
2. Makefile文件放在工程目录下
3. main函数所在文件建议放在`./main/src`下
4. 所有源文件放在`src`文件夹下
5. 所有头文件放在`inc`文件夹下
6. 子目录最深到`./*/*/src/`OR`./*/*/inc/`,更深的目录需要更改Makefile文件
7. 工程名字是`target`,随父目录名称更改而更改

> target
>
> > main
> >
> > > src
> >
> > pack
> >
> > > pack-1
> > >
> > > > inc
> > > >
> > > > src
> > >
> > > pack-2
> > >
> > > > inc
> > > >
> > > > src
> > >
> > >·····
> > 
> > Makefile
> > 

