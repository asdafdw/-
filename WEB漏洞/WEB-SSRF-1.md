![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/57cc4e12b9bc3060bf614a9ebaf63b1e.png)

## 原理：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99a8c988145d83168db07a88a1ff6b72.png)

### 协议

各个协议调用探针：http,file,dict,ftp,gopher 等
漏洞攻击：端口扫描，指纹识别，漏洞利用，内网探针等
http://192.168.64.144/phpmyadmin/
file:///D:/www.txt
dict://192.168.64.144:3306/info
ftp://192.168.64.144:21
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87b2e760215ddd21514f44f8345ee750.png)

#### 原理

利用一个可以发起网络请求的服务当作跳板来攻击内部其他服务

#### 出现点

常出现在：1.能够对外发起网络请求的地方 2.请求远程服务器资源的地方 3.数据库内置功能 4.邮件系统 5.文件处理 6.在线处理工具

```
网页中数据库验证的功能点
有远程图片加载的地方
网站提供的各种下载功能点
未公开的api实现及调用URL的功能
简单来说：所有目标服务器会从自身发起请求的功能点，且我们可以控制地址的参数，都可能造成SSRF漏洞
```

#### 协议

```
file：在有回显的情况下，利用 file 协议可以读取任意内容
dict：泄露安装软件版本信息，查看端口，操作内网redis服务等
gopher：gopher支持发出GET、POST请求：可以先截获get请求包和post请求包，再构造成符合     gopher协议的请求。gopher协议是ssrf利用中一个最强大的协议(俗称万能协议)。可用于反弹     shell
http/s：探测内网主机存活
https://blog.csdn.net/Gherbirthday0916/article/details/129944434
```

#### 函数

**引发ssrf漏洞的PHP函数**

file_get_contents：文件写入字符串，当url是内网文件的时候，会先去把这个文件的内容读出来再写入，导致了文件读取。

fsockopen(主机名称，端口号码，错误号的接受变量，错误提示的接受变量，超时时间)

curl_exec()：执行一个curl会话

#### 绕过

**利用@**：http://example<span class="label label-primary">@127.0.0.1。例如</span>：http://www.baidu.com<span class="label label-primary">@10.10.10.10与http</span>://10.10.10.10 请求是相同的

**添加端口号**：http://127.0.0.1:8080

**利用短地址**：http://dwz.cn/11SMa

**ip 地址进制转换**

**DNS解析** http://127.0.0.1.xip.io/可以指向任意ip的域名：xip.io

#### 利用

利用伪协议对内网信息进行探测

- 具体利用的方式：file协议查看文件、dict协议探测端口、ophergopher协议 支持GET&POST请求，同时在攻击内网ftp、redis、telnet、Memcache上有极大作用。利用 gopher协议访问redis反弹shell

#### 防御

禁止跳转；禁用不需要的协议；黑名单内网ip

```
 限制协议为HTTP、HTTPS，请求端口只能为web端口
2- 地址做白名单处理（限制不能访问内网的ip）
3- 禁止30x跳转
4- 过滤返回信息
5- 去除URL中的特殊字符
6- 统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态
```

#### 无回显怎么办

一般来说，目标出网的情况下，验证采用dnslog的方式，看能不能在Dnslog收到请求

DNS预解析可以绕过CSP进行解析，结合DNSLOG我们即可窃取在CSP保护下的Cookie

如果目标不出网，则可以根据返回包的特征来进行判断，如目标端口开放响应时间200Ms，未开放则响应时间1000ms等