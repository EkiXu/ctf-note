# Reverse Execrise Area

## 0x00 前言

逆向鲨我，用各种奇怪的技巧存储方式存数据，写函数，

感觉自己学的可能是假的C语言。。。。。。

收货：

- strings 的用法
- IDA pro Patch Program通过修改汇编代码改变程序逻辑
- sprintf的用法
- 64位int分割32位数据的写法
- gdb调试的基本操作
- upx 简单脱壳

Todo:

- Ollydbg 调试方法
- 其他C的奇怪写法
- ESP定律 脱壳方法

## 0x01 re1

入门逆向题，可以放进ida pro里在strings view里找flag

也可以用strings工具找

```bash
strings re1.exe |grep {*}
```

## 0x02 game

一个翻转的小游戏

拖进IDA 发现只要满足条件（7个灯全亮）程序就会输出flag（加密过）

思路

### 静态分析

找到程序中输出flag的程序段，自己写脚本输出一遍

```cpp
#include<stdio.h>
int main(){
  char v[200];
  v[59] = 18;
  v[60] = 64;
  v[61] = 98;
  v[62] = 5;
  v[63] = 2;
  v[64] = 4;
  v[65] = 6;
  v[66] = 3;
  v[67] = 6;
  v[68] = 48;
  v[69] = 49;
  v[70] = 65;
  v[71] = 32;
  v[72] = 12;
  v[73] = 48;
  v[74] = 65;
  v[75] = 31;
  v[76] = 78;
  v[77] = 62;
  v[78] = 32;
  v[79] = 49;
  v[80] = 32;
  v[81] = 1;
  v[82] = 57;
  v[83] = 96;
  v[84] = 3;
  v[85] = 21;
  v[86] = 9;
  v[87] = 4;
  v[88] = 62;
  v[89] = 3;
  v[90] = 5;
  v[91] = 4;
  v[92] = 1;
  v[93] = 2;
  v[94] = 3;
  v[95] = 44;
  v[96] = 65;
  v[97] = 78;
  v[98] = 32;
  v[99] = 16;
  v[100] = 97;
  v[101] = 54;
  v[102] = 16;
  v[103] = 44;
  v[104] = 52;
  v[105] = 32;
  v[106] = 64;
  v[107] = 89;
  v[108] = 45;
  v[109] = 32;
  v[110] = 65;
  v[111] = 15;
  v[112] = 34;
  v[113] = 18;
  v[114] = 16;
  v[115] = 0;
  v[2] = 123;
  v[3] = 32;
  v[4] = 18;
  v[5] = 98;
  v[6] = 119;
  v[7] = 108;
  v[8] = 65;
  v[9] = 41;
  v[10] = 124;
  v[11] = 80;
  v[12] = 125;
  v[13] = 38;
  v[14] = 124;
  v[15] = 111;
  v[16] = 74;
  v[17] = 49;
  v[18] = 83;
  v[19] = 108;
  v[20] = 94;
  v[21] = 108;
  v[22] = 84;
  v[23] = 6;
  v[24] = 96;
  v[25] = 83;
  v[26] = 44;
  v[27] = 121;
  v[28] = 104;
  v[29] = 110;
  v[30] = 32;
  v[31] = 95;
  v[32] = 117;
  v[33] = 101;
  v[34] = 99;
  v[35] = 123;
  v[36] = 127;
  v[37] = 119;
  v[38] = 96;
  v[39] = 48;
  v[40] = 107;
  v[41] = 71;
  v[42] = 92;
  v[43] = 29;
  v[44] = 81;
  v[45] = 107;
  v[46] = 90;
  v[47] = 85;
  v[48] = 64;
  v[49] = 12;
  v[50] = 43;
  v[51] = 76;
  v[52] = 86;
  v[53] = 13;
  v[54] = 114;
  v[55] = 1;
  v[56] = 117;
  v[57] = 126;
  v[58] = 0;
  for ( int i = 0; i < 56; ++i )
  {
    *(v+2 + i) ^= *(v+59 + i);
    *(v+2 + i) ^= 0x13u;
  }
  printf("%s",v+2);
  return 0;
}
```

### 实时修改内存

利用CheatEngine修改内存中存储灯亮的值，使之满足条件然后输出flag

### IDA Pro Patch Program

把标号3-7的汇编条件jnz 修改为jz

相当于变成

