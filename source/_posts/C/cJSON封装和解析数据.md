---
title:  cJSON封装和解析数据
date: 2020-08-20 12:25:03
abbrlink: C-CPP-0000-0001
author: XiaoPb
img: 
coverImg: /medias/banner/4.jpg
top: false
cover: false
toc: true
mathjax: false
summary: 
  cJSON的源码简洁精炼，使用方便容易！并且只需要两个文件cJSON.h/.c,非常适合用来学习数据结构的链表内容。其JSON的简洁性更决定其在IOT方面有大量用户。
tags:
  - IOT
  - cJSON
  - JSON
  - 链表
  - C/C++
categories:
  - IOT
password:
---

# JSON 与 cJSON

## JSON —— 轻量级的数据格式

> **JSON**[1] 全称 JavaScript Object Notation，即 JS 对象简谱，是一种轻量级的数据格式。
>
> 它采用完全独立于编程语言的文本格式来存储和表示数据，语法简洁、层次结构清晰，易于人阅读和编写，同时也易于机器解析和生成，有效的提升了网络传输效率。

## JSON —— 语法规则

> JSON 对象是一个无序的"名称/值"键值对的集合：
>
> - 以"`{`"开始，以"`}`"结束，允许嵌套使用；
> - 每个名称和值成对出现，名称和值之间使用"`:`"分隔；
> - 键值对之间用"`,`"分隔
> - 在这些字符前后允许存在无意义的空白符；
>
> 对于键值，可以有如下值：
>
> - 一个新的 json 对象
> - 数组：使用"`[`"和"`]`"表示
> - 数字：直接表示，可以是整数，也可以是浮点数
> - 字符串：使用引号`"`表示
> - 字面值：false、null、true 中的一个(必须是小写)

示例如下：

```json
{
        "name": "XiaoPb",
        "age":  23,
        "heigh": 170.500000,
        "address":      
  			{
                "country":      "China",
                "zip-code":     111111
        },
        "skill": ["c/cpp", "python", "stm32", "makefile"],
        "student": true
}
```

## cJSON

cJSON 是一个使用 C 语言编写的 JSON 数据解析器，具有超轻便，可移植，单文件的特点，使用 MIT 开源协议。

cJSON 项目托管在 Github 上，仓库地址如下：

> https://github.com/DaveGamble/cJSON

使用 Git 命令将其拉取到本地：

```bash
git clone https://github.com/DaveGamble/cJSON.git 
```

从 Github 拉取 cJSON 源码后，文件非常多，但是其中 cJSON 的源码文件只有两个：

> - cJSON.h
> - cJSON.c

使用的时候，只需要将这两个文件复制到工程目录，然后包含头文件`cJSON.h`即可，如下：

```c
#include "cJSON.h"
```

# cJSON 数据结构和设计思想

**cJSON 的设计思想从其数据结构上就能反映出来。**

cJSON 使用 cJSON 结构体来表示**一个 JSON 数据**，定义在`cJSON.h`中，源码如下：

```c
/* The cJSON structure: */
typedef struct cJSON
{
    /* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
    struct cJSON *next;
    struct cJSON *prev;
    /* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
    struct cJSON *child;

    /* The type of the item, as above. */
    int type;

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw */
    char *valuestring;
    /* writing to valueint is DEPRECATED, use cJSON_SetNumberValue instead */
    int valueint;
    /* The item's number, if type==cJSON_Number */
    double valuedouble;

    /* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    char *string;
} cJSON;
```

cJSON 的设计很巧妙。

首先，**它不是将一整段 JSON 数据抽象出来，而是将其中的一条 JSON 数据抽象出来，也就是一个键值对**，用上面的结构体 `strcut cJSON` 来表示，其中用来存放值的成员列表如下：

- `String`：用于表示该键值对的名称；
- `type`：用于表示该键值对中值的类型；
- `valuestring`：如果键值类型(type)是字符串，则将该指针指向键值；
- `valueint`：如果键值类型(type)是整数，则将该指针指向键值；
- `valuedouble`：如果键值类型(type)是浮点数，则将该指针指向键值；

首先，**它不是将一整段 JSON 数据抽象出来，而是将其中的一条 JSON 数据抽象出来，也就是一个键值对**，用上面的结构体 `strcut cJSON` 来表示，其中用来存放值的成员列表如下：

