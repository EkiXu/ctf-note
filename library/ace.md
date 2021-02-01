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

```
env
ps
```

特殊变量

```
$IFS
$PWD
```


## 基于时间的命令盲注

```
sleep $(hostname |cut -c 1| tr a 5)
```