#  CRYPTO Execrise Area

## 0x00 前言

密码学好考验数学知识。。。。。

收货

- 云影密码
- base64
- 栅栏密码+变种栅栏密码（w）
- 凯撒密码
- 转轮机密码
- 培根密码
- quipquip.com 词频分析暴力破解古典密码
- RSA的加密过程 rsatool openssl的基本使用方式

Todo

- ECC椭圆曲线加密过程
- 转轮机密码编程实现

## 0x01 幂数加密

。。。。。说好的幂数加密

结果是个“云影密码”。。。。。。。

> 01248 密码：该密码又称为云影密码，使用 01248 四个数字，其中 0 用来表示间隔，其他数字以加法可以表示出 如：28=10，124=7，18=9，再用 1->26 表示 A->Z。

## 0x02 base64

base64解码

## 0x03 Caesar

‘c'-'o'=-12 key=12的凯撒密码

## 0x04 Morse

纯莫尔斯电码解码

## 0x05 Railfence

说好的栅栏密码呢。。。。

题目描述“5只小鸡”提示每组5字，但是拖进解密的地方出来的还是密文。

考虑到解密后的明文应该是以“cyber”开头的

但这几个字母在密文中又不是“均匀”分布的。。。。。。

最后发现得用栅栏密码的变种做。。。。。。

```
c       c       e       h       g  
 y     a e     f n     p e     o o 
  b   e   {   l   c   i   r   g   }
   e p     r i     e c     _ o     
    r       a       _       g
```

栅栏密码在线解密： https://www.qqxiuzi.cn/bianma/zhalanmima.php

栅栏密码变种在线解密： http://rumkin.com/tools/cipher/railfence.php

## 0x06 转轮机加密

转轮机加密

密钥对应转轮行数和读的顺序

密文对应相应行第一位的字母

得到下表

```
< NACZDTRXMJQOYHGVSFUWIKPBEL <
< FHTEQGYXPLOCKBDMAIZVRNSJUW <
< QGWTHSPYBXIZULVKMRAFDCEONJ <
< KCPMNZQWXYIHFRLABEUOTSGJVD <
< SXCDERFVBGTYHNUMKILOPJZQAW <
< EIURYTASBKJDFHGLVNCMXZPQOW <
< VUBMCQWAOIKZGJXPLTDSRFHENY <
< OSFEZWAXJGDLUBVIQHKYPNTCRM <
< QNOZUTWDCVRJLXKISEFAPMYGHB <
< OWTGVRSCZQKELMXYIHPUDNAJFB <
< FCUKTEBSXQYIZMJWAORPLNDVHG <
< NBVCXZQWERTPOIUYALSKDJFHGM <
< PNYCJBFZDRUSLOQXVETAMKGHIW <
```

还得猜是有意义的字符串。。。。。

## 0x07 easy_RSA

RSA加密过程

n=p\times q*n*=*p*×*q*

\phi(n)=(p-1)(q-1)*ϕ*(*n*)=(*p*−1)(*q*−1)

e \in (1,\phi(n))*e*∈(1,*ϕ*(*n*))

ed\equiv 1 \pmod {\phi(n)} \Leftrightarrow ed=k\phi(n)+1*e**d*≡1(mod**ϕ**(*n*))⇔*e**d*=*k**ϕ*(*n*)+1

a^{\phi(n)}\equiv 1 \pmod n*a**ϕ*(*n*)≡1(mod**n**)

a^{k\phi(n)+1} \equiv a \pmod n*a**k**ϕ*(*n*)+1≡*a*(mod**n**)

a^{ed} \equiv a \pmod n*a**e**d*≡*a*(mod**n**)

若$c\equiv a^e \pmod n$则$c^d\equiv {a^{ed}} \equiv a \pmod n$

也就是说求模n下e的乘法n逆元d

用mathematica

```mathematica
In:=ExtendedGCD[17, (473398607161 - 1)*(4511491 - 1)]
Out:={1, {125631357777427553, -1}}
```

中间那个数就是d了

## 0x08 Normal_RSA

给了公钥和密文

要求明文

用openssl提取一下n

```bash
openssl rsa -pubin -text -modulus -in warmup -in pubkey.pem
RSA Public-Key: (256 bit)
Modulus:
    00:c2:63:6a:e5:c3:d8:e4:3f:fb:97:ab:09:02:8f:
    1a:ac:6c:0b:f6:cd:3d:70:eb:ca:28:1b:ff:e9:7f:
    be:30:dd
Exponent: 65537 (0x10001)
Modulus=C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD
writing RSA key
-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMJjauXD2OQ/+5erCQKPGqxsC/bNPXDr
yigb/+l/vjDdAgMBAAE=
-----END PUBLIC KEY-----
```

