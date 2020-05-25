# AWD

## 

ssh 

```bash
passwd {username}
```

scp

```bash
scp root@107.172.27.254:/home/test.txt .   //下载文件

scp test.txt root@107.172.27.254:/home  //上传文件

scp -r root@107.172.27.254:/home/test .  //下载目录

scp -r test root@107.172.27.254:/home   //上传目录
```

sql

```

select "123" into outfile '/tmp/out.txt'
```

流量探测

https://github.com/wupco/weblogger