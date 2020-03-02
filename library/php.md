# PHP积累

## php 三种写法

```
<? echo ("这是一个 PHP 语言的嵌入范例\n"); ?>
<?php echo("这是第二个 PHP 语言的嵌入范例\n"); ?>
<script language="php"> 
echo ("这是类似 JavaScript 及 VBScript 语法
的 PHP 语言嵌入范例");
</script>
<% echo ("这是类似 ASP 嵌入语法的 PHP 范例"); %>
```

php://filter简单理解：

php://filter 是php中独有的一个协议，可以作为一个中间流来处理其他流，可以进行任意文件的读取；根据名字，filter，可以很容易想到这个协议可以用来过滤一些东西；

使用不同的参数可以达到不同的目的和效果：

| 名称                      | 描述                                                         | 备注 |
| ------------------------- | ------------------------------------------------------------ | ---- |
| resource=<要过滤的数据流> | 指定了你要筛选过滤的数据流                                   | 必选 |
| read=<读链的筛选列表>     | 可以设定一个或多个过滤器名称，以管道符"\|"分割               | 可选 |
| write=<写链的筛选列表>    | 可以设定一个或多个过滤器名称，以管道符"\|“分割               | 可选 |
| <；两个链的筛选列表>      | 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。 |      |

## 反序列化魔术方法

```php
__construct()//当一个对象创建时被调用
__destruct() //当一个对象销毁时被调用
__toString() //当一个对象被当作一个字符串使用
__sleep()//在对象在被序列化之前运行
__wakeup()//将在反序列化之后立即被调用
__get()//获得一个类的成员变量时调用
__set()//设置一个类的成员变量时调用
__invoke()//调用函数的方式调用一个对象时的回应方法
```

## public、protected与private在序列化时的区别

protected 声明的字段为保护字段，在所声明的类和该类的子类中可见，但在该类的对象实例中不可见。因此保护字段的字段名在序列化时，字段名前面会加上\0*\0的前缀。这里的 \0 表示 ASCII 码为 0 的字符(不可见字符)，而不是 \0 组合。这也许解释了，为什么如果直接在网址上，传递\0*\0username会报错，因为实际上并不是\0，只是用它来代替ASCII值为0的字符。必须用python传值才可以。

```php
<?php
class Name{
    protected $username = 'nonono';
    protected $password = 'yesyes';
    public function __construct($username,$password){
    	$this->username = $username;
        $this->password = $password;
    }
    public function __wakeup(){
    	$this->username = "guests";
    }
    public function fun(){
    	echo $this->username;echo "<br>";echo $this->password;
    }
}
$a = serialize(new Name("admin",100));
echo $a;
?>
```

```
O:4:"Name":2:{s:11:"\0*\0username";s:5:"admin";s:11:"\0*\0password";i:100;}
```

private 声明的字段为私有字段，只在所声明的类中可见，在该类的子类和该类的对象实例中均不可见。因此私有字段的字段名在序列化时，类名和字段名前面都会加上\0的前缀。字符串长度也包括所加前缀的长度。其中 \0 字符也是计算长度的。

可以不看：
这里 表示的是声明该私有字段的类的类名，而不是被序列化的对象的类名。因为声明该私有字段的类不一定是被序列化的对象的类，而有可能是它的祖先类。字段名被作为字符串序列化时，字符串值中包括根据其可见性所加的前缀。

```php
<?php
class Name{
    private $username = 'nonono';
    private $password = 'yesyes';
    public function __construct($username,$password){
    	$this->username = $username;
        $this->password = $password;
    }
    public function __wakeup(){
    	$this->username = "guests";
    }
    public function fun(){
    	echo $this->username;echo "<br>";echo $this->password;
    }
}
$a = serialize(new Name("admin",100));
echo $a;
?>
O:4:"Name":2:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

## 3.``__wakeup()``方法绕过

作用：
与``__sleep()``函数相反，``__sleep()``函数，是在序序列化时被自动调用。``__wakeup()``函数，在反序列化时，被自动调用。
绕过：
当反序列化字符串，表示属性个数的值大于真实属性个数时，会跳过``__wakeup`` 函数的执行。
上面的代码，序列化后的结果为

```
O:4:"Name":2:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

其中name后面的2，代表类中有2个属性，但如果我们把2改成3，就会绕过__wakeup()函数。

```
O:4:"Name":3:{s:14:"\0Name\0username";s:5:"admin";s:14:"\0Name\0password";i:100;}
```

## ``.htaccess``文件利用

- 正确解析绕过
  
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
- 增加使用php解析的文件后缀(.jpg)
    
    ```
    application/x-httpd-php .jpg
    ```
- 增加使用php解析的文件
  
    ```
    <FilesMatch "<filename>">
    SetHandler application/x-httpd-php
    </FilesMatch>
    ```
- 

- **利用php_value注入php配置**

    - 在所有php前后注入恶意php文件
        
        ```
        php_value auto_prepend_file "<phpFileDir>"
        php_value auto_append_file "<phpFileDir>"
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