n=0xC2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD

比较小，用yafu分解一下

```bash
yafu.exe
factor(87924348264132406875276140514499937145050893665602592992418171647042491658461)


fac: factoring 87924348264132406875276140514499937145050893665602592992418171647042491658461
fac: using pretesting plan: normal
fac: no tune info: using qs/gnfs crossover of 95 digits
div: primes less than 10000
fmt: 1000000 iterations
rho: x^2 + 3, starting 1000 iterations on C77
rho: x^2 + 2, starting 1000 iterations on C77
rho: x^2 + 1, starting 1000 iterations on C77
pm1: starting B1 = 150K, B2 = gmp-ecm default on C77
ecm: 30/30 curves on C77, B1=2K, B2=gmp-ecm default
ecm: 74/74 curves on C77, B1=11K, B2=gmp-ecm default
ecm: 149/149 curves on C77, B1=50K, B2=gmp-ecm default, ETA: 0 sec

starting SIQS on c77: 87924348264132406875276140514499937145050893665602592992418171647042491658461

==== sieving in progress (1 thread):   36224 relations needed ====
====           Press ctrl-c to abort and save state           ====
35719 rels found: 17849 full + 17870 from 191606 partial, (2263.87 rels/sec)

SIQS elapsed time = 94.7100 seconds.
Total factoring time = 108.5340 seconds


***factors found***

P39 = 319576316814478949870590164193048041239
P39 = 275127860351348928173285174381581152299

ans = 1
```

p,q找到了，然后用rsatool生成一下密钥文件

```bash
python rsatool.py -o private.pem -e 65537 -p 319576316814478949870590164193048041239 -q 275127860351348928173285174381581152299
```

然后用openssl解码一下就行了

```bash
openssl rsautl -decrypt -in flag.enc -inkey private.pem
```

## 0x09 不仅仅是Morse

先拖到 https://morsecode.scphillips.com/translator.html

Morse码解密一下

```
M A Y # B E # H A V E # A N O T H E R # D E C O D E H H H H A A A A A B A A B B B A A B B A A A A A A A A B A A B A B A A A A A A A B B A B A A A B B A A A B B A A B A A A A B A B A A B A A A B B A B A A A B A A A B A A B A B B A A B B B A B A A A B A B A B B A A A B B A B A A A B A A B A A B A A A A B B A B B A A B B A A B A A B A A A B A A B A A B A A B A B A A B B A B A A A A B B A B A A B B A
```

根据题目暗示“ 一种食物 ”和一大串的AB联想到培根密码。。。

培根密码解密脚本

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
import re
# 密文转化为指定格式
s = 'AAAAABAABBBAABBAAAAAAAABAABABAAAAAAABBABAAABBAAABBAABAAAABABAABAAABBABAAABAAABAABABBAABBBABAAABABABBAAABBABAAABAABAABAAAABBABBAABBAABAABAAABAABAABAABABAABBABAAAABBABAABBA'
a = s.lower()
# 字典
CODE_TABLE = {
    'a':'aaaaa','b':'aaaab','c':'aaaba','d':'aaabb','e':'aabaa','f':'aabab','g':'aabba',
    'h':'aabbb','i':'abaaa','j':'abaab','k':'ababa','l':'ababb','m':'abbaa','n':'abbab',
    'o':'abbba','p':'abbbb','q':'baaaa','r':'baaab','s':'baaba','t':'baabb','u':'babaa',
    'v':'babab','w':'babba','x':'babbb','y':'bbaaa','z':'bbaab'
}
# 5个一组进行切割并解密
def peigendecode(peigen):
    msg =''
    codes = re.findall(r'.{5}', a)
    for code in codes:
        if code =='':
            msg += ' '
        else:
            UNCODE =dict(map(lambda t:(t[1],t[0]),CODE_TABLE.items()))
            msg += UNCODE[code]
    return msg
flag = peigendecode(a)
print('flag is ',flag)
```

## 0x0A 混合编码

raw->base64->unicode->ASCII->base64->ASCII

## 0x0B easyChallenge

用pycdc反编译得到

```bash
import base64

def encode1(ans):
    s = ''
    for i in ans:
        x = ord(i) ^ 36
        x = x + 25
        s += chr(x)

    return s


def encode2(ans):
    s = ''
    for i in ans:
        x = ord(i) + 36
        x = x ^ 36
        s += chr(x)

    return s


def encode3(ans):
    return base64.b32encode(ans)

