---
layout: post
title: 'Different iOS SDK Version Compile'
date: '2015-10-08'
header-img: "img/home-bg.jpg"
tags:
     - iOS
     
author: '778477'
---

**前言**
- - -
我们在开发feature的时候依赖 iOS9 SDK中的新framework，注意这个framework只有iOS9才有。但是我们有两套打包环境

* Xcode6 - iOS8
* Xcode7 - iOS9

如果不做任何处理的话，在第一套打包环境下会出现 'XX.h' file not found.

我们希望编写的代码能够兼容这两套打包环境. 

Google 区分iOS系统版本 得到以下结果：

[iPhone开发技巧之环境篇（7）--- 区分不同版本的iPhone](http://www.yifeiyang.net/iphone-development-techniques-of-environmental-chapter-7-distinguish-between-different-versions-of-the-iphone/)

[区分ios设备，os版本，sdk版本](http://justsee.iteye.com/blog/1621533)

我们只关心文章最后的段落:iPhone OS SDK版本，我们希望在 iOS9下 编译我们的feature代码，在iOS9之前的系统版本不编译，不执行。

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_2_2
    #if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_3_1
        // iPhone OS SDK 4.0 以后版本的处理
    #else
        // iPhone OS SDK 3.0 ~ 4.0 版本的处理
    #endif
#else
    // iPhone OS SDK 3.0 之前版本的处理
#endif
```

但是，这还是有一个坑。 我们来看一下 具体的宏定义在 ```AvailabilityInternal.h```中。

```
 #ifdef __IPHONE_OS_VERSION_MIN_REQUIRED
    /* make sure a default max version is set */
    #ifndef __IPHONE_OS_VERSION_MAX_ALLOWED
        #define __IPHONE_OS_VERSION_MAX_ALLOWED     __IPHONE_9_0
    #endif
    /* make sure a valid min is set */
    #if __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_2_0
        #undef __IPHONE_OS_VERSION_MIN_REQUIRED
        #define __IPHONE_OS_VERSION_MIN_REQUIRED    __IPHONE_2_0 
    #endif
```
 
我查看的是iOS 9.0的定义，里面定义 ```__IPHONE_OS_VERSION_MAX_ALLOWED``` 为 ```__IPHONE_9_0```(90000)
 
相信每个iOS SDK发布，都会增加 一个宏来表示版本号
 
```
#define __IPHONE_2_0     20000
#define __IPHONE_2_1     20100
#define __IPHONE_2_2     20200
#define __IPHONE_3_0     30000
#define __IPHONE_3_1     30100
#define __IPHONE_3_2     30200
#define __IPHONE_4_0     40000
#define __IPHONE_4_1     40100
#define __IPHONE_4_2     40200
#define __IPHONE_4_3     40300
#define __IPHONE_5_0     50000
#define __IPHONE_5_1     50100
#define __IPHONE_6_0     60000
#define __IPHONE_6_1     60100
#define __IPHONE_7_0     70000
#define __IPHONE_7_1     70100
#define __IPHONE_8_0     80000
#define __IPHONE_8_1     80100
#define __IPHONE_8_2     80200
#define __IPHONE_8_3     80300
#define __IPHONE_8_4     80400
#define __IPHONE_9_0     90000
/* __IPHONE_NA is not defined to a value but is uses as a token by macros to indicate that the API is unavailable */
```

问题来了：如果兼容代码中使用的 宏定义为 ```__IPHONE_8_4```，而我编译的iOS SDK使用的是 iOS8.3的话，```__IPHONE_8_4```其实是没定义的。

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_8_4
```
这个比较语句其实取 ```__IPHONE_OS_VERSION_MAX_ALLOWED``` = ```__IPHONE_8_3``` = 80300 和 ```__IPHONE_8_4```所表示的数字做比较，是没有作用的。正确做法 **是直接使用数字**

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED > 80400
```