- `String`：用于表示该键值对的名称；
- `type`：用于表示该键值对中值的类型；
- `valuestring`：如果键值类型(type)是字符串，则将该指针指向键值；
- `valueint`：如果键值类型(type)是整数，则将该指针指向键值；
- `valuedouble`：如果键值类型(type)是浮点数，则将该指针指向键值；

其次，一段完整的 JSON 数据中由很多键值对组成，并且涉及到键值对的查找、删除、添加，所以**使用链表来存储整段 JSON 数据**，如上面的代码所示：

- `next`指针：指向下一个键值对
- `prev`指针指向上一个键值对

最后，因为 JSON 数据支持嵌套，所以一个**键值对的值会是一个新的 JSON 数据对象（一条新的链表）**，也有可能是一个数组，方便起见，在 cJSON 中，数组也表示为一个数组对象，用链表存储，所以：

在键值对结构体中，当该键值对的值是一个嵌套的 JSON 数据或者一个数组时，由`child`指针指向该条新链表。

#  JSON 数据封装

## 封装方法

封装 JSON 数据的过程，其实就是创建链表和向链表中添加节点的过程。

首先来讲述一下链表中的一些术语：

- 头指针：指向链表头结点的指针；
- 头结点：不存放有效数据，方便链表操作；
- 首节点：第一个存放有效数据的节点；
- 尾节点：最后一个存放有效数据的节点；

明白了这几个概念之后，我们开始讲述创建一段完整的 JSON 数据，即如何创建一条完整的链表。

- ① 创建头指针：

```c
cJSON *human = NULL;
```

- ② 创建头结点，并将头指针指向头结点：

```c
human = cJSON_CreateObject();
```

- ③ 尽情的向链表中添加节点：

```c
cJSON_AddNullToObject(cJSON * const object, const char * const name);

cJSON_AddTrueToObject(cJSON * const object, const char * const name);

cJSON_AddFalseToObject(cJSON * const object, const char * const name);

cJSON_AddBoolToObject(cJSON * const object, const char * const name, const cJSON_bool boolean);

cJSON_AddNumberToObject(cJSON * const object, const char * const name, const double number);

cJSON_AddStringToObject(cJSON * const object, const char * const name, const char * const string);

cJSON_AddRawToObject(cJSON * const object, const char * const name, const char * const raw);

cJSON_AddObjectToObject(cJSON * const object, const char * const name);

cJSON_AddArrayToObject(cJSON * const object, const char * const name);
```

## 输出 JSON 数据

上面讲述，一段完整的 JSON 数据就是一条长长的链表，那么，如何打印出这段 JSON 数据呢？

cJSON 提供了一个 API，可以将整条链表中存放的 JSON 信息输出到一个字符串中：

```c
(char *) cJSON_Print(const cJSON *item);
```

使用的时候，只需要接收该函数返回的指针地址即可。

## 封装数据和打印数据示例

单纯的讲述方法还不够，下面用一个例子来说明，封装出开头给出的那段 JSON 数据：