```cpp
if ( byte_532E28[0] == 1
      && byte_532E28[1] == 1
      && byte_532E28[2] == 1
      && byte_532E28[3] != 1
      && byte_532E28[4] != 1
      && byte_532E28[5] != 1
      && byte_532E28[6] != 1
      && byte_532E28[7] != 1 )
    {
      sub_457AB4();
    }
```

然后解法就很好得到了

## 0x03 Hello,CTF

运行程序发现要让我们输入序列号，但是我们没有啊

拖进IDA

可以看到序列号正确与否是这样判断的

```cpp
strcpy(&v13, "437261636b4d654a757374466f7246756e");
  while ( 1 )
  {
    memset(&v10, 0, 0x20u);
    v11 = 0;
    v12 = 0;
    sub_40134B(aPleaseInputYou, v6);
    scanf(aS, v9);
    if ( strlen(v9) > 0x11 )
      break;
    v3 = 0;
    do
    {
      v4 = v9[v3];
      if ( !v4 )
        break;
      sprintf(&v8, asc_408044, v4);
      strcat(&v10, &v8);
      ++v3;
    }
    while ( v3 < 17 );
    if ( !strcmp(&v10, &v13) )
      sub_40134B(aSuccess, v7);
    else
      sub_40134B(aWrong, v7);
  }
```

根据sprintf的用法

> int sprintf(char *str, const char *format, ...)
>
>  发送格式化输出到 **str** 所指向的字符串 

所以可以逆推序列号的hex是这玩意

```
43 72 61 63 6b 4d 65 4a 75 73 74 46 6f 72 46 75 6e
```

转成ASCII就完事了

## 0x04  open-source 

根据源码内容直接整就完了

```cpp
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
	
    int first = 0xcafe;
    int second=25;
    unsigned int hash = first * 31337 + (second % 17) * 11 + strlen("h4cky0u") - 1615810207;

    printf("Get your key: ");
    printf("%x\n", hash);
    return 0;
}

```

## 0x05  simple-unpack 

题目~~暗~~明示了

使用 upx -d 把程序脱壳然后同0x01处理

## 0x06  logmein  

真算法逆向

拖进IDA pro分析

```cpp
void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)
{
  size_t v3; // rsi
  int i; // [rsp+3Ch] [rbp-54h]
  char s[36]; // [rsp+40h] [rbp-50h]
  int v6; // [rsp+64h] [rbp-2Ch]
  __int64 v7; // [rsp+68h] [rbp-28h]
  char v8[8]; // [rsp+70h] [rbp-20h]
  int v9; // [rsp+8Ch] [rbp-4h]

  v9 = 0;
  strcpy(v8, ":\"AL_RT^L*.?+6/46");
  v7 = 0x65626D61726168LL;
  v6 = 7;
  printf("Enter your guess: ");
  __isoc99_scanf("%32s", s);
  v3 = strlen(s);
  if ( v3 < strlen(v8) )
    fail();
  for ( i = 0; i < strlen(s); ++i )
  {
    if ( i >= strlen(v8) )
      fail();
    if ( s[i] != (char)(*((_BYTE *)&v7 + i % v6) ^ v8[i]) )
      fail();
  }
  sub_4007F0();
}
```

这次仍然是对程序中存储的密文进行解密所以理论上也可gdp调试在程序运行的时候拿到flag

但是因为他是一个个算的。

所以这里我们用还是自己写一个类似的解密脚本

其中

```cpp
*((_BYTE *)&v7 + i % v6
```

相当于把v7作为一个字符串数组 又因为是小段存储方式，所以解码的时候位置得倒过来（因为这个一开始怎么都解不出来）

```cpp
#include<stdio.h>
#include<string.h>
int main(){
	size_t v3; // rsi
	int i; // [rsp+3Ch] [rbp-54h]
	char s[36]; // [rsp+40h] [rbp-50h]
	int v6; // [rsp+64h] [rbp-2Ch]
	//__int64 v7; // [rsp+68h] [rbp-28h]
	char v8[16]; // [rsp+70h] [rbp-20h]
	int v9; // [rsp+8Ch] [rbp-4h]
	
	v9 = 0;
	strcpy(v8, ":\"AL_RT^L*.?+6/46");
	char v7[7] = {0x68,0x61,0x72,0x61,0x6D,0x62,0x65};
	//strrev(v7);
	v6 = 7;
	//__isoc99_scanf("%32s", s);
	//v3 = strlen(s);
	for ( i = 0; i < strlen(v8); ++i ){
		s[i] = v7[i%v6]^v8[i];
	}
	//printf("%s",v7);
	printf("%s",s);
	return 0;
}
```

