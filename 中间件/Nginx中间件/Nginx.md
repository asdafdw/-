Nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务，由于其具有许多优越的特性，导致在全球范围内被广泛使用。

### 解析漏洞  

NGINX解析漏洞主要是由于NGINX配置文件以及PHP配置文件的错误配置导致的。这个漏洞与NGINX、PHP版本无关，属于用户配置不当造成的解析漏洞。具体来说，由于nginx.conf的配置导致nginx把以’.php’结尾的文件交给fastcgi处理，对于任意文件名，在后面添加/xxx.php（xxx为任意字符）后，即可将文件作为php解析。

        当攻击者访问/phpinfo.jpg/abc.php时，Nginx将查看URL，看到他以.php结尾，并将路径传递给php fastcgi处理程序，php看到/phpinfo.jpg/abc.php不存在，便删除去最后的/abc.php，看到phpinfo.jpg存在，而后以php的形式执行.jpg的内容。
    
        这里涉及到php的有一个选择：cgi.fix_pathinfo，该配置默认为1，开启状态，表示对文件路径进行“修理”。当php遇到文件路径“/1.aaa/2.bbb/3.cccc"文件时，若“/1.aaa/2.bbb/3.cccc"不存在，则会去掉最后的”/3.ccc",然后判断“/1.aaa/2.bbb”是否存在，若不存在，则继续去掉“/2.bbb”,以此类推。
##### 利用

访问 localhost /uploadfiles/nginx.png  则 正常显示

增加 /.php 后缀，则解析为php文件

[Nginx解析漏洞（nginx_parsing_vulnerability）_nginx任意文件解析漏洞漏洞利用-CSDN博客](https://blog.csdn.net/m0_63521991/article/details/135887165)

### CVE-2013-4547漏洞文件名逻辑漏洞

##### 版本：

- 影响范围: Nginx 0.8.41 ~ 1.4.3
- 影响范围: Nginx 1.5.0 ~ 1.5.7

##### 原理：

[Nginx 文件名逻辑漏洞（CVE-2013-4547）复现 - Junglezt - 博客园](https://www.cnblogs.com/Junglezt/p/18119782)

**CVE-2013-4547漏洞**是由于非法字符空格和截止符导致Nginx在解析URL时的有限状态机混乱，导致攻击者可以通过一个非编码空格绕过后缀名限制。假设服务器中存在文件‘123.png '，则可以通过访问如下网址让服务器认为'123.png '的后缀为php

```
http://192.168.146.1/123.png \0.php
```

##### 利用：

[Nginx 文件名逻辑漏洞(CVE-2013-4547) - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/284531.html)

1.上传一个图片webshell【图片马】

2.上传时抓包进行重命名：’phpinfo.png    '，记得后面有空格（此时访问上传的图片马，可以正常访问，但不能执行）

3.使用0x00截断将图片马解析为PHP文件，访问该url并抓包修改http://192.168.146.134/uploadfiles/phpinfo.png%20a.php

【%20a都是占位符号，%20对应空格，是为了在修改hex表时方便找到位置，最终替换时删除%20并把a的hex值改为00（0x00截断原理）】

4.a作为占位符，将%20删除后进入Hex中把表示a的61修改为00，最后发送请求

### 权限提升漏洞(CVE-2016-1247)

##### 版本：

下述版本之前均存在此漏洞：
Debian: Nginx1.6.2-5+deb8u3
Ubuntu 16.04: Nginx1.10.0-0ubuntu0.16.04.3
Ubuntu 14.04: Nginx1.4.6-1ubuntu3.6
Ubuntu 16.10: Nginx1.10.1-0ubuntu1.1

##### 利用与修复

[Nginx权限提升漏洞(CVE-2016-1247) 分析 - 知道创宇](https://blog.knownsec.com/index.html%3Fp=3857.html)

### 命令执行/DDOS（CVE-2021-23017）

也可以执行ddos,rce暂时无poc和exp

https://github.com/M507/CVE-2021-23017-POC

【这个好像不能用了，想用的时候自己去搜poc】

由于Nginx在处理DNS响应时存在安全问题，当在配置文件中使用 “resolver ”指令时，远程攻击者可以通过伪造来自DNS服务器的UDP数据包，构造DNS响应造成1-byte内存覆盖，从而导致拒绝服务或任意代码执行。

该漏洞仅在配置了一个或多个“resolver”指令的情况下存在，而默认情况下没有配置

##### 利用

从网址处下载poc.py,用windows系统执行命令：

```
python3 poc.py -t 目标地址 -r dns服务器地址
```

### 越界读取缓存漏洞（CVE-2017-7529）

##### 版本：

Nginx 0.5.6 – 1.13.2

##### 原理：

Nginx在反向代理站点的时候，通常会将一些文件进行缓存，特别是静态文件。缓存的部分存储在文件中，每个缓存文件包括“文件头”+“HTTP返回包头”+“HTTP返回包体”。如果二次请求命中了该缓存文件，则Nginx会直接将该文件中的“HTTP返回包体”返回给用户。

##### 利用：

[【Vulhub靶场】Nginx 中间件漏洞复现_nginx中间件-CSDN博客](https://blog.csdn.net/M372109150/article/details/138397287)