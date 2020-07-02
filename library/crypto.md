# Crypto

> Tools: CyberChef

## 古典密码

- 各种单表代换
  - 猪圈
  - 象形文字
   
- 弗吉尼亚密码

- 词频分析
  https://www.boxentriq.com/code-breaking/frequency-analysis 

通解 [quipquip.com](https://quipqiup.com/)

### codemoji

### 栅栏加密

## 现代密码

### 各种XOR

### RSA

### DH

### ECC

### DES

### AES

#### ECB

分块加密 

#### CBC

## hash签名

### md5

利用点

- ``0e``开头弱类型比较
- fastcoll 选择前缀碰撞
- ffifdyop md5后，276f722736c95d99e921722cf9ed621 再转成字符串： ``'or'6<trash>``

## Token

### JWT

jwt由三部分构成,以点号隔开

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

利用点

- 标头空加密
- 签名密钥泄露

## 编码

- base family:base85 base64 base58 base32 base16 以及编码表替换
- rot13
- 与佛论禅
- 社会主义核心价值观编码


## 现代密码

### RSA

RSA加密过程

$n=p\times q$

$\phi(n)=(p-1)(q-1)$

$ e\in (1,\phi(n))$

$ ed\equiv 1 \pmod {\phi(n)} \Leftrightarrow ed=k\phi(n)+1$

$a^{\phi(n)}\equiv 1 \pmod n$

$a^{k\phi(n)+1} \equiv a \pmod n$

$a^{ed} \equiv a \pmod n$

若$c\equiv a^e \pmod n$则$c^d\equiv {a^{ed}} \equiv a \pmod n$

$assume\ dp\equiv d(mod\ (p-1)),dq\equiv d(mod\ (q-1))$