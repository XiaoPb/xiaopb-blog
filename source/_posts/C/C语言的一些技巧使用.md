---
title:  C语言的一些技巧使用
date: 2020-08-24 12:30:03
abbrlink: C-CPP-0000-0003
author: XiaoPb
img: 
coverImg: /medias/banner/7.jpg
top: true
cover: true
toc: true
mathjax: false
summary: 
  C语言编译器内置宏的使用、LOG输出组件的设计以及printf输出终端的控制符。
tags:
  - C/C++
  - 宏
  - printf
  - debug-log
categories:
  - C/C++
password:
---

# C语言的一些技巧使用

## 编译器内置宏的使用

​		嵌入式进行调试时，经常使用`printf`进行`debug`信息输出，而为了尽快的定位问题位置，经常会需要修改一下输出格式，如果使用函数进行封装，会占用栈资源，为了节省资源使用，可以使用`#define`进行函数封装，为了快速定位问题位置，可以使用编译器内置的一些宏进行输出，使调试，方便快捷。

|       宏       |           定义           |
| :------------: | :----------------------: |
| `__FUNCTION__` | 调用该宏所在函数的函数名 |
|   `__LINE__`   |    调用该宏所在的行号    |
|   `__FILE__`   |   调用该宏所在的文件名   |
|   `__DATE__`   |    编译该工程时的日期    |
|   `__TIME__`   |    编译该工程时的时间    |

例子：

```c
#include "stdio.h"

int main(void)
{
	printf("%s\n", __FUNCTION__);
	printf("%d\n", __LINE__);
	printf("%s\n", __FILE__);
	printf("%s\n", __DATE__);
	printf("%s\n", __TIME__);
}
```
输出：

```powershell
main
6
c:\Code\C\test.c
Aug 24 2020
14:05:15
```

​	利用这几个编译器内置的宏，可以方便快速的定制我们自己的`debug`输出信息。

1. 定制版本信息

   `#define LOG_V(fmt, ...) printf("[%s %s]: "fmt, __DATE__, __TIME__,##__VA_ARGS__)`

2. debug输出当前信息时包括所在函数以及文件信息

   ```c
   #define LOG(fmt, ...) printf("<%s>[%s]: "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__)
   #define DRV_S "main" //在不同文件中自个定制当前文件的标志，方便快速定位文件
   ```

例子：

```c
#include "stdio.h"
#define LOG(fmt, ...) printf("<%s>[%s]: "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__)
#define DRV_S "main"
void test(void)
{
	LOG("[%d]hello world!\r\n",2);
}
int main(void)
{
	LOG("[%d]hello world!\r\n",1);
	test();
}
```

输出：

```powershell
<main>[main]: [1]hello world!
<main>[test]: [2]hello world!
```



## LOG简易组件快速搭建使用

```c
#ifndef __XP_LOG_H__
#define __XP_LOG_H__

#define USE_LOG 1 //是否使用LOG输出
#define USE_COLOR 1 //是否使用彩色输出

#if USE_LOG

#if USE_COLOR
#define LOG_START(c) printf("\033["#c"m ")
#define LOG_END      printf("\033[0m\r\n")
#else
#define LOG_START(c) printf(" ")
#define LOG_END      printf("\r\n")
#endif

#define LOG_I(fmt, ...) \
		do{	\
			LOG_START(32);			\
			printf("[I/%s/%s] "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__);	\
			LOG_END;			\
		}while(0)
#define LOG_D(fmt, ...) \
		do{	\
			LOG_START(0);			\
			printf("[D/%s/%s] "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__);	\
			LOG_END;			\
		}while(0)
#define LOG_W(fmt, ...) \
		do{	\
			LOG_START(33);			\
			printf("[W/%s/%s] "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__);   \
			LOG_END;			\
		}while(0)
#define LOG_E(fmt, ...) \
		do{	\
			LOG_START(31);			\
			printf("[E/%s/%s] "fmt,DRV_S,__FUNCTION__,##__VA_ARGS__);	\
			LOG_END;			\
		}while(0)
#else
#define LOG_I(fmt, ...) 
#define LOG_D(fmt, ...) 
#define LOG_W(fmt, ...) 
#define LOG_E(fmt, ...) 
#endif

#endif
```

```c
#include "stdio.h"
#include "xp_log.h"

#define DRV_S "drv.iic"

void test(void)
{
	LOG_I("test");
	LOG_W("test");
	LOG_D("test");
	LOG_E("test");
}
int main(void)
{
	LOG_I("main");
	LOG_W("main");
	LOG_D("main");
	LOG_E("main");
	test();
}
```

```powershell
 [I/drv.iic/main] main
 [W/drv.iic/main] main
 [D/drv.iic/main] main
 [E/drv.iic/main] main
 [I/drv.iic/test] test
 [W/drv.iic/test] test
 [D/drv.iic/test] test
 [E/drv.iic/test] test
```

​	可见，通过这样封装输出可以快速判断在哪个文件哪个函数的输出信息，是什么类型的信息，开启颜色后，还可以通过颜色判别输出信息类型。

## printf重定义

```c
#include "stdio.h"
#include "xp_log.h"

#define DRV_S "drv.iic"
#undef printf
#define printf(fmt, ...) LOG_I(fmt,##__VA_ARGS__)
void test(void)
{
	printf("test");
}
int main(void)
{
	printf("main");
	test();
}
```

```powershell
 [I/drv.iic/main] main
 [I/drv.iic/test] test
```

​	可见，通过这样先用`undef`取消`printf`的定义再重新定义`printf`，可以减少很多代码上的修改。

## printf终端输出控制符

```c
printf("\033[1;33m Hello World. \033[0m \n");  
颜色列表如下:  
#define none 		 "\033[0m"  
#define black        "\033[0;30m"  
#define dark_gray    "\033[1;30m"  
#define blue         "\033[0;34m"  
#define light_blue   "\033[1;34m"  
#define green        "\033[0;32m"  
#define light_green  "\033[1;32m"  
#define cyan         "\033[0;36m"  
#define light_cyan   "\033[1;36m"  
#define red          "\033[0;31m"  
#define light_red    "\033[1;31m"  
#define purple       "\033[0;35m"  
#define light_purple "\033[1;35m"  
#define brown        "\033[0;33m"  
#define yellow       "\033[1;33m"  
#define light_gray   "\033[0;37m"  
#define white        "\033[1;37m"  

字背景颜色范围:40--49            字颜色: 30--39  
40: 黑                        30: 黑  
41: 红                        31: 红  
42: 绿                        32: 绿  
43: 黄                        33: 黄  
44: 蓝                        34: 蓝  
45: 紫                        35: 紫  
46: 深绿                      36: 深绿  
47: 白色                      37: 白色  

输出特效格式控制：  
\033[0m  关闭所有属性    
\033[1m   设置高亮度    
\03[4m   下划线    
\033[5m   闪烁    
\033[7m   反显    
\033[8m   消隐    
\033[30m   --   \033[37m   设置前景色    
\033[40m   --   \033[47m   设置背景色  

光标位置等的格式控制：  
\033[nA  光标上移n行    
\03[nB   光标下移n行    
\033[nC   光标右移n行    
\033[nD   光标左移n行    
\033[y;xH设置光标位置    
\033[2J   清屏    
\033[K   清除从光标到行尾的内容    
\033[s   保存光标位置    
\033[u   恢复光标位置    
\033[?25l   隐藏光标    
\33[?25h   显示光标
```

