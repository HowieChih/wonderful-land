---
layout: post
title:  "Fundamental | the C - bit fields mixed with padding"
date:   2020-11-16 09:25:00 +0800
---
There are three principles to follow when analyzing memory usage while bit fields mixed with padding after compiler processing:

1、对于alignment为K的任意类型的变量，他的起始地址（initial address）必须为K的倍数（initial address & structure length must be multiples of K），对于基本类型，size和alignment是一致的，比如char为1(byte)，int为4(bytes)。所以对于以下结构，64位机器上，sizeof是8 bytes：

```c
struct {
    char c;
    int i;
} base;
```

内存分布如下：

| byte index | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
| ---------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|            | c    | -    | -    | -    | i    | i    | i    | i    |

2、``int f: 1`` says that the bit-field f must be within an int. If entire bytes of space remains, a following char within the same struct will be packed inside this int, even if it is not a bit-field.

比如对于以下结构，在64位机器上，sizeof是4 bytes：

```c
struct {
    int a : 1;
    int b : 2;
    char c;
} base;
```

内存分布如下：

```
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 
|a b b x x x x x|c c c c c c c c|x x x x x x x x|x x x x x x x x|               

<------------------------------ int ---------------------------->
```

而且bit field存储类型的判定，遵循的规则应该是struct能尽可能多的容纳数据。

比如下面结构存储类型会被判定位int

```
struct {
    short a : 1;
    int b : 2;
}
```

bit field struct里没有指定位的字段才会根据规则[1]分配起始位置，如果都是bit field，不会根据规则[1]分配起始位置，而是紧跟前面字段之后。

比如下面的结构sizeof为2，short的长度，不会说short b要从第二个字节开始。

```
struct {
    char a : 1;
    short b : 1;
} base;
```

总结就是：以尽可能大的一个数据类型，塞下更多结构字段。塞下更多结构字段的时候，普通类型比如char c要按照起始位置规则排，而bit field字段就紧跟。

当按上面总结放不下所有字段数据时，扩容规则就是再扩一个已判定类型的空间。比如sizeof是4

```
struct {
    short a : 8;
    short b : 1;
    char c;
} base;
```

```
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7  
|a a a a a a a a|b - - - - - - -|c c c c c c c c|- - - - - - - -|                
<----------- short--------------><----------- short-------------->
```

3、当结构里只有一个字段时，该结构的sizeof值以该字段类型为准。

```
// 4 bytes
struct {
    int a;
}

// 2 bytes
struct { 
    short a : 1;
}
```

ref: https://stackoverflow.com/questions/54054427/different-between-c-struct-bitfields-on-char-and-on-int

**to read**: http://www.catb.org/esr/structure-packing/