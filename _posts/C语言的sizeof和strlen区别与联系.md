---
title: C语言的sizeof和strlen区别与联系
date: 2012-06-14 09:57:00
categories:
 - IT Technology
tags:
 - C
toc: false
---

sizeof指的是占有空间的大小，包括字符串结束的\0。strlen是计算字符串长度，以`\0`作为结束标志，并且`\0`不计入数值。

<!-- more -->

```cpp
#include <stdio.h>  
#include <iostream>  
using namespace std;  
  
void fun(char a[100]){  
     cout << sizeof(a) << endl;     
     // 参数里的数组也是按指针传值的。所以复制的是一个指针， 输出4   
     cout << strlen(a) << endl;    
     // 5   
}   
  
int main()  
{  
    char *p1 = "Hello world!";  
    char p2[] = "Hello world!";  
    char p3[] = {'H','e','l','l','o',' ','w','o','r','l','d','!'};  
    char p4[] = {'H','e','l','l','o','\0','w','o','r','l','d','!'};  
    char p5[100] = {'H','e','l','l','o','\0','w','o','r','l','d','!'};  
    char *p6 = "Hello world!";  
    cout << int(p1) << "\t" << int(p6) << endl;    
    // 两个值相同，所以p1和p6两个指针指向同一块静态存储区。  
     
    //p1[2]='T';    报错！内存错误，静态存储区内只能存储常量。   
    
    cout << sizeof(p1) << endl;   
    // 字符串存储在静态存储区，而栈中存放的是一个指针，在32位机上，输出4   
      
    cout << sizeof(p2) << endl;  
    // p2数组全部存放在栈空间上，包括\0共占用13个字节的空间，输出13  
       
    cout << sizeof(p3) << endl;   
    // p3是一个字符数组，共含有12个元素，所以输出12  
       
    cout << sizeof(p4) << endl;    
    // p4是一个字符数组，共含有12个元素，所以输出12  
      
    cout << sizeof(p5) << endl;    
    // 栈中开辟了100字节大小的内存，所以输出100   
      
    cout << sizeof(short) << endl;   
    // short = 2字节   
    cout << sizeof(int) << endl;    
    // int = 4字节   
    cout << sizeof(long) << endl;    
    // long=4字节   
    cout << sizeof(long long) << endl;    
    // 8   
    cout << sizeof(__int64) << endl;   
    //8   
    cout << sizeof(float) << endl;    
    // 4   
    cout << sizeof(double) << endl;    
    // 8   
          
    cout << strlen(p1) << endl;  
    // 静态存储区内，在！后含有了\0所以输出12   
    cout << strlen(p2) << endl;  
    // 12   
    cout << strlen(p3) << endl;  
    // 不含有\0  所以这种输出方式 disaster 灾难！！输出结果不确定，这里是28   
    cout << strlen(p4) << endl;    
    // \0之前含有Hello所以长度为5   
    cout << strlen(p5) << endl;    
    // 5     
    fun(p5);   
      
    system("pause");   
    return 0;   
}   
```
