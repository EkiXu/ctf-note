# 协议相关

## HTTP

- Header

Header|	解释|	示例|
|--|--|--|
Accept|	指定客户端能够接收的内容类型|	Accept: text/plain, text/html,application/json|
Accept-Charset|	浏览器可以接受的字符编码集。|	Accept-Charset: iso-8859-5|
Accept-Encoding|	指定浏览器可以支持的web服务器返回内容压缩编码类型。|	Accept-Encoding: compress, gzip|
Accept-Language|	浏览器可接受的语言|	Accept-Language: en,zh|
Accept-Ranges|	可以请求网页实体的一个或者多个子范围字段|	Accept-Ranges: bytes|
Authorization|	HTTP授权的授权证书	|Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==|
Cache-Control|	指定请求和响应遵循的缓存机制|	Cache-Control: no-cache|
Connection|	表示是否需要持久连接。（HTTP 1.1默认进行持久连接）|	Connection: close|
Cookie|	HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。|	Cookie: $Version=1; Skin=new;|
Content-Length|	请求的内容长度|	Content-Length: 348|
Content-Type|	请求的与实体对应的MIME信息|	Content-Type: application/x-www-form-urlencoded|
Date|	请求发送的日期和时间|	Date: Tue, 15 Nov 2010 08:12:31 GMT|
Expect|	请求的特定的服务器行为|	Expect: 100-continue|
From|	发出请求的用户的Email|	From: user@email.com|
Host|	指定请求的服务器的域名和端口号|	Host: www.zcmhi.com|
If-Match|	只有请求内容与实体相匹配才有效|	If-Match: “737060cd8c284d8af7ad3082f209582d”|
If-Modified-Since|	如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码|	If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
If-None-Match|	如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变|	If-None-Match: “737060cd8c284d8af7ad3082f209582d”|
If-Range|	如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag|	If-Range: “737060cd8c284d8af7ad3082f209582d”|
If-Unmodified-Since|	只在实体在指定时间之后未被修改才请求成功|	If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
Max-Forwards|	限制信息通过代理和网关传送的时间|	Max-Forwards: 10|
Pragma|	用来包含实现特定的指令|	Pragma: no-cache|
Proxy-Authorization|	连接到代理的授权证书|	Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==|
Range|	只请求实体的一部分，指定范围|	Range: bytes=500-999|
Referer|	先前网页的地址，当前请求网页紧随其后,即来路|	Referer: http://www.zcmhi.com/archives…|
TE|	客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息|	TE: trailers,deflate;q=0.5|
Upgrade|	向服务器指定某种传输协议以便服务器进行转换（如果支持）|	Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11|
User-Agent|	User-Agent的内容包含发出请求的用户信息|	User-Agent: Mozilla/5.0 (Linux; X11)|
Via|	通知中间网关或代理服务器地址，通信协议|	Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)|
Warning|	关于消息实体的警告信息|	Warn: 199 Miscellaneous warning|
X-REAL-IP|||
Client-IP|||


## TLS

## Gopher

