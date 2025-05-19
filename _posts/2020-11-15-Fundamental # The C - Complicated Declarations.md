---
layout: post
title:  "Fundamental | the C - Complicated Declarations"
date:   2020-11-15 09:25:00 +0800
---
```c
char (*(*x())[])()
char name() function return char
(*name) pointer to 
name[] array contains
*x() function return pointer
```



```
char (*(*x[3])())[5]
char name[5]  array of 5 chars
(*name()) function return pointer to  
*x[3] array contains 3 pointer to
```



从外向里：后一步只需将前一步的 name 展开，其余不要

从里向外：每一级之间都是上下级关系，不是 等于（这里的等于可以替换为表述：是... ） 的关系。比如(*x[3])()是数组里的3个指针都指向函数，而不是函数返回指针

上下级关系的描述通常有：函数返回、指针指向、数组包含

online tool to describe: https://cdecl.org/