```c
#include <stdio.h>
#include "xp_log.h" //LOG打印组件
#include "cJSON.h"

#define DRV_S "main" //配合LOG打印组件使用

#define SKILL_NUM 4 //skill数目

#undef bool  //取消bool相关定义
/* 定义一个bool枚举类型代替二进制 */
typedef enum bool{
	false = 0,
	true  = 1
}bool;
/* 定义一个国家信息结构体 */
typedef struct xp_address
{
	char country[10];
	int  zip_code;
}xp_address;
/* 定义一个个人信息结构体 */
typedef struct xp_human
{
	char name[10];
	int  age;
	double  heigh;
	xp_address* address;
	char skill[SKILL_NUM][10];
	bool student;
}xp_human;

/* 根据内容初始化结构体 */
xp_address my_addr ={
	.country = "China",
	.zip_code = 111111
};
xp_human my_human = {
	.name = "XiaoPb",
	.age  = 23,
	.heigh = 170.5,
	.address = &my_addr,
	.skill   = {"c/cpp","python","stm32","makefile"},
	.student = true
};

char cjson_get_str[512];


cJSON *create_json_human(const xp_human* my_human)
{
	unsigned int i = 0;
	cJSON *address = NULL; 
	cJSON *skill = NULL;
	cJSON *human = NULL;

	LOG_I("cJSON creat human start!");
	/* 创建一个JSON数据对象(链表头结点) */
	human = cJSON_CreateObject();
	if(human == NULL)
	{
		LOG_E("cjson creat obj error!");
		return;
	}
  /* 添加一条字符串类型的JSON数据(添加一个链表节点) */
	cJSON_AddStringToObject(human,"name",my_human->name);
  /* 添加一条整数类型的JSON数据(添加一个链表节点) */
	cJSON_AddNumberToObject(human,"age",my_human->age);
  /* 添加一条浮点类型的JSON数据(添加一个链表节点) */
	cJSON_AddNumberToObject(human,"heigh",my_human->heigh);
	
  /* 添加一个嵌套的JSON数据（添加一个链表节点） */
	address = cJSON_CreateObject();
	if(address == NULL)
	{
		LOG_E("cjson creat obj error!");
		return;
	}
	cJSON_AddStringToObject(address,"country",my_human->address->country);
	cJSON_AddNumberToObject(address,"zip-code",my_human->address->zip_code);
	cJSON_AddItemToObject(human,"address",address);

  /* 添加一个数组类型的JSON数据(添加一个链表节点) */
	skill = cJSON_CreateArray();
	if(skill == NULL)
	{
		LOG_E("cjson creat array error!");
		return;
	}
	for (i = 0; i < SKILL_NUM; i++)
	{
		cJSON_AddItemToArray(skill,cJSON_CreateString(&my_human->skill[i][0]));
	}
	cJSON_AddItemToObject(human,"skill",skill);

  /* 添加一个值为 False/True 的布尔类型的JSON数据(添加一个链表节点) */
	if(my_human->student == false){
		cJSON_AddFalseToObject(human, "student");
	}
	else{
		cJSON_AddTrueToObject(human, "student");
	}

	LOG_I("cJSON creat human success!");
	return human;
}

int main(void)
{
	cJSON* my_cjson_human = NULL;

	LOG_W("cjson test start!");

	my_cjson_human = create_json_human((const xp_human *)&my_human);
  /* 打印JSON对象(整条链表)的所有数据 */
	LOG_I("human\r\n%s",cJSON_Print(my_cjson_human));
   /* 打印JSON对象(整条链表)的所有数据(精简模式打印) */
	snprintf(cjson_get_str,512,"%s",cJSON_PrintUnformatted(my_cjson_human));
	LOG_I("human\r\n%s",cjson_get_str);
	LOG_W("cjson test stop!");
	cJSON_Delete(my_cjson_human);

	return 0;
}
```