后来看了大佬的WP 发现也可以这样写

```cpp
#include<stdio.h>
#include<string.h>
int main(){
	size_t v3; // rsi
	int i; // [rsp+3Ch] [rbp-54h]
	char s[36]; // [rsp+40h] [rbp-50h]
	int v6; // [rsp+64h] [rbp-2Ch]
	//__int64 v7; // [rsp+68h] [rbp-28h]
	char v8[16]; // [rsp+70h] [rbp-20h]
	int v9; // [rsp+8Ch] [rbp-4h]
	
	v9 = 0;
	strcpy(v8, ":\"AL_RT^L*.?+6/46");
	//char v7[7] = {0x68,0x61,0x72,0x61,0x6D,0x62,0x65};
	long long v7=0x65626D61726168;
	char *p=(char *)&v7;//转成字符数组
	v6 = 7;
	//__isoc99_scanf("%32s", s);
	//v3 = strlen(s);
	for ( i = 0; i < strlen(v8); ++i ){
		s[i] = p[i%v6]^v8[i];
	}
	//printf("%s",v7);
	printf("%s",s);
	return 0;
}
```

## 0x07  insanity 

题目说这是一道简单题，果然是一道简单题

又是一道string能看到的明文题

## 0x08  no-strings-attached 

拖进IDA 分析一波发现又是个解密题

本来想和前几题一样静态分析写个解密脚本的

但是在IDA里

数据长这样

