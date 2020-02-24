# MISC Exercise Area

## 0x00 前言

Misc题是真的杂。。。。但是会有很多收获

收货：

- linux镜像的挂载

- strings工具的用法 和管道符“|”和gerp的基础用法
- 二维码拼接
- 与佛论禅
- base64隐写
- stegSolve解题“套路”
- wireshark的基础使用方法

Todo:
- wireshark 流量分析的高级过滤方法
## 0x01 this_is_flag

没错，题目里给了flag

## 0x02 ext3

linux的ext3镜像

先用strings看一下

```bash
strings ext3 |gerp flag
.flag.txt.swp
flag.txtt.swx
~root/Desktop/file/O7avZhikgKgbF/flag.txt
.flag.txt.swp
flag.txtt.swx
.flag.txt.swp
flag.txtt.swx
```

虽然没看到flag

但是应该就是在flag.txt里了

```bash
mount ext3 /mnt/
find /mnt -name "flag.txt"
./O7avZhikgKgbF/flag.txt
cat ./O7avZhikgKgbF/flag.txt
ZmxhZ3tzYWpiY2lienNrampjbmJoc2J2Y2pianN6Y3N6Ymt6an0=
```

base64解码一下得到flag

## 0x03  give_you_flag 

用StegSolve检查帧

发现有一帧有缺了角的二维码

补起来扫码即可

## 0x04 pdf

据题目所说

图片下真的有东西。。。。。

用Adobe Acrobat 拖一下图片就行了

不知道为啥用strings没法搞，可能要学习一下pdf的编码姿势

## 0x05  stegano 

看一下属性信息，里面有个base64编码的假flag。。。。

提示有莫斯码

Ctrl+A 全选复制到记事本以后发现有这么一串东西

```
BABA BBB BA BBA ABA AB B AAB ABAA AB B AA BBB BA AAA BBAABB AABA ABAA AB BBA BBBAAA ABBBB BA AAAB ABBBB AAAAA ABBBB BAAA ABAA AAABB BB AAABB AAAAA AAAAA AAAAB BBA AAABB
```

应该就是morse码了

## 0x06 simpleRAR

用Winrar 打开RAR就说文件头被破坏了

查一下RAR编码后的头文件格式

用Winhex改一下就可以了

（可以自己用PNG压缩看RAR里的PNG的头文件是怎么给的）

然后得到一个png

提示双图层用ps还打不开

file一下发现是个gif。。。。。

```bash
file secret.png
secret.png: GIF image data, version 89a, 280 x 280
```

用ps分离一下图层

结果都还是白的。。。

用StegoSolve看一下

发现各有半个二维码

还得补全定位符。。。。

（好毒瘤）

## 0x07  坚持60s 

这是个逆向题？

用jd-gui反编译找一下flag就完了

或者也可真·坚持60s

## 0x08 gif

一堆黑白图

没有能表示空格的 排除莫斯码

转成01串然后再转ASCII码

看了大佬的WP

发现还可以用脚本自动找01。。。。。

```cpp
from PIL import Image

suf = '.jpg'
ch = 0
for i in range(104):
    s = str(i) + suf
    im = Image.open(s,'r')
    pix = im.load()#导入像素
    #im.show() open the image
    check = pix[0,0][0]
    if check == 255:
        ch=ch*2
    if check != 255:
        ch=ch*2+1
    if (i+1) % 8 == 0:
        print (chr(ch),end='')
        ch=0
print("\n")
```

## 0x09 掀桌子

这个题。。。。

有点脑洞

长度118字幕范围在a-f

可能又是hex转ascii码

但是直接转好像不行

注意到ascii ∈(0,128)

取个模试试？

```python
#coding=utf-8
hexstr="c8e9aca0c6f2e5f3e8c4efe7a1a0d4e8e5a0e6ece1e7a0e9f3baa0e8eafae3f9e4eafae2eae4e3eaebfaebe3f5e7e9f3e4e3e8eaf9eaf3e2e4e6f2"
flag=""
while len(hexstr):
        flag = flag+chr(int(hexstr[:2],16)%128)
        hexstr = hexstr[2:]
print (flag)
```