编译运行([参考我另一篇关于Makefile的配置](http://xiaopb.xyz/2020/08/30/c-cpp/c-2/))：

```bash
make
```

实验结果如图：

```bash
gcc -g -Wall -O2  -I ./pack/cjson/inc -I ./pack/xp_log/inc -c -o debug/test.o main/src/test.c
g++ -g -Wall -O2 -o cjson-test debug/test.o debug/cJSON.o debug/xp_log.o
./cjson-test.exe
[W/main/main] cjson test start!
[I/main/create_json_human] cJSON creat human start!
[I/main/create_json_human] cJSON creat human success!
[I/main/main] human
{
        "name": "XiaoPb",
        "age":  23,
        "heigh":        170.500000,
        "address":      {
                "country":      "China",
                "zip-code":     111111
        },
        "skill":        ["c/cpp", "python", "stm32", "makefile"],
        "student":      true
}
[I/main/main] human
{"name":"XiaoPb","age":23,"heigh":170.500000,"address":{"country":"China","zip-code":111111},"skill":["c/cpp","python","stm32","makefile"],"student":true}
[W/main/main] cjson test stop!
```

## 解析数据和打印数据示例

下面用一个例子来说明，解析出上面封装好的数据字符串：

```c
/* 将解析到的数据存入该结构体内 */
xp_address user_addr;
xp_human user_human = {
	.address = &user_addr
};
void get_json_human(xp_human* user_human,const char *cjson_str)
{
	unsigned int i = 0;
	const char *error_ptr = NULL;
	cJSON *human = NULL; 
	cJSON *address = NULL;
	cJSON *skill = NULL;
	
	LOG_I("cJSON get human start!");
	/* 根据输入字符串创建一个JSON数据对象 */
	human = cJSON_Parse(cjson_str);
	if(human == NULL)
	{
		error_ptr = cJSON_GetErrorPtr();
        if (error_ptr != NULL)
        {
            LOG_I("Error before: %s\n", error_ptr);
        }
		return;
	}
  /* 在总JSON数据对象中查找name名字的JSON数据对象，并把其字符串复制到结构体元素中 */
	snprintf(user_human->name,10,"%s",cJSON_GetObjectItem(human, "name")->valuestring); 
  
  /* 在总JSON数据对象中查找age名字的JSON数据对象，并把其数值赋值到结构体元素中 */
	user_human->age = cJSON_GetObjectItem(human, "age")->valueint;
  
  /* 在总JSON数据对象中查找heigh名字的JSON数据对象，并把其数值赋值到结构体元素中 */
	user_human->heigh = cJSON_GetObjectItem(human, "heigh")->valuedouble;
  
	/* 在总JSON数据对象中查找address名字的JSON数据对象 */
	address = cJSON_GetObjectItem(human, "address");
  
  /* 在address的JSON数据对象中查找country名字的JSON数据对象，并把其字符串复制到结构体元素中 */
	snprintf(user_human->address->country,10,"%s",cJSON_GetObjectItem(address, "country")->valuestring);
  
  /* 在address的JSON数据对象中查找zip-code名字的JSON数据对象，并把其数值赋值到结构体元素中 */
	user_human->address->zip_code = cJSON_GetObjectItem(address, "zip-code")->valueint;

  /* 在总JSON数据对象中查找skill名字的JSON数据对象 */
	skill = cJSON_GetObjectItem(human, "skill");
  
	for (i = 0; i < cJSON_GetArraySize(skill); i++)
	{
    /* 在skill的JSON数据数组对象中提取第i个数据，并把其数值赋值到结构体元素中 */
		snprintf(&user_human->skill[i][0],10,"%s",cJSON_GetArrayItem(skill, i)->valuestring);
	}
  
  /* 在总JSON数据对象中查找student名字的JSON数据对象，并把其数值赋值到结构体元素中 */
	user_human->student = cJSON_GetObjectItem(human, "student")->valueint;

  /* 删除总JSON数据对象 */
	cJSON_Delete(human);
	LOG_I("cJSON get human success!");
}

/* 根据结构体内容定制输出函数 */
void pr_human_data(xp_human *user_human)
{
	unsigned int i;
	LOG_I("user human data start!");

	LOG_I("user name : %s",user_human->name);
	LOG_I("user age  : %d",user_human->age);
	LOG_I("user heigh: %.1f",user_human->heigh);
	LOG_I("user address country: %s",user_human->address->country);
	LOG_I("user address zip-code: %d",user_human->address->zip_code);
	for (i = 0; i < SKILL_NUM; i++)
	{
		LOG_I("user skill(%d): %s",i,&user_human->skill[i][0]);
	}
	LOG_I("user student: %s",user_human->student?"true":"false");

	LOG_I("user human data end!");
}
int main(void)
{
	cJSON* my_cjson_human = NULL;

	LOG_W("cjson test start!");

	my_cjson_human = create_json_human((const xp_human *)&my_human);
	LOG_I("human\r\n%s",cJSON_Print(my_cjson_human));
	snprintf(cjson_get_str,512,"%s",cJSON_PrintUnformatted(my_cjson_human));
	LOG_I("human\r\n%s",cjson_get_str);
	// [1]
	get_json_human(&user_human,cjson_get_str);
	pr_human_data(&user_human);
	// [2]
	LOG_W("cjson test stop!");
	cJSON_Delete(my_cjson_human);

	return 0;
}
```
从[1]到[2]的运行结果：

```bash
[I/main/get_json_human] cJSON get human start!
[I/main/get_json_human] cJSON get human success!
[I/main/pr_human_data] user human data start!
[I/main/pr_human_data] user name : XiaoPb
[I/main/pr_human_data] user age  : 23
[I/main/pr_human_data] user heigh: 170.5
[I/main/pr_human_data] user address country: China
[I/main/pr_human_data] user address zip-code: 111111
[I/main/pr_human_data] user skill(0): c/cpp
[I/main/pr_human_data] user skill(1): python
[I/main/pr_human_data] user skill(2): stm32
[I/main/pr_human_data] user skill(3): makefile
[I/main/pr_human_data] user student: true
[I/main/pr_human_data] user human data end!
```



### 参考资料

[1]JSON: https://www.json.org/

[2]mculover666:[妙哉！cJSON设计思想解读及封装JSON数据方法示例](https://mp.weixin.qq.com/s?__biz=MzUyMTE0NTA2Ng==&mid=2247484273&idx=1&sn=1b43b0241c7dde121edfccf01637f5b9&chksm=f9dedcf4cea955e23ca747962bf482bf1b6c25fb7d8f60c0324d7b682dd4c4c95364d499de37&scene=21#wechat_redirect)