- 格式

    ```
    gopher://127.0.0.1:70/_ + TCP/IP数据
    ```

    ``_``可以是任意字符，作为连接符占位

    一个示例
    
    ```
    
    GET /?test=123 HTTP/1.1
    Host: 127.0.0.1:2222
    Pragma: no-cache
    Cache-Control: no-cache
    DNT: 1
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
    Accept-Encoding: gzip, deflate
    Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
    Connection: close


    ```
    
    编码后

    ```
    %47%45%54%20%2f%3f%74%65%73%74%3d%31%32%33%20%48%54%54%50%2f%31%2e%31%0d%0a%48%6f%73%74%3a%20%31%32%37%2e%30%2e%30%2e%31%3a%32%32%32%32%0d%0a%50%72%61%67%6d%61%3a%20%6e%6f%2d%63%61%63%68%65%0d%0a%43%61%63%68%65%2d%43%6f%6e%74%72%6f%6c%3a%20%6e%6f%2d%63%61%63%68%65%0d%0a%44%4e%54%3a%20%31%0d%0a%55%70%67%72%61%64%65%2d%49%6e%73%65%63%75%72%65%2d%52%65%71%75%65%73%74%73%3a%20%31%0d%0a%55%73%65%72%2d%41%67%65%6e%74%3a%20%4d%6f%7a%69%6c%6c%61%2f%35%2e%30%20%28%57%69%6e%64%6f%77%73%20%4e%54%20%31%30%2e%30%3b%20%57%69%6e%36%34%3b%20%78%36%34%29%20%41%70%70%6c%65%57%65%62%4b%69%74%2f%35%33%37%2e%33%36%20%28%4b%48%54%4d%4c%2c%20%6c%69%6b%65%20%47%65%63%6b%6f%29%20%43%68%72%6f%6d%65%2f%38%33%2e%30%2e%34%31%30%33%2e%36%31%20%53%61%66%61%72%69%2f%35%33%37%2e%33%36%0d%0a%41%63%63%65%70%74%3a%20%74%65%78%74%2f%68%74%6d%6c%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%68%74%6d%6c%2b%78%6d%6c%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%6d%6c%3b%71%3d%30%2e%39%2c%69%6d%61%67%65%2f%77%65%62%70%2c%69%6d%61%67%65%2f%61%70%6e%67%2c%2a%2f%2a%3b%71%3d%30%2e%38%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%73%69%67%6e%65%64%2d%65%78%63%68%61%6e%67%65%3b%76%3d%62%33%3b%71%3d%30%2e%39%0d%0a%41%63%63%65%70%74%2d%45%6e%63%6f%64%69%6e%67%3a%20%67%7a%69%70%2c%20%64%65%66%6c%61%74%65%0d%0a%41%63%63%65%70%74%2d%4c%61%6e%67%75%61%67%65%3a%20%7a%68%2d%43%4e%2c%7a%68%3b%71%3d%30%2e%39%2c%65%6e%2d%55%53%3b%71%3d%30%2e%38%2c%65%6e%3b%71%3d%30%2e%37%0d%0a%43%6f%6e%6e%65%63%74%69%6f%6e%3a%20%63%6c%6f%73%65%0d%0a%0d%0a
    ```

    test
    ```
    curl gopher://127.0.0.1:2222/_%47%45%54%20%2f%3f%74%65%73%74%3d%31%32%33%20%48%54%54%50%2f%31%2e%31%0d%0a%48%6f%73%74%3a%20%31%32%37%2e%30%2e%30%2e%31%3a%32%32%32%32%0d%0a%50%72%61%67%6d%61%3a%20%6e%6f%2d%63%61%63%68%65%0d%0a%43%61%63%68%65%2d%43%6f%6e%74%72%6f%6c%3a%20%6e%6f%2d%63%61%63%68%65%0d%0a%44%4e%54%3a%20%31%0d%0a%55%70%67%72%61%64%65%2d%49%6e%73%65%63%75%72%65%2d%52%65%71%75%65%73%74%73%3a%20%31%0d%0a%55%73%65%72%2d%41%67%65%6e%74%3a%20%4d%6f%7a%69%6c%6c%61%2f%35%2e%30%20%28%57%69%6e%64%6f%77%73%20%4e%54%20%31%30%2e%30%3b%20%57%69%6e%36%34%3b%20%78%36%34%29%20%41%70%70%6c%65%57%65%62%4b%69%74%2f%35%33%37%2e%33%36%20%28%4b%48%54%4d%4c%2c%20%6c%69%6b%65%20%47%65%63%6b%6f%29%20%43%68%72%6f%6d%65%2f%38%33%2e%30%2e%34%31%30%33%2e%36%31%20%53%61%66%61%72%69%2f%35%33%37%2e%33%36%0d%0a%41%63%63%65%70%74%3a%20%74%65%78%74%2f%68%74%6d%6c%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%68%74%6d%6c%2b%78%6d%6c%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%6d%6c%3b%71%3d%30%2e%39%2c%69%6d%61%67%65%2f%77%65%62%70%2c%69%6d%61%67%65%2f%61%70%6e%67%2c%2a%2f%2a%3b%71%3d%30%2e%38%2c%61%70%70%6c%69%63%61%74%69%6f%6e%2f%73%69%67%6e%65%64%2d%65%78%63%68%61%6e%67%65%3b%76%3d%62%33%3b%71%3d%30%2e%39%0d%0a%41%63%63%65%70%74%2d%45%6e%63%6f%64%69%6e%67%3a%20%67%7a%69%70%2c%20%64%65%66%6c%61%74%65%0d%0a%41%63%63%65%70%74%2d%4c%61%6e%67%75%61%67%65%3a%20%7a%68%2d%43%4e%2c%7a%68%3b%71%3d%30%2e%39%2c%65%6e%2d%55%53%3b%71%3d%30%2e%38%2c%65%6e%3b%71%3d%30%2e%37%0d%0a%43%6f%6e%6e%65%63%74%69%6f%6e%3a%20%63%6c%6f%73%65%0d%0a%0d%0a
    
    ->
    HTTP/1.1 200 OK
    Host: 127.0.0.1:2222
    Date: Tue, 26 May 2020 03:53:05 GMT
    Connection: close
    X-Powered-By: PHP/7.3.15-3
    Content-type: text/html; charset=UTF-8

    123
    ```


- 默认端口:``70``


### 编码脚本

```python
#!/usr/bin/python
# -*- coding:utf8 -*-

import getopt
import sys
import re

def togopher():
    try:
        opts,args = getopt.getopt(sys.argv[1:], "hf:s:", ["help", "file=", "stream="])
    except:
        print """
        Usage: python togopher.py -f <filename>
               python togopher.py -s <Byte stream>
               python togopher.py -h
        """
        sys.exit()
    
    if len(opts) == 0:
        print "Usage: python togopher.py -h"

    for opt,value in opts:
        if opt in ("-h", "--help"):
            print """
            Usage: 
            -h     --help     帮助
            -f     --file     数据包文件名
            -s     --stream   从流量包中得到的字节流
            """
            sys.exit()
        if opt in ("-f", "--file"):
            if not value:
                print "Usage: -f <filename>"
                sys.exit()
            words = ""
            with open(value, "r") as f:
                for i in f.readlines():
                    for j in i:
                        if re.findall(r'\n', j):
                            words += "%0d%0a"
                        else:
                            temp = str(hex(ord(j)))
                            if len(temp) == 3:
                                words += "%0" + temp[2]
                            else:
                                words += "%" + temp[2:]
            print words

        if opt in ("-s", "--stream"):
            if not value:
                print "Usage: -s <Bytg stream>"
                sys.exit()
            a = [value[i:i+2] for i in xrange(0, len(value), 2)]
            words = "%" + "%".join(a)
            print words

if __name__ == "__main__":
    togopher()
```

### 参考资料

对万金油gopher协议的理解与应用

https://k-ring.github.io/2019/05/31/%E5%AF%B9%E4%B8%87%E9%87%91%E6%B2%B9gopher%E5%8D%8F%E8%AE%AE%E7%9A%84%E7%90%86%E8%A7%A3%E4%B8%8E%E5%BA%94%E7%94%A8/