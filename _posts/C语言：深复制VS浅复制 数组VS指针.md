---
title: C语言：深复制VS浅复制 数组VS指针
date: 2012-04-19 16:45:00
categories:
 - IT Technology
tags:
 - C
toc: false
---

Shallow copy VS. deep copy

计算机是32位的，编译器是dev-cpp。

<!-- more -->

C源码：

```c
#include <stdio.h>  
  
struct student{  
       char *name1;  
       char name2[20];  
}stu;  
  
int main()  
{  
    struct student *p;  
    p = &stu;  
    //strcpy((*p).name1,"first");// 编译正确，运行错误，因为没有分配空间。  
    strcpy((*p).name2,"second");  // 深复制：复制数组所有的字符   
    printf("name1 = %s \t name2 = %s\n",(*p).name1,(*p).name2);  
      
    p->name1 = "third";   // 浅复制，只复制内存地址   
    printf("name1 = %s \t name2 = %s\n",(*p).name1,(*p).name2);  
    //p->name2 = "fourth";  //编译错误： incompatible types in assignment of `const char[7]' to `char[20]'   
    char tp[20]="fifth";    
    //p->name2 = tp;  //编译错误：ISO C++ forbids assignment of arrays   
    // 深复制和浅复制的区别   
    strcpy((*p).name2,tp);  
    p->name1 = tp;  
    tp[0]='F';//修改tp字符串的值  
    printf("name1 = %s \t name2 = %s\n",(*p).name1,(*p).name2); // 浅复制的值发生改变，深复制不会  
      
    // 字符串数组和指针的区别  
    printf("sizeof(char *name) = %d\nsizeof(char name[20]) = %d\n", sizeof(p->name1), sizeof(p->name2)); // 32位计算机4和20   
    system("PAUSE");  
    return 0;  
}  
```

运行结果为：

```c
name1 = (null)   name2 = second  
name1 = third    name2 = second  
name1 = Fifth    name2 = fifth  
sizeof(char *name) = 4  
sizeof(char name[20]) = 20  
```
