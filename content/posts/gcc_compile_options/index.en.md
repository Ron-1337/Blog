---
title: "Gcc Compiler Protection Options"
date: 2025-01-15
draft: false
description: "Gcc Compiler Protection Options"
summary: "Gcc Compiler Protection Options"
tags: ["notes", "ctf"]
categories: ["notes"]
showAuthor: false
---

> This article is excerpted from [linux program protection mechanism & gcc compile options](https://www.jianshu.com/p/91fae054f922)
>
> Original author: HAPPYers

{{< alert >}}
**WARNING！** This is a machine translated version of the [Chinese page](/posts/gcc_compile_options/)
{{< /alert >}}

## Summary

```
NX: -z execstack / -z noexecstack (off / on)
Canary: -fno-stack-protector / -fstack-protector / -fstack-protector-all (off / on / all on)
PIE: -no-pie / -pie (off / on)
RELRO: -z norelro / -z lazy / -z now (off / partially on / fully on)
```

## Canary

In version 4.2, gcc added the -fstack-protector and -fstack-protector-all compilation parameters to support stack protection. In version 4.9, the -fstack-protector-strong compilation parameter was added to make the protection wider.
Compilation control options

```shell
gcc -o test test.c // By default, Canary protection is not enabled
gcc -fno-stack-protector -o test test.c // Disable stack protection
gcc -fstack-protector -o test test.c // Enable stack protection, but only insert protection code for functions with char arrays in local variables
gcc -fstack-protector-all -o test test.c // Enable stack protection and insert protection code for all functions
```

## Fortify

Fortify is actually a very light check, used to check whether there is a buffer overflow error. This is useful for programs that use a lot of string or memory manipulation functions, such as memcpy, memset, stpcpy, strcpy, strncpy, strcat, strncat, sprintf, snprintf, vsprintf, vsnprintf, gets, and wide character variants.

- ```_FORTIFY_SOURCE``` is set to 1, and the compiler is set to optimization 1 (gcc -O1), and the above situation occurs, then the program will be checked when compiled but the program functionality will not be changed
- ```_FORTIFY_SOURCE``` is set to 2, some checks will be added, but this may cause the program to crash.

```gcc -D_FORTIFY_SOURCE=1``` will only check when compiling (especially for some header files #include <string.h>)
```gcc -D_FORTIFY_SOURCE=2``` will also check when the program is executed (if buffer overflow is detected, the program will be terminated)

```shell
gcc -o test test.c // By default, this check is not turned on
gcc -D_FORTIFY_SOURCE=1 -o test test.c // Weaker check
gcc -D_FORTIFY_SOURCE=2 -o test test.c // Stronger check
```

## NX(DEP)

NX stands for No-eXecute. The basic principle of NX (DEP) is to mark the memory page where the data is located as non-executable. When the program overflows successfully and transfers to the shellcode, the program will try to execute instructions on the data page. At this time, the CPU will throw an exception instead of executing malicious instructions.
The gcc compiler has the NX option enabled by default. If you need to disable the NX option, you can add the ```-z execstack``` parameter to the gcc compiler

```shell
gcc -o test test.c // Enable NX protection by default
gcc -z execstack -o test test.c // Disable NX protection
gcc -z noexecstack -o test test.c // Enable NX protection
```

## PIE(ASLR)

The address space layout randomization mechanism has the following three conditions

```
0 - means turning off process address space randomization.
1 - means randomizing the base address of mmap, stack and vdso pages.
2 - means adding stack (heap) randomization on the basis of 1.
```

- How to turn off PIE in Linux is as follows

```shell
sudo -s echo 0 > /proc/sys/kernel/randomize_va_space
```

- gcc compilation options

```shell
gcc -o test test.c // By default, PIE is not enabled
gcc -fpie -pie -o test test.c // Enable PIE, strength is 1 at this time
gcc -fPIE -pie -o test test.c // Enable PIE, maximum strength is 2 at this time
gcc -fpic -o test test.c // Enable PIC, strength is 1 at this time, PIE will not be enabled
gcc -fPIC -o test test.c // Enable PIC, maximum strength is 2 at this time, PIE will not be enabled
```

### Description

PIE was first implemented by RedHat people, who added the -pie option to the linker, so that objects compiled with -fPIE can get position-independent executable programs through the linker. fPIE and fPIC are somewhat different. Please refer to [-fPIC option in Gcc and Open64](http://writeblog.csdn.net/2009/11/20/10065/).

The -fpic option in gcc is used when compiling shared libraries when the target machine supports it. The compiled code will access memory through the constant address in the Global Offset Table, and the dynamic loader will parse the GOT table entry when the program starts executing (note that the dynamic loader is part of the operating system, and the linker is part of GCC). The -fPIC option in gcc is specially processed for some special models, such as being suitable for dynamic linking and avoiding errors such as exceeding the GOT size limit. Open64 only supports PIC compilation that does not cause GOT table overflow.

The -fpie and -fPIE options in gcc are very similar to fpic and fPIC, but the difference is that in addition to generating position-independent code, they can also assume that the code belongs to the program. Usually these options are used together with the -pie option when GCC is linked. The fPIE option can only be used when compiling executable code, not for compiling libraries. Therefore, if you want a PIE program, you need to add the -pie option to ld in addition to the -fPIE option in gcc to generate this code. That is, use gcc -fpie -pie to compile the program. Using either one alone will not achieve the desired effect.

## RELRO

GCC, GNU linker, and Glibc-dynamic linker work together to implement a technology called relro: read only relocation. The general implementation is that the linker specifies a binary area that has been processed by the dynamic linker after relocation as read-only.
Set the symbol redirection table to read-only or parse and bind all dynamic symbols at program startup to reduce attacks on the GOT (Global Offset Table). RELRO is "Partial RELRO", which means that we have write permission to the GOT table.

gcc compile options

```shell
gcc -o test test.c // By default, it is Partial RELRO
gcc -z norelro -o test test.c // Disable, i.e. No RELRO
gcc -z lazy -o test test.c // Enable partially, i.e. Partial RELRO
gcc -z now -o test test.c // Enable all, i.e. Full RELRO
```