## 0x0A 如来十三掌

看格式是与佛论禅。。。。

[http://keyfc.net/bbs/tools/tudoucode.aspx](http://keyfc.net/bbs/tools/tudoucode.aspx) 

但是还得加个佛曰：

解码得到

```
MzkuM3gvMUAwnzuvn3cgozMlMTuvqzAenJchMUAeqzWenzEmLJW9
```

看起来有点像base64，但是解码是乱码。。。

根据题目联想到rot13

```
ZmxhZ3tiZHNjamhia3ptbmZyZGhidmNraWpuZHNrdmJramRzYWJ9
```

这回base64解码不会再乱了。。。。

## 0x0B  base64stego 

打开压缩包发现加过密了

但是没有给密码提示

可能是个伪加密，果然用winrar修复一下就不用密码了。。。。

里面一堆base64，解码得到没有啥用的一段文档

题名暗示是base64隐写了

然后跑脚本。。。。

```python
def get_base64_diff_value(s1, s2):
    base64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    res = 0
    for i in xrange(len(s1)):
        if s1[i] != s2[i]:
            return abs(base64chars.index(s1[i]) - base64chars.index(s2[i]))
    return res

def solve_stego():

    with open('stego.txt', 'rb') as f:
        file_lines = f.readlines()
    bin_str = ''
    for line in file_lines:
        steg_line = line.replace('\n', '')
        norm_line = line.replace('\n', '').decode('base64').encode('base64').replace('\n', '')
        diff = get_base64_diff_value(steg_line, norm_line)

        pads_num = steg_line.count('=')
        if diff:
            bin_str += bin(diff)[2:].zfill(pads_num * 2)

        else:
            bin_str += '0' * pads_num * 2

    res_str = ''

    for i in xrange(0, len(bin_str), 8):

        res_str += chr(int(bin_str[i:i+8], 2))
    print (res_str)

solve_stego()
```

## 0x0C Base_sixty_four_point_five

流量分析题。。。。

用wireshark自带的文件导出工具把文件都存下来

然后得到一堆的php和objects

php基本上就是一句话木马和服务器交流的过程

其中奇数序号的应该就是服务器返回的内容

在1(3).php看到有意思的东西

```
->|./	2017-12-08 11:38:58	0	0777
../	2017-12-08 11:39:10	4096	0777
1.php	2017-12-08 11:33:16	33	0666
flag.txt	2017-12-08 11:35:29	17	0666
hello.zip	2017-12-08 09:32:36	224	0666
|<-
```

我们试着找找看flag.txt和hello.zip

在1(25)中又发现多了个6666.jpg

```
->|./	2017-12-08 11:42:11	0	0777
../	2017-12-08 11:39:10	4096	0777
1.php	2017-12-08 11:33:16	33	0666
6666.jpg	2017-12-08 11:42:11	102226	0666
flag.txt	2017-12-08 11:35:29	17	0666
hello.zip	2017-12-08 09:32:36	224	0666
|<-
```

可能是用菜刀传上去的

注意到有个特别大的php

里面好像有一串奇怪的hex

果然是个jpg

但是我们要的压缩包呢。。。。。

再试试用binwalk看一下有没有其他文件

```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
663085        0xA1E2D         xz compressed data
664045        0xA21ED         xz compressed data
812025        0xC63F9         xz compressed data
814001        0xC6BB1         xz compressed data
1238637       0x12E66D        xz compressed data
1240937       0x12EF69        xz compressed data
1391563       0x153BCB        xz compressed data
1393067       0x1541AB        xz compressed data
1406647       0x1576B7        xz compressed data
1412887       0x158F17        xz compressed data
1422689       0x15B561        Zip archive data, encrypted at least v2.0 to extract, compressed size: 52, uncompressed size: 40, name: flag.txt
```

果然有

然后用foremost分离一下

拿到zip,用jpg里的密码解密一下就行了





