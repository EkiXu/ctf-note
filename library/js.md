# JavaScript

## Fuzz

- ``Object.getOwnPropertyNames(this)`` 得到依赖包名

- ``Error().stack`` 抛出栈错误

## 绕过

- 数组绕过

- `\x70\x72\x6f\x74\x6f\x74\x79\x70\x65` 编码绕过


## 弱类型比较

```
[1] == 1
'1' == 1
```

## 函数“缺陷”

- setTimeout(callback, delay[, ...args])

    当 delay 大于 2147483647 或小于 1 时，则 delay 将会被设置为 1。 非整数的 delay 会被截断为整数。

    如果 callback 不是函数，则抛出 TypeError。


## 原型链污染

### 参考资料

https://3nd.xyz/2019/09/10/Summary/Javascript-Prototype-Attack/#0x07-%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5

## VM虚拟机逃逸

## Buffer

随机泄露内存
```js
Buffer(100)
```

升级版
```js
for (var step = 0; step < 100000; step++) {
    var buf = (new Buffer(100)).toString('ascii');
    if (buf.indexOf("target") !== -1)  break;
}
buf;
```

### 参考资料

https://github.com/ChALkeR/notes/blob/master/Buffer-knows-everything.md

## NPM 包管理安全检测

```
npm audit
```

## 拓展资料

nodejs 语法手册 http://nodejs.cn/api/

Vulnerablity Database: 

https://snyk.io/vuln

https://github.com/advisories

https://www.npmjs.com/advisories