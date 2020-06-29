# Forensics

## Windows 取证

取证大师

## Andorid 取证

``sdcard``

- ``/Android/data``

## Linux 取证

- ``/etc/passwd`` , ``/etc/shadow``
- ``/etc/hosts`` , ``/proc/net/arp``


## 内存取证 Volatility

```bash
volatility -f <file> imageinfo #猜测镜像类型
```

得到Profile后进入 volshell 
```bash
volatility -f mem.vmem --profile=Win7SP1x64 volshell
```

``ps()``来列进程,也可以用``pslist`` 插件

一些常用命令

```bash
#查看所有进程
volatility -f <file> --profile=Win7SP1x64 psscan 

#扫描所有的文件列表
volatility -f <file> --profile=Win7SP1x64 filescan 

volatility -f <file> --profile=Win7SP1x64 filescan | grep "doc\|docx\|rtf"
volatility -f <file> --profile=Win7SP1x64 filescan | grep "jpg\|jpeg\|png\|tif\|gif\|bmp"
volatility -f <file> --profile=Win7SP1x64 filescan | grep "Desktop"

#扫描 Windows 的服务
volatility -f <file> --profile=Win7SP1x64 svcscan 

#查看网络连接
volatility -f <file> --profile=Win7SP1x64 connscan 

#查看命令行上的操作
volatility -f <file> --profile=Win7SP1x64 cmdscan 

#根据进程的 pid dump出指定进程到指定的文件夹dump_dir
volatility -f <file> --profile=Win7SP1x64 memdump -p 120  --dump-dir=dump_dir
#dump 出来的进程文件用foremost来分离里面的文件

#查看命令行输入
volatility -f <file> --profile=Win7SP1x64 cmdline

#查看系统用户名
volatility -f <file> --profile=Win7SP1x64 printkey -K "SAM\Domains\Account\Users\Names"

#查看网络连接
volatility -f <file> --profile=Win7SP1x64 netscan

#查看记事本内容
volatility -f <file> --profile=Win7SP1x64 notepad #只能读XP的
volatility -f <file> --profile=Win7SP1x64 editbox 

#查看ie记录
volatility -f <file> --profile=Win7SP1x64 iehistory
#利用更为强大的yarascan
volatility -f <file> --profile=Win7SP1x64 yarascan -Y "/(URL|REDR|LEAK)/" -p <iexplore.exe pid>

volatility -f <file> --profile=Win7SP1x64 dumpfiles -Q 0x00000000053e9658 --dump-dir=./  
```

### 参考资料

官方wiki: 

https://github.com/volatilityfoundation/volatility/wiki/Command-Reference

内存取证工具 volatility 使用说明：

https://www.restran.net/2017/08/10/memory-forensics-tool-volatility/


## 文件恢复

### Zlib

```
choosing windowBits
But zlib can decompress all those formats:

to (de-)compress deflate format, use wbits = -zlib.MAX_WBITS
to (de-)compress zlib format, use wbits = zlib.MAX_WBITS
to (de-)compress gzip format, use wbits = zlib.MAX_WBITS | 16
```