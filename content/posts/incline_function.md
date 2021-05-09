---
title: "char*,const char*,string,memset等用法"
date: 2019-08-02T11:16:54+08:00
draft: false
tags: ["basic method","char*","const char*"]
categories: ["C++相关"]
author: "Jason·Gan"
---  
* 前言：之前用到一些C++的库时，会涉及到一些数据类型的转换问题，由于一些不常用导致不是很熟悉，经过此次教训后，想要记录下来，下次翻找就会方便点。  

## char*，const char*和string 三者转换  

* const char* 转 string。直接赋值即可，string有较大的包容性，不像const char*这么严格： 
```
const char* test="Hello world";   
string s=test;  
```  
* string转为const char*。使用方法c_str():  
```
string test="Hello world";
const char* ch=test.c_str();  
```
* char* 转为const char*。直接赋值：
```
char* test="hello world";
const char* ch=test;
```
* const char* 转为char* 。用const_cast<char*>
```
const char* test="hello wordl";
char* ch=const_cast<char*>(test);
```
* char*转为string:  
```
char* test="Hello world";
string s=test;  //我说啦string的包容性比较好
```
* string转为char*。分两步走，先把string转为const char*，再把const char * 转为char *：
```
string s="Hello world";
const char* ch=s.c_str();
char* test=const_cast<char*>(ch);
```  
## memset,memcpy,strcpy,memcmp等C函数的用法 
###  Memset
* 函数声明：```extern void *memset(void *buffer, int c, int count); ``` 在头文件<string.h>中。  
  用法:把buffer所指内存区域的前count个字节设置成字符c，返回指向buffer的指针。用来对一段内存空间全部设置为某个字符：  
  ```
  char a[10];
  memset(a, '\0', sizeof(a));
  ```
### Memcpy
* 函数声明：```extern void *memcpy(void* test,void* src,unsigned int count);``` 在头文件<string.h>中  
  用法：由src所指内存区域复制count个字节到test所指内存区域。src和test所指内存区域不能重叠，函数返回指向test的指针.可以拿它拷贝任何数据类型的对象。
  ```
  char a[10],b[5];
  memcpy(b, a, sizeof(b));
  ```
### Strcpy
* 函数声明：```extern char *strcpy(char *test,char *src);```在头文件<string.h>中  
  用法：把src所指由NULL结束（'\0'）的字符串复制到dest所指的数组中。src和test所指内存区域不可以重叠且dest必须有足够的空间来容纳 src的字符串.返回指向test的指针。
  ```
  char test[3];
  char src[6]="Hello";
  strcpy(test,src);
  ```
  这样是可以复制的，理论上不合规定，因为得确保test指向空间得足够存放src的东西，但实际上是可以复制成功，前提是src有一个结束符。Strcpy的源码是先复制，再判断是否遍历到了结束符'\0'，这样就会动态的扩展了test指向空间,也就是这样不会报错。

* 三者区别  
  memset   主要应用是初始化某个内存空间。                  
  memcpy   是用于copy源空间的数据到目的空间中，且指定复制的长度。                  
  strcpy   用于字符串copy，遇到‘\0’，将结束，不指定长度。

### Memcmp
* 函数声明：```extern int memcmp(void *buf1, void *buf2, unsigned int count);``` 包含在头文件<string.h>中  
  用法：比较内存区域buf1和buf2的前count个字节。    
  **当buf1< buf2时，返回值<0**  
  **当buf1=buf2时，返回值=0**    
  **当buf1>buf2时，返回值>0**
  






