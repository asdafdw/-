[Web中间件漏洞总结——IIS篇_iis6.0 put-CSDN博客](https://blog.csdn.net/Hacker_LaoYi/article/details/144630901)

Internet Information Services（IIS，以前称为Internet Information Server）互联网信息服务是[Microsoft]公司提供的可扩展[Web服务器](https://en.wikipedia.org/wiki/Web_server)，支持[HTTP](https://en.wikipedia.org/wiki/HTTP)，[HTTP/2](https://en.wikipedia.org/wiki/HTTP/2)，[HTTPS](https://en.wikipedia.org/wiki/HTTPS)，[FTP](https://en.wikipedia.org/wiki/File_Transfer_Protocol)，[FTPS](https://en.wikipedia.org/wiki/FTPS)，[SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol)和[NNTP](https://en.wikipedia.org/wiki/Network_News_Transfer_Protocol)等。起初用于[Windows NT](https://en.wikipedia.org/wiki/Windows_NT)系列，随后内置在Windows 2000、Windows XP Professional、Windows Server 2003和后续版本一起发行，但在Windows XP Home版本上并没有IIS。IIS目前只适用于Windows系统，不适用于其他操作系统。

IS对应windows系统版本：
Windows Server 2000 —— IIS 5.0
Windows XP SP1 —— IIS 5.1
Windows XP SP2，SP3 —— IIS 5.1
Windows Server 2003，XP porfessional —— IIS 6.0
Windows Vista Ultimate —— IIS 7.0
Windows 7 —— IIS 7，IIS 7.5
Windows Server 2008 SP2，SP3 —— IIS 7.0
Windows Server 2008 R2，部分Windows 7 —— IIS 7.5
Windows Server 2008 SP2，SP3 —— IIS 7.0
Windows Server 2012，Windows 8 —— IIS 8.0
Windows Server 2012 R2 —— IIS 8.5
windows Server 2016，Windows10 —— IIS 10

### 短文件名

##### 版本：

IIS 1.0，Windows NT 3.51 
IIS 3.0，Windows NT 4.0 Service Pack 2 
IIS 4.0，Windows NT 4.0选项包
IIS 5.0，Windows 2000 
IIS 5.1，Windows XP Professional和Windows XP Media Center Edition 
IIS 6.0，Windows Server 2003和Windows XP Professional x64 Edition 
IIS 7.0，Windows Server 2008和Windows Vista 
IIS 7.5，Windows 7（远程启用<customErrors>或没有web.config）

IIS 7.5，Windows 2008（经典管道模式）

注意：IIS使用.Net Framework 4时不受影响

##### 详细利用：

[IIS短文件名泄露漏洞修复_microsoft iis 版本信息泄露-CSDN博客](https://blog.csdn.net/bylfsj/article/details/102405360)

[IIS短文件名漏洞复现 - 雨中落叶 - 博客园](https://www.cnblogs.com/yuzly/p/11221685.html)

我们可以在启用.net的IIS下使用GET方法暴力列举短文件名，原因是攻击者使用通配符“*”和“?”发送一个请求到IIS,当IIS接收到一个文件路径中包含“~”请求时，它的反应是不同的，即返回的HTTP状态码和错误信息不同。基于这个特点，可以根据HTTP的响应区分一个可用或者不可用的文件。如下图所示不同IIS版本返回信息的不同：

我们可以在启用.net的IIS下使用GET方法暴力列举短文件名，原因是攻击者使用通配符“*”和“?”发送一个请求到IIS,当IIS接收到一个文件路径中包含“~”请求时，它的反应是不同的，即返回的HTTP状态码和错误信息不同。基于这个特点，可以根据HTTP的响应区分一个可用或者不可用的文件。如下图所示不同IIS版本返回信息的不同：



[![图片.png](https://i-blog.csdnimg.cn/blog_migrate/b666c7b614dbee970f4e9a045369f49b.jpeg)](https://image.3001.net/images/20180522/15269717282363.png)

### IIS6.0文件解析

##### 原理：

```
IIS6.0解析漏洞分两种：
1、目录解析：
以xx.asp命名的文件夹里的文件都将会被当成ASP文件执行。
2、文件解析：
xx.asp;.jpg 像这种畸形文件名在;后面的直接被忽略，也就是说当成xx.asp文件执行。
IIS6.0 默认的可执行文件除了asp还包含这三种 .asa .cer .cdx。
- 基于文件名
IIS6.0默认不解析;号后面的内容，例如1.asp;.jpg会当成1.asp解析，相当于分号截断。
- 基于文件夹
IIS6.0会将/*.asp/文件夹下的文件当成asp解析。
```

##### 利用

[【文件上传绕过】——解析漏洞_IIS6.0解析漏洞_iis6.0站上的目录路径检测解析绕过上传漏洞-CSDN博客](https://blog.csdn.net/weixin_45588247/article/details/118879619)

绕过检测上传一个含有后门的代码并执行

##### 修复方案

取消网站目录脚本执行权限：

![img](https://i-blog.csdnimg.cn/img_convert/fd7a76da82b4857b45e2437c85988e2b.png)

- 禁止创建文件夹

- 重命名上传文件为时间戳+`.jpg`或随机数+`.jpg`等

### IIS7.0/7.5文件解析

##### 原理

IIS7.*在FastCGI运行php的情况下，php默认配置`cgi.fix_pathinfo=1`，导致在任意文件后面添加`/.php`，服务器就会解析成php

##### 利用

访问.jpg文件无效，访问.jpg/.php成功

##### 修复方案：

- 将`cgi.fix_pathinfo`设置为0并重启php-cgi程序

### Http.sys远程代码执行漏洞(CVE-2015-1635)

[IIS Http.sys远程代码执行漏洞(CVE-2015-1635)复现_cve-2015-1635修复-CSDN博客](https://blog.csdn.net/qq_40640917/article/details/122615505)

远程执行代码漏洞存在于 HTTP 协议堆栈 (HTTP.sys) 中，当 HTTP.sys 未正确分析经特殊设计的 HTTP 请求时会导致此漏洞。 成功利用此漏洞的攻击者可以在系统帐户的上下文中执行任意代码。

若要利用此漏洞，攻击者必须将经特殊设计的 HTTP 请求发送到受影响的系统。 通过修改 Windows HTTP 堆栈处理请求的方式，安装更新可以修复此漏洞。

##### 版本：

Windows Server 2012 R2

Windows Server 2012

Windows Server 2008 R2

Windows 8.1

Windows 8

Windows 7

**漏洞判定：**

一、是否受影响版本。

二、有没有安装补丁KB3042553

##### 利用：

在请求头中添加参数“Range: bytes=0-18446744073709551615”

当回复出现Requested Range Not Satisfiable时则说明该漏洞是存在的

MSF详细利用看：

[IIS Http.sys远程代码执行漏洞(CVE-2015-1635)复现_cve-2015-1635修复-CSDN博客](https://blog.csdn.net/qq_40640917/article/details/122615505)

##### 修复：

1.升级补丁KB3042553

2.禁用IIS内核缓存（可能降低IIS性能）

### IIS6.0 WEBDAV远程代码执行漏洞 CVE-2017-7269

##### 原理:

在Windows Server 2003的IIS6.0的WebDAV服务的ScStoragePathFromUrl函数存在缓存区溢出漏洞，攻击者通过一个以`"If: <http://"`开始的较长header头的PROPFIND请求执行任意代码，控制目标主机。

##### 利用：

[Web中间件漏洞总结——IIS篇_iis6.0 put-CSDN博客](https://blog.csdn.net/Hacker_LaoYi/article/details/144630901)

### IIS6.0 put漏洞

##### 原理：

IS6.0 server在web服务扩展中开启了WebDAV（Web-based Distributed Authoring and Versioning）。WebDAV是一种HTTP1.1的扩展协议。它扩展了HTTP 1.1，在GET、POST、HEAD等几个HTTP标准方法以外添加了一些新的方法，如PUT。

该扩展也存在缺陷，利用PUT方法可直接向服务器上传恶意文件，控制服务器。

##### 利用：

使用IISPutScanner工具

1.使用工具DotNetScan工具扫描目标地址，爆出IIS6.0 PUT漏洞

2.借助该工具上传文件，数据包格式选择PUT，选择asp一句话木马，点击提交数据包。

3.然后数据包格式选择MOVE，再次点击提交数据包，此时可得到webshell路径：

4.用菜刀连接WEBSHELL

##### 修复

- 关闭WebDAV服务扩展
- 关闭IIS来宾用户写入权限