```assembly
.rodata:08048AA8 s               dd 143Ah                ; DATA XREF: authenticate+11↑o
.rodata:08048AAC                 db  36h ; 6
.rodata:08048AAD                 db  14h
.rodata:08048AAE                 db    0
.rodata:08048AAF                 db    0
.rodata:08048AB0                 db  37h ; 7
.rodata:08048AB1                 db  14h
.rodata:08048AB2                 db    0
.rodata:08048AB3                 db    0
.rodata:08048AB4                 db  3Bh ; ;
.rodata:08048AB5                 db  14h
.rodata:08048AB6                 db    0
.rodata:08048AB7                 db    0
.rodata:08048AB8                 db  80h
.rodata:08048AB9                 db  14h
.rodata:08048ABA                 db    0
.rodata:08048ABB                 db    0
.rodata:08048ABC                 db  7Ah ; z
.rodata:08048ABD                 db  14h
.rodata:08048ABE                 db    0
.rodata:08048ABF                 db    0
.rodata:08048AC0                 db  71h ; q
.rodata:08048AC1                 db  14h
.rodata:08048AC2                 db    0
.rodata:08048AC3                 db    0
.rodata:08048AC4                 db  78h ; x
.rodata:08048AC5                 db  14h
.rodata:08048AC6                 db    0
.rodata:08048AC7                 db    0
.rodata:08048AC8                 db  63h ; c
.rodata:08048AC9                 db  14h
.rodata:08048ACA                 db    0
.rodata:08048ACB                 db    0
.rodata:08048ACC                 db  66h ; f
.rodata:08048ACD                 db  14h
.rodata:08048ACE                 db    0
.rodata:08048ACF                 db    0
.rodata:08048AD0                 db  73h ; s
.rodata:08048AD1                 db  14h
.rodata:08048AD2                 db    0
.rodata:08048AD3                 db    0
.rodata:08048AD4                 db  67h ; g
.rodata:08048AD5                 db  14h
.rodata:08048AD6                 db    0
.rodata:08048AD7                 db    0
.rodata:08048AD8                 db  62h ; b
.rodata:08048AD9                 db  14h
.rodata:08048ADA                 db    0
.rodata:08048ADB                 db    0
.rodata:08048ADC                 db  65h ; e
.rodata:08048ADD                 db  14h
.rodata:08048ADE                 db    0
.rodata:08048ADF                 db    0
.rodata:08048AE0                 db  73h ; s
.rodata:08048AE1                 db  14h
.rodata:08048AE2                 db    0
.rodata:08048AE3                 db    0
.rodata:08048AE4                 db  60h ; `
.rodata:08048AE5                 db  14h
.rodata:08048AE6                 db    0
.rodata:08048AE7                 db    0
.rodata:08048AE8                 db  6Bh ; k
.rodata:08048AE9                 db  14h
.rodata:08048AEA                 db    0
.rodata:08048AEB                 db    0
.rodata:08048AEC                 db  71h ; q
.rodata:08048AED                 db  14h
.rodata:08048AEE                 db    0
.rodata:08048AEF                 db    0
.rodata:08048AF0                 db  78h ; x
.rodata:08048AF1                 db  14h
.rodata:08048AF2                 db    0
.rodata:08048AF3                 db    0
.rodata:08048AF4                 db  6Ah ; j
.rodata:08048AF5                 db  14h
.rodata:08048AF6                 db    0
.rodata:08048AF7                 db    0
.rodata:08048AF8                 db  73h ; s
.rodata:08048AF9                 db  14h
.rodata:08048AFA                 db    0
.rodata:08048AFB                 db    0
.rodata:08048AFC                 db  70h ; p
.rodata:08048AFD                 db  14h
.rodata:08048AFE                 db    0
.rodata:08048AFF                 db    0
.rodata:08048B00                 db  64h ; d
.rodata:08048B01                 db  14h
.rodata:08048B02                 db    0
.rodata:08048B03                 db    0
.rodata:08048B04                 db  78h ; x
.rodata:08048B05                 db  14h
.rodata:08048B06                 db    0
.rodata:08048B07                 db    0
.rodata:08048B08                 db  6Eh ; n
.rodata:08048B09                 db  14h
.rodata:08048B0A                 db    0
.rodata:08048B0B                 db    0
.rodata:08048B0C                 db  70h ; p
.rodata:08048B0D                 db  14h
.rodata:08048B0E                 db    0
.rodata:08048B0F                 db    0
.rodata:08048B10                 db  70h ; p
.rodata:08048B11                 db  14h
.rodata:08048B12                 db    0
.rodata:08048B13                 db    0
.rodata:08048B14                 db  64h ; d
.rodata:08048B15                 db  14h
.rodata:08048B16                 db    0
.rodata:08048B17                 db    0
.rodata:08048B18                 db  70h ; p
.rodata:08048B19                 db  14h
.rodata:08048B1A                 db    0
.rodata:08048B1B                 db    0
.rodata:08048B1C                 db  64h ; d
.rodata:08048B1D                 db  14h
.rodata:08048B1E                 db    0
.rodata:08048B1F                 db    0
.rodata:08048B20                 db  6Eh ; n
.rodata:08048B21                 db  14h
.rodata:08048B22                 db    0
.rodata:08048B23                 db    0
.rodata:08048B24                 db  7Bh ; {
.rodata:08048B25                 db  14h
.rodata:08048B26                 db    0
.rodata:08048B27                 db    0
.rodata:08048B28                 db  76h ; v
.rodata:08048B29                 db  14h
.rodata:08048B2A                 db    0
.rodata:08048B2B                 db    0
.rodata:08048B2C                 db  78h ; x
.rodata:08048B2D                 db  14h
.rodata:08048B2E                 db    0
.rodata:08048B2F                 db    0
.rodata:08048B30                 db  6Ah ; j
.rodata:08048B31                 db  14h
.rodata:08048B32                 db    0
.rodata:08048B33                 db    0
.rodata:08048B34                 db  73h ; s
.rodata:08048B35                 db  14h
.rodata:08048B36                 db    0
.rodata:08048B37                 db    0
.rodata:08048B38                 db  7Bh ; {
.rodata:08048B39                 db  14h
.rodata:08048B3A                 db    0
.rodata:08048B3B                 db    0
.rodata:08048B3C                 db  80h
.rodata:08048B3D                 db  14h
.rodata:08048B3E                 db    0
.rodata:08048B3F                 db    0
.rodata:08048B40                 db    0
.rodata:08048B41                 db    0
.rodata:08048B42                 db    0
.rodata:08048B43                 db    0
```

很难导出来写。。。。

注意到程序中直接比较strcmp(ws,s2)

所以我们可以用gdb动态调试的方法截获s2也即计算出的明文

在IDA pro里分析函数

```assembly
text:08048708                 push    ebp
.text:08048709                 mov     ebp, esp
.text:0804870B                 sub     esp, 8028h
.text:08048711                 mov     dword ptr [esp+4], offset dword_8048A90 ; wchar_t *
.text:08048719                 mov     dword ptr [esp], offset s ; s
.text:08048720                 call    decrypt
.text:08048725                 mov     [ebp+s2], eax
```

可以看到s2的数据是从寄存器eax来的

所以我们只要查看寄存器eax的值就行了

```bash
gdb no-srtings-attached
(gdb) break decrypt
Breakpoint 1 at 0x804865c
(gdb) run
Starting program: /root/no-srtings-attached 
Welcome to cyber malware control software.
Currently tracking 1373268125 bots worldwide

Breakpoint 1, 0x0804865c in decrypt ()
(gdb) next
Single stepping until exit from function decrypt,
which has no line number information.
0x08048725 in authenticate ()
(gdb) x/sw $eax #sw s表示以字符串形式输出w表示word（4字节）形式(w_char)
0x804e800:	U"9447{you_are_an_international_mystery}"
```

## 0x09  csaw2013reversing2 

拖进IDA pro 核心代码如下

```cpp
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  int v3; // ecx
  CHAR *lpMem; // [esp+8h] [ebp-Ch]
  HANDLE hHeap; // [esp+10h] [ebp-4h]

  hHeap = HeapCreate(0x40000u, 0, 0);
  lpMem = (CHAR *)HeapAlloc(hHeap, 8u, MaxCount + 1);
  memcpy_s(lpMem, MaxCount, &unk_409B10, MaxCount);
  if ( sub_40102A() || IsDebuggerPresent() )
  {
    __debugbreak();
    sub_401000(v3 + 4, lpMem);
    ExitProcess(0xFFFFFFFF);
  }
  MessageBoxA(0, lpMem + 1, "Flag", 2u);
  HeapFree(hHeap, 0, lpMem);
  HeapDestroy(hHeap);
  ExitProcess(0);
}
```

点开sub_40102A()

```cpp
int sub_40102A()
{
  char v0; // t1

  v0 = *(_BYTE *)(*(_DWORD *)(__readfsdword(0x18u) + 48) + 2);
  return 0;
}
```

所以无论如何都进不去if

应该就是这样导致乱码了

所以我们这里直接Path Program

把调试中断的汇编指令int 3改成nop空指令

然后把jnz改成jmp无条件跳转

然后在把loc_401096 结束后jmp到loc_4010B9上

变成这样



然后就不会乱码了

PS:

直接点击窗体ctrl+c是可以复制窗体内容的。。。。

## 0x0A  getit 

IDA 反编译的得到

```cpp
  char v3; // al
  __int64 v5; // [rsp+0h] [rbp-40h]
  int i; // [rsp+4h] [rbp-3Ch]
  FILE *stream; // [rsp+8h] [rbp-38h]
  char filename[8]; // [rsp+10h] [rbp-30h]
  unsigned __int64 v9; // [rsp+28h] [rbp-18h]

  v9 = __readfsqword(0x28u);
  LODWORD(v5) = 0;
  while ( (signed int)v5 < strlen(s) )
  {
    if ( v5 & 1 )
      v3 = 1;
    else
      v3 = -1;
    *(&t + (signed int)v5 + 10) = s[(signed int)v5] + v3;
    LODWORD(v5) = v5 + 1;
  }
  strcpy(filename, "/tmp/flag.txt");
```

找一下t和s

```assembly
.data:00000000006010E0 t               db 'S'                  ; DATA XREF: main+65↑w
.data:00000000006010E0                                         ; main+C9↑o ...
.data:00000000006010E1 aHarifctf       db 'harifCTF{????????????????????????????????}',0
```

所以可以静态分析写解密脚本了

```cpp
#include <stdio.h>
#include <string.h>
char s[]="c61b68366edeb7bdce3c6820314b7498";
char t[]="SharifCTF{????????????????????????????????}";
int main()
{
  char v3; // al
  __int64 v5; // [rsp+0h] [rbp-40h]
  char filename[8]; // [rsp+10h] [rbp-30h]
  unsigned __int64 v9; // [rsp+28h] [rbp-18h]

  v5 = 0;
  while ( (signed int)v5 < strlen(s) )
  {
    if ( v5 & 1 )
      v3 = 1;
    else
      v3 = -1;
    t[v5+10] = s[v5] + v3;
    v5++;
  }
  printf("%s",t);
  return 0;
}
```

占个坑尝试一下动态调试的做法

## 0x0B Python_trade

pycdc逆向得到

```python
import base64

def encode(message):
    s = ''
    for i in message:
        x = ord(i) ^ 32
        x = x + 16
        s += chr(x)

    return base64.b64encode(s)

correct = 'XlNkVmtUI1MgXWBZXCFeKY+AaXNt'
flag = ''
print 'Input flag:'
flag = raw_input()
if encode(flag) == correct:
    print 'correct'
else:
    print 'wrong'
```

所以解密脚本就很好写了

```python
import base64

def decode(message):
    message=base64.b64decode(message)
    s = ''
    for i in message:
        x = ord(i) - 16
        x = x ^ 32
        s += chr(x)

    return s

correct = 'XlNkVmtUI1MgXWBZXCFeKY+AaXNt'
print decode(correct)
```

## 0x0C maze

拖进IDA pro 核心代码如下

```cpp
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  signed __int64 v3; // rbx
  signed int v4; // eax
  bool v5; // bp
  bool v6; // al
  const char *v7; // rdi
  __int64 v9; // [rsp+0h] [rbp-28h]

  v9 = 0LL;
  puts("Input flag:");
  scanf("%s", &s1, 0LL);
  if ( strlen(&s1) != 24 || strncmp(&s1, "nctf{", 5uLL) || *(&byte_6010BF + 24) != '}' )
  {
LABEL_22:
    puts("Wrong flag!");
    exit(-1);
  }
  v3 = 5LL;
  if ( strlen(&s1) - 1 > 5 )
  {
    while ( 1 )
    {
      v4 = *(&s1 + v3);
      v5 = 0;
      if ( v4 > 'N' )
      {
        v4 = (unsigned __int8)v4;
        if ( (unsigned __int8)v4 == 'O' )
        {
          v6 = sub_400650((_DWORD *)&v9 + 1);
          goto LABEL_14;
        }
        if ( v4 == 'o' )
        {
          v6 = sub_400660((int *)&v9 + 1);
          goto LABEL_14;
        }
      }
      else
      {
        v4 = (unsigned __int8)v4;
        if ( (unsigned __int8)v4 == '.' )
        {
          v6 = sub_400670(&v9);
          goto LABEL_14;
        }
        if ( v4 == '0' )
        {
          v6 = sub_400680((int *)&v9);
LABEL_14:
          v5 = v6;
          goto LABEL_15;
        }
      }
LABEL_15:
      if ( !(unsigned __int8)sub_400690((__int64)asc_601060, SHIDWORD(v9), v9) )
        goto LABEL_22;
      if ( ++v3 >= strlen(&s1) - 1 )
      {
        if ( v5 )
          break;
LABEL_20:
        v7 = "Wrong flag!";
        goto LABEL_21;
      }
    }
  }
  if ( asc_601060[8 * (signed int)v9 + SHIDWORD(v9)] != '#' )
    goto LABEL_20;
  v7 = "Congratulations!";
LABEL_21:
  puts(v7);
  return 0LL;
}
```

不会SHIDWORD(),

查了一下发现是IDA pro的宏定义

```cpp
#define SHIDWORD(x)  (*((int32*)&(x)+1))
```

注意到x是个int64 所以SHIDWORD(x)应该就是取它的前32位（小端存）

写个脚本验证一下

```cpp
#include<stdio.h>
int main(){
	long long x=(15LL<<32)+23LL;
	printf("%lld %d\n",(unsigned int)x,(*((int*)&(x)+1)));
	return 0;
}
//输出 23 15
```

因为要让

```cpp
asc_601060[8 * (signed int)v9 + SHIDWORD(v9)] == '#'
```

v9前32位应该表示的是列，而v9后32位是行

然后又有四个if条件，感觉真就是个迷宫

```cpp
bool __fastcall sub_400650(_DWORD *a1)//O (_DWORD *)&v9 + 1
{
  int v1; // eax

  v1 = (*a1)--;//左
  return v1 > 0;
}
bool __fastcall sub_400660(int *a1)//o (int *)&v9 + 1
{
  int v1; // eax

  v1 = *a1 + 1;//右
  *a1 = v1;
  return v1 < 8;
}
bool __fastcall sub_400670(_DWORD *a1)//. &v9
{
  int v1; // eax

  v1 = (*a1)--;//上
  return v1 > 0;
}
bool __fastcall sub_400680(int *a1)//0 (int *)&v9
{
  int v1; // eax

  v1 = *a1 + 1;//下
  *a1 = v1;
  return v1 < 8;
}
```

然后我们把asc_601060里的东西按8个一行整理一下

```
  ******
*   *  *
*** * **
**  * **
*  *#  *
** *** *
**     *
********
```

然后就可以得到{}里面的flag部分了

o0oo00O000oooo..OO