flag = ' '
print 'Please Input your flag:'
flag = raw_input()
final = 'UC7KOWVXWVNKNIC2XCXKHKK2W5NLBKNOUOSK3LNNVWW3E==='
if encode3(encode2(encode1(flag))) == final:
    print 'correct'
else:
    print 'wrong'
```

改一下就得到解码脚本

```python
import base64

def decode1(ans):
    s = ''
    for i in ans:
        x = ord(i) - 25
        x = x ^ 36
        s += chr(x)

    return s


def decode2(ans):
    s = ''
    for i in ans:
        x = ord(i) ^ 36
        x = x - 36
        s += chr(x)

    return s


def decode3(ans):
    return base64.b32decode(ans)

final = 'UC7KOWVXWVNKNIC2XCXKHKK2W5NLBKNOUOSK3LNNVWW3E==='
print decode1(decode2(decode3(final)))
```

## 0x0C easyECC

椭圆曲线加密算法。。。。

原理还是不太清楚

但是可以利用下面的脚本计算公钥

```python
#coding=utf-8
import collections
import random

EllipticCurve = collections.namedtuple('EllipticCurve', 'name p a b g n h')

curve = EllipticCurve(
   'secp256k1',
   # Field characteristic.
   p=int(input('p=')),
   # Curve coefficients.
   a=int(input('a=')),
   b=int(input('b=')),
   # Base point.
   g=(int(input('Gx=')),
      int(input('Gy='))),
   # Subgroup order.
   n=int(input('k=')),
   # Subgroup cofactor.
   h=1,
)

# Modular arithmetic ##########################################################
def inverse_mod(k, p):
   """Returns the inverse of k modulo p.
  This function returns the only integer x such that (x * k) % p == 1.
  k must be non-zero and p must be a prime.
  """
   if k == 0:
       raise ZeroDivisionError('division by zero')

   if k < 0:
       # k ** -1 = p - (-k) ** -1 (mod p)
       return p - inverse_mod(-k, p)

   # Extended Euclidean algorithm.
   s, old_s = 0, 1
   t, old_t = 1, 0
   r, old_r = p, k

   while r != 0:
       quotient = old_r // r
       old_r, r = r, old_r - quotient * r
       old_s, s = s, old_s - quotient * s
       old_t, t = t, old_t - quotient * t

   gcd, x, y = old_r, old_s, old_t

   assert gcd == 1
   assert (k * x) % p == 1

   return x % p

# Functions that work on curve points #########################################
def is_on_curve(point):
   """Returns True if the given point lies on the elliptic curve."""
   if point is None:
       # None represents the point at infinity.
       return True

   x, y = point

   return (y * y - x * x * x - curve.a * x - curve.b) % curve.p == 0

def point_neg(point):
   """Returns -point."""
   assert is_on_curve(point)

   if point is None:
       # -0 = 0
       return None

   x, y = point
   result = (x, -y % curve.p)

   assert is_on_curve(result)

   return result

def point_add(point1, point2):
   """Returns the result of point1 + point2 according to the group law."""
   assert is_on_curve(point1)
   assert is_on_curve(point2)

   if point1 is None:
       # 0 + point2 = point2
       return point2
   if point2 is None:
       # point1 + 0 = point1
       return point1

   x1, y1 = point1
   x2, y2 = point2

   if x1 == x2 and y1 != y2:
       # point1 + (-point1) = 0
       return None

   if x1 == x2:
       # This is the case point1 == point2.
       m = (3 * x1 * x1 + curve.a) * inverse_mod(2 * y1, curve.p)
   else:
       # This is the case point1 != point2.
       m = (y1 - y2) * inverse_mod(x1 - x2, curve.p)

   x3 = m * m - x1 - x2
   y3 = y1 + m * (x3 - x1)
   result = (x3 % curve.p,
             -y3 % curve.p)

   assert is_on_curve(result)

   return result

def scalar_mult(k, point):
   """Returns k * point computed using the double and point_add algorithm."""
   assert is_on_curve(point)

   if k < 0:
       # k * point = -k * (-point)
       return scalar_mult(-k, point_neg(point))

   result = None
   addend = point

   while k:
       if k & 1:
           # Add.
           result = point_add(result, addend)

       # Double.
       addend = point_add(addend, addend)

       k >>= 1

   assert is_on_curve(result)

   return result

# Keypair generation and ECDHE ################################################
def make_keypair():
   """Generates a random private-public key pair."""
   private_key = curve.n
   public_key = scalar_mult(private_key, curve.g)

   return private_key, public_key

private_key, public_key = make_keypair()
print("private key:", hex(private_key))
print("public key: (0x{:x}, 0x{:x})".format(*public_key))
```