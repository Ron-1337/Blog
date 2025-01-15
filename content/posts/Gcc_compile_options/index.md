---
title: "Gcc 编译保护选项"
date: 2025-01-15
draft: false
description: "Gcc 编译保护选项"
summary: "Gcc 编译保护选项"
tags: ["notes", "ctf"]
categories: ["notes"]
showAuthor: false
---

> 本文摘自[linux程序保护机制&gcc编译选项](https://www.jianshu.com/p/91fae054f922)
>
> 原作者: HAPPYers

## 总结

```
NX：-z execstack / -z noexecstack (关闭 / 开启)
Canary：-fno-stack-protector /-fstack-protector / -fstack-protector-all (关闭 / 开启 / 全开启)
PIE：-no-pie / -pie (关闭 / 开启)
RELRO：-z norelro / -z lazy / -z now (关闭 / 部分开启 / 完全开启)
```

## Canary

gcc在4.2版本中添加了-fstack-protector和-fstack-protector-all编译参数以支持栈保护功能，4.9新增了-fstack-protector-strong编译参数让保护的范围更广。
编译控制选项

```shell
gcc -o test test.c // 默认情况下，不开启Canary保护
gcc -fno-stack-protector -o test test.c //禁用栈保护
gcc -fstack-protector -o test test.c //启用堆栈保护，不过只为局部变量中含有 char 数组的函数插入保护代码
gcc -fstack-protector-all -o test test.c //启用堆栈保护，为所有函数插入保护代码
```

## Fortify

fority其实非常轻微的检查，用于检查是否存在缓冲区溢出的错误。适用情形是程序采用大量的字符串或者内存操作函数，如memcpy，memset，stpcpy，strcpy，strncpy，strcat，strncat，sprintf，snprintf，vsprintf，vsnprintf，gets以及宽字符的变体。

- ```_FORTIFY_SOURCE``` 设为1，并且将编译器设置为优化1(gcc -O1)，以及出现上述情形，那么程序编译时就会进行检查但又不会改变程序功能
- ```_FORTIFY_SOURCE``` 设为2，有些检查功能会加入，但是这可能导致程序崩溃。

```gcc -D_FORTIFY_SOURCE=1``` 仅仅只会在编译时进行检查 (特别像某些头文件 #include <string.h>)
```gcc -D_FORTIFY_SOURCE=2``` 程序执行时也会有检查 (如果检查到缓冲区溢出，就终止程序)

```shell
gcc -o test test.c // 默认情况下，不会开这个检查
gcc -D_FORTIFY_SOURCE=1 -o test test.c // 较弱的检查
gcc -D_FORTIFY_SOURCE=2 -o test test.c // 较强的检查
```

## NX(DEP)

NX即No-eXecute（不可执行）的意思，NX（DEP）的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。
gcc编译器默认开启了NX选项，如果需要关闭NX选项，可以给gcc编译器添加```-z execstack```参数

```shell
gcc -o test test.c // 默认情况下，开启NX保护
gcc -z execstack -o test test.c // 禁用NX保护
gcc -z noexecstack -o test test.c // 开启NX保护
```

## PIE(ASLR)

内存地址随机化机制（address space layout randomization)，有以下三种情况

```
0 - 表示关闭进程地址空间随机化。
1 - 表示将mmap的基址，stack和vdso页面随机化。
2 - 表示在1的基础上增加栈（heap）的随机化。
```

- Linux关闭PIE的方法如下

```shell
sudo -s echo 0 > /proc/sys/kernel/randomize_va_space
```

- gcc编译选项

```shell
gcc -o test test.c // 默认情况下，不开启PIE
gcc -fpie -pie -o test test.c // 开启PIE，此时强度为1
gcc -fPIE -pie -o test test.c // 开启PIE，此时为最高强度2
gcc -fpic -o test test.c // 开启PIC，此时强度为1，不会开启PIE
gcc -fPIC -o test test.c // 开启PIC，此时为最高强度2，不会开启PIE
```

### 说明

PIE最早由RedHat的人实现，他在连接起上增加了-pie选项，这样使用-fPIE编译的对象就能通过连接器得到位置无关可执行程序。fPIE和fPIC有些不同。可以参考[Gcc和Open64中的-fPIC选项](http://writeblog.csdn.net/2009/11/20/10065/).

gcc中的-fpic选项，使用于在目标机支持时，编译共享库时使用。编译出的代码将通过全局偏移表(Global Offset Table)中的常数地址访存，动态装载器将在程序开始执行时解析GOT表项(注意，动态装载器操作系统的一部分，连接器是GCC的一部分)。而gcc中的-fPIC选项则是针对某些特殊机型做了特殊处理，比如适合动态链接并能避免超出GOT大小限制之类的错误。而Open64仅仅支持不会导致GOT表溢出的PIC编译。

gcc中的-fpie和-fPIE选项和fpic及fPIC很相似，但不同的是，除了生成为位置无关代码外，还能假定代码是属于本程序。通常这些选项会和GCC链接时的-pie选项一起使用。fPIE选项仅能在编译可执行码时用，不能用于编译库。所以，如果想要PIE的程序，需要你除了在gcc增加-fPIE选项外，还需要在ld时增加-pie选项才能产生这种代码。即gcc -fpie -pie来编译程序。单独使用哪一个都无法达到效果。

## RELRO

GCC, GNU linker以及Glibc-dynamic linker一起配合实现了一种叫做relro的技术: read only relocation。大概实现就是由linker指定binary的一块经过dynamic linker处理过 relocation之后的区域为只读.
设置符号重定向表格为只读或在程序启动时就解析并绑定所有动态符号，从而减少对GOT（Global Offset Table）攻击。RELRO为” Partial RELRO”，说明我们对GOT表具有写权限。

gcc编译选项

```shell
gcc -o test test.c // 默认情况下，是Partial RELRO
gcc -z norelro -o test test.c // 关闭，即No RELRO
gcc -z lazy -o test test.c // 部分开启，即Partial RELRO
gcc -z now -o test test.c // 全部开启，即Full RELRO
```