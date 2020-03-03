# 隐写术

## 图片隐写

> 常用到的工具 binwalk foremost stegSolve stegdetect outguess JPHS Photoshop

### 直接隐写

#### exif隐写

直接在图片属性中放置flag

### 文件内隐写

在图片文件尾部放置flag 解决方法 使用winhex打开查找

扩展： 在文件尾嵌入zip，rar等文件 解决方法 使用binwalk检测 foremost 分离

### 加密隐写

LSB隐写 原理 LSB隐写就是修改RGB颜色分量的最低二进制位也就是最低有效位（LSB），而人类的眼睛不会注意到这前后的变化，此方法中每个像素可以携带3比特的信息。解决方法 使用stegSolve 查看各通道显示的图像 查看异常通道 使用Analyse-&gt;DataExtract 查看是否存在flag

扩展 以图象方式嵌入flag或对应二维码进行进一步解密

### outguess 隐写

原理 利用JPEG的DCT系数的冗余 解决方法 使用outguess 工具解密

### 文件“上”隐写

#### 部分显示图片

原理 PNG IHDR 前8字节的内容可以更改一张图片的高度或者宽度使得一张图片显示不完整从而达到隐藏信息的目的 解决办法 Kali中不可以打开，提示文件头错误，而Windows自带的图片查看器可以打开，就提醒了我们IHDR被人篡改过 利用winhex 修改文件头使图片完全显示

#### GIF 时间轴隐藏

原理 由于GIF的动态特性，由一帧帧的图片构成，所以每一帧的图片，多帧图片间的结合，都成了隐藏信息的一种载体。解决方法 利用Photoshop 或者 StegSolve的 Analyse -&gt; Frame Brower

### 双图片隐写

双图片异或和 解决方法 StegSolve的 Analyse -&gt; Imgine Combiner

### 双图层

解决方法 PS分离

## 文本隐写

### 二进制隐写

直接winhex打开看。。。。

### base64隐写

解密脚本

```python
#!/usr/bin/python
import sys

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

加密脚本

```python
# coding:UTF8
import base64
import string


def decode(flag1):  # 把需要隐藏的密文变成二进制字符串
    list_flag1 = list(flag1)  # 把密文转为list例如：['I', ' ', 'a', 'm', ' ', 'a', ' ', 'C', 'T', 'F', 'e', 'r']
    list_dec_flag1 = []  # 十进制密文例如：[73, 32, 97, 109, 32, 97, 32, 67, 84, 70, 101, 114]
    for j in range(len(list_flag1)):  # 把ascii码密文转换为十进制密文list
        list_dec_flag1.append(ord(list_flag1[j]))
    list_bin_flag1 = []
    # 二进制密文例如：['1001001', '100000', '1100001', '1101101', '100000', '1100001', '100000', '1000011','1010100'...]
    for j in range(len(list_dec_flag1)):  # 把十进制密文转化为二进制密文
        list_bin_flag1.append((bin(list_dec_flag1[j])[2:]).zfill(8))
    str_bin_flag1 = ''.join(list_bin_flag1)  # 把二进制密文list拼接成str
    # 例如：010010010010000001100001011011010010000001100001001000000100001101010100010001100110010101110010
    list_bin_every_flag1 = list(str_bin_flag1)  # 把str二进制密文转换成list
    # 例如：['1', '0', '0', '1', '0', '0', '1', '1', '0', '0', '0', '0', '0', '1', '1', '0', '0', '0', '0', '1'...]
    return list_bin_every_flag1


if __name__ == '__main__':
    dic = string.uppercase+string.lowercase+string.digits+'+/'
    # a = raw_input()
    flag = ':-)There is no flag,but a cute key:\"hmz\"'
    list_bin_every_flag = decode(flag)  # list二进制密文
    # print len(list_bin_every_flag)
    # 例如：['1', '0', '0', '1', '0', '0', '1', '1', '0', '0', '0', '0', '0', '1', '1', '0', '0', '0', '0', '1'...]
    tip = 0  # 定义一个指针指向要写入base64的list二进制密文
    # tt = 0
    #stego = open('stego.txt','w')
    with open('stego.txt', 'rb') as h:  # 打开明文
        file_lines = h.readlines()  # 把明文读取成一行
        # 例如：['#include <stdio.h>\r\n', '#include <stdlib.h>\r\n', 'main(){int i,n[]={(((1 <<1)<< (1<<1)...]
        for line in file_lines:  # 接下来是正文了
            normal_line = line.replace('\r\n', '')  # 每一行的明文
            # print normal_line
            # 例如：#include <stdio.h>\r
            equal_sign_num = 3 - (len(normal_line) % 3)  # 每行base64加密后的等号数量
            if equal_sign_num == 3:  # 如果是3的倍数说明这一句没法进行隐写
                equal_sign_num = 0  # 设其等号数量为0
            # print 'equal_sign_num', equal_sign_num
            # tt += equal_sign_num * 2
            # print tt, 'tt'
            list_normal_line = decode(normal_line)  # 把明文也装换为list_bin_every_明文
            # print list_normal_line
            if equal_sign_num == 1:  # 一个等号
                for i in range(2):
                    # print 'tip', tip, 'list_bin_every_flag[tip]',list_bin_every_flag[tip]
                    list_normal_line.append(list_bin_every_flag[tip])
                    tip += 1
                # print list_normal_line
            elif equal_sign_num == 2:  # 两个等号
                for i in range(4):
                    # print 'tip', tip, 'list_bin_every_flag[tip]', list_bin_every_flag[tip]
                    list_normal_line.append(list_bin_every_flag[tip])
                    tip += 1
                # print list_normal_line
            # print tt
            str_bin_normal_line = ''.join(list_normal_line)
            # print str_bin_normal_line
            b64 = ''
            for i in range(0, len(str_bin_normal_line), 6):
                b64 += dic[int(str_bin_normal_line[i: i+6], 2)]  # 以6位为单位对照base64编码表
            if equal_sign_num == 1:
                b64 += '='
            elif equal_sign_num == 2:
                b64 += '=='
            print b64

```

### snow 隐写

### pyc 隐写

### whitespace隐写

## 音频隐写

### 音频LSB隐写

### 频谱图隐写

### SSTV信号

## 文件系统隐写

### NTFS 隐写



