# XCTF-Final 2019

## LFI 2019

剔除一些没用的代码，核心部分如下
```php
<?php
    // Here, kawaii path-san for you! //
    function path_sanitizer($dir, $harden=false){
        $dir = (string)$dir;
        $dir_len = strlen($dir);
        // Deny LFI/RFI/XSS //
        $filter = ['.', './', '~', '.\\', '#', '<', '>'];
        foreach($filter as $f){
            if(stripos($dir, $f) !== false){
                return false;
            }
        }
        // Deny SSRF and all possible weird bypasses //这里ban了所有协议
        $stream = stream_get_wrappers();
        $stream = array_merge($stream, stream_get_transports());
        $stream = array_merge($stream, stream_get_filters());
        foreach($stream as $f){
            $f_len = strlen($f);
            if(substr($dir, 0, $f_len) === $f){
                return false;
            }
        }
        // Deny length // 
        if($dir_len >= 128){
            return false;
        }
		// Easy level hardening //
		if($harden){
			$harden_filter = ["/", "\\"];
			foreach($harden_filter as $f){
				$dir = str_replace($f, "", $dir);
			}
		}

        // Sanitize feature is available starting from the medium level //
        return $dir;
    }

    // The new kakkoii code-san is re-implemented. // 这部分将上传文件中所有数字字母替换
    function code_sanitizer($code){
        // Computer-chan, please don't speak english. Speak something else! //
        $code = preg_replace("/[^<>!@#$%\^&*\_?+\.\-\\\'\"\=\(\)\[\]\;]/u", "*Nope*", (string)$code);
        return $code;
    }

    // Errors are intended and straightforward. Please do not ask questions. //
    class Get {
        private $filename;
        function __construct($filename){
            $this->filename = path_sanitizer($filename);
        }
        function get(){
            if($this->filename === false){
                return ["msg" => "blocked by path sanitizer", "type" => "error"];
            }
            // wtf???? //
            if(!@file_exists($this->filename)){
                // index files are *completely* disabled. //
                if(stripos($this->filename, "index") !== false){
                    return ["msg" => "you cannot include index files!", "type" => "error"];
                }

                // hardened sanitizer spawned. thus we sense ambiguity //如果这两个读到的相等就被ban了
                $read_file = "./files/" . $this->filename;
                $read_file_with_hardened_filter = "./files/" . path_sanitizer($this->filename, true);

                if($read_file === $read_file_with_hardened_filter ||
                    @file_get_contents($read_file) === @file_get_contents($read_file_with_hardened_filter)){
                    return ["msg" => "request blocked", "type" => "error"];
                }
                // .. and finally, include *un*exploitable file is included. //
                @include("./files/" . $this->filename);
                return ["type" => "success"];
            }else{
                return ["msg" => "invalid filename (wtf)", "type" => "error"];
            }
        }
    }
    class Put {
        private $filename;
        private $content;
        private $dir = "./files/";
        function __construct($filename, $data){
            global $seed;
            if((string)$filename === (string)@path_sanitizer($data['filename'])){
                $this->filename = (string)$filename;
            }else{
                $this->filename = false;
            }
            $this->content = (string)@code_sanitizer($data['content']);
        }
        function put(){
            // just another typical file insertion //
            if($this->filename === false){
                return ["msg" => "blocked by path sanitizer", "type" => "error"];
            }
            // check if file exists //
            if(file_exists($this->dir . $this->filename)){
                return ["msg" => "file exists", "type" => "error"];
            }
            file_put_contents($this->dir . $this->filename, $this->content);
            // just check if file is written. hopefully. //
            if(@file_get_contents($this->dir . $this->filename) == ""){
                return ["msg" => "file not written.", "type" => "error"];
            }
            return ["type" => "success"];
        }
    }

    // Hello, JSON! //
    $parsed_url = explode("&", $_SERVER['QUERY_STRING']);
    if(count($parsed_url) >= 2){
        header("Content-Type:text/json");
        switch($parsed_url[0]){
            case "get":
                $get = new Get($parsed_url[1]);
                $data = $get->get();
                break;
            case "put":
                $put = new Put($parsed_url[1], $_POST);
                $data = $put->put();
                break;
            default:
                $data = ["msg" => "Invalid data."];
                break;
        }
        die(json_encode($data));
    }
?>
```

``code_sanitizer``可以利用无数字字母的php脚本绕过

``path_sanitizer``主要是让正常读和经过``path_sanitizer``处理后不一样

如果是linux可以直接用``eki\``的形式绕过

二对于Windows的文件读取，在使用FindFirstFile这个API的时候，

字符``>``被替换成``?``，字符``<``被替换成``*``，而符号`` ` ``（双引号）被替换成一个``.``字符。

那么我们上传一个文件，名字设为``eki``。然后，通过``"/eki``即可读取。此时：

```php
$read_file = "./files/./eki";
$read_file_with_hardened_filter = "./files/.eki";
file_get_contents($read_file) = '实际文件内容';
file_get_contents($read_file_with_hardened_filter) = false //文件不存在
```

### 参考文章

XCTF final 2019 Writeup By ROIS

https://xz.aliyun.com/t/6655#toc-2


我的WafBypass之道（upload篇）

https://www.cnblogs.com/hookjoy/p/6579646.html


FindFirstFile利用

https://www.cnblogs.com/Akkuman/p/6962984.html