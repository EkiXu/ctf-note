# I-SOON 2019 Log

## 0x00 前言

一场十分艰难的比赛。。。。

web pwn无头绪

时间全花在misc上了

还实力眼瞎了好几次。。。。。

最后只拿到了rank 40......

道阻且长

收货

- wireshark 流量分析的进阶过滤操作

- mimikatz64工具的使用

Todo

- php curl 操作
- request 写exp....
- ....一堆要补的web知识

## 0x01 吹着贝斯扫二维码

看文件头是一堆jpg，发现要拼图。。。。。

根据四个定位定位标识的加一些边框把二维码拼出来拿到解密方法

```
BASE Family Bucket ??? 85->64->85->13->16->32
```

把zip文件的注释按照得到的解密方法解一遍就完事了

## 0x02 music

根据提示信息拿mp3stego用密钥接触压缩包密码

```
密码是123qwe123
```

然后用silenteye对压缩包里的wav进行分析，是个音频的LSB隐写

## 0x03 Attack

流量分析， tcp.stream eq 824拿到一个zip包里面有flag.txt

提示Administrator的密码

tcp.stream eq 833拿到dmp

file一下是个minidump

mimikatz64 可以对其进行分析得到所有用户的密码

用Adminstrator的密码解密

## 0x04 funny-php

逆向题

```php
function encode($str){

	$str1=array();
	$str1=unpack("C*",$str);
	for($_0=0;$_0<count($str1);$_0++){
		$_c=$str1[$_0];
		$_=$_.$_c;
	}

	$_d=array();
	for($_1=0;$_1<strlen($_);$_1++){
		$_d[$_1]=substr($_,$_1,1);		
		$_e=ord($_d[$_1])+$_1;
		$_f=chr($_e);
		$__=$__.$_f;
		if($__%100==0)
			$__=base64_encode($__);
	}
	$__=strrev(str_rot13(base64_encode($__)));

	return $__;
	
}
```
比赛的时候只写了个部分解密的脚本
```
function decode($chiper){
    
    $_ftmp='';
	$_='';
	$str='';
    $_d=array();
	$__=base64_decode(str_rot13(strrev($chiper)));
	
	print($__.'<br/>');
    for($i=0;$i<strlen($__);$i++){
		$_f=substr($__,$i,1);		
		$_e=ord($_f)-$i;
		$_=$_.chr($_e);
    }
	print($_);
	/*while(strlen($__)>0){
        if(base64_decode($__)%100==0&&base64_decode($__)!=NULL){
            $__=base64_decode($__);
		}
        $_ftmp=substr($__,strlen($__)-1,1).$_ftmp;
     	$__=substr($__,0,strlen($__)-1);
    }
	
	//$_ftmp=$__;
    print($_ftmp.'<br/>');
	for($i=0;$i<strlen($_ftmp);$i++){
		$_f=substr($_ftmp,$i,1);
		$_e=ord($_f);
		$_d[$i]=chr($_e-$i);
		$_=$_.$_d[$i];
    }
	print($_.'<br/>');
	$str1=array();
	
	$str1=explode(';',$_);
	
	$str=pack("C*",$str1);
	return $str;*/
}
```
在
```
	for($_0=0;$_0<count($str1);$_0++){
		$_c=$str1[$_0];
		$_=$_.$_c;
	}
```
之前得到这么一串数字

```
10210897103123101971151219510111099111100101125
```
根据ascii码的特征分割一下

```c
#include<stdio.h>
char str[]={102,108,97,103,123,101,97,115,121,95,101,110,99,111,100,101,125,0};
int main(){
	printf(str);
	return 0;
}
```
