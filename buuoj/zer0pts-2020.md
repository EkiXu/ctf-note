# Zer0pts2020

## Can you guess it?

给了源码
```php
<?php
include 'config.php'; // FLAG is defined in config.php

if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}

$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Can you guess it?</title>
  </head>
  <body>
    <h1>Can you guess it?</h1>
    <p>If your guess is correct, I'll give you the flag.</p>
    <p><a href="?source">Source</a></p>
    <hr>
<?php if (isset($message)) { ?>
    <p><?= $message ?></p>
<?php } ?>
    <form action="index.php" method="POST">
      <input type="text" name="guess">
      <input type="submit">
    </form>
  </body>
</html>
```

发现``hash_equals``绕不过去，只能想办法从别的地方入手

问题出在``$_SERVER['PHP_SELF']``

>$_SERVER['PHP_SELF'] 表示当前 php 文件相对于网站根目录的位置地址，与 document root 相关。
>
>假设我们有如下网址，$_SERVER['PHP_SELF']得到的结果分别为：
>
>http://ctf.ieki.xyz/php/ ：/php/index.php
>http://ctf.ieki.xyz/php/index.php ：/php/index.php
>http://ctf.ieki.xyz/php/index.php?test=foo ：/php/index.php
>http://ctf.ieki.xyz/php/index.php/test/foo ：/php/index.php/test/foo
>
>```php
>$url = "http://".$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'];
>```


用Unicode可以绕过

payload

```
http://2086784a-a065-458e-8566-2d1a200b1f41.node3.buuoj.cn/index.php/config.php/%E5%95%A5?source
```